---
name: tc-close
description: Close the finished TC-review threads in the QA Discord channel — `/tc-close`. Scans the channel for threads you created in which someone wrote pass, passed, reviewed, approve or approved (any letter case, whole word), adds a green check emoji to the thread's first post and closes (archives) the thread. `/tc-close dry` only lists what it would close. Use when the user types /tc-close or asks to clean up, close or sweep the Passed / Approved threads on Discord. Do NOT use for creating threads (tc-pyr), replying Passed (tc-pyr review), or any other channel.
argument-hint: "[dry]"
metadata:
  proactive_triggers:
    - /tc-close
    - ปิด thread
---

# /tc-close — close the threads that are done

Each `/tc-pyr xxx` creates a Discord thread `PYR-xxx <Jira name>` in the QA channel for TC review. When the review is answered with a pass word, the thread is finished. This skill closes those threads and marks them with a green check.

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

`/tc-close dry`: do Step 1 and 2 only, then show the list and stop. Nothing is changed.

## Rules

- MUST touch only active threads that you created, in the QA channel.
- MUST match whole words, case-insensitive, and apply the negation guard.
- MUST add ✅ to the first post and archive the thread. MUST NOT delete a thread or a message, lock a thread, or post any message.
- MUST NOT change anything outside Discord (no Jira, Qase or sheet writes).
- One run closes each thread once. A thread already archived is not listed.
- **Language:** explain, ask and answer in Thai, always (questions, options, reports, confirmations). Code, commands, paths, error text and ticket terms stay exact.

## Report (chat, short)

A short list: closed threads (name and link), skipped threads with the quoted message and why, failed ones with the step that failed, and the counts (scanned / closed / skipped / failed).

## QA closing

- [ ] Only your own active threads in the QA channel were looked at.
- [ ] Each closed thread has a whole-word match and no negation, ✅ on its first post, and is archived.
- [ ] Skipped and failed threads are listed with their reason.
