---
name: tc-ols
description: Prepare test cases for an OLS ticket — `/tc-ols 1234` means OLS-1234. Applies the OLS rules (Core / Edge, OLS Thai glossary) on top of Helix tc-fe-prep / tc-api-prep. Also `/tc-ols 1234 format` (move TCs into the release template) and `/tc-ols 1234 run` (manual execution). Use when the user types /tc-ols or gives an OLS-xxx ticket and wants TCs. Do NOT use for Pyrite tickets (tc-pyr), TC review (Helix tc-review), filing bugs, retest or test data.
argument-hint: "<number> [format | run]   e.g. 1234"
metadata:
  proactive_triggers:
    - /tc-ols
    - เขียน TC OLS
    - write test cases OLS
---

# /tc-ols — TC prep for one OLS ticket

`/tc-ols 1234` = prepare the test cases of **OLS-1234**. OLS only. Pyrite has its own skill, `tc-pyr`; its rules (TC-01 ids, Pyrite Type values, Qase Test plan) do not apply here.

**Session start:** follow Helix `references/session-start-context.md` (installed with Helix).

## Step 1 — Read the command

| Input | Job | Delegate to |
|---|---|---|
| `/tc-ols 1234` | **Create** | Helix `tc-fe-prep-workflow` (UI story) or `tc-api-prep-workflow` (API / Swagger) |
| `/tc-ols 1234 format` | **Format** | Helix `tc-change-format-workflow` |
| `/tc-ols 1234 run` | **Run** | [references/execute.md](references/execute.md) |
| `/tc-ols OLS-1234` | Same as `1234` | |
| Number missing | Ask one line: `OLS-?` | |
| Another project's key (PYR-xxx …) | Refuse in one line, route to `tc-pyr` | |

Read [references/ols.md](references/ols.md). Its rules override the generic Helix defaults wherever they differ.

## Step 2 — Run

1. Announce one line: `Using **tc-ols** → OLS-1234 · {job} → {workflow}.`
2. Load the delegated workflow and follow it completely, with the OLS profile as overlay. Say in one line when the profile changes the output.
3. Reuse check: if the ticket already has a sheet / folder, update in place; never duplicate.
4. Stop at the workflow's end. Do not add posts it does not call for.

## Rules

- MUST start every question to the PO or anyone else with its source label: `AC_xx:`, `EC_xx:` or `Business Rule: <name>` (several sources: list each; none, e.g. prototype only: `Prototype: <screen>`). Same text in chat, gate and any post. Never ask an unlabelled question.
- MUST take the ticket key from the command. Never guess it.
- MUST NOT apply any Pyrite rule (see `tc-pyr`).
- MUST NOT write secrets into sheet, Jira, repo or chat.
- MUST NOT invent a rule that is not in the profile; ask once and propose it for the profile.
- No Qase Test plan for OLS: the OLS Qase layout is not specified yet (profile, "Not yet specified").
- Automation (Playwright) is not part of this flow unless asked.
- TC review is not here: use Helix `tc-review` (`/tc-review`).

## Communication (shortest possible)

- **Language:** explain, ask and answer in Thai, always (questions, options, reports, confirmations). Code, commands, paths, error text and ticket terms stay exact. Text that a skill posts or writes (TC content, Jira comment, Discord messages, Qase results) follows the format and language that skill defines, not this rule.
- One question per turn, one line, options inline. No preamble.
- Never ask for what the profile, ticket or an earlier answer already gives.
- Report results in a few lines: counts, links, open issues. No recap, no offers of follow-up work.
- Tables only when the user must decide something (e.g. the Core / Edge confirmation table).

## Refusal-first

| Request | Do |
|---|---|
| PYR key or no key | Refuse, one line: `tc-pyr` for Pyrite, ask for the number otherwise. |
| Review TCs | Not here — Helix `tc-review`. |
| File bug, retest, test data | Route to `create-bug-workflow`, `retest-bug-workflow`, `test-data-prep-workflow`. |

## QA closing (mandatory before "done")

Follow Helix `references/qa-closing-shared.md`, plus:

- [ ] Ticket key came from the command; OLS profile read.
- [ ] Output follows the profile (Core / Edge mapping, OLS glossary, ID format).
- [ ] No Pyrite rule applied; nothing posted that the workflow does not call for; no secret written.
