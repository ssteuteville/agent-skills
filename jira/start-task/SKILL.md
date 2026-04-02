---
name: jira-start-task
description: Start working on a Jira task — browse the active sprint, pick a task, create a branch, assign yourself, transition the issue, and review the description. Use when the user says "start a task", "pick up a ticket", "grab a ticket", "begin work on a ticket", "assign me a task", "start PROJ-123", "work on PROJ-123", or references beginning work on a Jira issue.
---

# Start Task

Pick up a Jira task and prepare the local repo to work on it. Handles sprint browsing, branch creation, assignment, and status transitions via Atlassian MCP tools. If the MCP server is not connected, read [resources/cli-fallback.md](resources/cli-fallback.md) for CLI instructions.

## Prerequisites

Every MCP call requires a `cloudId` — use the site hostname (e.g., `skyslope.atlassian.net`). If that fails, call `getAccessibleAtlassianResources` to discover the correct cloud ID.

## Step 1: Determine the user's role

Check in this order:

1. The current conversation — did the user state their role?
2. Project-level `AGENTS.md`, `CLAUDE.md`, or docs referenced from those files
3. Memory (if available)

If no role is found, default to **developer**.

The role determines:
- **Which statuses to show** when browsing the sprint (developer → "To Do"; QA → "Ready for QA")
- **Which transition to apply** when starting work (developer → "In Progress"; QA → "Testing")

If you have to ask the user their role, suggest they define it in `AGENTS.md` or commit it to memory so future sessions don't need to ask.

## Step 2: Identify the task

### Case A — Specific task provided

The user gave a full issue key (e.g., `PROJ-123`). Skip to Step 3.

### Case B — Ambiguous project

The user mentioned a task but the project is unclear (e.g., just a number, or the branch/context doesn't reveal the project). Use `getVisibleJiraProjects` to list available projects and ask the user which one. Once resolved, go to Step 3.

### Case C — Browse the sprint

The user wants to pick a task but didn't specify one.

1. Determine the project. If unknown, ask the user. Use `getVisibleJiraProjects` to help if needed.
2. Search for available tasks using `searchJiraIssuesUsingJql`:
   ```
   project = <PROJECT> AND sprint in openSprints() AND status = "<role-appropriate status>"
   ```
   Fields: `summary`, `status`, `description`
3. If the board's statuses don't clearly map to the user's role, ask what status to filter by. Then suggest the user document their board's status meanings in `AGENTS.md` or a referenced doc (e.g., "To Do = ready for dev, Code Review = waiting for PR review") so future sessions can map statuses automatically. Alternatively, offer to commit the status mappings to memory.
4. Present tasks as a clean list — **not** a table:

```
- **PROJ-123** Title of the task
  One sentence summarizing the description.

- **PROJ-456** Another task title
  One sentence summary of what this task involves.
```

5. Let the user ask questions about any task or pick one to start. Once they pick, go to Step 3.

## Step 3: Get full issue details

Use `getJiraIssue`:

- `cloudId` — site hostname or cloud UUID
- `issueIdOrKey` — e.g., `PROJ-123`
- `fields` — `["summary", "description", "status", "assignee", "issuetype"]`
- `responseContentFormat` — `"markdown"`

Hold onto the response — you'll need the description, current status, and assignee for later steps.

## Step 4: Create a branch

### 4a. Detect the default branch

```bash
gh repo view --json defaultBranchRef --jq '.defaultBranchRef.name'
```

### 4b. Check working tree cleanliness

```bash
git status --porcelain
```

If dirty, warn the user that starting a task requires a clean working tree. Do **not** proceed with branch creation unless the user explicitly says to continue anyway.

### 4c. Fetch latest and checkout default branch

Run each of these separately, in order:

```bash
git fetch origin
```

```bash
git checkout <default-branch>
```

```bash
git pull origin <default-branch>
```

### 4d. Determine branch name

Start with the issue key as the base name (e.g., `PROJ-123`). Check for existing branches:

```bash
git branch --list 'PROJ-123*'
```

```bash
git branch -r --list 'origin/PROJ-123*'
```

- **No existing branch** → use `PROJ-123`
- **Branch already exists** → if the user is working on a subset of the task or the existing branch is from prior work, create `PROJ-123_<suffix>` where the suffix describes the scope (e.g., `PROJ-123_api_layer`). Ask the user for a suffix if the intent isn't clear from context.
- **User specified partial scope upfront** → use a suffix proactively (e.g., `PROJ-123_frontend`)

Sanitize suffixes: replace spaces with `_`, strip characters that are invalid in git branch names (e.g., `~`, `^`, `:`, `\`, `..`). Use only alphanumeric characters, hyphens, and underscores.

### 4e. Create and checkout the branch

```bash
git checkout -b <branch-name>
```

## Step 5: Assign the task

1. Get the current user's account ID:

   Call `atlassianUserInfo()` and extract the account ID from the response.

2. Check the current assignee from Step 3:
   - **Already assigned to you** → skip silently
   - **Assigned to someone else** → warn the user and ask whether to reassign
   - **Unassigned** → proceed

3. Assign using `editJiraIssue`:
   - `cloudId` — site hostname or cloud UUID
   - `issueIdOrKey` — e.g., `PROJ-123`
   - `fields` — `{ "assignee": { "accountId": "<account-id>" } }`

## Step 6: Transition the task

1. Get available transitions using `getTransitionsForJiraIssue`:
   - `cloudId` — site hostname or cloud UUID
   - `issueIdOrKey` — e.g., `PROJ-123`

2. Find the transition that matches the user's role:
   - **Developer**: look for "In Progress", "In Development", "Start Progress", or similar
   - **QA**: look for "Testing", "In QA", "Start Testing", or similar

3. If no transition is an obvious match, present the available transitions and ask the user which one to use. Suggest documenting the board's status meanings in `AGENTS.md` or a referenced doc so future sessions can map statuses automatically. Alternatively, offer to commit the mappings to memory.

4. If the task is already in the target status, skip and note this to the user.

5. Apply the transition using `transitionJiraIssue`:
   - `cloudId` — site hostname or cloud UUID
   - `issueIdOrKey` — e.g., `PROJ-123`
   - `transition` — `{ "id": "<transition-id>" }`

## Step 7: Present the task

Display the full task details from Step 3:

- Issue key and title
- Current status (post-transition)
- Assignee (should now be the current user)
- Full description in markdown

## Step 8: Gather context

Ask the user: "Do you have any additional context for this task? (design docs, Slack threads, related PRs, etc.)"

Wait for the user's response. If they provide context, acknowledge it and incorporate it into your understanding of the task.

## Step 9: Suggest plan mode

Suggest switching to plan mode to explore the codebase and design an implementation approach before writing code.

If the user asks to plan without switching to plan mode, do your best to behave as if in plan mode — explore the codebase, read relevant files, and outline an implementation strategy without making edits.
