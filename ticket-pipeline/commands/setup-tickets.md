---
description: Scaffold the current repo for the ticket pipeline — issue/PR templates, handoffs + tickets folders, a PROCESS doc, and a CLAUDE.md starter
---
Set up the **current repository** to use the ticket pipeline. The plugin's template files live at `${CLAUDE_PLUGIN_ROOT}/templates/`.

Steps (use the Bash tool):
1. Create folders: `mkdir -p .github/ISSUE_TEMPLATE docs/tickets handoffs`.
2. Copy each template **only if the destination does not already exist** (check first; if it exists, ask the manager before replacing):
   - `${CLAUDE_PLUGIN_ROOT}/templates/feature-ticket.md` → `.github/ISSUE_TEMPLATE/feature-ticket.md`
   - `${CLAUDE_PLUGIN_ROOT}/templates/pull_request_template.md` → `.github/pull_request_template.md`
   - `${CLAUDE_PLUGIN_ROOT}/templates/PROCESS.md` → `docs/PROCESS.md`
   - `${CLAUDE_PLUGIN_ROOT}/templates/handoffs-README.md` → `handoffs/README.md`
   - create an empty `docs/tickets/.gitkeep`
3. If `CLAUDE.md` does NOT exist in the repo root, copy `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE.starter.md` → `CLAUDE.md`, then tell the manager to fill in the project-specific architecture sections. If `CLAUDE.md` exists, leave it and remind them to add a "How we work" section pointing to `docs/PROCESS.md` if it's missing.
4. Verify each file landed, then print a short summary and the next step: **"Run `/draft-ticket <what to build>` to create your first ticket."**
