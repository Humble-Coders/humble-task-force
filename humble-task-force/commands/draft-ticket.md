---
description: Draft a ticket the team way — interview the manager for the questions a dev would ask, then produce a ready-to-create issue
argument-hint: <short description of the work>
---
You are helping a manager draft a development ticket. Goal: a ticket so self-sufficient that a junior developer (and their Claude Code) can execute it **cold**, without a call.

## 1. Ground yourself (silently)
- Read the project's `CLAUDE.md` and any spec docs (e.g. `docs/PRD.md`) relevant to: **$ARGUMENTS**
- **If this work implements a feature brief** (the cofounder just ran `/read-brief <#>`, or names one): read it
  — `gh issue view <brief#>` + `docs/briefs/B<brief#>-*.md`. It's the Product Owner's intent and guardrails;
  draft the ticket to deliver it (see "If this ticket implements a brief" below).
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
- If it implements a brief, tell the manager it's ready for the Product Owner's **`/review-ticket <#>`** before
  build; otherwise it's ready for **`/start-ticket <#>`**.

## If this ticket implements a brief
- **Deliver the brief.** Its capabilities become the ticket's Scope / Acceptance Criteria.
- **Honor the non-negotiables** (product + `[hard]` technical steers). Never silently override one — if you
  believe a non-negotiable should change, **flag it for the Product Owner** in the ticket and your summary,
  rather than working around it.
- **`[preference]` steers** are strong defaults; if you deviate, **say why in Context** so `/review-ticket`
  can surface it for the Product Owner to bless or veto.
- **Stamp the link:** put `**Brief:** #<n>` at the top of the ticket body (and the saved doc) so
  `/review-ticket` can find the yardstick.

## For UI / visual tickets — always
If the work is UI-facing (a screen, component, theme, or any visual change), do BOTH of these:
- **Settle the design source in the interview.** Ask: *is there an approved design (Figma / screenshots /
  mockup) the UI must match exactly, or should the dev design it against the brand kit?* A design almost
  always exists but **won't paste cleanly into a GitHub issue** — so state in the ticket that **the PM
  provides the design assets** and that the **dev's Claude must ask for them at `/start-ticket` before
  building**, and that the bar is to **match the design exactly** (not approximate or "improve" it).
- **Embed the UI standards.** Read `${CLAUDE_PLUGIN_ROOT}/templates/ui-standards.md` and paste it into the
  ticket as a `## 🖼️ UI standards` section, folding the load-bearing ones into Acceptance Criteria. Adapt the
  platform names to the project and drop any line that genuinely doesn't apply. Every UI ticket must carry:
  light + dark themes; native components (flag when a design can't be done natively, then proceed);
  edge-to-edge + safe-area / notch / nav-bar insets; responsive across sizes (incl. desktop reflow); keyboard
  handling (correct type, Next/Done actions, numeric pads, keep the field visible); correct ellipsis /
  truncation; loading / empty / error states; accessibility; i18n strings; and the project's UI architecture.

Follow the project's `CLAUDE.md` and `docs/PROCESS.md` if present. The goal is a ticket a junior dev can run cold.
