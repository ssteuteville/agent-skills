# acli jira workitem comment update

Update an existing comment on a work item.

## Usage

```
acli jira workitem comment update [flags]
```

## Flags

| Flag | Short | Type | Description |
|------|-------|------|-------------|
| `--key` | | string | Work item key (required) |
| `--id` | | string | Comment ID to update (required) |
| `--body` | `-b` | string | New comment body in plain text only |
| `--body-adf` | | string | Path to a JSON file (or process substitution) containing ADF |
| `--body-file` | `-F` | string | Path to a plain text file containing comment body |
| `--notify` | | bool | Notify users about the change |
| `--visibility-group` | | string | Restrict visibility to a specific group |
| `--visibility-role` | | string | Restrict visibility to a specific project role |
| `--help` | `-h` | bool | Show help |

## Body Input (provide one)

- `--body` — inline plain text string only (ADF JSON passed here is rendered as literal text)
- `--body-adf` — path to a `.json` file with ADF content; **the only reliable way to get rich formatting**. Supports process substitution: `--body-adf <(echo '<ADF_JSON>')`
- `--body-file` — path to a plain text file

## Examples

```bash
# Update with plain text
acli jira workitem comment update --key SK-100 --id 10001 --body "Updated text"

# Update with ADF from a file
acli jira workitem comment update --key SK-100 --id 10001 --body-adf comment.json

# Update with ADF via process substitution (no temp file)
acli jira workitem comment update --key SK-100 --id 10001 --body-adf <(echo '{"version":1,"type":"doc","content":[{"type":"paragraph","content":[{"type":"text","text":"Rich text","marks":[{"type":"strong"}]}]}]}')

# Update with restricted visibility
acli jira workitem comment update --key SK-100 --id 10001 --body "Internal note" --visibility-role "Administrators"

# Update and notify watchers
acli jira workitem comment update --key SK-100 --id 10001 --body "Important update" --notify
```
