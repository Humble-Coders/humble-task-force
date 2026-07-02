---
description: Product Owner review of a drafted ticket — does it deliver the brief's intent? Check in product language, then sign off or send back
argument-hint: <ticket-issue-number>
---
You are helping the **Product Owner** review the drafted ticket in GitHub issue #$1, **before** it goes to
build. There is one question: **does this ticket's scope + decisions deliver what the brief asked for?** Judge
in **product language** — you are NOT reviewing code quality here (that's `/manager-review`, later, on the PR).

## 1. Load the yardstick + the plan (silently)
- Run `gh issue view $1` and read the ticket. Find its **`Brief: #<n>`** link.
- Run `gh issue view <n>` and read `docs/briefs/B<n>-*.md` — this brief is the **yardstick**. If the ticket
  links no brief, ask the Product Owner to state the intent inline and check against that instead.
- Read `docs/PRD.md` + `CLAUDE.md` for context.

## 2. Produce the review — product-first, specific, cite the ticket
- **Intent coverage** — for each capability / "must do" in the brief: is it in the ticket's Scope /
  Acceptance Criteria? **met / missing / partial**, citing the ticket line. If the brief spans several tickets,
  `gh issue list --search "Brief: #<n>"` finds the siblings — judge whether this ticket delivers its slice, and
  note capabilities that look unaddressed (they may live in a sibling, or in none — say which).
- **Non-negotiables** — for each brief non-negotiable and each `[hard]` technical steer:
  **honored / violated / flagged-for-change**. A violation is a blocker.
- **Deviations from your steers** — where the ticket deviated from a `[preference]` steer; surface each plainly
  so you can **bless or veto** it.
- **Silent scope changes** — anything the ticket defers, marks out-of-scope, or adds beyond the brief —
  translated into product terms ("this means users won't be able to …").
- **Product-risk decisions** — the few technical decisions that carry product consequences, each in plain
  language + **why it matters to you**.
- **Open product questions** — anything that genuinely needs the Product Owner's ruling.
- **Verdict** — **✅ Sign off** or **🔄 Send back**, with a short, ordered list of **specific product-level
  asks** for the cofounder (product outcomes, not technical prescriptions).

Do NOT re-litigate pure-technical choices (threading, file layout, naming) — you may flag one for
`/manager-review` if it's worth noting, but it does not gate your sign-off. Keep this quick — it's a 2-minute
product read, not a code review.

## 3. Close the loop
Offer to post the verdict + asks as a comment on the ticket so the cofounder can action them:
`gh issue comment $1 --body "<asks>"`. On **Send back**, the cofounder revises and you re-review; on **Sign
off**, the ticket is ready for the developer's **`/start-ticket $1`**.
