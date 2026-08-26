---
description: Manager review of a PR — check the diff + handoff against the ticket's acceptance criteria
argument-hint: <pr-number>
---
You are helping a manager review pull request #$1.

1. Run `gh pr view $1` and `gh pr diff $1` to read the PR and its full diff.
2. Read the linked ticket (`gh issue view <linked-number>`) and the committed handoff report under `handoffs/`.
3. Read the project's `CLAUDE.md` and the relevant spec sections so you judge against the project's standards.
4. **Run the Crash Gate below first.** It is blocking: a violation is "request changes" no matter how
   good the rest of the PR is.
5. Produce a review covering:
   - **Crash gate:** pass, or the exact violations with `file:line`.
   - **Acceptance criteria:** go through each one — met / not met / unclear — citing the code.
   - **Correctness & risks:** real bugs, security issues, or architecture violations, each with `file:line`.
   - **Report vs reality:** flag anything the handoff claims that the diff does not support.
   - **Verify by hand:** the exact steps for the manager to run it and confirm.
   - **Verdict:** approve or request changes, with the top issues to fix first.

Be skeptical and specific. Prefer citing `file:line` over general comments.

---

# The Crash Gate — errors that kill the app instead of surfacing

This gate exists because a shipped release crashed for every user on a weak connection, and the
`catch` block written to handle that exact failure had never once executed. Code review passed it.
Tests passed it. It only showed up in TestFlight crash logs.

**The principle, which outlives any one framework:** when an error crosses a boundary — between two
languages, two processes, two runtimes — some boundaries *deliver* the error and some *terminate the
program*. A boundary that terminates does so before any handler runs, so the handler is dead code
that reads as if it works. Review must therefore ask, of every error path in the diff: **can this
error actually arrive where it is caught?** Never assume it can because someone wrote a `catch`.

## What to check, in order

### 1. Is a boundary crossed at all?

Scan the diff for these, and if none appear, note "no boundary crossed" and move on:

- A Kotlin/Native ↔ Swift call (KMP + SKIE): shared `suspend fun`s, shared interfaces implemented in
  Swift, non-suspend shared functions called from Swift.
- A JNI, FFI, or native-module call.
- A callback or continuation resumed from foreign code.
- An `async` boundary where the failure is resumed rather than returned.

### 2. For KMP ↔ Swift, apply these rules exactly

Kotlin/Native delivers an exception to Swift **only if it is an instance of a type listed in the
function's `@Throws`.** Three consequences that reviewers get wrong:

- **Sibling types do not count.** `@Throws(HlException::class)` does not deliver a `LoginException`
  even though both extend `RuntimeException`. It terminates. Check the *declared types cover the
  thrown types*, not merely that some `@Throws` is present.
- **It applies in both directions.** For a Kotlin interface whose iOS implementation is **Swift**,
  the boundary is the **interface method**. If the interface member has no `@Throws`, nothing its
  Swift implementation raises can be delivered — the process dies. This is the single highest-yield
  check in this gate; it is what shipped the crash described above.
- **A raw `NSError` can never be declared.** A Swift implementation that does `throw NSError(...)`
  or rethrows a Firestore/Functions/URLSession error unconverted sends Kotlin a type no `@Throws`
  list can name. It is fatal on every Kotlin-crossing path regardless of annotations. Swift
  implementations of shared interfaces must raise a **shared Kotlin exception type** (typically via
  `.asError()`), always.

Also reject, because no annotation can rescue them:

- A `require`/`check`/`throw` inside an **`init {}` block** of a class Swift constructs directly.
- A **property getter** that can throw — `.first()`, `!!`, a force-cast, an unguarded index — on a
  type exported to Swift. Make it total (`firstOrNull() ?: fallback`) or move validation into a
  nullable factory.
