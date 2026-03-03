# Jira Agent Skills

Agent skills for interacting with Jira via the Atlassian CLI.

## Prerequisites

1. **Install `acli`** — follow the [official installation guide](https://developer.atlassian.com/cloud/acli/guides/install-acli/)
2. **Authenticate** — run `acli auth login` and complete the OAuth flow before using any skill

Verify your setup:

```bash
acli --version
acli jira workitem search --jql "assignee = currentUser()" --limit 1
```

If both commands succeed, you're ready to use the Jira skills.

## Skills

### [comment](comment/)

CRUD operations for Jira issue comments. Supports rich text formatting via Atlassian Document Format (ADF) with headings, bold, italic, strikethrough, inline code, links, bullet/ordered lists, code blocks, blockquotes, and horizontal rules.

Key details:
- Uses a two-step create-then-update workflow for rich formatting (the `create` command doesn't reliably render ADF marks).
- Uses process substitution `<(echo '...')` with `--body-adf` to avoid temp files.
- All agent-created comments are prefixed with bold **[Agent]**.
- Includes resource files documenting every flag for each subcommand (`create`, `list`, `update`, `delete`, `visibility`) and a full ADF reference.

### [skill-maintenance](skill-maintenance/)

Maintenance skill for keeping the above skills up to date. Run this when `acli` is upgraded or a skill starts producing unexpected errors. See [skill-maintenance/SKILL.md](skill-maintenance/SKILL.md) for the workflow.
