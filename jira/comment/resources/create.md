# acli jira workitem comment create

Add a comment to a work item or multiple work items using the default visibility for the project.

**Warning**: Although `--body` and `--body-file` claim to accept ADF, inline marks (bold, italic, links, etc.) are silently stripped. For rich-formatted comments, create a plain-text placeholder then update with `--body-adf`. See the SKILL.md two-step workflow.

## Usage

```
acli jira workitem comment create [flags]
```

## Flags

| Flag | Short | Type | Description |
|------|-------|------|-------------|
| `--body` | `-b` | string | Comment body in plain text (ADF marks are not preserved) |
| `--body-file` | `-F` | string | Path to a plain text file (ADF marks are not preserved) |
| `--edit-last` | `-e` | bool | Edit the last comment from the same author instead of creating new |
| `--editor` | | bool | Open text editor to write the body (interactive, not for automation) |
| `--filter` | | string | Filter ID of work items to comment |
| `--jql` | | string | JQL query for work items to comment |
| `--key` | `-k` | string | A list of work item keys to comment |
| `--json` | | bool | Generate JSON output |
| `--ignore-errors` | | bool | Ignore errors and continue |
| `--help` | `-h` | bool | Show help |

## Target Selection (mutually exclusive)

Exactly one of `--key`, `--jql`, or `--filter` is required to identify which work items to comment on.

## Examples

```bash
# Plain text comment
acli jira workitem comment create --key "SK-100" --body "Simple text comment"

# Comment on multiple issues via JQL
acli jira workitem comment create --jql "project = SK AND sprint in openSprints()" --body "Sprint note"
```
