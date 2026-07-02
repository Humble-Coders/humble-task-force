---
description: Capture a feature/screen/milestone's intent as a brief — interview the Product Owner, then create the brief issue + doc that seeds a ticket
argument-hint: <feature / screen / milestone to build>
---
You are helping the **Product Owner** turn intent into a **feature brief** for: **$ARGUMENTS**

A brief is the **what & why** of one feature/screen/milestone — the guardrails a cofounder drafts a ticket
against, and the yardstick `/review-ticket` measures that ticket by. It is **product-first**, but the Product
Owner may be an engineer with technical steers up front, so capture those too. **Never put secrets in a brief.**

## 1. Ground yourself (silently)
- Read `docs/PRD.md` and `CLAUDE.md` so the brief is consistent with the product and its architecture.
- Read any related briefs (`docs/briefs/`), tickets, or existing code this feature touches.

## 2. Interview the Product Owner — product questions, a few at a time
Drive the discussion; for each area give a recommendation and let them decide (use AskUserQuestion if
available). Cover only what matters for THIS feature:
- **Problem & outcome** — the user problem, and what "done" feels like.
- **Who it's for** and the context they hit it in.
- **Capabilities** — what the feature must let a user do (user terms, not implementation).
- **What "good" looks like** — how they'll judge it.
- **Non-negotiables** — hard lines (product or technical) the ticket must honor.
- **Technical steers (optional)** — if they have engineering suggestions, capture each as `[hard]`
  (must follow) or `[preference]` (strong default, overridable with a documented reason). If they'd rather
  delegate the "how", leave this empty.
- **Happy to defer** — what they're fine NOT doing now.
- **References** — designs (note "provided by the PO"), legacy app, PRD sections.

Ask only what genuinely needs their judgment; don't force technical detail they'd rather hand to the cofounders.

## 3. Write the brief
Fill the sections in `${CLAUDE_PLUGIN_ROOT}/templates/brief.md` (or `.github/ISSUE_TEMPLATE/brief.md` if the
repo has one). Keep it to ~1 page — sharp, not exhaustive. **Product intent only; do not design the
implementation.** A brief may be milestone-sized (it can later spawn several tickets).

## 4. Create the brief — issue + committed doc
Use the Bash tool:
- Ensure the label exists: `gh label create brief --color 5319E7 --description "Product Owner feature brief" 2>/dev/null || true`.
- Create the issue and capture its number `N`:
  `gh issue create --title "[Brief] <feature>" --label brief --body "<brief body>" [--milestone "<milestone>"]`.
- Save a committed copy to `docs/briefs/B<N>-<slug>.md` (create the folder if needed) and commit it
  (`docs: brief B<N> — <feature>`).

## 5. Next step
Tell the Product Owner: hand a cofounder **`/read-brief <N>`** — they'll absorb the intent, agree a build plan,
then draft the ticket. You close the loop later with **`/review-ticket <ticket#>`**.
