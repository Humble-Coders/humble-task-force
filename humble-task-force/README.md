# Humble Task Force

A portable **product-owner ↔ manager ↔ developer pipeline** for Claude Code. Install once, use in every repo.

Three roles, **two review gates**: the **Product Owner** frames each feature and signs off the *plan*; the **Manager** (your delivery lead / cofounder) turns it into a ticket and reviews the *code*; the **Developer** builds it.

## Commands

| Command | Who | What it does |
|---|---|---|
| `/setup-tickets` | Anyone | Scaffolds the current repo: issue/PR/brief templates, the `brief` label, `docs/PROCESS.md`, `docs/briefs/`, `docs/tickets/`, `handoffs/`. |
| `/draft-prd <idea>` | **Product Owner** | Interviews you to produce the product spec at `docs/PRD.md` (vision, scope, architecture, constraints). |
| `/draft-architecture` | **Product Owner** | Reads the PRD + codebase and writes the project's `CLAUDE.md` (architecture + rules every dev's Claude reads). |
| `/draft-brief <feature>` | **Product Owner** | Captures a feature's **what & why** (+ optional technical steers) as a brief — a GitHub issue (`brief` label) + `docs/briefs/B<#>-*.md`. Seeds the ticket and is the yardstick for review. |
| `/read-brief <brief#>` | **Manager** | Deep product walkthrough of a brief + Q&A "training mode", then proposes a **build plan** — before any ticket is drafted. |
| `/draft-ticket <what to build>` | **Manager** | Drafts a ticket **against the brief**, **interviews you** for the dev-facing decisions, stamps `Brief: #<n>`, creates the issue. For **UI tickets** it also settles the design source and embeds the [UI standards](./templates/ui-standards.md). |
| `/review-ticket <ticket#>` | **Product Owner** | **Gate 1** — checks the ticket delivers the brief's intent, in product language (coverage, non-negotiables, silent scope changes, product-risk decisions). Sign off or send back. |
| `/start-ticket <issue#>` | Developer | Reads the ticket (+ linked brief) + `CLAUDE.md` + PRD, gives a walkthrough, **offers Q&A training mode**, then plans and confirms before coding. |
| `/handoff <issue#>` | Developer | Writes `handoffs/ticket-<#>.md` from the **real git diff**. |
| `/manager-review <PR#>` | **Manager** | **Gate 2** — checks the PR diff + handoff against the ticket's acceptance criteria; flags risks at `file:line`. |

## The full project lifecycle
```
                ── foundation (once) ──            ── per feature ──
Product Owner:  /draft-prd → /draft-architecture → /draft-brief ──┐ BRIEF #n
Manager:                          /read-brief → build plan → /draft-ticket ──┐ TICKET (Brief: #n)
Product Owner:           GATE 1: /review-ticket ◀────────────────────────────┘
                                    │ sign off
Developer:                          └▶ /start-ticket → build → /handoff → PR ──┐
Manager:                                         GATE 2: /manager-review ◀──────┘ → merge
```
The **PRD** and **CLAUDE.md** are the foundation — every brief and ticket references them. Build them first.

## Principles
- **Foundation first** — the PRD and `CLAUDE.md` ground everything; create them before briefs.
- **Intent before implementation** — the Product Owner's brief is the *what & why*; the Manager owns the *how*, within the brief's non-negotiables.
- **Two gates** — the Product Owner signs off the *plan* (`/review-ticket`); the Manager reviews the *code* (`/manager-review`). Each checks an artifact against the intent above it.
- **Definition of Ready** — a ticket must be runnable by a junior dev cold.
- **Secrets never go in briefs, tickets, or `CLAUDE.md`** — reference the secure source, never the value.
- **Training before building** — a cofounder absorbs a brief, and a developer a ticket, before any code is written.
- **Handoff from the real diff** — reports are generated from `git diff`, not memory.
