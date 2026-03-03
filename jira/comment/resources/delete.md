# acli jira workitem comment delete

Delete a comment from a work item.

## Usage

```
acli jira workitem comment delete [flags]
```

## Flags

| Flag | Short | Type | Description |
|------|-------|------|-------------|
| `--key` | | string | Work item key (required) |
| `--id` | | string | Comment ID to delete (required) |
| `--help` | `-h` | bool | Show help |

## Examples

```bash
# Delete a specific comment
acli jira workitem comment delete --key SK-100 --id 10001
```

## Notes

- Both `--key` and `--id` are required.
- Get the comment ID from `acli jira workitem comment list --key <KEY> --json`.
- Deletion is permanent and cannot be undone.
