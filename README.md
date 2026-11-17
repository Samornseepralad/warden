# warden

Claude skills for QA test-case prep, one skill per project.

| Command | Does |
|---|---|
| `/tc-pyr xxx` | Prepare TCs for **PYR-xxx**: write TCs, show an HTML draft and wait for Approve, then create them in Qase (Pyrite), build the Qase Test plan `PYR-xxx <ticket name>`, fill the Drive TC sheet and tracking sheet |
| `/tc-pyr xxx update` | `updateTCSheet`: sync all, in order: Qase TCs → Test plan → Drive sheet → tracking sheet |
| `/tc-pyr xxx review` | Review TCs against the prototype (layout and flow) and the ticket (all wording), plus the PYR rules then proposes fixes and, on Approve, updates Qase TC, Test plan, Drive sheet and tracking sheet |
| `/tc-pyr xxx run` | Manual execution from the TC sheet |
| `/qase-run xxx` | Run the Qase Test plan of PYR-xxx: Test run named like the plan, one mp4 per case (named by Test Title) on Drive, link in each actual result |
| `/test-task xxx` | Test card PYR-xxx by AC/EC: short plan table, **Approve**, mp4 per AC_xx / EC_xx, Jira comment (mp4 links), move card Done / In Progress |
| `/tc-ols xxxx` | Prepare TCs for **OLS-xxxx** with the OLS rules (Core / Edge, OLS glossary) |
| `/tc-ols xxxx format` | Move OLS TCs into the release template |
| `/tc-ols xxxx run` | Manual execution from the TC sheet |

TC review for OLS, or without PYR rules: Helix `/tc-review`.

## Skills in detail

### `/tc-pyr xxx` — create TCs for PYR-xxx
1. Read the ticket from Jira (AC/EC, business rules, details).
2. **Ticket review**: compare the ticket with the prototype for layout and flow (Figma / PRD only if linked; absent means absent). **All button names and UI text come from the ticket only**; the prototype is a rough picture for developers, not a wording source. **Scope check**: list AC/EC that belong to another card (ticket section "ไม่อยู่ใน ticket นี้:") or go beyond the ticket, as short bullets for the PO.
3. Write the TCs (Helix `tc-fe-prep-workflow` / `tc-api-prep-workflow`) with the PYR rules. Button names and UI text come from the ticket only; the prototype only guides layout and flow. A button the ticket does not name becomes a question for the PO, not a name borrowed from the prototype.
4. **HTML draft** opens for review: TCs, mismatches, a "ต้องถาม PO" section. Reply **Approve** or **แก้ไข**. Nothing is written before Approve.
5. On Approve: Qase TCs → Qase Test plan `PYR-xxx <ticket name>` → Drive TC sheet → tracking sheet.
6. **Sign-off:** Jira field **TC Status** = `DONE`, then a **Discord thread** `PYR-xxx <Jira name>` in the QA channel. First message: one reviewer picked at random from Samorn, Nipit and Sujaree (never the person who created the TCs), `need review tc`, and `TC: Link` to the Drive TC sheet. The thread name and message are shown in the HTML gate before posting.

### `/tc-pyr xxx review` — review the TCs
- Checks the TCs against the **prototype for layout and flow** and against the **ticket for all wording** (button names, UI text) and rules, plus PYR format rules and the Test plan.
- Reports Good vs Need Improve in chat. The review itself writes nothing.
- If some findings are fixable, it drafts the fixes in an HTML and asks once: **update everywhere?** On Approve it updates all four places (same as `update`), so no separate `update` is needed afterwards. Findings that need the PO are listed, never applied.
- **Sign-off (after Approve):** sets the `Summary` tab column F of the ticket's row and the Jira card field **TC Status** to `QA-REVIEWED`, replies `Passed` in the ticket's existing Discord thread (QA channel), and posts the PO review request in the PO channel (`@PO need po review tc`, bullet `PYR-xxx` Jira link + Jira name, `Test Cases Summary: Link`). The exact messages are shown in the HTML gate before posting. Without a Discord connection, the messages are given as text to paste.

