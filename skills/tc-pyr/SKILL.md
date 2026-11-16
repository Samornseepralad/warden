---
name: tc-pyr
description: Prepare test cases for a Pyrite ticket — `/tc-pyr 783` means PYR-783. Writes the TCs from the ticket's AC/EC, creates them in Qase project Pyrite, builds the Qase Test plan "PYR-xxx + ticket name", fills the Drive TC sheet and the tracking sheet. Also `/tc-pyr 783 update` (updateTCSheet) and `/tc-pyr 783 run` (manual execution). Use when the user types /tc-pyr, gives a PYR-xxx ticket and wants TCs, says "เขียน TC PYR-xxx" or "updateTCSheet". Do NOT use for OLS tickets (tc-ols), TC review (Helix tc-review), filing bugs, retest or test data.
argument-hint: "<number> [update | run]   e.g. 783"
metadata:
  proactive_triggers:
    - /tc-pyr
    - เขียน TC PYR
    - updateTCSheet
    - write test cases PYR
---

# /tc-pyr — TC prep for one Pyrite ticket

`/tc-pyr 783` = prepare the test cases of **PYR-783**. Pyrite only. OLS has its own skill, `tc-ols`.

**Session start:** follow Helix `references/session-start-context.md` (installed with Helix).

## Step 1 — Read the command

| Input | Meaning |
|---|---|
| `/tc-pyr 783` | Ticket `PYR-783`, job **create** |
| `/tc-pyr 783 update` or `updateTCSheet` | Job **update**: apply changed TCs everywhere |
| `/tc-pyr 783 run` | Job **run**: manual execution, [references/execute.md](references/execute.md) |
| `/tc-pyr PYR-783` | Same as `783` |
| Number missing | Ask one line: `PYR-?` |
| Another project's key (OLS-xxx …) | Refuse in one line, route to `tc-ols` |

Read [references/pyr.md](references/pyr.md). Its rules override the generic Helix defaults wherever they differ.

## Step 2 — Create (default job)

1. Announce one line: `Using **tc-pyr** → PYR-783 · create.`
2. Fetch the ticket from Jira (summary = the **ticket name**, AC/EC, Jira link).
3. Reuse check (profile "Reuse rule"): existing Drive subfolder + sheet, Qase cases / suite, Qase Test plan for this key. Update in place; never duplicate.
4. Delegate TC writing to Helix `tc-fe-prep-workflow` (UI story) or `tc-api-prep-workflow` (API / Swagger) and follow it completely, with the profile as overlay.
5. Create the TCs in Qase project Pyrite.
6. **Create / update the Qase Test plan** named `PYR-783 <ticket name>` with exactly these cases (profile "Qase Test plan"). Always, not on request. Verify the case count.
7. Create the Drive subfolder + TC sheet; update the tracking sheet row.
8. Stop at the definition of done. No Jira comment, no CSV attach unless the user asks.

## Step 3 — Update

Apply the changed TCs to **every** place: Qase cases, Qase Test plan (re-sync cases and title), Drive TC sheet, tracking sheet. All must match.

## Rules

- MUST take the ticket key from the command. Never guess it.
- MUST create or update the Test plan after every create / update, titled `PYR-xxx <ticket name>`.
- MUST keep Qase cases, Test plan, Drive sheet and any other artifact of one ticket identical.
- MUST NOT write secrets (Qase token, passwords) into sheet, Qase, Jira, repo or chat. Token comes from `QASE_API_TOKEN`.
- MUST NOT invent a rule that is not in the profile. If the profile is silent on something that changes the output, ask once and propose the answer for the team to add to the profile.
- Automation (Playwright) is not part of this flow: do not offer to write or run it unless asked.
- TC review is not here: use Helix `tc-review` (`/tc-review`).

## Communication (shortest possible)

- One question per turn, one line, options inline. No preamble.
- Never ask for what the profile, ticket or an earlier answer already gives.
- Report results in a few lines: counts, links (Qase suite, Test plan, sheet), open issues. No recap, no offers of follow-up work.
- Tables only when the user must decide something. The brevity rule applies to chat, not to TC content.

## Refusal-first

| Request | Do |
|---|---|
| OLS key or no key | Refuse, one line: `tc-ols` for OLS, ask for the number otherwise. |
| Review TCs | Not here — Helix `tc-review`. |
| Post a Jira comment | Refuse; PYR definition of done excludes it unless the user explicitly asks. |
| File bug, retest, test data | Route to `create-bug-workflow`, `retest-bug-workflow`, `test-data-prep-workflow`. |

## QA closing (mandatory before "done")

Follow Helix `references/qa-closing-shared.md`, plus:

- [ ] Ticket key came from the command; profile read.
- [ ] Output follows the profile (Type values, ID format, wording, ordering).
- [ ] Test plan exists once, titled `PYR-xxx <ticket name>`, holds exactly the ticket's cases, count verified.
- [ ] Qase cases, Test plan, sheet and tracking row match each other.
- [ ] Nothing posted outside the definition of done; no secret written anywhere.
