---
description: Generate or update this project's CLAUDE.md (architecture + rules) from the PRD and the codebase
---
You are producing the project's **`CLAUDE.md`** — the file every developer's Claude Code reads automatically on every ticket. It must capture the architecture and the non-obvious rules so the whole team (and their Claude) build consistently.

## 1. Gather (silently)
- Read `docs/PRD.md` if present — it's the source of truth for architecture decisions.
- Read the codebase structure (build files, module/folder layout, key configs) so `CLAUDE.md` reflects what actually exists, not an ideal.

## 2. Interview the manager — only on what isn't already settled
Don't re-ask anything the PRD or code already answers. Ask, with recommendations, about: coding conventions, what developers must NOT do, security rules, how the modules/layers fit, and infra/deploy facts. **Never put secrets in `CLAUDE.md`** — reference where they live instead.

## 3. Write CLAUDE.md
Write or update `CLAUDE.md` at the repo root with:
- **What this project is** — one paragraph.
- **Architecture** — layers, stack, data flow, and the rules a developer must follow (be specific and strict).
- **Key rules** — non-negotiables, conventions, security.
- **How we work (ticket workflow)** — point to `docs/PROCESS.md` and the `/draft-ticket`, `/start-ticket`, `/handoff`, `/manager-review` commands.
- **References** — spec docs, external systems.

Keep it tight and concrete — it's read on every ticket. If `CLAUDE.md` already exists, update it in place rather than overwriting wholesale.

## 4. Next step
Tell the manager: run **`/draft-ticket <first thing to build>`**.
