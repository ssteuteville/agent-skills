# Commands Reference

Literal commands for the gather steps in [../SKILL.md](../SKILL.md). Built and tested with `gh version 2.55`, `git version 2.40`, and `acli version 1.3.13-stable`.

`<ISO-date>` throughout is the window start computed in Step 1 of [../SKILL.md](../SKILL.md), formatted per the table in that step.

## GitHub

`gh search` runs against the GitHub commit/PR search index, which has a few-minute lag. Acceptable for standup prep.

### Cross-repo commits (default branch)

```bash
gh search commits --author=@me --owner=skyslope --committer-date=">=<ISO-date>" --json repository,sha,commit,url --limit 20
```

Flags:
- `--author=@me` — the authenticated user.
- `--owner=skyslope` — repeat for additional SkySlope orgs (e.g. `--owner=skyslope --owner=skyslope-engineering`). Confirm the org list with the user on first run; persist to memory if useful.
- `--committer-date=">=<ISO-date>"` — quotes are required; the `>=` prefix is part of the value.
- `--json repository,sha,commit,url` — `gh` only supports whole-object selection; the agent should ignore unused subfields of `commit` (only the message subject is needed for grouping).
- `--limit 20` — typical engineer ships <20 commits/day across all repos. If the result count equals the limit, warn the user that results may be clipped and re-run with a higher limit.

### PRs (catches feature-branch work)

```bash
gh search prs --author=@me --owner=skyslope --updated=">=<ISO-date>" --json repository,number,title,state,url,updatedAt --limit 15
```

Flags:
- Omit `--state` to include both open and closed (which covers merged) — `gh search prs` only accepts `open|closed`, not `all`.
- `--updated=">=<ISO-date>"` — covers any PR that saw activity in the window, even if opened earlier.
- `--limit 15` — same clipping rule as above; warn and re-run if the result count equals the limit.

### Local-only branches (current repo)

```bash
git log --author="$(git config user.email)" --since="<ISO-date>T00:00:00" --all --pretty=format:"%h %s" --no-merges
```

Flags:
- `--all` — refs across all local and remote-tracking branches, not just the checked-out branch.
- `--no-merges` — drop merge commits.
- `--pretty=format:"%h %s"` — short SHA + subject; enough for grouping under ticket prefixes.

### Grouping commits by repo

After `gh search commits` returns its JSON, group in-process by `.repository.nameWithOwner`. Do not pipe to `jq` via Bash — per `~/.claude/CLAUDE.md`, run the `gh` command and the transform separately, or do the grouping inline from the parsed JSON.

### Auth recovery

If `gh search` returns zero across known-active orgs, surface these to the user. Do **not** run the refresh automatically.

```bash
gh auth status
```

```bash
gh auth refresh -h github.com -s repo
```

SkySlope uses SAML SSO; the refresh is required after the org enables or rotates SSO enforcement.

## Jira

### MCP (preferred)

Tool: `searchJiraIssuesUsingJql`

Parameters (both calls):
- `cloudId` — see SKILL.md "Resolved IDs"
- `fields` — `["summary", "status", "issuetype", "updated"]`
- `responseContentFormat` — `"markdown"`

JQL #1 — status moves yesterday:

```
assignee = currentUser() AND project = SK AND status changed DURING ("<start>", "<end>") BY currentUser()
```

`<start>` and `<end>` are the window bounds in `"YYYY-MM-DD HH:mm"` format (the `DURING` clause requires double-quoted timestamps with this exact shape).

JQL #2 — today's WIP:

```
assignee = currentUser() AND project = SK AND status = "In Progress"
```

### CLI fallback

See [jira-fallback.md](jira-fallback.md) for the `acli` equivalents of both queries.

## Slack

### Mentions and DMs to the user

Tool: `slack_search_public_and_private`. Use the display name from SKILL.md "Resolved IDs" (`SHANE`). Issue both queries in parallel.

```
to:@SHANE after:<ISO-date>
```

```
@SHANE after:<ISO-date>
```

### Unread threads in channels

Tool: `slack_read_channel`. Issue both channel reads in parallel; channel IDs are in SKILL.md "Resolved IDs".

- Skunks engineering DM
- `#skunk-code-review`

Filter the returned messages to those that mention the user or are a direct reply to one of their messages. **Then read all qualifying threads in a single parallel batch of `slack_read_thread` calls** — do not chain follow-ups one at a time.

## Calendar

Tool: `mcp__claude_ai_Google_Calendar` (use whichever list-events action the MCP exposes — typically `list_events` or equivalent).

Window: today, 00:00 → 23:59 local.

Filter rules:
- Include all-hands, sprint ceremonies (standup itself excluded), 1:1s with manager or direct reports.
- Exclude blocks labeled "Focus", "Lunch", or all-day OOO entries.
- Surface anything tagged `[Blocker]`, `[Decision]`, or similar in the title.
