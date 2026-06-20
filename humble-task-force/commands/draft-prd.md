---
description: Produce a product spec (PRD) for this project by interviewing the manager
argument-hint: <product / project name or one-line idea>
---
You are helping a manager produce a **PRD** (product requirements document) for: **$ARGUMENTS**

Goal: a spec a team can build from — vision, scope, architecture decisions, and constraints — written so tickets can later reference it. This is the foundation of the whole workflow; take it seriously.

## 1. Understand what exists (silently)
- If this is a rebuild or has existing code/docs, read them to ground yourself. Otherwise start fresh.

## 2. Interview the manager — drive the discussion, ONE area at a time
For each area: explain the trade-offs in plain language, give a **clear recommendation**, and ask the manager to choose (use AskUserQuestion if available). Settle each before moving on; don't overwhelm — a few sharp questions at a time. Cover, as relevant to this product:
- **Vision & goals** — what it is, who it's for, what success looks like.
- **Scope & non-goals** — what's in v1, what's explicitly out.
- **Users & roles** — who uses it, permissions.
- **Architecture & backend** — data model, services, integrations, hosting, platforms.
- **Key constraints** — offline, scale, money/correctness, security, compliance.
- **Open decisions** — surface the forks the manager must settle; recommend, don't invent.

If a decision is genuinely the manager's and you can't infer it, ask — don't guess.

## 3. Write the PRD
Write `docs/PRD.md` with: Overview/Vision · Goals & Non-Goals · Target users · System Architecture · Feature spec by area · Non-functional requirements · Open decisions · **Decision log** (every locked choice, so it survives). Use Mermaid diagrams where they clarify.

## 4. Next step
Tell the manager: run **`/draft-architecture`** to turn this into the project's `CLAUDE.md`, then **`/draft-ticket <first thing>`** to create the first ticket.
