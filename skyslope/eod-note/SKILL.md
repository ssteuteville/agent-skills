---
name: skyslope-eod-note
description: End-of-day personal log capturing what shipped today, what was learned (decisions, design rationale, gotchas), and what's still open. Audience is the user themselves — technical detail is welcome. Pulls from Jira, GitHub, Slack, and local git, then drafts/sends to the user's self-DM. Use when the user says "eod note", "end of day summary", "wrap-up note", "log my day", or "/eod".
---

# EOD Note

A private end-of-day log — captures decisions and rationale that would otherwise vanish from working memory by tomorrow morning. The morning-standup counterpart is at [../daily-standup-prep/SKILL.md](../daily-standup-prep/SKILL.md).

Each step below references the relevant section of [resources/commands.md](resources/commands.md) and [resources/sources.md](resources/sources.md) on demand — do not pre-load the full files.

This skill assumes an interactive session: Step 3's "anything else?" prompt and Step 5's draft-vs-send prompt both expect a user reply. If invoked non-interactively, both prompts default to "skip" / "draft" respectively without blocking.

## Prerequisites

- **MCP-first.** Atlassian and Slack MCPs are the primary path. Attempt the calls in Step 2 directly; if a call returns an auth error, run the MCP's `authenticate` tool and surface the prompt to the user. Never silently skip a source.
- For Jira specifically, fall back to `acli` if the Atlassian MCP is unavailable (no dedicated fallback file yet; reuse the patterns in [../../jira/comment/resources/cli-fallback.md](../../jira/comment/resources/cli-fallback.md) for the `--jql` flag shape).
- `gh` CLI authenticated against the SkySlope GitHub org. If a `gh search` returns zero across known-active repos, surface the auth-recovery commands in [resources/commands.md#auth-recovery](resources/commands.md#auth-recovery) to the user — do not run the refresh automatically.
- **No silent posting.** Even self-DM requires explicit user confirmation. Never send without an explicit go-ahead.
- **`[agent]` prefix** on any Slack message that gets posted, per `~/.claude/CLAUDE.md`.
- **Empty buckets render `_None_`**, not omitted. Do not fabricate items to fill a bucket — especially Learned, which is qualitative and tempting to pad.
- **Audience guidance** (technical detail welcome, no cross-team softening) lives in [resources/output-template.md](resources/output-template.md). Read it before rendering.

### Resolved IDs

These are the canonical SkySlope IDs used throughout the skill — inlined here so each run does not require re-reading `~/.claude/jira.md` or `~/.claude/slack.md`. If an ID becomes invalid, those two files are the source of truth.

- Jira cloud ID: `a1ce6525-ad2b-4088-a36d-393ece6a7835`, project `SK`
- User Slack display name: `SHANE`
- User self-DM: `UABEDKSDR`
- Skunks engineering DM: `C0A9BMWC6JU`
- `#skunk-code-review`: `C0ATH0WH9NW`
- `#skunk-den`: `C05SKMFG69Y` — read-only for this skill; per `~/.claude/slack.md`, never post here without explicit confirmation.

## Step 1: Resolve the time window

Default: today, midnight local → now.

If the user runs `/eod` on a Friday and asks for "this week", extend the start to Monday 09:00 of the same week. Otherwise stay on today.

Per-tool date formats (same as standup-prep):

| Tool | Format | Example |
|---|---|---|
| `gh search ... --committer-date` / `--updated` / `--merged-at` | `>=YYYY-MM-DD` | `>=2026-05-04` |
| JQL `status changed DURING (...)` | `"YYYY-MM-DD HH:mm"` (both bounds, double-quoted) | `"2026-05-04 00:00"` |
| `git log --since` | `YYYY-MM-DDTHH:MM:SS` (ISO 8601, local TZ) | `2026-05-04T00:00:00` |
| Slack search `after:` | `YYYY-MM-DD` | `2026-05-04` |

## Step 2: Gather in parallel

Issue every call below in a single message with parallel tool invocations. Typical run is 6–8 independent calls. Use the literal commands in [resources/commands.md](resources/commands.md); do not improvise flags.

### Jira (MCP)

Three `searchJiraIssuesUsingJql` calls:

1. Tickets the user moved to a closing status today (Done / In Review).
2. Tickets the user commented on today (the comment text is the qualitative signal).
3. Tickets currently assigned to the user in `Blocked` or `Needs Input`.

See [resources/commands.md#jira](resources/commands.md#jira) for the exact JQL.

### GitHub

Three independent calls — same parallel batch as Jira/Slack.

1. **Merged PRs by the user today** — for the Shipped bucket.
2. **Open PRs by the user with unresolved review comments** — for the Open Questions bucket.
3. **`git log` with full bodies** (`%h %s%n%b`) — local commits authored today across all branches. The body is the *primary* signal for Learned (rationale prose lives there).

See [resources/commands.md#github](resources/commands.md#github) for exact commands.

### Slack (MCP)

- `slack_search_public_and_private` for `from:me after:<date> in:<channel>` for each Skunks channel (`C0A9BMWC6JU`, `C0ATH0WH9NW`, `C05SKMFG69Y`) — captures rationale prose the user typed. Pass `count: 20` per query and watch for the 60k-char overflow that bit standup-prep; see [resources/commands.md#slack](resources/commands.md#slack).
- `slack_read_channel` on each of those channels to scan threads the user replied to today.
- For thread context, **collect all qualifying thread IDs first, then batch the `slack_read_thread` calls in parallel** — do not chain follow-ups.

## Step 3: Synthesize into three buckets

See [resources/sources.md](resources/sources.md) for the full extraction logic per bucket. Summary:

- **Shipped** — pure status filtering. Jira tickets that moved to Done today + merged PRs + commits not tied to a ticket. Hyperlinked.
- **Learned** — qualitative LLM extraction from prose:
  - Commit message bodies (`%b`) — look for `Why:`, `Decided:`, `Note:`, or any prose explaining trade-offs.
  - Jira comments authored by the user today.
  - User Slack messages > 200 chars (likely contain rationale).
  - Each `Learned` bullet is **one sentence the agent writes**, not a quote. **Always cite the source link** — no orphan claims.
- **Open Questions** — PRs with unresolved review threads, Jira tickets in Blocked / Needs Input, Jira tickets where someone @-mentioned the user without a reply, Slack threads the user replied to today with no follow-up activity from anyone else.

**After auto-extraction, ask the user once:** *"Anything else worth capturing? (one-sentence answer fine, or 'nope')"*

If they provide an answer, fold it into Learned with `(self)` as the source marker instead of a hyperlink. If they say "nope" or skip, move on. **Do not re-prompt** — the goal is a 30-second EOD ritual.

## Step 4: Render

Use the canonical format in [resources/output-template.md](resources/output-template.md). Show the rendered note in chat. Do not send anywhere yet.

## Step 5: Send to self-DM (after confirmation)

After the user reviews the rendered note and confirms, send to their Slack self-DM (`UABEDKSDR`):

- **Draft vs. send:** check memory for an `eod-note-send-preference` key (values: `direct` or `draft`). If set, honor it silently. If not set, ask the user once which they prefer and offer to save the response under that key.
- Note: this is **separate** from `standup-send-preference` — the user may have different defaults for EOD vs standup. Do not read or write the standup key here.
- **Always prefix** the message body with `[agent]`.
- **Alternative destinations** (copy-paste only, post elsewhere) require an additional explicit confirmation, separate from the self-DM confirmation.

Never send without an explicit go-ahead, even for the default self-DM path.
