# Pyrite (PYR-xxx) profile

Applies to every ticket with key `PYR-xxx`. Used by the `tc-pyr` skill.

## Targets
- Qase project: https://app.qase.io/project/Pyrite — API project code is `Pyrite` (not `PYR` / `PYRITE`).
- Qase API token: read from the environment variable `QASE_API_TOKEN` (each person creates their own; setup in this repo's README, section "Qase token"). Send it as header `Token: $QASE_API_TOKEN`. Never print, copy or store the value anywhere (chat, sheet, Qase, Jira, repo). If the variable is empty or the API returns 401, tell the user to follow the README setup; do not ask them to paste the token.
- Tracking sheet ("Pyrite QA"): the one entry point. Its link is team-private and is NOT stored in this public repo. Each person gives it once; keep it in your own memory or local notes. If it is missing, ask once and remember it.
- Drive output folder and TC sheet template: read them from the `Config` tab of the Tracking sheet (columns: Key, Value; keys `DRIVE_FOLDER` and `TC_TEMPLATE`). If the tab or a key is missing, ask once; never guess a link.

## Create-TC flow
1. Create the TCs in Qase project Pyrite (suite chosen per feature; e.g. Authentication › Register).
2. In the Drive folder, create subfolder `PYR-xxx` and put the TC sheet inside, copied from the template format.
3. Create (or update) the Qase **Test plan** for the ticket — see "Qase Test plan" below.
4. Update the tracking sheet row for the ticket.
5. Keep Qase cases, Test plan, sheet and any CSV identical.

**Reuse rule:** before creating anything, check for an existing Drive subfolder `PYR-xxx` + sheet, Qase cases/suite and Qase Test plan for that ticket. If found, update in place (same ids/links). Create only what is missing.

**`updateTCSheet`:** after a review or change, apply the updated TCs to every place (Qase cases, Qase Test plan, TC sheet, other artifacts of that ticket) so all match. Confirm the ticket if ambiguous.

**Definition of done:** Qase cases + Qase Test plan + Drive TC sheet + tracking sheet updated. Then report done and stop. Do NOT post a Jira comment or attach CSVs to Jira unless asked.

## Qase Test plan
Always. As soon as the ticket's TCs exist in Qase, put them in a Test plan. One plan per ticket; never one plan for several tickets.

- **Title:** `PYR-xxx <ticket name>` — the ticket key, one space, the Jira summary exactly as written (e.g. `PYR-783 Register with email`). Always both parts. Never the key alone.
- **Description:** Jira link, one line. No secrets.
- **Cases:** every Qase case id of that ticket, in the final TC order (Happy Path → Failure Scenario → Edge Case → Exceptional Case). Nothing from other tickets.
- **API** (header `Token: $QASE_API_TOKEN`, project code `Pyrite`, base `https://api.qase.io/v1`):
  1. Find: `GET /plan/Pyrite?limit=100&offset=…` (page through). Match a plan whose title equals `PYR-xxx` or starts with `PYR-xxx ` (with the space, so PYR-78 never matches PYR-783).
  2. Missing → `POST /plan/Pyrite` with `{"title","description","cases":[ids]}`.
  3. Found → `PATCH /plan/Pyrite/{id}` with the same body (fixes the title if the ticket was renamed). `cases` replaces the whole list, so always send the full current list. Never create a second plan for the same key.
  4. Verify: `GET /plan/Pyrite/{id}`; case count must equal the ticket's TC count. Report plan id and link `https://app.qase.io/plan/Pyrite/{id}`.
- Any later add / remove / re-create of a case (`update`, `updateTCSheet`) re-syncs the plan in the same run.
- A Test plan is not a Test run. Do not start a run or set case results.

## TC format rules
- **Test Case ID:** `TC-01`, `TC-02`, … (prefix `TC-`, 2-digit padding; ask only if a ticket passes 99 TCs). Qase has no ID field (auto ids).
- **Test Case Type** (sheet column and Qase custom field id 4): `Happy Path`, `Failure Scenario`, `Edge Case`, `Exceptional Case` — never OLS Core/Edge.
  - Qase option ids: 1 Happy Path, 2 Failure Scenario, 3 Edge Case, 4 Exceptional Case.
  - Positive flow → Happy Path; rejected/invalid input or error → Failure Scenario; boundary/limit/unusual condition → Edge Case; unexpected state / system exception → Exceptional Case.
- **Ordering:** Happy Path → Failure Scenario → Edge Case → Exceptional Case; inside a type keep AC/EC order; renumber IDs 1..N after sorting. The Qase API cannot set case position, so reorder by overwriting existing case slots in place, not delete/recreate.
- **Test Data / Test Step / Expected Result:** equal item count, numbered 1, 2, 3 with no gaps, and **no `-` placeholder lines**. Merge or split steps so every step has its own data and expected result; if a count truly cannot match, flag it instead of padding.
- **Test Title:** start with "ตรวจสอบ". Describe what is being checked only; never put the expected result in the title.
- **Precondition and Test Data:** short sentences, one idea per line, no long explanations.
- **Writing style (every TC field):** natural Thai, semi-formal. No special symbols or unusual characters (no emoji, arrows, decorative bullets, odd quotes or brackets used as decoration). Plain numbers and normal punctuation only.
- **Glossary:** English-term-first. Use the ticket's own wording (keep English where the ticket writes English: checkbox, badge, dropdown, permission, scope, role, account, login, tab, filter, status). Do not raise glossary-vs-ticket collisions as questions; still show the term table for information.
- **No Figma / PRD link in the ticket = absent.** Do not ask for them. Use ticket text only and record "no Figma / no PRD" in the pre-draft review and Remark.
