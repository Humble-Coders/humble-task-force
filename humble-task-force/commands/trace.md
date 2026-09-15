---
description: Trace one feature end to end through the real code - UI to logic to network to database - and publish it as a clean visual flow a non-engineer can audit
argument-hint: <feature> (e.g. "the complete auth process", "order import", "how an indent is approved")
---
A manager asks: *what actually happens when someone does X?* Answer it by reading the code,
then publish one visual that a non-engineer can follow and an engineer cannot fault.

The diagram is the easy half. **The trace is the product.** A beautiful picture of a path you
guessed at is worse than no picture, because it will be believed. Every rule below exists to
keep that from happening.

**You are not the architect's advocate.** The default failure of this command is a page that
reads as validation: you describe what the code does, everything you describe looks
deliberate, and the manager comes away reassured rather than informed. Reading code makes
every choice look intentional, because you are reading the choice that won. Your job is to
surface what each step costs, at the step where it is paid.

## The one rule

**Never draw a hop you have not read.** Not inferred from a name, not assumed from a pattern,
not filled in from how these things usually work. If you cannot follow a step to actual source,
you have two honest options: keep digging, or draw it and label it as unverified in the diagram
itself. Silently smoothing over a gap is the failure mode of this command.

---

## 1. Pin down what is being traced

`$ARGUMENTS` names a feature in the user's words. Turn it into an entry point and an end state
before reading anything else: *"from pressing Sign in, to the main window opening."*

If the request spans several distinct journeys (auth is really three: password sign-in, first
time by code, forced password creation), **trace all of them** and show where they converge.
The convergence is usually the most valuable thing on the page - see step 4.

Do not ask the user to narrow it down unless it is genuinely ambiguous. Read the code and find
out.

## 2. Start at the finger, not at the model

Find the actual UI element: the button, the menu item, the field. Search the UI layer for the
label the user would recognise, then follow the handler outward. Starting at a data model and
working backwards produces a tidy architecture picture that nobody's finger ever traces.

Then follow the chain hop by hop, opening each file as you go:

- **UI** - which handler fires
- **Client logic** - view model, controller, use case. What is validated here and what is not
- **Network** - the exact call: REST path, RPC name, function name, endpoint
- **Server** - and this is the half that matters most

## 3. Read the server side properly

This is where the command earns its keep, because it is the part a manager has never been shown
and cannot read for themselves. Do not stop at "it calls the API".

- **Database functions and procedures**: open the migration or definition. Who may execute it?
  What does it check? Does it read the caller's identity from the session, or trust a parameter
  the client sent? That distinction is the whole security story.
- **Row-level security / access rules** on every table touched, read or written.
- **Serverless and edge functions**: what privilege do they run with? A function holding an
  admin or service key is the most security-critical code in most projects. Read all of it.
- **Triggers and cascades**: a write that fires an audit trigger, a sequence, or a notification
  is part of the flow even though no application code mentions it.
- **External systems**: payment providers, mail, storage, another team's queue. Mark the
  boundary where data leaves your control.

**Check for superseded definitions.** Grep the whole migration history for each function name,
not just the first hit. A later migration replacing an earlier one is normal, and reading only
the earlier one gives a confidently wrong answer. Trace the version that is actually live, and
mention the supersede if a reader of the older file would be misled.

## 4. Find the joins, the gates and the forks

While reading, watch for the structural facts that a step-by-step list hides:

- **Where separate journeys converge.** Two ways in, one check both must pass: that shared gate
  is the architecture, and it deserves to be the centre of the picture rather than a step in a
  list.
- **Where a decision forks the path**, and on what value.
- **Where the real authorisation happens**, as opposed to where it appears to. If the client
  hides a menu by role and the server checks it again, say that the hiding is cosmetic. If the
  server does *not* check it again, you have found something important - report it plainly.
- **What happens when it fails.** A flow drawn only as its happy path is half a flow. Show the
  meaningful failures: rejected, rate limited, unreachable, not permitted.

## 5. Interrogate every step for what it costs

This is the section that makes the page worth a manager's time. Walk each step again and ask
what it gives up. Most will give up nothing worth reporting; some will, and those are the
reason the page exists.

Probe for the specific things, not for a general sense of unease:

- **What happens when this step fails?** Is the failure visible, or does it turn into an empty
  screen, a silent skip, a default value? A failure that looks like "no data" is the single
  most common real defect in business software.
- **Does this step hold a privilege it does not need?** Admin keys, service accounts, wide
  grants, broad scopes.
- **What is checked only on the client?** Say plainly whether the server checks it again. If it
  does not, that is a finding, not a note.
- **What can a second person do at the same time?** Two people, one row, one counter, one
  sequence, one file. Check-then-act across a network is not atomic.
- **What grows?** A query with no limit, a table with no retention, a list fetched whole.
  Name the size at which it stops working, and compare it to the real numbers in this business.
