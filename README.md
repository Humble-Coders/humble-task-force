# Humble Coders — Claude Code plugin marketplace

This repo is a [Claude Code plugin marketplace](https://docs.claude.com/en/docs/claude-code/plugins). It ships **Humble Task Force** (`humble-task-force`) — a portable product-owner↔manager↔developer workflow with two review gates.

## Install (every teammate does this once)

```
/plugin marketplace add Humble-Coders/humble-task-force
/plugin install humble-task-force@humble-coders
```

That's it — the full pipeline (`/draft-prd`, `/draft-architecture`, `/draft-brief`, `/read-brief`, `/draft-ticket`, `/review-ticket`, `/start-ticket`, `/handoff`, `/manager-review`, `/setup-tickets`) is now available in **every** repo you open.

> Replace `Humble-Coders/humble-task-force` with this repo's actual `owner/name` if it differs.

## What's inside

- [`humble-task-force/`](./humble-task-force) — the plugin (commands + repo-scaffold templates). See its [README](./humble-task-force/README.md).

## Updating

Push changes here, then teammates run `/plugin marketplace update humble-coders` to pull the latest commands.
