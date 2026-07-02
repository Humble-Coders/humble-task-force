---
description: Absorb a Product Owner's feature brief (deep walkthrough + Q&A), agree a build plan, then draft the ticket
argument-hint: <brief-issue-number>
---
You are helping a **cofounder (Manager)** take on the feature brief in GitHub issue #$1. Your job: make them
**feel exactly what the Product Owner wants built and why**, answer everything they're unsure about, then agree
a build plan — all **before** any ticket is drafted.

## 1. Load the context (silently)
- Run `gh issue view $1` and read the brief in full. If it isn't labelled `brief`, say so and confirm the number.
- Also read its committed copy in `docs/briefs/` if present, plus `docs/PRD.md`, `CLAUDE.md`, and the existing
  code/tickets this feature touches — so everything you say is grounded in what already exists.

## 2. Deep walkthrough — convey the intent, not just the words
Explain the feature richly, in **product language**: the user problem, who it's for, what "good" looks like,
and — most importantly — **why it matters to the Product Owner**. Call out the **non-negotiables** and any
**technical steers** (which are `[hard]` vs `[preference]`). Aim for the cofounder to end up wanting to build
what the Product Owner wants, for the same reasons.

## 3. Q&A (training mode) — DO THIS BEFORE PLANNING
- Ask, in these words or similar: **"Ask me anything about this feature before we plan — the user, the intent,
  the constraints, how it fits the system. Or say 'ready' and I'll propose a build plan."**
- **Wait. Do NOT plan until they say they're ready.** Answer grounded strictly in the brief + PRD + code; loop
  until they're ready. If a question exposes a genuine gap or conflict in the brief, note it — it may need to
  go back to the Product Owner.

## 4. Propose a build plan — then decide together
- Propose how to build it: the technical shape, how to **break it into ticket(s)** (a milestone brief may need
  several), sequencing, the key decisions to make, and the risks. Honor the brief's non-negotiables and
  `[hard]` steers; treat `[preference]` steers as strong defaults.
- Discuss and adjust with the cofounder until the plan is agreed. **This is where the cofounder makes the
  technical decisions the Product Owner delegated** — own them.

## 5. Draft the ticket
- When the plan is set, move to **`/draft-ticket <feature>`** for each ticket — it drafts against **Brief #$1**,
  honors its non-negotiables, and stamps `Brief: #$1` into the ticket. From there it goes to the Product Owner
  for **`/review-ticket`**.

Ground everything in the brief, the PRD, and `CLAUDE.md`. If the brief is ambiguous or conflicts with the spec,
surface it — and if it touches a **non-negotiable**, it goes back to the Product Owner, not around them.
