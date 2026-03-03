# acli jira workitem comment list

List comments for a work item.

## Usage

```
acli jira workitem comment list [flags]
```

## Flags

| Flag | Short | Type | Default | Description |
|------|-------|------|---------|-------------|
| `--key` | | string | | Work item key (required) |
| `--json` | | bool | false | Output in JSON format |
| `--limit` | | int | 50 | Maximum number of comments to return per page |
| `--order` | | string | `+created` | Order by field: `created` or `updated`. Prefix `+` for ascending, `-` for descending |
| `--paginate` | | bool | false | Fetch all pages of results (ignores `--limit`) |
| `--help` | `-h` | bool | | Show help |

## JSON Output Shape

```json
{
  "comments": [
    {
      "author": "Display Name",
      "body": "Plain-text rendering of the comment",
      "id": "123456",
      "visibility": "public"
    }
  ],
  "isLast": false,
  "maxResults": 50,
  "startAt": 0,
  "total": 1
}
```

## Examples

```bash
# List all comments (table format)
acli jira workitem comment list --key SK-100

# List as JSON (for parsing)
acli jira workitem comment list --key SK-100 --json

# Most recent comments first
acli jira workitem comment list --key SK-100 --order "-created"

# Get all comments with pagination
acli jira workitem comment list --key SK-100 --paginate
```
