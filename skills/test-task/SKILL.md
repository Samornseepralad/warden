---
name: test-task
description: Test one Pyrite Jira card by its AC/EC — `/test-task PYR-783` (or `/test-task 783`). Plans the tests from AC/EC plus business rules and other details, shows a short plan table in chat, waits for Approve, then runs headless Playwright, records one mp4 per AC_xx / EC_xx, posts the result as a Jira comment (bullets with mp4 links only) and moves the card (Passed → Done, Failed → In Progress). Use when the user types /test-task or asks to test a PYR card on dev. Do NOT use for OLS cards, for writing TCs (tc-pyr), for running a Qase Test plan (qase-run), or for filing bugs.
argument-hint: "<PYR-xxx | number>   e.g. 783"
metadata:
  proactive_triggers:
    - /test-task
    - test card PYR
---

# /test-task — test one Pyrite card by AC/EC

`/test-task 783` = test **PYR-783** against its AC/EC on dev and report in Jira. Pyrite only (OLS cards use Helix `testing-ticket-workflow`).

**Session start:** follow Helix `references/session-start-context.md` (installed with Helix).

## Step 1 — Ticket and plan

1. Key = `PYR-<number>`. Missing → ask one line `PYR-?`. Non-PYR key → refuse, one line.
2. Read the Jira card: summary, AC/EC, business rules, other details, assignee, status, comments. Read the prototype (and Figma / PRD only if linked; absent = absent, never ask) as in `tc-pyr` references/pyr.md "Ticket review" if the tc-pyr skill is installed.
3. Scope check: apply `tc-pyr` references/pyr.md "Scope check" (section "ไม่อยู่ใน ticket นี้:"). AC/EC that belong to another card or exceed the ticket are **not planned**; list them as short bullets under the plan table (`ต้องถาม PO`) and test them only if the user includes them at Approve.
4. Plan one test per in-scope AC and EC (`AC_01`, `EC_01`, … exactly as the ticket labels them). Anything in business rules or details that adds behavior and has no AC/EC → extra planned rows, marked `extra`.

## Step 2 — Show the plan, wait for Approve (always)

Show a **short table in chat**, nothing else long:

| ID | What to check (one line) | Source |
|---|---|---|
| AC_01 | … | AC |
| EC_01 | … | EC |
| X_01 | … | business rule (extra) |

Under the table, the `ต้องถาม PO` bullets (if any), one line: environment, and open points if any. Then ask once: `Approve / แก้ไข?`

- **Approve** → start testing.
- **แก้ไข** → change the table, show it again, ask again.
- **No approval = no test.** Silence or a question is not approval. Approval for one card never covers another.

## Step 3 — Test

Rules, environment, recording: [references/video-evidence.md](references/video-evidence.md). Real UI steps, no shortcut through an API for the action under test.

For each planned row, in plan order:

1. Run it, record **one mp4**, file name exactly the row ID: `AC_01.mp4`, `EC_02.mp4` (extra rows: `X_01.mp4`).
2. Judge `Passed` or `Failed`. Before `Failed`, re-run once and rule out data / environment causes. Keep the video of the final attempt.
3. Upload the mp4 to Drive and get its link (video-evidence.md, "Store").
4. Blocked by environment / VPN / account (not a product result): stop that row, do not record Passed or Failed, say so in chat and ask.

## Step 4 — Report in Jira, move the card

Needs the reviewer list and Drive folder from the Tracking sheet `Config` tab (keys `QA_REVIEWERS` = e-mails, `VIDEO_FOLDER`). Missing → ask once. Mention people with Jira accountIds (look up by e-mail with `lookupJiraAccountId`); never write e-mails in plain text in the comment.

The comment is **bullets with `.mp4` links only** (no screenshots, no long prose).

**All Passed**
```
Test on dev: Passed
• AC_01: <mp4 link>
• AC_02: <mp4 link>
• EC_01: <mp4 link>
need review results: @reviewer1 @reviewer2 @reviewer3
```
Then transition the card to **Done**.

**Any Failed**
```
Test Failed
• AC_02: <mp4 link of the failing run>
• (every other row as a bullet with its mp4 link, Passed or Failed)
@assignee
```
Tag the card's assignee (accountId from the card). Transition the card back to **In Progress**. Say in the first line of each failed bullet which row failed (`AC_02 Failed:`), still with the mp4 link only as evidence.

Post the comment first, then transition. If either call fails, say which and stop; do not retry blindly. Report in chat: counts, comment link, new status.

## Rules

- MUST start every question to the PO or anyone else with its source label: `AC_xx:`, `EC_xx:` or `Business Rule: <name>` (several sources: list each; none, e.g. prototype only: `Prototype: <screen>`). Same text in chat, gate and any post. Never ask an unlabelled question.
- MUST show the plan table and get Approve before any test starts.
- MUST name videos `AC_xx` / `EC_xx` (`X_xx` for extra), mp4 only.
- MUST post bullets with mp4 links only; no screenshots or webm.
- MUST move the card: Passed → Done, Failed → In Progress. Never Done with any Failed row.
- MUST NOT test or post for a card the user did not name in the command.
- MUST NOT write passwords, tokens or e-mails of reviewers into the comment text, repo or chat. Videos must not show them.
- MUST NOT file bugs automatically; for Failed, ask once whether to hand it to `create-bug-workflow`.
- The PYR rule "no Jira comment" belongs to `tc-pyr` (TC prep). Here the Jira comment is the deliverable.

## Language

Explain, ask and answer in Thai, always (questions, options, reports, confirmations). Code, commands, paths, error text and ticket terms stay exact. Text that a skill posts or writes (TC content, Jira comment, Discord messages, Qase results) follows the format and language that skill defines, not this rule.

## QA closing

Follow Helix `references/qa-closing-shared.md`, plus:

- [ ] Plan table was approved before the first test.
- [ ] One mp4 per row, named by its ID, link opens (checked once).
- [ ] Comment matches the Passed / Failed format; right people tagged.
- [ ] Card status matches the result.