### `/tc-pyr xxx update` — sync TC changes everywhere
Updates, in order: Qase TCs → Test plan → Drive sheet → tracking sheet, after an HTML review and Approve.

**After `review` + Approve, do not run `update` again.** `review` already updated all four places; a second run adds nothing.

Use `update` for TC changes that do **not** come from `review`. Most common first:
1. The PO answered about an AC/EC flagged as out of scope, and TCs must be added or removed (the most frequent case, because the scope check usually leaves questions for the PO).
2. The ticket was changed later (AC/EC changed after the TCs were created or reviewed).
3. You edited some TCs by hand (file or Qase) and want everything synced.

| | `review` | `update` |
|---|---|---|
| Purpose | Check the whole TC set for quality | Sync a change you already know about |
| Walks every TC in the prototype | Yes | No |
| Updates the four places | Yes, after Approve (when there are fixable findings) | Yes, after Approve |
| Use after the other | Never needed | Never needed after an approved review |

### `/qase-run xxx` — run the Test plan
- Finds the plan of PYR-xxx (never creates one) and creates or reuses a Test run **named exactly like the plan**.
- Runs each case headless, records one **mp4 per case named by its Test Title**, uploads it to the Drive folder (`VIDEO_FOLDER`), and posts the result in Qase with the Drive link in the actual result.
- Does not complete the run and does not file bugs on its own.

### `/test-task xxx` — test one card by AC/EC
- Plans tests from AC/EC (business rules and details become extra rows; out-of-scope items are listed for the PO).
- Shows a **short plan table in chat** and waits for **Approve** before any test.
- Records one mp4 per AC_xx / EC_xx, posts a Jira comment with bullets and mp4 links only.
- Passed: `Test on dev: Passed`, tag the reviewers (`QA_REVIEWERS`), move the card to **Done**. Failed: `Test Failed` with mp4, tag the assignee, move the card back to **In Progress**.

### `/tc-ols xxxx` — OLS
Create TCs with the OLS rules (Core / Edge, OLS glossary), `format` (move into the release template) and `run`. No Qase Test plan for OLS yet.

## Layout

```
skills/tc-pyr/   SKILL.md + references/{pyr,execute}.md
skills/tc-ols/   SKILL.md + references/{ols,execute}.md
skills/qase-run/ SKILL.md + references/video-evidence.md
skills/test-task/ SKILL.md + references/video-evidence.md   (video-evidence.md copies must stay identical)
```

## Prerequisite: Helix

**Install Helix first.** The skills delegate to Helix workflows: `tc-fe-prep-workflow`, `tc-api-prep-workflow`, `tc-change-format-workflow` (OLS), `create-bug-workflow`, `retest-bug-workflow`, and use its reference files (`session-start-context.md`, `qa-closing-shared.md`).

Helix: https://github.com/Thitic9203/helix (install steps are in its README).

## Install

Symlink every skill folder (run again after pulling a release that adds skills):

```bash
for d in "$PWD"/skills/*; do ln -sfn "$d" ~/.claude/skills/$(basename "$d"); done
```

Upgrading from the old single `tc-by-project` skill: remove its link first (`rm ~/.claude/skills/tc-by-project`).

## Auto-update (set up once)

GitHub cannot push files into your machine, so each person lets Claude Code pull `main` at every session start. After this, every merged change reaches you without any manual step.

Add to `~/.claude/settings.json` (merge into an existing `hooks` block if you have one). Replace the path with where you cloned this repo:

```json
{
  "hooks": {
    "SessionStart": [
      {
        "hooks": [
          {
            "type": "command",
            "command": "git -C \"$HOME/GitHub/warden\" pull --ff-only -q >/dev/null 2>&1 || true"
          }
        ]
      }
    ]
  }
}
```

- Keep your clone on `main` with no local edits. A fast-forward-only pull never overwrites local work; if it cannot pull, it silently skips.
- Because the skills are symlinks into the clone, the pulled files are used right away.
- Not on the latest? Run `warden-up` (below) or `git -C <your-clone> pull` by hand.

### Quick update: `warden-up`

