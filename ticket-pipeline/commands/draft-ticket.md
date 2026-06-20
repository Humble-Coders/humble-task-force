---
description: Draft a ticket the team way — interview the manager for the questions a dev would ask, then produce a ready-to-create issue
argument-hint: <short description of the work>
---
You are helping a manager draft a development ticket. Goal: a ticket so self-sufficient that a junior developer (and their Claude Code) can execute it **cold**, without a call.

## 1. Ground yourself (silently)
- Read the project's `CLAUDE.md` and any spec docs (e.g. `docs/PRD.md`) relevant to: **$ARGUMENTS**
- If `.github/ISSUE_TEMPLATE/feature-ticket.md` exists, match its sections exactly. Otherwise use: Story/Why · Context · 🔑 Access & prerequisites · Scope · Acceptance Criteria · Out of scope · Dependencies · References · Kickoff prompt (`/start-ticket <#>`).
- Read any existing code/tickets this work touches.

## 2. Draft the ticket
Write a first draft with every section.

## 3. Interview the manager (the key step) — DO THIS BEFORE FINALIZING
- Find every place a developer would get blocked or have to ask the manager: **access/credentials, repo placement, hosting/deploy target, which service or module, ambiguous scope, external dependencies, test setup.**
- Present these as a short list of decisions **with your recommended answer for each**, and ask the manager to confirm or adjust (use AskUserQuestion if available). **STOP and wait — do not finalize until the manager answers.**
- Only surface questions that genuinely need the manager's judgment; skip the obvious.

## 4. Bake in the answers
- Fold the manager's decisions into **Context** and **🔑 Access & prerequisites**.
- **NEVER put real secrets in the ticket.** State *what* the dev needs and *where to get it* (team password manager / "ask the manager via secure channel").

## 5. Save + offer to create
- Save the finished draft to `docs/tickets/<id>-<slug>.md` (create the folder if needed).
- Offer to create the GitHub issue: `gh issue create --title "..." --body "$(tail -n +5 <draft>)" [--milestone "<milestone>"]`.

Follow the project's `CLAUDE.md` and `docs/PROCESS.md` if present. The goal is a ticket a junior dev can run cold.
