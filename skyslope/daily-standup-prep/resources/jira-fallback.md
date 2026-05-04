# CLI Fallback for Daily Standup Prep — Jira

Use these instructions when the Atlassian MCP server is not available. Requires `acli` installed and authenticated. See [../../../jira/README.md](../../../jira/README.md). Built and tested with `acli version 1.3.13-stable`.

These commands cover only the two JQL queries used by [../SKILL.md](../SKILL.md) Step 2a. For comment / transition / branch operations, see other skills under `jira/`.

## Search by JQL

Reference: `acli jira workitem search --help`

```bash
acli jira workitem search --jql "<JQL-STRING>" --fields "summary,status,issuetype,updated" --json
```

Flags:
- `--jql` — the literal JQL string. Quote it with double quotes; escape any inner double quotes with `\"`.
- `--fields` — comma-separated field names. Match what the MCP path requests so the synthesis logic doesn't branch on data source.
- `--json` — required for downstream parsing.

## Query 1: status moves yesterday

```bash
acli jira workitem search --jql "assignee = currentUser() AND project = SK AND status changed DURING (\"<START>\", \"<END>\") BY currentUser()" --fields "summary,status,issuetype,updated" --json
```

Substitute `<START>` and `<END>` with `"YYYY-MM-DD HH:mm"` strings (already inside the outer double-quoted JQL, so escape them as shown).

## Query 2: today's WIP

```bash
acli jira workitem search --jql "assignee = currentUser() AND project = SK AND status = \"In Progress\"" --fields "summary,status,issuetype,updated" --json
```

## Post-fetch transformation

The JSON response shape is:

```json
{
  "issues": [
    {
      "key": "SK-1234",
      "fields": {
        "summary": "...",
        "status": { "name": "Done" },
        "issuetype": { "name": "Task" },
        "updated": "2026-05-03T17:42:01.000-0700"
      }
    }
  ]
}
```

Map each issue to the same shape the MCP path produces, so Step 3 synthesis logic in `../SKILL.md` doesn't branch on data source:

- `key` → ticket ID for grouping commits
- `fields.summary` → bullet text
- `fields.status.name` → bucket selection (Done / In Review / In Progress / etc.)

## Troubleshooting

1. Check the installed CLI version: `acli --version`
2. Compare against the version at the top of this file.
3. If they differ, the CLI may have introduced breaking changes to `--jql` parsing or output shape. Run the skill at [../../../jira/skill-maintenance/SKILL.md](../../../jira/skill-maintenance/SKILL.md) to re-validate and update this skill against the current CLI.
