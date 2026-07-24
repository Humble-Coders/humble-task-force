---
description: File a clean bug report as a GitHub issue — asks platform + severity, uploads any screenshot, and creates the labelled issue on the target repo
argument-hint: [what went wrong] [--repo owner/repo]
---
You are helping a **tester** file a bug as a GitHub issue. The tester describes what they saw in plain
words — your job is to turn it into a **clear, well-labelled issue** on the target repo, with any
screenshot attached. **You are NOT fixing the bug** (that's `/fetch-bug` on the developer's side) and you
are NOT touching any code — you are only capturing the report cleanly.

## 1. Gather the report
Take the bug description from **$ARGUMENTS**. Then make sure you have the following — **ask only for what's
missing**, a couple at a time (use AskUserQuestion if available):

- **Platform** *(required)* — where the bug happened: **Desktop**, **Web**, **Android**, **iOS**, or
  **Other / multiple**. If the tester already said which, don't re-ask.
- **Severity** *(required)* — **Critical** (blocks core use / data loss) · **High** (a major feature is
  broken) · **Medium** (works but behaves wrong) · **Low** (cosmetic / minor).
- **Screenshot(s)** *(optional)* — ask for a **file path** to any screenshot on their machine
  (e.g. `~/Desktop/bug.png`). Skip if they have none. *(A pasted/dragged image only works if your Claude
  Code saved it to a real file you can read; if you're not sure you have a readable file path, ask them to
  save the screenshot and give you the path.)*
- **Target repo** — the `owner/repo` to file against. If a `--repo owner/repo` is in the arguments, use it.
  Otherwise default to the current directory's repo (`gh repo view --json nameWithOwner`); if you're not in
  a repo, ask for `owner/repo`.

Do **not** ask for reproduction steps in this version.

## 2. Clean up the description
Rewrite the tester's words into a clear, neutral report: **what happened**, and **what they expected**
instead (only if they said so). Preserve every concrete detail — screen names, values, on-screen error
text — and **invent nothing** (no repro steps you weren't given, no assumed causes). Keep it tight.

## 3. Attach screenshots (if any) — via release-asset hosting
The GitHub CLI can't attach an image to an issue directly, so host each screenshot as a **release asset** on
the target repo and embed its URL — this renders inline for repo members (verified). For each image:

```bash
REPO="<owner/repo>"
# Lazy-create the hosting release once (idempotent — safe to run every time):
gh release view bug-attachments --repo "$REPO" >/dev/null 2>&1 || \
  gh release create bug-attachments --repo "$REPO" --title "Bug report attachments" \
    --notes "Auto-managed by /report-bug — hosts images embedded in bug issues. Do not delete."
# Upload with a unique name, preserving the original extension (EXT below):
NAME="bug-$(date +%s)-<index>.<EXT>"
cp "<tester-file-path>" "/tmp/$NAME"
gh release upload bug-attachments --repo "$REPO" "/tmp/$NAME" --clobber
# Resolve the download URL to embed in the issue body:
gh release view bug-attachments --repo "$REPO" --json assets \
  --jq '.assets[] | select(.name=="'"$NAME"'") | .url'
```

Embed each as `![screenshot](<that url>)` in the issue body. If an upload fails, **tell the tester and file
the issue without the image** rather than aborting.

## 4. Create the issue
Make sure the labels exist (idempotent), then create:

```bash
gh label create bug --repo "$REPO" --color d73a4a --description "Something isn't working" 2>/dev/null || true
gh label create "platform:<x>" --repo "$REPO" --color 0e8a16 2>/dev/null || true
gh label create "severity:<x>"  --repo "$REPO" --color fbca04 2>/dev/null || true
gh issue create --repo "$REPO" --title "<short specific summary>" \
  --label bug --label "platform:<x>" --label "severity:<x>" --body "<body>"
```

Body layout:
- **Platform:** `<x>`  ·  **Severity:** `<x>`
- **What happened** — the cleaned description.
- **Screenshot(s)** — the embedded image(s), if any.
- a small footer: `_Filed via /report-bug._`

Title is a short, specific summary (e.g. "Desktop — inventory table columns overlap on narrow window"),
**not** the raw paragraph.

## 5. Confirm
Show the tester the **issue URL** and a one-line recap (title · platform · severity · image yes/no), and
tell them a developer can pick it up with **`/fetch-bug <number>`**.

## Guardrails
- **Never** put secrets, tokens, passwords, or API keys in the issue. If the tester's text or a screenshot's
  filename contains one, leave it out and say so.
- Only file the issue once you have the platform + severity + target repo. Don't edit any code or repo files
  other than creating the issue and (if needed) the `bug-attachments` release.
