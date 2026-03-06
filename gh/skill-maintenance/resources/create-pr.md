# Create PR Skill Maintenance Playbook

Maintenance guide for the `gh-create-pr` skill located at `../../create-pr/`.

## Skill Files

| File | Purpose |
|------|---------|
| `../../create-pr/SKILL.md` | Main skill instructions and workflows |

## Testing Conventions

- Use a dedicated test branch for PR creation tests. Name it something obvious like `agent-test/maintenance-check`.
- Always close and delete test PRs when done.
- Never merge test PRs.

## Step 1: Version Check

```bash
gh --version
```

Compare against the version in `../../create-pr/SKILL.md` (look for the "Built and tested with" line). If they differ, expect potential behavioral changes.

## Step 2: Audit CLI Help

Run each of these and compare against the skill's SKILL.md:

```bash
gh pr create --help
gh pr view --help
gh repo view --help
```

For each command:
1. Read the skill file at `../../create-pr/SKILL.md`.
2. Run the `--help` command.
3. Compare flags, descriptions, and defaults against what the skill uses.
4. Note any new flags, removed flags, renamed flags, or changed behavior.
5. Pay special attention to `--title`, `--body`, `--base`, and any flags the skill relies on.

If discrepancies are found, search the web for gh CLI release notes between the two versions.

## Step 3: Update Skill Files

Update `../../create-pr/SKILL.md`:
- Update the CLI version line.
- Update any workflow steps affected by flag changes.
- Update command examples if syntax has changed.

## Step 4: Test Flow

Ask the user which repo to test in, then run through this sequence.

### 4a. Create a Test Branch

```bash
git checkout -b agent-test/maintenance-check
git commit --allow-empty -m "test: maintenance check for create-pr skill"
git push -u origin HEAD
```

### 4b. Test Context Gathering

Verify the context-gathering commands from Step 1 of the skill:

```bash
git branch --show-current
git log --oneline main..HEAD
git diff main...HEAD --stat
git diff main...HEAD
```

Confirm all four produce reasonable output.

### 4c. Test PR Creation

Run the PR creation flow:

```bash
gh pr create --title "test: maintenance check" --body "$(cat <<'EOF'
## Notes

- Automated maintenance check — will be closed immediately.
EOF
)"
```

Verify the PR was created successfully. Record the PR number.

### 4d. Test Jira Slug Detection

If the skill's Jira slug regex `^[A-Z][A-Z0-9]+-\d+` is still relevant, verify the regex logic by checking the branch name parsing. This is a documentation/logic check — no CLI commands needed.

### 4e. Verify PR Exists

```bash
gh pr view --json number,title,body,url
```

Confirm the title, body, and URL are correct.

## Step 5: Cleanup

1. Close the test PR:

```bash
gh pr close <PR_NUMBER> --delete-branch
```

2. Verify the branch was deleted:

```bash
git branch -a | grep agent-test/maintenance-check
```

3. Switch back to the previous branch:

```bash
git checkout main
```

4. Update the CLI version in `../../create-pr/SKILL.md` to the current installed version.
