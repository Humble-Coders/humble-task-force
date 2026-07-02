# Humble Task Force

A portable **manager ↔ developer ticket pipeline** for Claude Code. Install once, use in every repo.

## Commands

| Command | Who | What it does |
|---|---|---|
| `/setup-tickets` | Anyone | Scaffolds the current repo: issue/PR templates, `docs/PROCESS.md`, `handoffs/`, `docs/tickets/`. |
| `/draft-prd <idea>` | **Manager** | Interviews you to produce the product spec at `docs/PRD.md` (vision, scope, architecture, constraints). |
| `/draft-architecture` | **Manager** | Reads the PRD + codebase and writes the project's `CLAUDE.md` (architecture + rules every dev's Claude reads). |
| `/draft-ticket <what to build>` | **Manager** | Drafts a ticket, **interviews you** for the decisions a developer would otherwise ask about, bakes them in, and creates the GitHub issue. For **UI tickets** it also settles the design source and embeds the [UI standards](./templates/ui-standards.md) (light+dark, native components, edge-to-edge/insets, responsive, keyboard, a11y…). |
| `/start-ticket <issue#>` | Developer | Reads the ticket + `CLAUDE.md` + PRD, gives a plain-language walkthrough, **offers a Q&A "training mode"**, then plans and confirms before coding. |
| `/handoff <issue#>` | Developer | Writes `handoffs/ticket-<#>.md` from the **real git diff**. |
| `/manager-review <PR#>` | **Manager** | Checks the PR diff + handoff against the ticket's acceptance criteria; flags risks at `file:line`. |

## The full project lifecycle
```
/setup-tickets → /draft-prd → /draft-architecture → /draft-ticket → /start-ticket → build → /handoff → PR → /manager-review → merge
   (scaffold)      (PRD)         (CLAUDE.md)          (issue)        (dev: train+plan)  (dev)             (manager)
```
The **PRD** and **CLAUDE.md** are the foundation — every ticket references the PRD, and every `/start-ticket` and `/draft-ticket` reads `CLAUDE.md`. Build them first.

## Principles
- **Foundation first** — the PRD and `CLAUDE.md` ground everything; create them before tickets.
- **Definition of Ready** — a ticket must be runnable by a junior dev cold.
- **Secrets never go in tickets or `CLAUDE.md`** — reference the secure source, never the value.
- **Training before building** — a developer can ask anything before code is written.
- **Handoff from the real diff** — reports are generated from `git diff`, not memory.
