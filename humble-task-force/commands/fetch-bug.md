---
description: Developer-side — read a bug issue, validate it against the real code, propose fix approaches, then implement the chosen one (no PR)
argument-hint: <issue-number> [--repo owner/repo]
---
You are helping a **developer** act on bug issue #$1 — filed by a tester via `/report-bug`. Unlike the
tester, **you have the code in front of you**, so you don't just relay the report — you **validate it
against the source, explain the root cause, and fix it**.

## 1. Read the bug
- Run `gh issue view $1` (add `--repo owner/repo` if one was passed). Read the description, **platform**,
  **severity**, and any screenshot. If there's an embedded image, note its URL — you can open it to see
  what the tester saw.
- Restate the bug to the developer in **plain language**: what the tester observed, on which platform,
  and how severe — so you're both looking at the same thing before touching code.

## 2. Validate it against the code — the whole point of this command
- Locate the relevant code by the symptom (screen name, on-screen text, the error, the platform).
- Reach an **evidence-based judgement**, citing `file:line`:
  - **Confirmed** — you can see the defect in the code and explain the exact mechanism.
  - **Likely** — the code strongly points to it, but you'd confirm by running it (say that explicitly).
  - **Can't reproduce from code** — the report doesn't match what the code does, or key detail is missing.
    Say precisely what you'd need from the tester.
- Explain the **root cause** to the developer in plain language — not just *where*, but *why* it happens.

## 3. Propose fix approaches
Offer **2–3 distinct approaches**, each with what it changes, its trade-offs, and its risk — typically a
**quick targeted fix** vs. a **deeper root-cause fix**, and anything sensible in between. Recommend one, but
let the **developer choose** (use AskUserQuestion if available). If the bug turns out **not** to be real,
say so plainly and propose closing the issue with an explanation instead of inventing a change.

## 4. Implement the chosen fix
- Make the change following the project's conventions and `CLAUDE.md`. Keep it scoped to the bug.
- **Verify** it where practical — build and/or run the relevant tests; for a UI bug, run the app or describe
  exactly how to see it fixed.
- **Do NOT open a pull request and do NOT push.** Leave the fix in the working tree. Summarize:
  - what you changed (`file:line`), and why it resolves the root cause,
  - how you verified it (build/test output, or the manual check),
  - anything the developer should double-check.
- Mention that the developer can reference `Closes #$1` when **they** commit — the commit and PR are theirs
  to make, not yours.

Be skeptical and specific. Validating against the real code — and being honest about your confidence — is
what makes this more than a bug-forwarder.
