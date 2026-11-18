---
name: focus-target-update
description: |
  Post my daily target result in the team's Discord Target channel. Reads that day's Target rows from the Focus work sheet (read-only), collects my evidence from Jira (TC done, TC reviewed = QA REVIEWED, RTT retest, test story Done) and from the Discord review channel (TC review requests sent, Reviewed), judges each target row (ได้ตามเป้า / บางส่วน / ไม่ได้ตามเป้า), and after I approve posts one bullet message. Never writes the sheet; the team fills it from the Discord reply.
  Use when I say อัปเดตเป้า, แจ้งเป้าวันนี้, update target, /focus-target-update [date].
  Do NOT use for other people's results, for writing any Google Sheet, or for creating Target rows.
---

# Focus target update

Reply in my language (Thai). Never fill a number from memory: if a source fails, say "ดึงไม่ได้" instead of showing 0.
The only write is the Discord post, and it needs my explicit approval of the exact text in chat first. **Never write the Google Sheet or Jira.**

## Settings (Tracking sheet `Config` tab, same place as the other warden skills)

Team-private values stay out of this public repo. Keys: `TARGET_SHEET` (URL of the Focus work sheet), `TARGET_TAB` (tab with the daily targets), `TARGET_DISCORD_CHANNEL` (channel URL where results are posted), `REVIEW_DISCORD_CHANNEL` (channel URL where TC review requests are posted as threads). Missing key → ask once.

Target tab layout: column A date, column B target text, then one column per QA headed `QA - <full name>` (use the column of the person running the skill, matched against their Jira display name; read it for reference only), and a Note column.

Team rule: reply **before 18:00 of that day, no retroactive updates**.

Discord has no plugin here, so read and post through the logged-in Chrome tab (osascript JS for reading, cmux-cua for key presses). Never click voice channels.

**Prerequisite (once per machine):** in Chrome menu *View → Developer → Allow JavaScript from Apple Events* must be on, otherwise the osascript JS calls fail; cmux-cua needs its macOS permissions granted.

## 1. Day

`TZ=Asia/Bangkok date`. Day = today unless I name another. Window = that day 00:00–24:00 Asia/Bangkok.
- Before 18:00 today: warn that the result may be incomplete.
- After 18:00, or a past day: warn about the no-retroactive rule; post only if I explicitly say to post after that warning.

## 2. Read the Target rows

`get_values` on the target tab; keep rows where the date equals the day. No rows → report and stop (never create rows). Parse the number from the target text ("อย่างน้อยคนละ 3 ใบ" → 3; a range "1-2 ใบ" → lower bound, say so in the table). Rows ending "ถ้ามี" are optional.

