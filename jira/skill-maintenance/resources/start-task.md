# Maintenance Playbook — jira-start-task

## Test Target

Ask the user for:
- A **Jira project key** with an active sprint
- A **test issue key** in that sprint (or create a throwaway task)

All test actions use the prefix `**[Agent testing]**` when commenting (if applicable) to distinguish from real agent activity.

## Test Flow

### 1. Sprint Browsing (MCP)

Search for tasks in the active sprint:

```
searchJiraIssuesUsingJql with jql: "project = <PROJECT> AND sprint in openSprints() AND status = 'To Do'"
```

Verify:
- Results include the test issue
- Fields `summary`, `status`, `description` are populated
- JQL function `openSprints()` is supported

### 2. Sprint Browsing (CLI fallback)

```bash
acli jira workitem search --jql "project = <PROJECT> AND sprint in openSprints() AND status = 'To Do'" --fields "key,summary,status,description" --json
```

Verify:
- Same results as MCP path
- JSON output is parseable
- `--fields` flag filters correctly

### 3. Issue Details (MCP)

```
getJiraIssue with issueIdOrKey: "<TEST-KEY>", fields: ["summary", "description", "status", "assignee", "issuetype"]
```

Verify:
- All requested fields are present
- `responseContentFormat: "markdown"` renders description as markdown

### 4. Assignment (MCP)

```
atlassianUserInfo() → get account ID
editJiraIssue with fields: { "assignee": { "accountId": "<id>" } }
```

Verify:
- `atlassianUserInfo` returns an account ID
- Assignment updates in Jira UI
- Re-running with same user is a no-op (no error)

### 5. Assignment (CLI fallback)

```bash
acli jira workitem assign --key "<TEST-KEY>" --assignee "@me"
```

Verify:
- `@me` token resolves to the authenticated user
- Assignment updates in Jira UI

### 6. Transitions (MCP)

```
getTransitionsForJiraIssue with issueIdOrKey: "<TEST-KEY>"
```

Pick a transition (e.g., "In Progress") and apply:

```
transitionJiraIssue with transition: { "id": "<transition-id>" }
```

Verify:
- Available transitions list is accurate
- Transition updates the issue status in Jira UI

### 7. Transitions (CLI fallback)

```bash
acli jira workitem transition --key "<TEST-KEY>" --status "In Progress" --yes
```

Verify:
- Status name matching works
- `--yes` flag skips confirmation

## Cleanup

1. Restore the test issue to its original status
2. Restore the test issue's original assignee (or unassign)
3. Delete any test branches created locally: `git branch -d <test-branch>`

## Known Quirks

- `sprint in openSprints()` requires the board to have an active sprint — future-only sprints won't match
- `@me` in acli resolves to the OAuth-authenticated user, not necessarily the Jira user if accounts differ
- Some Jira workflows restrict transitions based on assignee or required fields — `transitionJiraIssue` may need additional `fields` if the workflow enforces them