- **What is not written down?** A change with no audit row, no timestamp, no actor.
- **What cannot be undone**, and is there a confirmation in front of it?
- **Who gets locked out?** Rate limits, circuit breakers and lockouts protect against abuse and
  create a way to deny service to legitimate users. Say who can trigger that, and for how long.
- **What happens to someone already inside?** Access checks usually run at the door. If
  permission is revoked mid-session, does anything actually stop them, and what do they see?
- **What is the blast radius if this one step is wrong?** One record, one customer, or the
  whole table.

**Verify before you claim.** A risk you assert without reading the code that would disprove it
is worse than one you miss, because it sends people chasing a problem that is not there and it
spends the credibility of everything else on the page. Before writing "the server does not
check X", find the place it would be checked and confirm it is absent. If you looked and could
not tell, write that you could not tell.

**Do not manufacture balance.** If a step is genuinely well built, say so in one line and move
on. A page with three invented concerns is as useless as a page with none, and it is obvious.
Weight matters too: do not give a naming inconsistency the same visual weight as a missing
authorisation check.

**Name the trade, not just the flaw.** Most drawbacks are the price of something the team chose
deliberately. "The invite stays valid until the password is set, so a new employee is never
stranded halfway; the cost is a replay window on a code that has already been used once" tells
a manager something. "The invite is not invalidated immediately" reads as an accusation and
invites a pointless defence.

## 6. Write two registers for every step

Each step carries both, and neither substitutes for the other:

- **Plain language**, from the person's side of the screen: *"They pick their email, type a
  password, press Sign in."* No class names, no jargon. A reader who knows the business and not
  the code must follow the whole page.
- **The technical line**: real file names, real function names, real table and endpoint names,
  in a monospace face. An engineer must be able to jump straight to any step in the codebase.

Where a step exists for a non-obvious reason, add one short note explaining **why**. Those notes
are what turn a diagram into an audit. Draw them from comments and commit messages in the code
rather than inventing a rationale.

## 7. Build the visual

Load the `artifact-design` skill, then write and publish an HTML artifact. Design it properly;
it is going in front of a client or a board, not into a terminal.

**Honour the project's own identity.** If the repo has a theme, brand or token file, take the
palette from it so the audit looks like it belongs to the product. Check the project's own
rules too (typography, punctuation, house style) and follow them.

What the visual must do:

- **Colour-code the tiers** - on the user's machine / crossing the network / on the server - and
  make the trust boundary a visible crossing rather than a label. A manager should be able to
  see, without reading a word, how much of the flow they control.
- **Number the steps** so a conversation can refer to "B4".
- **Give converging paths a real join**, and forks a real split. Lanes side by side merging into
  one emphasised gate block reads instantly; a flat list of the same steps does not.
- **Mark the security-critical steps distinctly** from the ordinary ones.
- **Put each drawback on its own step, not only in a list at the end.** A risk collected into a
  summary is read as an abstraction; the same sentence attached to step B3 is read as a fact
  about B3. Give risk notes a distinct, heavier treatment than explanatory notes, so a reader
  skimming the page sees where the soft spots are without reading a word. Where a step carries
  a real cost, say in one sentence **what could go wrong and who it affects** - not a label.
- Stay readable on a laptop and in both light and dark themes.

Avoid a generic left-to-right box-and-arrow chart. The shape should carry the specific structure
of *this* flow.

## 8. Close with findings, then with limits

Two short sections after the diagram, and do not skip either.

**What an auditor should take away.** Lead with the drawbacks, then the strengths - a manager
reading the top of this section is deciding where to spend attention, and strengths do not need
any. Label each item: worth fixing / accepted risk / housekeeping / sound. Name the specific
thing in plain language, say what it costs and who it affects, and say it plainly without
softening. Rank by consequence, not by how confident you feel.

Then one short paragraph of **honest verdict on the approach as a whole**: what this design is
good at, what it is structurally bad at, and what a different approach would have bought or
lost. Every design trades something. A trace that cannot name what this one traded has not
understood it yet. If the honest verdict is that the approach is sound and the trades were made
knowingly, say exactly that - being real is not the same as being negative.

**What this trace does not cover.** The adjacent paths you did not follow, and any hop you could
not verify. An audit that does not state its own boundary invites the reader to assume it covers
everything. This section is what makes the rest trustworthy.

## 9. Hand it over

Give the user the artifact link and three or four sentences: what you traced, the single most
important structural fact you found, and anything that needs their decision. Do not re-narrate
the diagram - they are about to look at it.

Then **offer** to save a markdown version into the repo (`docs/flows/<feature>.md`, mermaid plus
the findings) so it is versioned and reviewable in pull requests. Offer; do not write it unasked.

## Notes

- Nothing in this command modifies code. It reads, and it publishes a document.
- Real names throughout. A trace with placeholder names is not a trace.
- The page is an audit, not a pitch. Never write that the approach is best practice, robust,
  or industry standard. Show what it does and what that costs, and let the reader judge.
- If the feature turns out not to exist yet, or exists only as a stub, say so immediately and
  stop. Do not draw the flow the code is going to have.
