# Attach File to PR Skill Maintenance Playbook

Maintenance guide for the `gh-pr-attach-file` skill at `../../pr-attach-file/`.

## Skill Files

| File | Purpose |
|------|---------|
| `../../pr-attach-file/SKILL.md` | Sibling-branch push + PR-body splice flow |

## Testing Conventions

- Use a throwaway test branch (e.g. `agent-test/attach-check`) with an open draft PR.
- Always close and delete both the test PR and the `*_screenshots` sibling branch when done.
- Use trivial test files (a 1x1 PNG, a small text PDF) so the orphan branch stays small.

## Step 1: Version Check

```bash
gh --version
```

```bash
git --version
```

## Step 2: Audit CLI Help

```bash
gh pr edit --help
```

```bash
gh pr view --help
```

```bash
gh api --help
```

```bash
git worktree --help
```

Confirm:
- `gh pr edit` still accepts `--body-file`.
- `gh pr view --json body,number,url` still returns those fields.
- `git worktree add` still accepts `--detach` and the `add <path> <branch>` form.
- `git checkout --orphan` still exists.

## Step 3: Update Skill Files

Update `../../pr-attach-file/SKILL.md` if any of the above flags changed, or if GitHub's `blob/<branch>/<path>?raw=true` URL form has been deprecated.

## Step 4: Test Flow

### 4a. Create a test PR

```bash
git checkout -b agent-test/attach-check
```

```bash
git commit --allow-empty -m "test: attach-file maintenance check"
```

```bash
git push -u origin HEAD
```

```bash
gh pr create --draft --title "test: attach-file maintenance check" --body "Body before attachment."
```

Record `PR_NUMBER`.

### 4b. Prepare test files

```bash
printf '\x89PNG\r\n\x1a\n' > /tmp/attach-test.png
```

```bash
echo "hello" > /tmp/attach-test.txt
```

### 4c. Run the skill on the image

Invoke the skill with `/tmp/attach-test.png` and an alt text. Verify:

1. A sibling branch `agent-test/attach-check_screenshots` is created.
2. The file lands at the root of that branch as `attach-test.png`.
3. The PR body is updated with `![<alt>](https://github.com/<owner>/<repo>/blob/agent-test/attach-check_screenshots/attach-test.png?raw=true)`.
4. Opening the PR in a browser renders the image inline.

### 4d. Run the skill on the non-image file

Invoke again with `/tmp/attach-test.txt`. Verify:

1. The same sibling branch picks up the new file (no re-orphan).
2. The PR body gains a `[<caption>](url)` line (linked, not embedded).
3. The link displays the text content when clicked.

### 4e. Verify re-run behavior

Run a third time with a PNG of the same basename but different content. Confirm the existing top-level file is overwritten in a new commit on the screenshots branch, and the PR body gains a fresh markdown line pointing to the same URL.

## Step 5: Cleanup

```bash
gh pr close <PR_NUMBER> --delete-branch
```

```bash
git push origin --delete agent-test/attach-check_screenshots
```

```bash
git worktree list
```

If `.git/tmp-attach` is still listed:

```bash
git worktree remove .git/tmp-attach --force
```

```bash
git checkout main
```

```bash
rm -f /tmp/attach-test.png /tmp/attach-test.txt /tmp/pr-body.md
```
