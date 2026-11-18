---
name: qase-run
description: Run the Qase Test plan of a Pyrite ticket — `/qase-run 783` means the plan of PYR-783. Creates a Test run named exactly like the Test plan, runs every case by hand-driven headless Playwright, records each case as an mp4 named by its Test Title, uploads it to the Drive folder, records the result in Qase with the video link in the actual result. Use when the user types /qase-run or asks to run the PYR-xxx Test plan / Test run. Do NOT use for OLS, for writing TCs (tc-pyr), for testing a Jira ticket by AC/EC (test-task), or for filing bugs.
argument-hint: "<number>   e.g. 783"
metadata:
  proactive_triggers:
    - /qase-run
    - run test plan PYR
---

# /qase-run — Test run from the Test plan of one Pyrite ticket

`/qase-run 783` = run the Qase Test plan of **PYR-783**. Pyrite only. Qase token: `QASE_API_TOKEN` (see README, "Qase token"). Project code `Pyrite`, base `https://api.qase.io/v1`, header `Token: $QASE_API_TOKEN`. Never print the token.

**Session start:** follow Helix `references/session-start-context.md` (installed with Helix).

## Step 1 — Find the plan

1. Key = `PYR-<number>`. Number missing → ask one line `PYR-?`. Non-PYR key → refuse, one line.
2. `GET /plan/Pyrite?limit=100&offset=…` (page through). Match the plan whose title equals `PYR-xxx` or starts with `PYR-xxx ` (with the space).
3. No plan → stop: `ไม่พบ Test plan ของ PYR-xxx` and point to `/tc-pyr xxx`. Never create a plan here.
4. Read the plan (`GET /plan/Pyrite/{id}`); list its cases (id, Test Title, Type) in plan order.

## Step 2 — Test run

- **Run title = the Test plan title, character for character** (`PYR-783 <ticket name>`).
- Look for an active run with that title (`GET /run/Pyrite?search=PYR-xxx&status=active`, exact title match). Found → reuse it (continue only cases without a result). Missing → `POST /run/Pyrite` with `{"title": <plan title>, "plan_id": <plan id>}`. Never two runs with the same title open at once.
- Verify the run holds the same case count as the plan. Report run id and link `https://app.qase.io/run/Pyrite/{run_id}`.
- Do not complete or abort the run on your own. Leave it active for review unless the user says to complete it.

## Step 3 — Run each case

Rules, environment and recording: [references/video-evidence.md](references/video-evidence.md). Execution rules (real UI steps, Expected per step, no shortcut through an API): same as Helix execute flow.

For every case in plan order:

1. Read the case: Precondition, Test Data, Steps, Expected Result (from Qase; the Drive TC sheet is the same data).
2. Record an mp4 of the whole case (video-evidence.md). **File name = the case's Test Title** + `.mp4`.
3. Judge per step against Expected Result: `passed`, `failed`, `blocked` (environment / data / precondition prevents the run) or `skipped` (not run).
4. Before recording `failed`, re-run once and rule out data / environment causes; keep the video of the final attempt.
5. Upload the mp4 (video-evidence.md, "Store") and get its Drive link.
6. Post the result: `POST /result/Pyrite/{run_id}` with
   `{"case_id": …, "status": "passed|failed|blocked|skipped", "comment": "<actual result>\n<Drive mp4 link>", "time_ms": …}`.
   The **actual result** text = one or two plain lines of what was seen, then the link on its own line. For `failed`, say which step and what differed. A link only, with no text, is not enough.
7. Re-read the result (`GET /result/Pyrite/{run_id}` or the run page) once per run and confirm every case has a result and a link.

Do not file bugs automatically. For failed cases ask once at the end whether to hand them to `create-bug-workflow`.

## Rules

- MUST start every question to the PO or anyone else with its source label: `AC_xx:`, `EC_xx:` or `Business Rule: <name>` (several sources: list each; none, e.g. prototype only: `Prototype: <screen>`). Same text in chat, gate and any post. Never ask an unlabelled question.
- MUST name the run exactly like the Test plan.
- MUST name each video exactly like its Test Title; mp4 only.
- MUST put the Drive mp4 link into the actual result (comment) of every case that was run.
- MUST NOT mark a case passed without running its steps.
- MUST NOT write the token or any password in Qase, Drive, video, repo or chat.
- MUST NOT edit TCs or the plan here. A TC that looks wrong → report it, route to `/tc-pyr xxx update`.
- No Jira comment from this skill.
- If the run starts to take long, say how many cases are done / left in one line; do not narrate each step.

## Language

Explain, ask and answer in Thai, always (questions, options, reports, confirmations). Code, commands, paths, error text and ticket terms stay exact. Text that a skill posts or writes (TC content, Jira comment, Discord messages, Qase results) follows the format and language that skill defines, not this rule.

## Report (chat, short)

Run id + link, counts (passed / failed / blocked / skipped), failed list with Test Title, anything not run and why.

## QA closing

Follow Helix `references/qa-closing-shared.md`, plus:

- [ ] Run title equals plan title; run case count equals plan case count.
- [ ] Every case run has a result, a video named by Test Title, and the link in its actual result.
- [ ] No secret in any video, result or file.
