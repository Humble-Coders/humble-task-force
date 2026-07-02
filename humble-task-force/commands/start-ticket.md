---
description: Start a ticket — read context, offer a walkthrough + Q&A (training mode), then plan, then code
argument-hint: <issue-number>
---
You are helping a developer start GitHub issue #$1. The developer may be junior or unfamiliar with parts of the system — your FIRST job is to make sure they understand the ticket before any code is written.

## 1. Load the context (silently)
- Run `gh issue view $1` and read the ticket in full.
- If the ticket links a **`Brief: #<n>`**, skim that brief (`gh issue view <n>`) for the product intent — the *why* behind the work — so the walkthrough conveys it.
- Read the project's `CLAUDE.md` and the spec sections the ticket references (e.g. `docs/PRD.md`).
- Read the existing code/modules the ticket touches, so everything you say is grounded in what already exists.

## 2. Walkthrough + Q&A (training mode) — DO THIS BEFORE PLANNING
- Give a short, plain-language walkthrough of the ticket: what we're building, why it matters, how it fits the bigger system, and any concept or term the ticket assumes.
- **If this is a UI ticket that references a design not attached to the issue, ask the PM for the design assets (screenshots / Figma) now, before planning** — the ticket's `🖼️ UI standards` require matching the provided design exactly, so you need it in hand first.
- Then STOP and ask, in these words or similar: **"Before we start — want me to explain any part of this, or any concept you're unsure about? Ask me anything, or just say 'ready' and I'll move to the plan."**
- **Wait for the developer's reply. Do NOT proceed to planning until they explicitly say they're ready.**
- Answer their questions clearly and simply, grounded strictly in the ticket, the spec, and the codebase — explain only as much as they need. After answering, ask if they have more. Loop until they're ready.

## 3. Plan
- Present a concise step-by-step plan and the files you expect to change. Ask the developer to confirm. **Do NOT write code yet.**

## 4. Build
- **Sync first** so you branch off the latest merged work: `git checkout main && git pull` (use `master` if that's the default branch).
- After approval, create a branch `ticket-$1-<short-slug>` off the updated main, then implement step by step, scoped to this ticket.

## 5. Hand off
- When the work is done and verified, run `/handoff $1`.

Follow the project's `CLAUDE.md` strictly. The developer can ask questions at ANY point — not just step 2 — so just answer them. If the ticket is ambiguous or conflicts with the spec, stop and ask before proceeding.
