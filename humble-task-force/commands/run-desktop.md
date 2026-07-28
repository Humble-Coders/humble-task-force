---
description: Build and launch the desktop app for the user to review — no questions, no confirmation, just run it
argument-hint: (no arguments)
---
Launch this project's **desktop app** so the user can look at it. **Ask nothing** — no
confirmations, no clarifying questions, no "shall I?". Just get it on screen and report the
outcome. Do not run tests, do not build other platforms, do not modify any files.

## 1. Find how this repo runs its desktop app (silently)
Work it out from the repo, in this order — take the first that matches:
- **Compose Desktop / Gradle** — a module applying `org.jetbrains.compose` with a
  `compose.desktop { application { … } }` block (commonly `desktopApp`). The task is
  `:<module>:run`. Confirm the module name from `settings.gradle.kts` / the build file rather
  than assuming it.
- **`.claude/launch.json`** — use the configuration whose name/port matches the desktop app.
- **Electron / Tauri / npm** — a `dev`/`start`/`tauri dev` script in `package.json`.
- **JVM `application` plugin** — `:<module>:run`.

If none match, say so plainly, name what you looked for, and stop — don't guess a command.

## 2. Clear any stale instance
A previously-launched copy is the most common cause of "I changed the code but nothing looks
different." Kill it first (best-effort, ignore failures) — e.g. for Compose Desktop:
```bash
pkill -f "<mainClass>" 2>/dev/null; pkill -f "<module>:run" 2>/dev/null || true
```

## 3. Launch it in the background
**Always run it in the background** with output teed to a log — a desktop app runs until the
user closes it, so a foreground run would block the whole session.
```bash
cd <repo-root> && export LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 && \
  ./gradlew :<module>:run --console=plain 2>&1 | tee /tmp/<app>-desktop-run.log
```
*(`--console=plain` keeps the log parseable; the UTF-8 locale avoids encoding failures on macOS.)*

## 4. Watch until it's up (or has failed)
Poll the log — don't just assume it launched. Stop as soon as either appears:
- **Launched:** the run task started (e.g. `> Task :<module>:run`), or the app's own startup
  logging begins.
- **Failed:** `BUILD FAILED`, `FAILED`, or compiler errors (`e: `).

Poll on a short sleep loop with a sensible ceiling (a cold Gradle build can take a couple of
minutes); never block indefinitely.

## 5. Report — short and honest
- **Launched:** one line that it's running and the window is open, plus anything notable the
  startup log already shows (which screen it's on, sign-in state, backend/config it connected
  to). Mention the log path and that they can ask you to stop it.
- **Failed to compile:** show the actual `e: file:line` errors — that's the useful part — not
  the Gradle preamble. Offer to fix them; don't start fixing unasked.
- **Failed at runtime:** show the exception and where it came from.

Never claim it's running if the log doesn't show it. If the process exits later (the user
closed the window), that's normal — say so if it comes up, don't treat it as an error.

## Notes
- The app keeps running across turns; when the user is done, stopping it is just killing that
  background process.
- If the user has uncommitted changes, that's fine — this runs the working tree as-is, which
  is usually exactly what they want to review.
