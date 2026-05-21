---
name: gh-pr-attach-file
description: Attach one or more local files (screenshots, diagrams, GIFs, PDFs, flamegraphs, anything) to a GitHub PR's description by committing them to a sibling `{branch}_screenshots` branch and splicing the raw GitHub URL into the PR body. Works for both public and private repos. Use when the user asks to "attach a file to my PR", "embed a screenshot in the PR", "add an image to the PR description", "put this PDF in the PR", or "show this in the PR body".
---

# Attach File to PR

Commit a local file to a sibling branch named `{current_branch}_screenshots` and splice the raw GitHub URL into the current PR's body. Image files render inline (`![](url)`); non-image files are linked (`[name](url)`). Works for public and private repos — private-repo URLs render for viewers authenticated to the repo.

How the file is *produced* is out of scope. Common sources: Playwright MCP screenshots, hand-taken screenshots, Figma exports, `pprof` SVGs, animated GIFs, generated PDFs. The user (or another skill) should hand this skill a path on disk.

## Prerequisites

- `gh` installed and authenticated. See [../README.md](../README.md).
- The current branch has an open PR. If not, run `/gh-create-pr` first.

Built and tested with `gh version 2.86.0` and `git version 2.50.1`.

## Step 1: Gather inputs

Ask the user (or accept up-front) for:

- One or more **absolute file paths** to attach.
- A short **alt text / caption** per file.
- Whether to **append** to the existing PR body or **replace a section** (e.g. the `## Visual` placeholder from `/gh-create-pr`).

Verify each file exists and is non-empty:

```bash
ls -lh "<PATH>"
```

Detect image vs non-image by extension (`.png .jpg .jpeg .gif .webp .svg` → image; everything else → link).

## Step 2: Gather repo + PR context

Run in parallel:

```bash
git branch --show-current
```

```bash
gh repo view --json nameWithOwner --jq .nameWithOwner
```

```bash
gh pr view --json number,url,body
```

Save as `BRANCH`, `OWNER_REPO`, `PR_NUMBER`, `PR_BODY`. If `gh pr view` errors with "no pull requests found", stop and tell the user to create a PR first.

Set `SCREENSHOTS_BRANCH="${BRANCH}_screenshots"`.

## Step 3: Push the file(s) to the sibling branch

Use a `git worktree` so the working tree is untouched. The screenshots branch is orphan-based to stay small and decoupled from feature history. The branch name keeps the historical `_screenshots` suffix even for non-screenshot files — it's a stable convention, not a description.

**If `origin/$SCREENSHOTS_BRANCH` already exists:**

```bash
git fetch origin "$SCREENSHOTS_BRANCH"
```

```bash
git worktree add .git/tmp-attach "$SCREENSHOTS_BRANCH"
```

**Otherwise create it as an orphan branch:**

```bash
git worktree add --detach .git/tmp-attach origin/main
```

Then from inside the worktree:

```bash
cd .git/tmp-attach && git checkout --orphan "$SCREENSHOTS_BRANCH"
```

Clear the inherited index so the orphan starts empty (robust against repos where `origin/main` has no tracked files):

```bash
cd .git/tmp-attach && git read-tree --empty
```

Copy each file to the **top level** of the worktree using its basename:

```bash
cp "<PATH>" ".git/tmp-attach/$(basename "<PATH>")"
```

(Run the `cp` once per file. If two source files share a basename, rename one before copying — or accept that the second copy overwrites the first.)

Commit and push from inside the worktree:

```bash
cd .git/tmp-attach && git add -A
```

```bash
cd .git/tmp-attach && git commit -m "attach files for $BRANCH"
```

```bash
cd .git/tmp-attach && git push -u origin "$SCREENSHOTS_BRANCH"
```

Clean up:

```bash
git worktree remove .git/tmp-attach --force
```

## Step 4: Build raw GitHub URLs

For each committed file, use the `blob/...?raw=true` form:

```
https://github.com/<OWNER_REPO>/blob/<SCREENSHOTS_BRANCH>/<filename>?raw=true
```

Example: a file `landing.png` on branch `web/web-landing_screenshots` of `skyslope/agent-calculator` →
`https://github.com/skyslope/agent-calculator/blob/web/web-landing_screenshots/landing.png?raw=true`

Note: branch names containing `/` (e.g. `web/web-landing_screenshots`) need no escaping — GitHub parses the path lazily.

Optional sanity check:

```bash
gh api "repos/<OWNER_REPO>/contents/<filename>?ref=<SCREENSHOTS_BRANCH>" --jq .download_url
```

## Step 5: Compose the markdown

- **Image** (`.png .jpg .jpeg .gif .webp .svg`):
  ```markdown
  ![<alt text>](<raw URL>)
  ```
- **Other** (PDF, MP4, anything else):
  ```markdown
  [<caption>](<raw URL>)
  ```

Note: MP4/WebM linked this way will *download* rather than render inline. For inline video playback in PR bodies, GitHub requires uploading via the web UI's drag-drop (which produces `user-attachments/assets/<uuid>` URLs) — that's not what this skill does. Recommend the `github-pr-video` skill for true inline video.

## Step 6: Splice into the PR body

Show the user the proposed new body and confirm append vs section-replace. Write the new body to a file:

```bash
gh pr edit <PR_NUMBER> --body-file /tmp/pr-body.md
```

## Step 7: Report

Output the PR URL and each raw attachment URL so the user can confirm rendering. GitHub may take a few seconds to cache new images on first load.

## Re-running

Re-invoke the skill anytime to add more attachments. Step 3 detects the existing `${BRANCH}_screenshots` branch and adds new files at its root (or overwrites on basename collision — rename the source if you want to keep both). Step 6 appends additional markdown lines to the PR body (unless the user picks section-replace).
