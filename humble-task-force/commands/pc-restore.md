---
description: Restore a /pc-handoff bundle on this PC — repos, secrets, and the literal conversation history, resumable with claude --resume
argument-hint: <path to the decrypted, extracted bundle directory>
---
Restore a `/pc-handoff` bundle on THIS machine so that `claude --resume` here continues the
original conversations literally — same transcripts, same memory, same project state. The bundle
also contains a standalone `restore.sh`; you are the smarter version of that script, with judgment.

The argument is the path to the **already decrypted and extracted** bundle directory (the one
containing `MANIFEST.json`). If the user gives you an `.enc` file instead, give them the decrypt
command from the bundle's RESTORE.md pattern and have them run it — the passphrase must never pass
through you.

## 1. Read the manifest, take stock

Read `MANIFEST.json` and `HANDOFF.md`. Present a short inventory: which repos, which un-versioned
directories, how many sessions, which secrets. Note `sourceHome` and `sourceCwd` — every path
rewrite below is derived from the manifest's source paths, longest first, then old home → new home.

## 2. Decide where things live (ask, don't guess)

For each top-level item, propose a destination — default: the same path relative to `$HOME` as on
the source machine — and confirm with the user before writing anything. If a destination already
exists, say what's there and never overwrite silently.

## 3. Repos

For each manifest repo: if the destination is already a clone of the same remote, `git fetch` and
check out the recorded branch; otherwise clone it (`gh repo clone` when authenticated, plain `git
clone` otherwise) and check out the recorded branch/HEAD. Apply any bundled patch from `patches/`
(`git apply`, then copy the untracked files in). Report each repo's final state: branch, sha, clean
or carrying the WIP patch.

## 4. Un-versioned directories, secrets, scratchpad

- Copy each `dirs/<name>` to its confirmed destination. These had NO version control on the source
  machine — suggest `git init` while you're at it; the bundle should not remain their only backup.
- Place each `secrets/<repo>/<relpath>` file into the restored repo at that relative path. Verify
  each landed inside a gitignored location (`git check-ignore`) — if one wouldn't be ignored at its
  destination, stop and flag it rather than planting a committable secret.
- Copy `scratchpad/` somewhere durable in the project (e.g. `<project>/.claude/scratch-restored/`,
  gitignored) and say where you put it — the original scratchpad location was ephemeral.

## 5. The conversation itself (the whole point)

- Compute `NEW_SLUG` from the confirmed project cwd (`/` → `-`).
- If `~/.claude/projects/$NEW_SLUG` already exists, back it up to a sibling
  (`$NEW_SLUG.pre-restore-<date>`) first.
- Copy `claude-project/` from the bundle to `~/.claude/projects/$NEW_SLUG/`.
- **Rewrite paths** in every `.jsonl` and every `memory/*.md`: for each manifest mapping, replace
  the old source path with the confirmed new path (longest first), then `sourceHome` → this
  machine's `$HOME`. Use python3 (read text, `str.replace`, write back) — not sed. Do the same for
  any `claude-todos/` files, into `~/.claude/todos/`.
- Sanity-check: grep one rewritten `.jsonl` for the OLD home path — zero hits expected. If hits
  remain, show where and fix before declaring success.

## 6. Re-auth and rebuild (files can't carry identity)

Walk the user through what the bundle cannot contain, checking each one's current state rather than
assuming: `gh auth status` (→ `gh auth login`), `firebase login`, cloud CLI credentials
(`aws sts get-caller-identity` or equivalent), SSH keys for any non-GitHub remotes, platform
toolchains (Xcode/Android SDK) if the project needs them, and any project-specific items HANDOFF.md
names. Then rebuild what the bundle excluded: `npm install` where `node_modules` was stripped, etc.

## 7. Finish line

Print, as the very last thing:

- `cd <new-project-path> && claude --resume` — and that the session picker will show the original
  conversations, which now continue on this machine as if nothing happened.
- A reminder to **delete the `.enc` archive and the extracted bundle from BOTH machines** now that
  the restore is done — the transcripts inside contain every credential ever pasted into chat.
- If HANDOFF.md lists credentials to rotate, repeat that list (names only).
