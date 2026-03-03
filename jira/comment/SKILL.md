---
name: jira-comment
description: Create, read, update, and delete Jira issue comments via the Atlassian CLI (acli). Use when the user wants to add, edit, list, view, or remove comments on a Jira issue or work item, or needs to post rich-formatted notes to Jira.
---

# Jira Comment CRUD

Manage Jira work item comments via `acli jira workitem comment`. For detailed flags and options for each command, read the corresponding file in [resources/](resources/).

## Prerequisites

`acli` must be installed and authenticated. See [../README.md](../README.md).

## Rich Text Formatting

**Do not use Markdown syntax in comments.** Jira will display raw Markdown characters (`**`, `#`, etc.) literally. Instead, use Atlassian Document Format (ADF) JSON to produce rich text.

See [resources/adf-reference.md](resources/adf-reference.md) for the full ADF node/mark catalogue.

### Critical: Create Does Not Render ADF Properly

The `create` command's `--body` and `--body-file` flags claim to accept ADF, but **inline marks (bold, italic, links, etc.) are silently stripped**. Only the `update` command's `--body-adf` flag reliably renders full ADF.

To create a rich-formatted comment, use a **two-step create-then-update** workflow:

```bash
# Step 1: Create a plain-text placeholder
acli jira workitem comment create --key <ISSUE_KEY> --body "placeholder"

# Step 2: Get the comment ID
acli jira workitem comment list --key <ISSUE_KEY> --json

# Step 3: Update with rich ADF using process substitution (no temp files needed)
acli jira workitem comment update --key <ISSUE_KEY> --id <COMMENT_ID> --body-adf <(echo '<ADF_JSON>')
```

Process substitution `<(echo '...')` feeds inline JSON to `--body-adf` without creating a temp file on disk. This is the recommended approach.

## Operations

### Create a Comment (Rich Text)

For rich-formatted comments, always use the two-step workflow above.

Steps:
1. Build ADF JSON per [resources/adf-reference.md](resources/adf-reference.md).
2. Create a plain-text placeholder: `acli jira workitem comment create --key <KEY> --body "placeholder"`
3. List comments to get the new ID: `acli jira workitem comment list --key <KEY> --json`
4. Update with ADF: `acli jira workitem comment update --key <KEY> --id <ID> --body-adf <(echo '<ADF_JSON>')`

### Create a Comment (Plain Text)

Reference: [resources/create.md](resources/create.md)

```bash
acli jira workitem comment create --key <ISSUE_KEY> --body "Plain text comment"
```

### List / Read Comments

Reference: [resources/list.md](resources/list.md)

```bash
# Table format
acli jira workitem comment list --key <ISSUE_KEY>

# JSON (for parsing comment IDs, authors, bodies)
acli jira workitem comment list --key <ISSUE_KEY> --json
```

Steps:
1. Run the list command with `--key`. Add `--json` when you need to parse the output programmatically.
2. Each comment in the JSON response has an `id`, `author`, `body` (plain-text rendering), and `visibility`.
3. Use the `id` field for subsequent update or delete operations.

### Update a Comment

Reference: [resources/update.md](resources/update.md)

```bash
# Plain text update
acli jira workitem comment update --key <ISSUE_KEY> --id <COMMENT_ID> --body "New text"

# Rich text update (ADF) — use process substitution:
acli jira workitem comment update --key <ISSUE_KEY> --id <COMMENT_ID> --body-adf <(echo '<ADF_JSON>')
```

Steps:
1. Get the comment ID by listing comments (see above).
2. For plain text, use `--body`. For rich formatting, build ADF JSON and use `--body-adf <(echo '...')`.
3. Optionally add `--notify` to alert watchers, or `--visibility-role`/`--visibility-group` to restrict access.

### Delete a Comment

Reference: [resources/delete.md](resources/delete.md)

```bash
acli jira workitem comment delete --key <ISSUE_KEY> --id <COMMENT_ID>
```

Steps:
1. Get the comment ID by listing comments.
2. Run the delete command with `--key` and `--id`.
3. Deletion is permanent.

### Check Visibility Options

Reference: [resources/visibility.md](resources/visibility.md)

```bash
# Available roles for a project
acli jira workitem comment visibility --role --project <PROJECT_KEY>

# Available groups
acli jira workitem comment visibility --group
```

Use these to discover valid values for `--visibility-role` and `--visibility-group` flags on create/update.

## ADF Quick-Start Template

Minimal ADF comment with a bold label and a paragraph:

```json
{
  "version": 1,
  "type": "doc",
  "content": [
    {
      "type": "paragraph",
      "content": [
        { "type": "text", "text": "Label: ", "marks": [{ "type": "strong" }] },
        { "type": "text", "text": "Description of the update." }
      ]
    }
  ]
}
```

For headings, lists, code blocks, blockquotes, and other elements, see [resources/adf-reference.md](resources/adf-reference.md).
