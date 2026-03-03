# Comment Skill Maintenance Playbook

Maintenance guide for the `jira-comment` skill located at `../../comment/`.

## Skill Files

| File | Purpose |
|------|---------|
| `../../comment/SKILL.md` | Main skill instructions and workflows |
| `../../comment/resources/create.md` | `comment create` flags and examples |
| `../../comment/resources/list.md` | `comment list` flags and examples |
| `../../comment/resources/update.md` | `comment update` flags and examples |
| `../../comment/resources/delete.md` | `comment delete` flags and examples |
| `../../comment/resources/visibility.md` | `comment visibility` flags and examples |
| `../../comment/resources/adf-reference.md` | ADF node/mark reference for rich formatting |

## Testing Conventions

- Test comments must start with bold **[Agent testing]** (not `[Agent]`) so they are distinguishable from real agent-created comments.
- Never leave more than 1 test comment on an issue at a time.
- Always delete test comments when done.
- No temp files — use process substitution `<(echo '...')` for ADF.
- Iterate with the user — ask them to verify rendering via screenshot after each test comment.

## Step 1: Version Check

```bash
acli --version
```

Compare against the version in `../../comment/SKILL.md` (look for the "Built and tested with" line). If they differ, expect potential behavioral changes.

## Step 2: Audit CLI Help

Run each of these and compare against the corresponding resource file:

```bash
acli jira workitem comment --help
acli jira workitem comment create --help
acli jira workitem comment list --help
acli jira workitem comment update --help
acli jira workitem comment delete --help
acli jira workitem comment visibility --help
```

For each command:
1. Read the resource file in `../../comment/resources/`.
2. Run the `--help` command.
3. Compare flags, descriptions, and defaults.
4. Note any new flags, removed flags, renamed flags, or changed behavior.
5. Check if there are new subcommands under `acli jira workitem comment`.

If discrepancies are found, search the web for acli release notes between the two versions.

## Step 3: Update Resource Files

Update each resource file in `../../comment/resources/` to match the current `--help` output. Then update `../../comment/SKILL.md`:
- Update the CLI version line.
- Update any workflow steps affected by flag changes.
- Update quirks/caveats if behavior has changed.

## Step 4: Test Flow

Ask the user for a Jira issue key to test on, then run through this sequence.

### 4a. Test Plain-Text Create

```bash
acli jira workitem comment create --key <ISSUE_KEY> --body "[Agent testing] plain text test"
```

Verify it appears. Get the comment ID:

```bash
acli jira workitem comment list --key <ISSUE_KEY> --json
```

Delete it:

```bash
acli jira workitem comment delete --key <ISSUE_KEY> --id <COMMENT_ID>
```

### 4b. Test Rich-Text Create (Two-Step Workflow)

Create placeholder:

```bash
acli jira workitem comment create --key <ISSUE_KEY> --body "[Agent testing]"
```

Get comment ID:

```bash
acli jira workitem comment list --key <ISSUE_KEY> --json
```

Update with ADF that exercises at least 5 features. Use process substitution (no temp files):

```bash
acli jira workitem comment update --key <ISSUE_KEY> --id <COMMENT_ID> --body-adf <(echo '<ADF_JSON>')
```

The ADF should test:
- Bold `[Agent testing]` prefix (strong mark)
- Headings (h2, h3)
- Italic, strikethrough, inline code, link marks
- Bullet list and ordered list
- Code block with language
- Blockquote
- Horizontal rule

**Ask the user to verify the rendering via screenshot.** If anything doesn't render:
1. Delete the comment.
2. Investigate (check if the ADF node is still supported, try variants).
3. Retry and ask for another screenshot.
4. Repeat until all features render correctly or are confirmed unsupported.

### 4c. Re-validate Known Quirks

These are known behaviors from when the skill was built. Confirm they still hold:

| Quirk | How to test |
|-------|-------------|
| `create --body` strips ADF inline marks | Create with inline ADF JSON via `--body`, ask user to check if marks render |
| `create --body-file` strips ADF inline marks | Create with ADF file via `--body-file`, ask user to check |
| `update --body` treats ADF as literal text | Update with ADF JSON string via `--body`, check if raw JSON appears |
| `update --body-adf` renders ADF correctly | Already tested in 4b |
| `update --body-adf <(echo '...')` works | Already tested in 4b |
| Headings work via `--body-adf` | Already tested in 4b |

If any quirk has changed (e.g. `create` now properly renders ADF), update the skill docs to reflect the new behavior and simplify workflows accordingly.

### 4d. Test Delete

Delete the test comment from 4b:

```bash
acli jira workitem comment delete --key <ISSUE_KEY> --id <COMMENT_ID>
```

## Step 5: Cleanup Checklist

1. List all comments and verify no test comments remain:

```bash
acli jira workitem comment list --key <ISSUE_KEY> --json
```

2. Confirm no comments have body starting with `[Agent testing]`.
3. Confirm no temp files were created on disk.
4. Update the CLI version in `../../comment/SKILL.md` to the current installed version.