- Framework exceptions that are not Kotlin exceptions at all: an invalid Firestore document path
  (a user-supplied string containing `/`), a `whereField(in:)` over the backend's limit, a
  non-positive `limit`, an `AVCaptureSession` used with no active connection. These raise
  Objective-C exceptions that Swift **cannot catch** — they must be made impossible by validation,
  and the validation belongs in shared code so all platforms agree.

### 3. Hunt the dead `catch`

For every `catch`, `do/catch`, or `runCatching` the diff adds or touches on a boundary path, ask:
*has this ever executed?* If the boundary above it does not declare the error's type, the answer is
no — and after the fix it will run **for the first time**. Require that each newly-live handler:

- surfaces something the user can actually see, and
- resets every in-flight flag it needs to (`isSaving`, `isLoading`, spinners, disabled buttons).

"No longer crashes, but the spinner never stops" is not a fix. This is the most common regression
when this class is repaired, so check it explicitly rather than trusting the diff to look right.

### 4. Check the platform's own failure mode

The same code has different consequences per platform, so judge severity per platform, not once:

- **Kotlin/Native (iOS):** an undeclared exception at a bridge **terminates the process**.
- **Android:** an uncaught exception in `viewModelScope.launch`, a composable body, or a click
  lambda **crashes the app**. A shared function that throws is safe on Android only if the call site
  actually catches it.
- **Desktop/JVM:** an uncaught exception on the Compose/AWT thread **kills the window**; in a
  background scope it is *silently swallowed* and the feature dies with no crash and no log — often
  worse than a crash because nobody reports it.
- Cancellation is not failure: every `catch (Throwable)` / `runCatching` on a cancellable path must
  rethrow `CancellationException`, or a superseded request renders as an error banner.

### 5. Demand real verification, and reject fake verification

- **Do not accept a generated-header grep as proof for `suspend` functions.** In Kotlin/Native a
  `suspend fun` exports with an `NSError` completion-handler parameter **whether or not** it carries
  `@Throws`, so the presence of `NSError` in the header proves nothing about the annotation. That
  check distinguishes annotated from un-annotated only for **non-suspend** functions. Treating it as
  general proof is a trap that makes an unfixed crash look verified.
- What counts as proof for a suspend boundary: the annotation is in the source **and** someone
  triggered the real failure on a device or simulator (airplane mode, revoked account, forced
  backend error) and saw a message instead of a termination. Require the handoff to say who ran it
  and what they saw.
- A build passing proves nothing here: this entire class is invisible to compilation, and largely
  invisible to unit tests, because the bridge only exists at runtime on-device.

### 6. Check the guardrail was not weakened

If the repo has a source-scanning guard for this class (an allowlist-bearing test), verify the diff
does not **add** entries to the allowlist to make a build pass. The allowlist may only shrink. A new
entry is a new instance of the bug wearing a suppression, and is grounds to request changes on its
own. If the repo has no such guard and the diff touches a boundary, say so in the review — the
absence is worth a follow-up ticket.

## Signature-change warning (so a correct fix is not rejected as risky)

Adding `@Throws` to a **`suspend`** function does not change its Swift signature — it is already
`async throws`, so no call site changes and the diff is genuinely low-risk. Adding it to a
**non-suspend** function *does* change the signature: Swift call sites need `try` and the build
breaks loudly until they have it. On Android/JVM, `@Throws` only adds a `throws` clause to the
bytecode signature — runtime behaviour is byte-for-byte identical, so **Android and Desktop cannot
regress from an annotation alone.** If either platform's behaviour changed visibly, something
semantic was altered by mistake; find it.

## How to report a gate finding

State the boundary, the type that cannot cross it, the user-facing trigger, and the consequence —
in that order, with `file:line`. For example:

> `sharedLogic/.../HlTokenProvider.kt:13` — `currentToken()` has no `@Throws`, and its iOS
> implementation is Swift (`HlTokenRepositoryImpl.swift:71`) which raises `HlException` on any
> network failure. Every HL-backed screen therefore terminates on weak signal instead of showing the
> offline state, and the handler at `HomeViewModel.swift:313` is dead code. **Request changes.**