Target kinds: **Clear RTT**, **TC Prep** (TC ready / sent to review), **Review TC** (I reviewed someone's TC), **TC Prep และส่งรีวิวให้ผ่าน** (prepared and review passed).

## 3. Evidence (mine only, inside the window)

**Jira** (Atlassian connector; `atlassianUserInfo` for me). JQL dates follow the Jira profile time zone, not Asia/Bangkok, so use a wide net — `<start>` = day − 1, `<end>` = day + 2 — and keep only changelog entries whose `created` (converted to Asia/Bangkok) is inside the window. Two queries, merged:
```
"QA Owner[People]" = currentUser() AND updated >= "<start>" AND updated < "<end>" ORDER BY updated DESC
"TC Status" CHANGED BY currentUser() DURING ("<start>", "<end>")
```
The second one finds cards I reviewed but do not own (I am not their QA Owner, so the first query misses them). For each candidate `getJiraIssue` with `expand=changelog,names` (large results are saved to a file; filter with `jq` on `author.accountId` = me and `created` inside the window). Field names to look for (ids come from `names`): `TC Status`, `Test Result Approval by QA`.
- **TC Prep** — `TC Status` → `DONE`, or my comment with a TC table
- **TC reviewed** — `TC Status` → `QA REVIEWED`
- **RTT cleared** — `Test Result Approval by QA` → `PASSED`/`FAILED`/`SKIPPED`, or status → Done after retest
- **Test story Done** — status → Done by me
- Bugs I opened: `reporter = currentUser() AND issuetype = Bug AND created …`

**Discord review channel** (read-only): each review request is a "started a thread: <CARD> …" message. Count (a) threads I started to send a TC for review → TC Prep "ส่งรีวิว"; (b) review requests addressed to me that I answered with Reviewed → Review TC; (c) reviewer replies Reviewed / ผ่าน / approved on my threads.
Recipe: JS-click the thread link (`a[role=link]` whose text starts with the card id) → the URL becomes `…/threads/<id>` and the page lists that thread's messages (skip the "started a thread" rows). In AppleScript JS strings never write `\n` (it becomes a real newline) — use `String.fromCharCode(10)`, and do not name a variable `key`. `time[datetime]` is **UTC → add 7 h for Asia/Bangkok**; a grouped follow-up message has no username and belongs to the previous author (mark it unconfirmed). `Reviewing` = started only; `Reviewed` = done. Report the Thai time of my replies. Replies after 18:00 are still counted but flagged `late` in the step 5 table; I decide there whether to keep or drop them. Afterwards put the tab back on the target channel URL.

## 4. Decide per row

Count = evidence items of that row's kind vs the parsed target N:
- count ≥ N → ✅ ได้ตามเป้า
- 1 ≤ count < N → ⚠️ ได้ตามเป้าบางส่วน
- count = 0 → ❌ ไม่ได้ตามเป้า
- optional ("ถ้ามี") row with no eligible work → omit
- "ส่งรีวิวให้ผ่าน" rows: count only TCs prepared **and** Reviewed.
- A card counts **once per target kind**: a TC that is Done and also sent to review is one TC Prep, not two.
- A row whose source could not be read ("ดึงไม่ได้") is **not judged** (not 0, not ❌): show it as "ดึงไม่ได้" and wait for me to decide before the overall result and the draft.
- If no row is left after omitting optional ones, report that and stop.

Overall: every shown row ✅ → **ได้ตามเป้าทั้งหมด**; none ✅ and none ⚠️ → **ไม่ได้ตามเป้า**; otherwise **ได้ตามเป้าบางส่วน**.

## 5. Confirm the judgement

Table in chat: `แถวเป้า | เป้า | ได้ | หลักฐาน (card / ลิงก์ / เวลาไทย) | ผล`, plus anything "ดึงไม่ได้" or unconfirmed. Ask me to fix unclear items before drafting.

## 6. Discord (the only write)

Draft one message: first line `<first name> <d Mon>: <overall result>` (first name taken from my target column header `QA - <full name>`, so it matches the sheet), then one bullet per target row of that day in sheet order: the row's target text, `(<ได้>/<เป้า>)`, the verdict. Under it one bullet per card, written as the **card id as a Discord masked link** `[ABC-123](<jira base>/browse/ABC-123)` (never a bare long URL; Jira base from the Atlassian connector) followed by `<what> = <status>` (TC = Done / ส่งรีวิว / Reviewed / Test done = Done|Pass|Fail|Block / Testing = In progress). Review TC rows too: one bullet per reviewed card, never several ids on one line. A thread title without a card id: list the title as-is, no link. No evidence: "- ไม่มีงาน" (or "ดึงไม่ได้"). **No reason lines.** Ratios and verdicts must match the step 5 table.

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

Show the draft, wait for Approve, then post. **This is the only posting recipe that worked; the obvious ones fail** (cmux `type_text` never lands; JS `execCommand insertText` leaves Discord's editor out of sync so Return clears the box without sending; Return can even switch the tab to another channel):
1. Save my current clipboard (`pbpaste > /tmp/clip.bak`) and tell me it will be replaced, then write the approved text to a file and `pbcopy < file`; check `pbpaste | wc -c`. Restore it at the end (`pbcopy < /tmp/clip.bak`).
2. Open a **dedicated new Chrome tab** on the target channel URL and use only that tab (a shared Discord tab got switched to other servers while working). Chrome pid + window id from cmux `list_windows`; `tell application "Google Chrome" to activate`.
3. Discord restores the last draft after load: read the box length (`[role=textbox]`, empty ≤ 2). If not empty, clear with **real keys** (cmux `hotkey cmd+a`, `press_key delete`, `delivery_mode:"foreground"`, pid + window id) — never clear or insert with JS.
4. JS only to read and focus the box.
5. `hotkey cmd+v` (foreground). Read the box back: exactly one copy, header line first, `location.pathname` still the target channel.
6. **Right before sending re-check `location.pathname`; abort if it is not the target channel.** Then `press_key return` (foreground).
7. **Verify it posted:** after 3–4 s the box is empty AND a message with the header text exists in `li[id^=chat-messages-]` (and `screencapture -x -o -l <windowId>` shows it). Box still holds the text → press Return again (max 3 tries). Box empty but message not visible → do **not** resend: reload the channel and look again; resend only if a reload still shows nothing.
8. Post nowhere else. Restore the clipboard. Leave the dedicated tab open and tell me its position. End the cmux session.

### Editing an already-posted message
Messages are edited, not deleted and reposted (keeps the timestamp). Same dedicated tab, Chrome frontmost.
1. Focus the empty main box (JS `.focus()`), `press_key up` (foreground) → my last message opens in edit mode ("escape to cancel • enter to save"; two `[role=textbox]` exist, the edit one is index 0).
1b. **Check you opened the right message**: the first line of the edit box must be the target-result header (`<first name> <d Mon>: …`). If it is not (something else was posted after it), press Escape and stop.
2. Clear the edit box with `hotkey cmd+shift+up` then `press_key delete` (`cmd+a` does not select there; plain `delete` removes one char). Edit box length ≤ 2.
3. **Never press Return while the edit box is empty** — Discord offers to delete the message.
4. `cmd+v` does not paste in the edit box. Dispatch a synthetic paste so Discord's own handler inserts the text: on `document.querySelectorAll('[role=textbox]')[0]`, `dispatchEvent(new ClipboardEvent('paste',{clipboardData:dt,bubbles:true,cancelable:true}))` with `dt=new DataTransfer(); dt.setData('text/plain', text)`. Build the JS string with Python `json.dumps` so Thai, emoji and newlines survive osascript. Read the box back.
5. `press_key return` (foreground); verify the new text and `(edited)`.

## Output

Short note in Thai: the verdict per target row, items "ดึงไม่ได้", the Discord message link or "ยังไม่ได้โพสต์". The sheet is left for the team.
