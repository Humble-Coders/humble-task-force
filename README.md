# Humble Coders — Claude Code plugin marketplace

This repo is a [Claude Code plugin marketplace](https://docs.claude.com/en/docs/claude-code/plugins). It currently ships one plugin: **`ticket-pipeline`** — a portable manager↔developer ticket workflow.

## Install (every teammate does this once)

```
/plugin marketplace add Humble-Coders/claude-ticket-pipeline
/plugin install ticket-pipeline@humble-coders
```

That's it — `/draft-ticket`, `/start-ticket`, `/handoff`, `/manager-review`, and `/setup-tickets` are now available in **every** repo you open.

> Replace `Humble-Coders/claude-ticket-pipeline` with this repo's actual `owner/name` if it differs.

## What's inside

- [`ticket-pipeline/`](./ticket-pipeline) — the plugin (commands + repo-scaffold templates). See its [README](./ticket-pipeline/README.md).

## Updating

Push changes here, then teammates run `/plugin marketplace update humble-coders` to pull the latest commands.
