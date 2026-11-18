---
name: focus-target-update
description: Report the daily target result of the QA running it — `/focus-target-update` (today) or `/focus-target-update 17-Nov`. Reads that day's Target rows from the Focus work sheet (read-only), collects the QA's evidence from Jira (TC done, TC reviewed = QA REVIEWED, RTT retest, test story Done) and from the Discord review channel (review requests sent, Reviewed), judges each target row (ได้ตามเป้า / บางส่วน / ไม่ได้ตามเป้า), shows the table, waits for Approve, then posts one bullet message in the Discord Target channel and checks it is visible. Use when the user types /focus-target-update or asks to report or update the daily target (อัปเดตเป้า, แจ้งเป้าวันนี้). Do NOT use for other people's results, for writing any Google Sheet or Jira, or for creating Target rows.
argument-hint: "[date]   e.g. 17-Nov (default today)"
metadata:
  proactive_triggers:
    - /focus-target-update
    - อัปเดตเป้า
    - แจ้งเป้าวันนี้
    - update daily target
---

# /focus-target-update — report the daily target

Reply in the user's language (Thai). Never fill a number from memory: if a source fails, say "ดึงไม่ได้" instead of showing 0.
The only write is the Discord post, and it needs the user's explicit approval of the exact text in chat first. **Never write the Google Sheet or Jira.**

## Settings (Tracking sheet `Config` tab, same place as the other warden skills)

Team-private values stay out of this public repo. Keys: `TARGET_SHEET` (URL of the Focus work sheet), `TARGET_TAB` (tab with the daily targets), `TARGET_DISCORD_CHANNEL` (channel URL where results are posted), `REVIEW_DISCORD_CHANNEL` (channel URL where TC review requests are posted as threads). Missing key → ask once.

Target tab layout: column A date, column B target text, then one column per QA headed `QA - <full name>` (use the column of the person running the skill, matched against their Jira display name; read it for reference only), and a Note column.

Team rule: reply **before 18:00 of that day, no retroactive updates**.

Discord has no plugin here, so read and post through the logged-in Chrome tab (osascript JS for reading, cmux-cua for key presses). Never click voice channels.

**Prerequisite (once per machine):** in Chrome menu *View → Developer → Allow JavaScript from Apple Events* must be on, otherwise the osascript JS calls fail; cmux-cua needs its macOS permissions granted.

## 1. Day

`TZ=Asia/Bangkok date`. Day = today unless the user names another. Window = that day 00:00–24:00 Asia/Bangkok.
- Before 18:00 today: warn that the result may be incomplete.
- After 18:00, or a past day: warn about the no-retroactive rule; post only if the user explicitly says to post after that warning.

## 2. Read the Target rows

`get_values` on the target tab; keep rows where the date equals the day. No rows → report and stop (never create rows). Parse the number from the target text ("อย่างน้อยคนละ 3 ใบ" → 3; a range "1-2 ใบ" → lower bound, say so in the table). Rows ending "ถ้ามี" are optional.

