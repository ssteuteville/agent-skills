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
