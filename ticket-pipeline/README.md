# ticket-pipeline

A portable **manager ↔ developer ticket pipeline** for Claude Code. Install once, use in every repo.

## Commands

| Command | Who | What it does |
|---|---|---|
| `/draft-ticket <what to build>` | Manager | Drafts a ticket, then **interviews you** for the decisions a developer would otherwise ask about (access, hosting, scope) with recommended answers, bakes them in, and creates the GitHub issue. |
| `/start-ticket <issue#>` | Developer | Reads the ticket + `CLAUDE.md` + spec, gives a plain-language walkthrough, **offers a Q&A "training mode"**, then plans and confirms before coding. |
| `/handoff <issue#>` | Developer | Writes `handoffs/ticket-<#>.md` from the **real git diff** (files changed, how to test, acceptance criteria, deviations). |
| `/manager-review <PR#>` | Manager | Checks the PR diff + handoff against the ticket's acceptance criteria; flags risks at `file:line`. |
| `/setup-tickets` | Anyone | Scaffolds the current repo: issue/PR templates, `docs/PROCESS.md`, `handoffs/`, `docs/tickets/`, and a `CLAUDE.md` starter. |

## Setting up a new project
1. `/setup-tickets` — drops in the templates, folders, and `docs/PROCESS.md`.
2. Fill in the project-specific `CLAUDE.md` (architecture).
3. `/draft-ticket <first thing to build>`.

## Principles
- **Definition of Ready** — a ticket must be runnable by a junior dev cold.
- **Secrets never go in tickets** — reference the secure source, never the value.
- **Training before building** — a developer can ask anything before code is written.
- **Handoff from the real diff** — reports are generated from `git diff`, not memory.
