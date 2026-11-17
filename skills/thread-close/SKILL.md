---
name: thread-close
description: Close the finished TC-review threads in the QA Discord channel — `/thread-close`. Scans the channel for threads you created in which someone wrote pass, passed, reviewed, approve or approved (any letter case, whole word), closes (archives) the thread and puts a green check emoji on the thread's post in the channel (the channel message the thread hangs from, not a message inside the thread). `/thread-close dry` only lists what it would close. Use when the user types /thread-close or asks to clean up, close or sweep the Passed / Approved threads on Discord. Do NOT use for creating threads (tc-pyr), replying Passed (tc-pyr review), or any other channel.
argument-hint: "[dry]"
metadata:
  proactive_triggers:
    - /thread-close
    - ปิด thread
---

# /thread-close — close the threads that are done

Each `/tc-pyr xxx` creates a Discord thread `PYR-xxx <Jira name>` in the QA channel for TC review. When the review is answered with a pass word, the thread is finished. This skill closes those threads and marks each one with a green check in the channel list.

**Session start:** follow Helix `references/session-start-context.md` (installed with Helix).

## Step 1 — Setup

- Channel: `DISCORD_QA_CHANNEL` from the `Config` tab of the Tracking sheet. Missing → ask once, never guess.
- "Yourself": the Discord username of the person running the skill, from `MENTION_SAMORN` / `MENTION_NIPIT` / `MENTION_SUJAREE` in `Config` (matched through the Atlassian user, as in `tc-pyr` "Create sign-off"). If the threads are created through the Discord MCP's own account, that account is the owner instead; ask once which one when no thread matches.
- Needs the Discord MCP. If it is not connected, say so in one line and stop. Never use another route.

## Step 2 — Find the threads

1. List the **active** (not archived) threads in the channel that **you created**. Threads made by other people are never touched.
2. For each thread read its messages (the thread's first post and every reply).
3. A thread **matches** when at least one message contains one of these words: `pass`, `passed`, `reviewed`, `approve`, `approved`.
   - Letter case does not matter: `Pass`, `PASSED`, `pAsSeD` are the same word.
   - **Whole word only.** `password`, `bypass`, `approver`, `unreviewed` do not match.
   - Messages count from anyone in the thread, including replies from reviewers.
4. **Negation guard.** If the matching message also contains a negation or failure word (`not`, `no`, `never`, `fail`, `failed`, `reject`, `rejected`, `ไม่`, `ยัง`), do not close the thread. List it as skipped with the message quoted, so the user can decide.
5. Several matches in one thread count as one thread.

## Step 3 — Close each matching thread

For every matching thread, in this order:

1. **Close the thread = archive it** (not lock, not delete). Archiving is reversible: anyone can reopen it by posting.
2. **Put the green check emoji ✅ (`white_check_mark`) on the thread's post in the channel itself** (the channel view of `DISCORD_QA_CHANNEL`): the channel message that the thread hangs from, which shows the thread card under it. **Not inside the thread**: never react to a message in the thread. A reaction on a channel message is accepted even after the thread is archived.
3. If the thread has no post in the channel to react to (for example a forum-style thread), close it, say `ไม่มีโพสต์ในช่องให้ใส่ ✅` in the report, and do not put the emoji anywhere else.
4. If the reaction already exists, do not add it twice. If archiving fails, do not add the emoji, report the thread as failed and continue with the next one. If only the emoji fails, report that the thread is closed without ✅.

`/thread-close dry` only lists what it would close. Use when the user types /thread-close or asks to clean up, close or sweep the Passed / Approved threads on Discord. Do NOT use for creating threads (tc-pyr), replying Passed (tc-pyr review), or any other channel.
argument-hint: "[dry]"
metadata:
  proactive_triggers:
    - /thread-close
    - ปิด thread
---

# /thread-close — close the threads that are done

Each `/tc-pyr xxx` creates a Discord thread `PYR-xxx <Jira name>` in the QA channel for TC review. When the review is answered with a pass word, the thread is finished. This skill closes those threads and marks each one with a green check in the channel list.

**Session start:** follow Helix `references/session-start-context.md` (installed with Helix).

## Step 1 — Setup

- Channel: `DISCORD_QA_CHANNEL` from the `Config` tab of the Tracking sheet. Missing → ask once, never guess.
- "Yourself": the Discord username of the person running the skill, from `MENTION_SAMORN` / `MENTION_NIPIT` / `MENTION_SUJAREE` in `Config` (matched through the Atlassian user, as in `tc-pyr` "Create sign-off"). If the threads are created through the Discord MCP's own account, that account is the owner instead; ask once which one when no thread matches.
- Needs the Discord MCP. If it is not connected, say so in one line and stop. Never use another route.

## Step 2 — Find the threads

1. List the **active** (not archived) threads in the channel that **you created**. Threads made by other people are never touched.
2. For each thread read its messages (the thread's first post and every reply).
3. A thread **matches** when at least one message contains one of these words: `pass`, `passed`, `reviewed`, `approve`, `approved`.
   - Letter case does not matter: `Pass`, `PASSED`, `pAsSeD` are the same word.
   - **Whole word only.** `password`, `bypass`, `approver`, `unreviewed` do not match.
   - Messages count from anyone in the thread, including replies from reviewers.
4. **Negation guard.** If the matching message also contains a negation or failure word (`not`, `no`, `never`, `fail`, `failed`, `reject`, `rejected`, `ไม่`, `ยัง`), do not close the thread. List it as skipped with the message quoted, so the user can decide.
5. Several matches in one thread count as one thread.

## Step 3 — Close each matching thread

For every matching thread, in this order:

1. Add the green check emoji ✅ (`white_check_mark`) as a reaction on the thread's **first post** (the starter post). Do this first, because Discord does not accept new reactions in an archived thread.
2. **Close the thread = archive it** (not lock, not delete). Archiving is reversible: anyone can reopen it by posting.
3. If the reaction already exists, do not add it twice. If archiving fails, keep the reaction, report the thread as failed, and continue with the next one.

`/thread-close dry`: do Step 1 and 2 only, then show the list and stop. Nothing is changed.

## Rules

- MUST touch only active threads that you created, in the QA channel.
- MUST match whole words, case-insensitive, and apply the negation guard.
- MUST archive the thread and put ✅ on its post in the channel, never inside the thread. MUST NOT delete a thread or a message, lock a thread, or post any message.
- MUST NOT change anything outside Discord (no Jira, Qase or sheet writes).
- One run closes each thread once. A thread already archived is not listed.
- **Language:** explain, ask and answer in Thai, always (questions, options, reports, confirmations). Code, commands, paths, error text and ticket terms stay exact.

## Report (chat, short)

A short list: closed threads (name and link), skipped threads with the quoted message and why, failed ones with the step that failed, and the counts (scanned / closed / skipped / failed).

## QA closing

- [ ] Only your own active threads in the QA channel were looked at.
- [ ] Each closed thread has a whole-word match and no negation, is archived, and has ✅ on its post in the channel (not inside the thread).
- [ ] Skipped and failed threads are listed with their reason.
