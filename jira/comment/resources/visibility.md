# acli jira workitem comment visibility

Get visibility options for work item comments. Use this to discover available roles and groups before restricting comment visibility.

## Usage

```
acli jira workitem comment visibility [flags]
```

## Flags

| Flag | Type | Description |
|------|------|-------------|
| `--role` | bool | List project roles available for visibility |
| `--group` | bool | List available groups for visibility |
| `--project` | string | Project key (required when using `--role`) |
| `--help` | bool | Show help |

## Examples

```bash
# List roles for a project
acli jira workitem comment visibility --role --project SK

# List available groups
acli jira workitem comment visibility --group
```
