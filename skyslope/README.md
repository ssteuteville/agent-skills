# SkySlope Agent Skills

SkySlope-specific skills wired to the SkySlope Atlassian site (`SK` Jira project), the SkySlope GitHub org, and the Skunks team Slack channels. Canonical IDs and contacts live in `~/.claude/jira.md` and `~/.claude/slack.md`.

## Prerequisites

- `gh` CLI installed and authenticated against the SkySlope GitHub org. SkySlope uses SAML SSO — you may need `gh auth refresh -h github.com -s repo` to grant org access.
- `acli` installed and authenticated, **or** the Atlassian MCP server connected.
- Slack and Google Calendar MCP servers connected (for skills that read either).
- Local references: `~/.claude/jira.md`, `~/.claude/slack.md`, `~/.claude/CLAUDE.md`.

## Skills

### [daily-standup-prep](daily-standup-prep/)

Aggregates yesterday's work + today's plan into a paste-ready Yesterday / Today / Blockers summary. Pulls Jira ticket movement on the SK project, cross-repo GitHub commits and PRs across the SkySlope org, Slack mentions and unread threads in Skunks team channels, and today's Calendar. Renders in chat for review and (after confirmation) DMs the result to the user's self-DM. Audience is the cross-functional team — output is hyperlinked and written in plain language for product / QA / scrum.

### [eod-note](eod-note/)

End-of-day personal log capturing what shipped today, what was learned (decisions, design rationale, gotchas), and what's still open. Pulls from Jira / GitHub / Slack / local git, mining commit bodies and prose for the qualitative *why* behind the day's work. Audience is the user themselves — technical detail is welcome. Renders in chat, then drafts/sends to self-DM (separate preference key from standup-prep).
