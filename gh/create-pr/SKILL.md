---
name: gh-create-pr
description: Create a GitHub pull request for the current branch using gh CLI. Diffs against the repo's base branch, extracts Jira slug from branch name if present, and generates a concise PR description. Use when the user asks to create a PR, open a pull request, submit changes for review, or push a PR to GitHub.
---

# Create PR

Create a GitHub PR for the current branch with an auto-generated description based on the diff against the base branch.

## Prerequisites

`gh` must be installed and authenticated. See [../README.md](../README.md).

Built and tested with `gh version 2.86.0`.

## Step 1: Determine base branch

If the user explicitly specified a base branch (e.g., "create a PR against dev"), use that and skip the detection below.

Otherwise, run these in parallel:

```bash
gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name'
```

```bash
git branch -r --list 'origin/main' 'origin/dev' 'origin/develop' 'origin/master'
```

Use the GitHub default branch as `BASE`. If multiple long-lived branches exist (e.g., both `origin/main` and `origin/dev` are present) and the user did not specify one, ask the user which branch to target before proceeding.

Refer to the chosen branch as `BASE` throughout the remaining steps.

## Step 2: Gather context

Run these in parallel:

```bash
git branch --show-current
```

```bash
git log --oneline <BASE>..HEAD
```

```bash
git diff <BASE>...HEAD --stat
```

```bash
git diff <BASE>...HEAD
```

Extract the branch name. If it matches `^[A-Z][A-Z0-9]+-\d+` (e.g., `PROJ-1234-add-auth`), extract the Jira slug (the matching prefix). This will be used in the PR description.

## Step 3: Write the PR title

Derive a concise title from the branch name or commit messages. Use imperative mood (e.g., "Add user authentication", "Fix login validation").

If a Jira slug was extracted, prefix the title: `PROJ-1234: Add user authentication`.

## Step 4: Write the PR description

Use this template:

```markdown
## Task

<!-- Jira link if slug was extracted, otherwise remove this section -->

## Notes

<!-- High-level bullet points covering:
- What changed and why
- Non-obvious design decisions or trade-offs
- Anything a reviewer should pay attention to
-->

## Visual

<!-- Leave empty for user to fill in -->
```

### Description guidelines

- Read the full diff from Step 2 to understand what changed
- Write 3-8 high-level bullet points in the Notes section — focus on *what* and *why*, not line-by-line changes
- Call out non-obvious decisions, trade-offs, or workarounds
- If a Jira slug was extracted, set the Task section to the Jira issue URL (e.g., `https://<org>.atlassian.net/browse/<SLUG>`). Ask the user for their Jira base URL if not already known.
- If no Jira slug, remove the Task section entirely
- Keep it concise — this is a starting point for the user to iterate on

## Step 5: Confirm with user

Present the title and description to the user before creating the PR. Wait for approval or edits.

## Step 6: Push and create PR

```bash
git push -u origin HEAD
```

```bash
gh pr create --title "<title>" --body "$(cat <<'EOF'
<description>
EOF
)"
```

Use `--base <BASE>` if needed. If the PR already exists, inform the user and provide the URL.

## Step 7: Report

Output the PR URL so the user can open it directly.
