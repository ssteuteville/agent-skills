---
name: jira-comment
description: Create, read, update, and delete Jira issue comments. Prefers Atlassian MCP tools when available, with acli CLI as fallback (see resources/cli-fallback.md).
---

# Jira Comment CRUD

Manage Jira issue comments via Atlassian MCP tools. If the MCP server is not connected, read [resources/cli-fallback.md](resources/cli-fallback.md) for CLI instructions.

## Prerequisites

Every MCP call requires a `cloudId` — use the site hostname (e.g., `skyslope.atlassian.net`). If that fails, call `getAccessibleAtlassianResources` to discover the correct cloud ID.

## Comment Prefix

Every comment **must** start with `**[Agent]**` as the first line to signal it was created by an AI agent.

## Create a Comment

Use `addCommentToJiraIssue` with `contentFormat: "markdown"`. Rich text (bold, links, lists, code blocks) works natively via standard markdown.

Parameters:
- `cloudId` — site hostname or cloud UUID
- `issueIdOrKey` — e.g., `PROJ-123`
- `commentBody` — markdown string starting with `**[Agent]**`
- `contentFormat` — `"markdown"`
- `commentVisibility` *(optional)* — `{"type": "role", "value": "Administrators"}` or `{"type": "group", "value": "jira-developers"}`

Example `commentBody`:

```markdown
**[Agent]**

Completed the migration. Key changes:
- Updated schema to v3
- Backfilled `created_at` column
- [PR link](https://github.com/org/repo/pull/42)
```

## Read / List Comments

Use `getJiraIssue` to retrieve comments embedded in the issue response:

- `cloudId` — site hostname or cloud UUID
- `issueIdOrKey` — e.g., `PROJ-123`
- `fields` — `["comment"]`
- `responseContentFormat` — `"markdown"`

## Troubleshooting

If `cloudId` is rejected, call `getAccessibleAtlassianResources` to list available Atlassian sites and their cloud IDs.
