# GitHub Agent Skills

Agent skills for interacting with GitHub via the `gh` CLI.

## Prerequisites

1. **Install `gh`** — follow the [official installation guide](https://cli.github.com/manual/installation)
2. **Authenticate** — run `gh auth login` and complete the auth flow before using any skill

Verify your setup:

```bash
gh --version
gh repo view --json name
```

If both commands succeed, you're ready to use the GitHub skills.

## Skills

### [create-pr](create-pr/)

Create a GitHub PR for the current branch with an auto-generated description based on the diff against main. Extracts a Jira slug from the branch name if present and includes it in the PR title and description.

Key details:
- Diffs against `main` to gather context (commit log, changed files, full diff).
- Detects Jira slugs in branch names (e.g., `PROJ-1234-add-auth`) and links to the Jira issue.
- Generates a concise title and structured description, then waits for user approval before creating.

### [pr-comment-resolution](pr-comment-resolution/)

Evaluate and resolve unresolved PR review comments. Fetches all open threads via GraphQL, categorizes each comment, gets user approval, applies fixes, and resolves bot threads. Human threads are replied to but left open for the reviewer.

Key details:
- Classifies threads by author (bot vs human) — only bot threads are auto-resolved.
- Evaluates each comment against the actual code before categorizing (already fixed, incorrect, valid, low priority, pre-existing).
- Requires explicit user approval before committing, pushing, or resolving anything.
- All agent replies are prefixed with bold **[Agent]**.

### [skill-maintenance](skill-maintenance/)

Maintenance skill for keeping the above skills up to date. Run this when `gh` is upgraded or a skill starts producing unexpected errors. See [skill-maintenance/SKILL.md](skill-maintenance/SKILL.md) for the workflow.