Target kinds: **Clear RTT**, **TC Prep** (TC ready / sent to review), **Review TC** (the user reviewed someone's TC), **TC Prep และส่งรีวิวให้ผ่าน** (prepared and review passed).

## 3. Evidence (the user's own, inside the window)

**Jira** (Atlassian connector; `atlassianUserInfo` for the user). JQL dates follow the Jira profile time zone, not Asia/Bangkok, so use a wide net — `<start>` = day − 1, `<end>` = day + 2 — and keep only changelog entries whose `created` (converted to Asia/Bangkok) is inside the window. Three queries, merged (verified on this Jira: `CHANGED BY` works on `status` but **not** on the custom field `TC Status` — it returns Bad Request — and the JQL name of that field needs the type suffix, `"TC Status[Dropdown]"`):
```
"QA Owner[People]" = currentUser() AND updated >= "<start>" AND updated < "<end>" ORDER BY updated DESC
status CHANGED BY currentUser() DURING ("<start>", "<end>")
"TC Status[Dropdown]" IN ("DONE", "QA REVIEWED") AND updated >= "<start>" AND updated < "<end>"
```
The second finds cards whose status the user moved but does not own; the third finds TC Status moves on cards the user does not own (e.g. TCs the user reviewed). The queries cannot filter by author for `TC Status`, so they are only candidate nets: the author filter is always the changelog check below. For each candidate `getJiraIssue` with `expand=changelog,names` (large results are saved to a file; filter with `jq` on `author.accountId` = the user and `created` inside the window). Field names to look for (ids come from `names`): `TC Status`, `Test Result Approval by QA`.
- **TC Prep** — `TC Status` → `DONE`, or the user's comment with a TC table
- **TC reviewed** — `TC Status` → `QA REVIEWED`
- **RTT cleared** — `Test Result Approval by QA` → `PASSED`/`FAILED`/`SKIPPED`, or status → Done after retest
- **Test story Done** — status → Done by the user
- Bugs the user opened: `reporter = currentUser() AND issuetype = Bug AND created …`

**Discord review channel** (read-only): each review request is a "started a thread: <CARD> …" message. Count (a) threads the user started to send a TC for review → TC Prep "ส่งรีวิว"; (b) review requests addressed to the user that the user answered with Reviewed → Review TC; (c) reviewer replies Reviewed / ผ่าน / approved on the user's threads.
Recipe: JS-click the thread link (`a[role=link]` whose text starts with the card id) → the URL becomes `…/threads/<id>` and the page lists that thread's messages (skip the "started a thread" rows). In AppleScript JS strings never write `\n` (it becomes a real newline) — use `String.fromCharCode(10)`, and do not name a variable `key`. `time[datetime]` is **UTC → add 7 h for Asia/Bangkok**; a grouped follow-up message has no username and belongs to the previous author (mark it unconfirmed). `Reviewing` = started only; `Reviewed` = done. Report the Thai time of the user's replies. Replies after 18:00 are still counted but flagged `late` in the step 5 table; the user decides there whether to keep or drop them. Afterwards put the tab back on the target channel URL.

## 4. Decide per row

Count = evidence items of that row's kind vs the parsed target N:
- count ≥ N → ✅ ได้ตามเป้า
- 1 ≤ count < N → ⚠️ ได้ตามเป้าบางส่วน
- count = 0 → ❌ ไม่ได้ตามเป้า
- optional ("ถ้ามี") row with no eligible work → omit
- "ส่งรีวิวให้ผ่าน" rows: count only TCs prepared **and** Reviewed.
- A card counts **once per target kind**: a TC that is Done and also sent to review is one TC Prep, not two.
- A row whose source could not be read ("ดึงไม่ได้") is **not judged** (not 0, not ❌): show it as "ดึงไม่ได้" and wait for the user to decide before the overall result and the draft.
- If no row is left after omitting optional ones, report that and stop.

Overall: every shown row ✅ → **ได้ตามเป้าทั้งหมด**; none ✅ and none ⚠️ → **ไม่ได้ตามเป้า**; otherwise **ได้ตามเป้าบางส่วน**.

## 5. Confirm the judgement

Table in chat: `แถวเป้า | เป้า | ได้ | หลักฐาน (card / ลิงก์ / เวลาไทย) | ผล`, plus anything "ดึงไม่ได้" or unconfirmed. Ask the user to fix unclear items before drafting.

## 6. Discord (the only write)

Draft one message: first line `<first name> <d Mon>: <overall result>` (first name taken from the user's target column header `QA - <full name>`, so it matches the sheet), then one bullet per target row of that day in sheet order: the row's target text, `(<ได้>/<เป้า>)`, the verdict. Under it one bullet per card, written as the **card id as a Discord masked link** `[ABC-123](<jira base>/browse/ABC-123)` (never a bare long URL; Jira base from the Atlassian connector) followed by `<what> = <status>` (TC = Done / ส่งรีวิว / Reviewed / Test done = Done|Pass|Fail|Block / Testing = In progress). Review TC rows too: one bullet per reviewed card, never several ids on one line. A thread title without a card id: list the title as-is, no link. No evidence: "- ไม่มีงาน" (or "ดึงไม่ได้"). **No reason lines.** Ratios and verdicts must match the step 5 table.

```
Jane 17 Nov: ได้ตามเป้าบางส่วน
- Clear RTT 1-2 ใบ (1/1) — ✅ ได้ตามเป้า
  - [ABC-101](https://<jira>/browse/ABC-101) Test done = Done
- TC Prep อย่างน้อย 3 ใบ (1/3) — ⚠️ ได้ตามเป้าบางส่วน
  - [ABC-205](https://<jira>/browse/ABC-205) TC = Done
- Review TC อย่างน้อย 3 ใบ (3/3) — ✅ ได้ตามเป้า
  - [ABC-310](https://<jira>/browse/ABC-310) Reviewed
  - [ABC-311](https://<jira>/browse/ABC-311) Reviewed
  - [ABC-312](https://<jira>/browse/ABC-312) Reviewed
```

Show the draft, wait for Approve, then post it and verify it is visible in the channel. Posting and editing recipes (the obvious ways fail, so follow them exactly): [references/discord-posting.md](references/discord-posting.md). Retry rule: box still holds the text → press Return again (max 3 tries); box empty but message not visible → do **not** resend, reload the channel first.

## Quick reference

| Situation | Do |
|---|---|
| Count ≥ target / 1…target−1 / 0 | ✅ ได้ตามเป้า / ⚠️ บางส่วน / ❌ ไม่ได้ตามเป้า |
| Optional row ("ถ้ามี") with no work | Omit it |
| Source unreadable | Show "ดึงไม่ได้", do not judge, wait for the user |
| Run after 18:00 or for a past day | Warn (no retroactive); post only on explicit confirm |
| Reply after 18:00 | Count it, flag `late`, user decides |
| Card id format | `[ABC-123](<jira base>/browse/ABC-123)`, one card per bullet |
| Sheet / Jira | Read only, never write |

## Common mistakes

- Treating an unreadable source as 0 → wrong ❌. Show "ดึงไม่ได้" instead.
- Counting one card twice in the same target kind (Done and also sent to review).
- `"TC Status" CHANGED BY …` in JQL → Bad Request; use `status CHANGED BY` or `"TC Status[Dropdown]" IN (…)` and filter the changelog by author.
- Filling or clearing the Discord box with JS → Discord's editor goes out of sync and Return sends nothing. Use the clipboard and real keys (references/discord-posting.md).
- Resending when the box is empty but the message is not visible → duplicate. Reload and look first.
- Pressing Return on an empty edit box → Discord offers to delete the message.
- Adding a reason line, or several card ids on one line.

## Output

Short note in Thai: the verdict per target row, items "ดึงไม่ได้", the Discord message link or "ยังไม่ได้โพสต์". The sheet is left for the team.
