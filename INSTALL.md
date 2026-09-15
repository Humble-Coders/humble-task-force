# Installing the Humble Task Force plugin

> **For a new teammate (or their Claude Code).** This installs the **Humble Task Force**
> ticket-pipeline plugin from its public marketplace. Follow the steps in order.

Repo (public): https://github.com/Humble-Coders/humble-task-force

---

## 1. Be on a surface that supports plugins

The `/plugin` command only works in:

- ✅ **Terminal CLI** — run `claude` in Terminal / iTerm
- ✅ **VS Code extension** — type `/plugins` in the prompt box

It does **not** work in the **web app** (claude.ai/code) or the **desktop app** GUI. If you're in
the desktop app, manage plugins via the **+** button → **Plugins** instead. The simplest path is a
plain terminal.

## 2. Install — two commands

In an interactive Claude Code terminal session, run:

```
/plugin marketplace add Humble-Coders/humble-task-force
/plugin install humble-task-force@humble-coders
```

The repo is **public**, so there's **no GitHub auth or org invite needed** for the plugin itself.

## 3. If you see "plugin isn't available in this environment"

You're either not on a terminal/VS Code surface (see step 1), or you're on an older Claude Code.
Update and retry:

```bash
claude --version        # check your version
claude update           # native install
# or: npm i -g @anthropic-ai/claude-code@latest
# or: brew upgrade claude-code
```

Restart the terminal, then repeat step 2.

## 4. Verify it worked

- Type `/` and you should see the new commands, or run `/plugin` and confirm
  **humble-task-force** is listed as installed.
- The commands you now have:

| Command | Who | What |
|---|---|---|
| `/setup-tickets` | anyone | scaffold a repo (templates, folders, process doc) |
| `/draft-prd` | manager | interview to `docs/PRD.md` |
| `/draft-architecture` | manager | PRD + code to `CLAUDE.md` |
| `/draft-brief` | product owner | capture a feature's intent as a brief |
| `/read-brief` | manager | absorb the brief, agree a build plan |
| `/draft-ticket` | manager | interview to a ready-to-create GitHub issue |
| `/review-ticket` | product owner | Gate 1: does the plan deliver the brief |
| `/start-ticket <#>` | developer | walkthrough + Q&A, then plan, then build |
| `/handoff <#>` | developer | write the handoff from the real git diff |
| `/manager-review <PR#>` | manager | Gate 2: review the PR against the ticket |
| `/rebase` | developer | safely pull main into the branch you are on |
| `/run-desktop` | anyone | launch the desktop app for review, no questions |
| `/report-bug` | tester | file a clean, labelled bug issue with screenshot |
| `/fetch-bug` | developer | validate a bug against the code and fix it |
| `/pc-handoff` | anyone | pack a project's brain to move PCs |
| `/pc-restore` | anyone | unpack that bundle on the new PC |

## 4b. Keeping it up to date

The plugin gets new commands over time. You are not notified, so check when a
teammate mentions one you do not have. In an interactive Claude Code terminal:

```
/plugin marketplace update humble-coders
/plugin update humble-task-force@humble-coders
```

The first refreshes the catalogue, the second installs the newer version. Or open
`/plugin`, find humble-task-force, and use the menu. Restart the session afterwards
so the new commands load.

To see which version you are on, run `/plugin` and read it off the entry. A command
that "does not exist" is almost always an out-of-date plugin rather than a typo.

## 5. To actually run the workflow (separate from installing the plugin)

The plugin is public, but your **project** is not. To work tickets you also need:

- **Access to the project repo** (e.g. `Humble-Coders/Aromex-KMP`) and `gh auth login` — because
  `/start-ticket` reads issues and `/handoff` opens PRs via the `gh` CLI.
- **Secrets** (test logins, credentials) come from your manager via a secure one-time link —
  **never** through the plugin or a ticket.

New here? After installing, read the project's `docs/PROCESS.md`, then `docs/PRD.md` and `CLAUDE.md`.
That's the whole pipeline.
