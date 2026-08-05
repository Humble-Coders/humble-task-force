---
description: Pack this project's entire brain — transcripts, memory, un-versioned dirs, scratch artifacts, secrets — into one encrypted bundle for another PC
argument-hint: (no arguments)
---
Move this project — **including the literal conversation history** — to another PC. The receiving
machine's Claude must not get a retelling; it gets the same transcripts, so `claude --resume` there
IS this chat. You are packing six layers into one staging folder, then handing the user a single
encryption command. Be thorough and do not economise on tokens where the instructions say to write.

## Ground rules (read first)

- **The transcripts are radioactive.** Every credential ever pasted into any chat of this project is
  inside the `.jsonl` files in plain text. The bundle therefore ALWAYS travels encrypted, and NEVER
  through a git repo, cloud drive, or anything that syncs — direct machine-to-machine only
  (AirDrop / scp / USB).
- **You never see the passphrase.** The user runs the encryption command themselves; openssl prompts
  them interactively. Do not ask for it, do not put it in any command you run.
- Nothing here modifies the project. You copy; you don't move.

## 1. Establish identity

- `SLUG` = the current working directory with every `/` replaced by `-` (e.g.
  `/Users/x/Projects/app` → `-Users-x-Projects-app`).
- `CLAUDE_PROJ=~/.claude/projects/$SLUG` — verify it exists and count the `*.jsonl` sessions. If it
  doesn't exist, stop and say so: there is nothing to hand off.
- Create a staging dir: `STAGE=$(mktemp -d)/pc-handoff-$(date +%Y%m%d)` — under mktemp, not the
  repo, so nothing can ever be committed by accident.

## 2. Layer 1+2 — conversation + memory (the brain itself)

Copy the **whole** `$CLAUDE_PROJ` directory into `$STAGE/claude-project/`: every session `.jsonl`
(all of them — the user chose all-sessions scope) and the `memory/` directory. If `~/.claude/todos/`
exists, also copy any file whose name contains one of this project's session ids into
`$STAGE/claude-todos/`.

## 3. Layer 3 — the repos and un-versioned directories

Identify **every local directory this project's work touches** — from `CLAUDE.md`, the memory files,
and the conversation itself. Typically: the main repo, sibling repos (a gateway, a backend), and any
directory that has come up in the work. For each one:

- **Git repo, clean and pushed** → do NOT copy it. Record in the manifest: path, remote URL, branch,
  HEAD sha. The other PC clones it.
- **Git repo, dirty or unpushed** → tell the user what's uncommitted and offer two options: push a
  WIP branch now (preferred), or bundle a patch (`git diff` + `git ls-files --others
  --exclude-standard` files copied under `$STAGE/patches/<repo>/`). Record which was done.
- **Not a git repo at all** → this bundle is the only way it travels; copy it wholesale into
  `$STAGE/dirs/<basename>/`, excluding `node_modules`, `build`, `.gradle`, `dist`, `.next`,
  `__pycache__` (all restorable by install/build). Say explicitly in your output that this directory
  has no version control and the bundle is currently its only backup.

## 4. Layer 4 — scratchpad artifacts

Sweep this project's session scratchpads (on macOS: `/private/tmp/claude-*/$SLUG/*/scratchpad`).
Copy the files the conversation **deliberately created** — scripts, checklists, verification
harnesses, payload samples — into `$STAGE/scratchpad/`. Skip build logs, downloaded artifacts, and
archives. When unsure, a file under ~1 MB that a chat wrote on purpose goes in.

## 5. Layer 5 — secrets (user opted these INTO the encrypted bundle)

From each repo, collect the gitignored secret files the project needs to run: `**/secrets/**`,
`.env*` (never `.env.example`), `*-sa.json`, `*.pem`, keystores. Place them under
`$STAGE/secrets/<repo-basename>/<relative-path>` so restore can put each one back exactly where it
came from. List every file you included in your output — the user should see what's travelling.

## 6. Layer 6 — HANDOFF.md (do not be brief)

Write `$STAGE/HANDOFF.md` as if the reader is a competent colleague who has NEVER seen this project
and the transcripts somehow failed to restore. Spend tokens. Cover, at minimum:

- What the product is and where every repo lives (remote URLs, branches, what each is for).
- The **deployed state** of everything: which functions/services are live where, at which commit,
  anything known to be stale.
- Open threads: PRs in flight, tickets in progress, decisions awaiting the user.
- Every environment quirk and hard-won lesson (mine the memory files and the conversation — the
  things that cost a day the first time).
- Credentials the new PC will need — **names and where to get them, never values** (the transcripts
  already carry any values pasted in chat; do not add more copies).
- What to do first on the new machine.

## 7. MANIFEST.json

Write `$STAGE/MANIFEST.json`: `schemaVersion: 1`, `createdAt`, `sourceHome`, `sourceCwd`, `slug`,
session ids, and an entry per item: what it is, its **source path** on this machine, its path inside
the bundle, and for repos the remote/branch/HEAD/dirty-handling. This is what restore uses to
rewrite paths — the source paths must be exact.

## 8. RESTORE.md + restore.sh (self-contained — the new PC has no plugin yet)

Write both into `$STAGE/`. They must work on a bare machine with only bash + python3 + git.

`restore.sh` must, in order: read MANIFEST; ask the user where the project should live on this
machine (default: same path under the new `$HOME`); clone each recorded repo at its branch (and
apply any bundled patch); place each `dirs/` directory; place each secret back to its recorded
target; compute the NEW slug from the new cwd, create `~/.claude/projects/<new-slug>/` (backing up
any existing one first, never overwriting), copy the transcripts + memory in, and **rewrite paths
inside every `.jsonl` and memory file** — replace each old source path with its new location,
longest paths first, then old home → new home (use python3 for the rewrite, not sed — BSD/GNU sed
differ). Finish by printing: a re-auth checklist (`gh auth login`, `firebase login`,
`aws configure`, SSH keys, plus whatever this project's HANDOFF says), any install steps
(`npm install` where node_modules was excluded), and finally:
`cd <new-project-path> && claude --resume`.

`RESTORE.md` explains the same for a human, starting from "you have the `.enc` file", including the
decrypt command and that `/pc-restore` (this plugin, if installed) can drive the whole thing instead
of `restore.sh`.

## 9. Hand encryption to the user, then clean up

Print the staging summary (layer-by-layer, with sizes), then give the user these to run themselves
— filling in real paths, one command per block:

```
tar -czf - -C "$(dirname "$STAGE")" "$(basename "$STAGE")" | openssl enc -aes-256-cbc -pbkdf2 -iter 200000 -salt -out ~/Desktop/pc-handoff-<project>-<date>.enc
```

Verify (should list the bundle contents; wrong passphrase = garbage error):

```
openssl enc -d -aes-256-cbc -pbkdf2 -iter 200000 -in ~/Desktop/pc-handoff-<project>-<date>.enc | tar -tzf - | head
```

Then a transfer example (`scp` form plus "or AirDrop / USB"), and the on-the-other-side commands:
decrypt + extract, then `bash <bundle>/restore.sh` (or `/pc-restore <bundle>` if the plugin is
installed there).

**Only after the user confirms the verify listing worked**: `rm -rf` the staging dir (it holds
plaintext secrets), and remind them to delete the `.enc` from BOTH machines once the restore is
done — and that rotating anything sensitive that ever appeared in chat remains a good idea
regardless.
