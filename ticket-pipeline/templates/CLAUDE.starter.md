# CLAUDE.md — <PROJECT NAME>

> Read this before working on any ticket. It captures the architecture and the rules that
> aren't obvious from the code. **Fill in the project-specific sections below.**

## What this project is
<!-- One paragraph: what the product/service is and who it's for. -->

## Architecture
<!-- The layers/modules, the tech stack, how data flows. The rules a developer must follow.
     Be specific — this is what every developer's Claude reads automatically. -->

## Key rules
<!-- Non-negotiables: conventions, security rules, what NOT to do. -->

## How we work (ticket workflow)
> Full playbook for managers and new joiners: **`docs/PROCESS.md`**.
- **Managers draft tickets** with **`/draft-ticket <what to build>`** — it interviews the manager
  for the decisions a developer would otherwise ask about, then produces a ready-to-create issue.
- A developer starts a ticket with **`/start-ticket <#>`** — it reads the ticket + this file +
  the spec, gives a plain-language walkthrough, offers a Q&A (training mode), then **plans and
  confirms before writing code.**
- When done, run **`/handoff <#>`** to write `handoffs/ticket-<#>.md` from the **real diff**, then
  open a PR (the PR template links the handoff).
- The manager reviews with **`/manager-review <PR#>`**.
- If a ticket conflicts with this file or the spec, **stop and ask** — don't guess.

## References
<!-- Spec docs, design notes, external systems. -->