Add to `~/.zshrc` (or `~/.bashrc`). Pulls the latest `main` and links any **new** skill folders (the hook above does not link new skills):

```bash
alias warden-up='git -C ~/GitHub/warden pull --ff-only && for d in ~/GitHub/warden/skills/*; do ln -sfn "$d" ~/.claude/skills/$(basename "$d"); done'
```

Use both: the hook for the routine pull, `warden-up` when a release adds a skill or when you want it right now.

### Limits
- Setup is once per person and per machine; nobody can set it up for another machine.
- macOS and Linux with zsh or bash. Windows needs different commands.
- The hook and alias assume the clone is at `~/GitHub/warden`. Change the path if yours differs.
- `--ff-only` never overwrites local work. If the clone is not on `main` or has local edits, it does nothing.
- Each machine also needs its own `ffmpeg`, Playwright, Qase token and prototype password file.

### After updating: which tab sees it
- **Claude Code sessions**: all sessions on the machine read the same files (skills are symlinks). An open session picks up edited content of an existing skill the next time it is invoked. A **new skill** or a changed skill **description** needs a new session, because the skill list is loaded at session start. The hook runs at session start only. When unsure, restart Claude Code.
- **Terminal tabs**: the alias applies to tabs opened after you edit `~/.zshrc`; in an open tab run `source ~/.zshrc` first.

## Team settings in the Tracking sheet

Team-private values stay out of this public repo. Add to the `Config` tab (Key, Value):

| Key | Value |
|---|---|
| `VIDEO_FOLDER` | Drive folder for mp4 evidence (`/qase-run`, `/test-task`) |
| `DISCORD_QA_CHANNEL`, `DISCORD_PO_CHANNEL` | Discord channels for the QA thread reply and the PO review request (`/tc-pyr xxx review`) |
| `MENTION_SAMORN`, `MENTION_NIPIT`, `MENTION_SUJAREE` | Discord username (or `<@id>` mention) of each QA member (`/tc-pyr xxx`); a username is resolved to a real mention at post time |
| `PO_MENTIONS` | PO to mention in the review request: Discord usernames, `<@id>` or role mentions, space separated (`/tc-pyr xxx review`) |
| `QA_REVIEWERS` | E-mails to tag for review, comma separated (`/test-task`) |
| `PROTOTYPE_URL`, `PROTOTYPE_USER` | Pyrite prototype (see below) |

Videos need `ffmpeg` and Playwright installed locally (`brew install ffmpeg`).

## Pyrite prototype login

`tc-pyr` compares each ticket with the Pyrite prototype. The URL and username live in the Tracking sheet `Config` tab (`PROTOTYPE_URL`, `PROTOTYPE_USER`). The password stays on your machine only; ask the team for it and never paste it in chat, sheet, Jira or this repo.

```bash
touch ~/.config/pyr-prototype.env && chmod 600 ~/.config/pyr-prototype.env
printf 'export PYR_PROTOTYPE_USER=<username>\nexport PYR_PROTOTYPE_PASSWORD=<password>\n' > ~/.config/pyr-prototype.env
```

Load it in `~/.zshrc` like the Qase token: `[ -f ~/.config/pyr-prototype.env ] && . ~/.config/pyr-prototype.env`.

## Qase token

`tc-pyr` creates PYR test cases and Test plans in Qase and needs your own Qase API token. Never share it in chat or commit it.

1. In Qase, open your profile, then **API tokens**, and create a token. Copy it once.
2. Save it outside any git repo:
   ```bash
   mkdir -p ~/.config && touch ~/.config/qase.env && chmod 600 ~/.config/qase.env
   echo 'export QASE_API_TOKEN=paste-your-token-here' > ~/.config/qase.env
   ```
3. Load it in `~/.zshrc` or `~/.bashrc`, then open a new terminal:
   ```bash
   [ -f ~/.config/qase.env ] && . ~/.config/qase.env
   ```
4. Check without printing it: `[ -n "$QASE_API_TOKEN" ] && echo set || echo missing`.

Rotate the token in Qase if it was ever pasted anywhere by mistake.
