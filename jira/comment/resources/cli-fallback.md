# CLI Fallback for Jira Comments

Use these instructions when the Atlassian MCP server is not available. Requires `acli` installed and authenticated. See [../../README.md](../../README.md). Built and tested with `acli version 1.3.13-stable`.

## Rich Text Warning

**Do not use Markdown syntax in CLI comments.** The CLI will display raw Markdown characters literally. Use Atlassian Document Format (ADF) JSON for rich text. See [adf-reference.md](adf-reference.md) for the full node/mark catalogue.

## Create a Comment

Reference: [create.md](create.md)

**Plain text:**

```bash
acli jira workitem comment create --key <ISSUE_KEY> --body "[Agent] Plain text comment"
```

**Rich text (ADF):** The CLI `create` command's `--body` and `--body-file` flags claim to accept ADF, but **inline marks (bold, italic, links) are silently stripped**. Use a two-step create-then-update workflow:

1. Create a placeholder: `acli jira workitem comment create --key <KEY> --body "[Agent]"`
2. Get the comment ID: `acli jira workitem comment list --key <KEY> --json`
3. Update with ADF: `acli jira workitem comment update --key <KEY> --id <ID> --body-adf <(echo '<ADF_JSON>')`

## Read / List Comments

Reference: [list.md](list.md)

```bash
acli jira workitem comment list --key <ISSUE_KEY>

acli jira workitem comment list --key <ISSUE_KEY> --json
```

Each comment in the JSON response has `id`, `author`, `body`, and `visibility`. Use `id` for subsequent update or delete operations.

## Update a Comment

Reference: [update.md](update.md)

```bash
acli jira workitem comment update --key <ISSUE_KEY> --id <COMMENT_ID> --body "New text"

acli jira workitem comment update --key <ISSUE_KEY> --id <COMMENT_ID> --body-adf <(echo '<ADF_JSON>')
```

Steps:
1. Get the comment ID by listing comments.
2. For plain text, use `--body`. For rich formatting, build ADF JSON and use `--body-adf <(echo '...')`.
3. Optionally add `--notify` to alert watchers, or `--visibility-role`/`--visibility-group` to restrict access.

## Delete a Comment

Reference: [delete.md](delete.md)

```bash
acli jira workitem comment delete --key <ISSUE_KEY> --id <COMMENT_ID>
```

Deletion is permanent. Get the comment ID by listing comments first.

## Check Visibility Options

Reference: [visibility.md](visibility.md)

```bash
acli jira workitem comment visibility --role --project <PROJECT_KEY>

acli jira workitem comment visibility --group
```

Use these to discover valid values for `--visibility-role` and `--visibility-group` flags on create/update.

## ADF Quick-Start Template

Minimal ADF comment with the required `[Agent]` prefix:

```json
{
  "version": 1,
  "type": "doc",
  "content": [
    {
      "type": "paragraph",
      "content": [
        { "type": "text", "text": "[Agent]", "marks": [{ "type": "strong" }] }
      ]
    },
    {
      "type": "paragraph",
      "content": [
        { "type": "text", "text": "Description of the update." }
      ]
    }
  ]
}
```

For headings, lists, code blocks, blockquotes, and other elements, see [adf-reference.md](adf-reference.md).

## Troubleshooting

1. Check the installed CLI version: `acli --version`
2. Compare against the version at the top of this file.
3. If they differ, the CLI may have introduced breaking changes. Run the `jira-skill-maintenance` skill to re-validate and update this skill against the current CLI.
