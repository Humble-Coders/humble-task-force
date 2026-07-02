---
description: Scaffold the current repo for the ticket pipeline — issue/PR templates, handoffs + tickets folders, and a PROCESS doc
---
Set up the **current repository** to use the ticket pipeline. The plugin's template files live at `${CLAUDE_PLUGIN_ROOT}/templates/`.

Steps (use the Bash tool):
1. Create folders: `mkdir -p .github/ISSUE_TEMPLATE docs/tickets docs/briefs handoffs`.
2. Copy each template **only if the destination does not already exist** (check first; if it exists, ask the manager before replacing):
   - `${CLAUDE_PLUGIN_ROOT}/templates/feature-ticket.md` → `.github/ISSUE_TEMPLATE/feature-ticket.md`
   - `${CLAUDE_PLUGIN_ROOT}/templates/brief.md` → `.github/ISSUE_TEMPLATE/brief.md`
   - `${CLAUDE_PLUGIN_ROOT}/templates/pull_request_template.md` → `.github/pull_request_template.md`
   - `${CLAUDE_PLUGIN_ROOT}/templates/PROCESS.md` → `docs/PROCESS.md`
   - `${CLAUDE_PLUGIN_ROOT}/templates/handoffs-README.md` → `handoffs/README.md`
   - `${CLAUDE_PLUGIN_ROOT}/templates/briefs-README.md` → `docs/briefs/README.md`
   - create an empty `docs/tickets/.gitkeep`
   - create the `brief` label (needs `gh` + a GitHub remote; harmless if offline):
     `gh label create brief --color 5319E7 --description "Product Owner feature brief" 2>/dev/null || true`
3. Verify each file landed, then print a summary and the **foundation next steps** (skip any whose file already exists):
   - "Run **`/draft-prd <your idea>`** to write the product spec (`docs/PRD.md`)."
   - "Then **`/draft-architecture`** to generate this project's `CLAUDE.md` from the PRD + code."
   - "Then, per feature: Product Owner runs **`/draft-brief <feature>`** → a cofounder runs **`/read-brief <#>`** and **`/draft-ticket`** → Product Owner runs **`/review-ticket <#>`** → build."

(For a manual `CLAUDE.md` skeleton instead of `/draft-architecture`, a starter is at `${CLAUDE_PLUGIN_ROOT}/templates/CLAUDE.starter.md`.)
