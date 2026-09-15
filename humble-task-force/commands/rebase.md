---
description: Safely pull main into the branch you are on - commits your work first, records a recovery point, never touches main, stops at conflicts instead of guessing
argument-hint: (no arguments)
---
Bring the latest `main` underneath the work on the **current branch**, without ever putting
that work at risk. This is the command a developer runs each morning.

The guarantee this command makes is narrow and worth stating plainly: **everything that was
committed before it ran can be recovered afterwards.** It achieves that by committing first
and by refusing the handful of git operations that actually destroy work. It does not promise
a conflict-free rebase; conflicts are normal and it stops for the human.

## Never, under any circumstance

These are hard refusals. If one looks like the answer, the answer is wrong - stop and explain
instead.

- **Never `git rebase --skip`.** It silently DROPS the developer's commit. It is the single
  fastest way to lose work in a rebase and it looks harmless.
- **Never resolve a conflict with `-X ours`, `-X theirs`, `git checkout --ours/--theirs`, or
  by guessing.** A conflict means two humans changed the same lines. Only a human knows which
  is right.
- **Never `git reset --hard` or `git checkout .` while the tree is dirty.** Uncommitted work
  is the only thing git cannot give back.
- **Never `git push --force` (bare) and never `--no-verify`.** `--force-with-lease` only, and
  a refused push is information, not an obstacle.
- **Never push to `main`, and never rebase while standing on `main`.**
- **Never `git stash` as a way to get past the dirty-tree check.** Commit instead: a commit is
  visible in `git log`, a stash is invisible and gets forgotten.

## 1. Refuse to run in the wrong place

```bash
git rev-parse --abbrev-ref HEAD
git rev-parse --git-dir >/dev/null   # is this even a repo
```

If the branch is `main` (or `master`): **stop.** Explain that this command pulls main INTO a
feature branch, and that being on main means either they have not started a branch yet, or
they have work sitting on main that needs moving off it first. Offer to create a branch for
them; do not proceed.

If a rebase or merge is already in progress (`.git/rebase-merge`, `.git/rebase-apply`,
`.git/MERGE_HEAD` exist): **stop.** Report the half-finished state and offer `--continue` or
`--abort`. Never start a second one on top.

## 2. Get their work committed

```bash
git status --porcelain
```

If anything is uncommitted (staged, unstaged, or both), **commit it** - do not stash, do not
ask a long question about it. Show the developer the file list first, then:

```bash
git add -A
git commit -m "wip: before syncing with main"
```

Tell them in one line that this is undoable with `git reset --soft HEAD~1`, which puts the
changes back exactly as they were, still uncommitted.

Untracked files that are clearly not theirs to commit (build output, `local.properties`,
`.DS_Store`) are a signal the repo's `.gitignore` is wrong. Mention it, add the rest, do not
silently commit a secrets file. **If a file looks like it carries credentials, stop and say
so** rather than committing it.

## 3. Record the recovery point

```bash
git rev-parse HEAD
git log --oneline -1
```

Print that SHA to the developer with the sentence: *"if anything looks wrong afterwards,
`git reset --hard <sha>` puts you back exactly here."* This matters more than it looks: the
fear of rebasing is what stops people rebasing, and a visible way back removes it.

Also count what is about to be replayed, so step 6 can verify nothing vanished:

```bash
git rev-list --count origin/main..HEAD
```

## 4. Show what is arriving

```bash
git fetch origin main
git log --oneline HEAD..origin/main
```

Summarise it in plain language - how many commits, and what they touch. A developer who knows
"three commits, all in the TDS module" understands any conflict that follows. If the list is
empty, say "already up to date" and stop; there is nothing to do.

## 5. Rebase

```bash
git rebase origin/main
```

**If it succeeds:** say so, briefly.

**If it stops on a conflict, stop and hand it to the human.** Do not resolve it unasked.
Report:
- the exact files, from `git diff --name-only --diff-filter=U`
- what each side wanted, in one sentence each, read from the conflict markers
- the two ways forward:

```bash
# fix the files, then
git add <files> && git rebase --continue

# or back out completely, losing nothing
git rebase --abort
```

Then offer to walk through the resolution with them. If they accept, resolve one file at a
time and show the result before continuing. **`git rebase --abort` is always available and
always safe** - say so, because a developer who feels trapped will do something worse.

## 6. Verify nothing was lost

Before declaring success:

```bash
git rev-list --count origin/main..HEAD   # compare with the count from step 3
git log --oneline origin/main..HEAD
git status
```

The count must match step 3's, unless commits legitimately merged away (which happens when
their work already landed on main via a PR - explain that if it is the case). **If the count
dropped and there is no explanation, say so loudly** and give them the recovery SHA from
step 3. Do not paper over it.

## 7. Offer the push, do not just do it

The rebase rewrote their commits, so the next ordinary push is refused as non-fast-forward.
That is expected. Tell them, show the command, and push **only if they say yes**:

```bash
git push --force-with-lease origin <their-branch>
```

Explain `--force-with-lease` in one line: it refuses if somebody else pushed to that branch
meanwhile, so it cannot flatten another person's work. If it does get refused, that refusal
means somebody else is on their branch - fetch and look, never escalate to bare `--force`.

## Report

Short. What came in, whether it was clean, what is on their branch now, and the recovery SHA.
If the project has a rule about where the daily rebase is documented (many do), point at it
rather than restating it.
