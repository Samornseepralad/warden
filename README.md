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
