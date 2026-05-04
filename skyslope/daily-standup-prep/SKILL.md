---
name: skyslope-daily-standup-prep
description: Aggregate yesterday's work + today's plan into a paste-ready standup summary. Pulls Jira ticket movement on the SK project, cross-repo GitHub commits and PRs across the SkySlope org, Slack mentions, and today's Calendar — outputs Yesterday / Today / Blockers. Use when the user says "standup prep", "what should I say in standup", or "morning summary".
---

# Daily Standup Prep

Aggregate signal from Jira, GitHub, Slack, and Google Calendar into a Yesterday / Today / Blockers summary. Render in chat first, then — only after explicit confirmation — DM the user their own copy in Slack.

Each step below references the relevant section of [resources/commands.md](resources/commands.md) on demand — do not pre-load the full commands file.

## Prerequisites

- **MCP-first.** Atlassian, Slack, and Google Calendar MCPs are the primary path. Attempt the calls in Step 2 directly; if a call returns an auth error, run the MCP's `authenticate` tool and surface the prompt to the user. Never silently skip a source.
- For Jira specifically, fall back to `acli` per [resources/jira-fallback.md](resources/jira-fallback.md) only if the Atlassian MCP is unavailable.
- `gh` CLI authenticated against the SkySlope GitHub org. If `gh search commits --owner=skyslope` returns zero across known-active repos, surface `gh auth status` output to the user and suggest `gh auth refresh -h github.com -s repo` (SAML/SSO). Do **not** run the refresh automatically.
- **No silent posting.** Even self-DM requires explicit user confirmation. Never send without an explicit go-ahead.
- **`[agent]` prefix** on any Slack message that gets posted, per `~/.claude/CLAUDE.md`.
- **Empty buckets render `_None_`**, not omitted. Do not fabricate items to fill a bucket.
- Fathom is not a source for this skill.

### Resolved IDs

These are the canonical SkySlope IDs used throughout the skill — inlined here so each run does not require re-reading `~/.claude/jira.md` or `~/.claude/slack.md`. If an ID becomes invalid, those two files are the source of truth.

- Jira cloud ID: `a1ce6525-ad2b-4088-a36d-393ece6a7835`, project `SK`
- User Slack display name: `SHANE`
- User self-DM: `UABEDKSDR`
- Skunks engineering DM: `C0A9BMWC6JU`
- `#skunk-code-review`: `C0ATH0WH9NW`

## Step 1: Resolve the time window

Determine "yesterday":

- **Tue–Fri:** 24h ending now.
- **Mon:** window starts at previous Friday 09:00 local. No prompt — this is the user's default.
- **After a holiday or PTO gap > 3 days:** extend through the gap. **Confirm the proposed window with the user before proceeding** — see [resources/time-windows.md](resources/time-windows.md) for detection guidance.

Convert the window start to an absolute date or timestamp. The same value gets formatted differently per tool:

| Tool | Format | Example |
|---|---|---|
| `gh search ... --committer-date` / `--updated` | `>=YYYY-MM-DD` | `>=2026-05-03` |
| JQL `status changed DURING (...)` | `"YYYY-MM-DD HH:mm"` (both bounds, double-quoted) | `"2026-05-03 09:00"` |
| `git log --since` | `YYYY-MM-DDTHH:MM:SS` (ISO 8601, local TZ) | `2026-05-03T09:00:00` |
| Slack search `after:` | `YYYY-MM-DD` | `2026-05-03` |

## Step 2: Gather in parallel

Issue every call in this step in a single message with parallel tool invocations. Across the four sources there are typically 8+ independent calls (2 Jira JQL, 3 GitHub, 2+ Slack, 1 Calendar) — all fully parallelizable. Use the literal commands in [resources/commands.md](resources/commands.md); do not improvise flags.

### Jira (MCP)

Two `searchJiraIssuesUsingJql` calls — see [resources/commands.md#jira](resources/commands.md#jira) for the exact JQL. If the Atlassian MCP is unavailable, switch to `acli` per [resources/jira-fallback.md](resources/jira-fallback.md).

### GitHub (cross-repo, all branches)

Three independent calls — issue them in the same parallel batch as Jira/Slack/Calendar. See [resources/commands.md#github](resources/commands.md#github) for exact flags.

1. **`gh search commits`** — cross-repo, default branch only. Catches anything merged to `main`/`master` across the SkySlope org.
2. **`gh search prs`** — catches feature-branch work tied to a PR (any branch, any state).
3. **Local `git log --all`** — catches commits on local branches in the current repo that haven't been pushed/PR'd yet.

**Caveat to surface to the user on first run:** `gh search commits` searches each repo's default branch only. Feature-branch commits are picked up via `gh search prs` (any branch with an open or recent PR) and the local `git log --all` (current repo only). Truly orphaned commits on remote feature branches without a PR will be missed — acceptable tradeoff.

### Slack (MCP)

- `slack_search_public_and_private` — two parallel queries (`to:@SHANE after:<date>` and `@SHANE after:<date>`) to catch both DMs and mentions.
- `slack_read_channel` on `C0A9BMWC6JU` and `C0ATH0WH9NW` — issue both in parallel.
- For thread context, **collect all qualifying thread IDs from the channel reads first, then batch the `slack_read_thread` calls in parallel** — do not chain one-at-a-time follow-ups.

Exact queries in [resources/commands.md#slack](resources/commands.md#slack).

### Calendar (MCP)

`mcp__claude_ai_Google_Calendar` list events for today; filter per [resources/commands.md#calendar](resources/commands.md#calendar).

## Step 3: Synthesize

Map the raw signals into the three buckets:

- **Yesterday:** Jira status moves (Done / In Review) + GitHub commits and PRs grouped by ticket prefix (`SK-1234`). When a commit's message starts with a ticket key that matches a Jira status move, fold the commit under that ticket — do not double-list.
- **Today:** In-Progress Jira tickets + today's notable meetings.
- **Blockers:** Tickets stuck in code review for more than 24h, unanswered Slack threads where the user was pinged, and open PRs with unresolved review comments. For PR comment detail, defer to the existing `gh-pr-comment-resolution` skill — do not duplicate its logic here.

**Audience reminder.** Standup is read by the full cross-functional team (product, QA, scrum, sometimes leadership) — translate engineering work into user-visible outcomes and avoid deep technical jargon. Hyperlink every ticket and PR so curious readers can drill in without you having to explain mechanics. See [resources/output-template.md](resources/output-template.md) for the audience guidance and the link patterns.

## Step 4: Render

Use the canonical format in [resources/output-template.md](resources/output-template.md). Show the rendered standup in chat. Do not send anywhere yet.

## Step 5: Send to self-DM (after confirmation)

After the user reviews the rendered output and confirms, send to their Slack self-DM (`UABEDKSDR`):

- **Draft vs. send:** check memory for a `standup-send-preference` key (values: `direct` or `draft`). If set, honor it silently. If not set, ask the user once which they prefer and offer to save the response under that key — per `~/.claude/CLAUDE.md`.
- **Always prefix** the message body with `[agent]`.
- **Alternative destinations** (copy-paste only, post to a channel) require an additional explicit confirmation, separate from the self-DM confirmation.

Never send without an explicit go-ahead, even for the default self-DM path.
