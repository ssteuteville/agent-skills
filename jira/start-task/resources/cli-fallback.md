# CLI Fallback — Start Task

Use these `acli` commands when the Atlassian MCP server is not connected.

## Search Sprint Tasks

```bash
acli jira workitem search --jql "project = PROJ AND sprint in openSprints() AND status = 'To Do'" --fields "key,summary,status,description" --json
```

Adjust the `status` value to match the user's role (e.g., `'Ready for QA'` for QA).

## List Projects

```bash
acli jira project list --json
```

## Get Issue Details

```bash
acli jira workitem view PROJ-123 --fields "*all" --json
```

## Assign

```bash
acli jira workitem assign --key "PROJ-123" --assignee "@me"
```

Use `@me` to self-assign — this replaces the `atlassianUserInfo()` MCP call entirely; no separate account-ID lookup is needed with the CLI path. To assign to another user, use their email address or account ID.

If the task is already assigned to someone else and the user wants to reassign, run the same command — it will overwrite the current assignee.

## Transition

```bash
acli jira workitem transition --key "PROJ-123" --status "In Progress" --yes
```

Replace `"In Progress"` with the target status name for the user's role. The `--yes` flag skips the confirmation prompt.

If you're unsure which statuses are available, omit `--status` and `--yes` to see the interactive list, or check the board configuration.

## Troubleshooting

- **Authentication errors**: Run `acli auth login` to re-authenticate.
- **"Sprint not found"**: The JQL function `sprint in openSprints()` requires the project to have an active sprint. Verify the board has a sprint in progress.
- **"Transition not available"**: The target status may not be reachable from the current status. Check the board's workflow configuration or ask the user which transition to use.
- **Version compatibility**: These commands were documented against `acli` v1.x. Run `acli --version` to check, and use the [skill-maintenance](../../skill-maintenance/) workflow if commands behave unexpectedly.
