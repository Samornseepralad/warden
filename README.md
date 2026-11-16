# warden

Claude skills for QA test-case prep, one skill per project.

| Command | Does |
|---|---|
| `/tc-pyr 783` | Prepare TCs for **PYR-783**: write TCs, show an HTML draft and wait for Approve, then create them in Qase (Pyrite), build the Qase Test plan `PYR-783 <ticket name>`, fill the Drive TC sheet and tracking sheet |
| `/tc-pyr 783 update` | `updateTCSheet`: sync all, in order: Qase TCs → Test plan → Drive sheet → tracking sheet |
| `/tc-pyr 783 review` | Review TCs with Helix `tc-review` plus the PYR rules |
| `/tc-pyr 783 run` | Manual execution from the TC sheet |
| `/tc-ols 1234` | Prepare TCs for **OLS-1234** with the OLS rules (Core / Edge, OLS glossary) |
| `/tc-ols 1234 format` | Move OLS TCs into the release template |
| `/tc-ols 1234 run` | Manual execution from the TC sheet |

TC review for OLS, or without PYR rules: Helix `/tc-review`.

## Layout

```
skills/tc-pyr/   SKILL.md + references/{pyr,execute}.md
skills/tc-ols/   SKILL.md + references/{ols,execute}.md
```

## Prerequisite: Helix

**Install Helix first.** The skills delegate to Helix workflows: `tc-fe-prep-workflow`, `tc-api-prep-workflow`, `tc-change-format-workflow` (OLS), `create-bug-workflow`, `retest-bug-workflow`, and use its reference files (`session-start-context.md`, `qa-closing-shared.md`).

Helix: https://github.com/Thitic9203/helix (install steps are in its README).

## Install

Symlink each skill folder:

```bash
ln -s "$PWD/skills/tc-pyr" ~/.claude/skills/tc-pyr
ln -s "$PWD/skills/tc-ols" ~/.claude/skills/tc-ols
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
- Not on the latest? Run `git -C <your-clone> pull` by hand.

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
