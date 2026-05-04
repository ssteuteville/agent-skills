# Commands Reference

Literal commands for the gather steps in [../SKILL.md](../SKILL.md). Built and tested with `gh version 2.55`, `git version 2.40`, and `acli version 1.3.13-stable`.

`<ISO-date>` throughout is the window start computed in Step 1 of [../SKILL.md](../SKILL.md), formatted per the table in that step.

## Jira

Tool: `searchJiraIssuesUsingJql`

Common parameters:
- `cloudId` — see SKILL.md "Resolved IDs"
- `fields` — `["summary", "status", "issuetype", "comment", "updated"]` (`comment` is required for query #2's client-side filter)
- `responseContentFormat` — `"markdown"`

### JQL #1: closed by me today (Shipped)

```
assignee = currentUser() AND project = SK AND status changed BY currentUser() DURING ("<start>", "<end>") AND status changed TO ("Done", "In Review", "Code Review")
```

`<start>` and `<end>` use `"YYYY-MM-DD HH:mm"` format. The Skunks board uses `Code Review` as a closing-style status; drop or adjust based on the board's actual statuses.

### JQL #2: tickets I touched today (Learned signal)

SkySlope's Atlassian does not have ScriptRunner installed, so `issueFunction in commented(...)` is unavailable. Use this JQL to scope, then filter comments client-side:

```
project = SK AND updated >= "<start>" AND (assignee = currentUser() OR reporter = currentUser() OR watcher = currentUser())
```

After fetching, scan each ticket's `comment` array for entries where `author.accountId == <currentUserId>` and `created >= <ISO-date>`. Get `<currentUserId>` from `atlassianUserInfo()` once at the start of the run.

**Coverage gap:** misses tickets the user only commented on without being assignee/reporter/watcher (e.g. drive-by review comments on a teammate's ticket). Acceptable v1 trade-off — those rationale fragments still surface via Slack stream C if the user discussed them there.

### JQL #3: currently blocked (Open Questions)

```
assignee = currentUser() AND project = SK AND status in ("Blocked", "Needs Input", "Waiting")
```

Adjust the status list to match the board. The intent is "tickets I own that are stuck on someone else."

### CLI fallback

If the Atlassian MCP is unavailable, mirror the `acli` `--jql` shape from [../../../jira/comment/resources/cli-fallback.md](../../../jira/comment/resources/cli-fallback.md). Wrap each JQL string in double quotes and escape inner double quotes:

```bash
acli jira workitem search --jql "<JQL-STRING>" --fields "summary,status,issuetype,comment,updated" --json
```

## GitHub

### Merged PRs by me today (Shipped)

```bash
gh search prs --author=@me --owner=skyslope --merged-at=">=<ISO-date>" --json repository,number,title,url,mergedAt --limit 5
```

Flags:
- `--merged-at=">=<ISO-date>"` — restricts to PRs merged in the window. Drops draft / open PRs entirely (those are Open Questions, not Shipped).
- Omit `--state` — `gh search prs` only accepts `open|closed`, not `all`. `--merged-at` already filters to closed-and-merged.
- `--limit 5` — typical engineer merges <5 PRs/day. If results equal the cap, warn and re-run with `--limit 10`.

### Open PRs by me with unresolved review comments (Open Questions)

```bash
gh search prs --author=@me --owner=skyslope --state=open --review=changes_requested --json repository,number,title,url,updatedAt --limit 10
```

Plus a parallel call for all my open PRs to count unresolved threads via GraphQL:

```bash
gh search prs --author=@me --owner=skyslope --state=open --json repository,number,title,url,updatedAt --limit 10
```

Then for each result, count unresolved review threads. The REST `gh api repos/{owner}/{repo}/pulls/{number}/reviews` does not expose thread-resolved state; use GraphQL:

```bash
gh api graphql -f query='query($owner:String!,$repo:String!,$number:Int!){repository(owner:$owner,name:$repo){pullRequest(number:$number){reviewThreads(first:50){nodes{isResolved,comments(first:1){nodes{author{login}}}}}}}}' -F owner=<owner> -F repo=<repo> -F number=<number>
```

Important: use **`-F`** (typed) for `number=<number>` because the query declares `$number:Int!`. The lowercase `-f` would coerce to string and fail GraphQL type-checking. Do not silently change `-F` to `-f` when editing.

**Issue all per-PR GraphQL calls in a single parallel batch** — one per open PR. Do not chain.

For PR comment triage detail, defer to the existing `gh-pr-comment-resolution` skill. This skill only counts and links — it does not resolve.

### Local commits with bodies (Learned signal)

```bash
git log --author="$(git config user.email)" --since="<ISO-date>T00:00:00" --all --max-count=30 --pretty=format:"%h %s%n%b%n---" --no-merges
```

Flags:
- `%b` — full commit body (the rationale prose lives here).
- `%n---` — separator to keep multi-paragraph bodies parseable.
- `--all` — refs across all local and remote-tracking branches.
- `--no-merges` — drop merge commits.
- `--max-count=30` — token cap. If output exceeds ~8k chars, process the first 30 commits and note the truncation in the rendered output.

When mining for Learned signal, look in `%b` for: `Why:`, `Decided:`, `Note:`, `Trade-off`, `Caveat`, or any sentence with an explanatory clause ("because…", "to avoid…", "since…").

### Auth recovery

If `gh search` returns zero across known-active orgs:

```bash
gh auth status
```

```bash
gh auth refresh -h github.com -s repo
```

Surface both to the user; do not run the refresh automatically. SkySlope uses SAML SSO and the refresh is required after the org enables or rotates SSO enforcement.

## Slack

### My messages with rationale prose

Tool: `slack_search_public_and_private`. Issue all three queries in parallel; pass `count: 20` per call.

```
from:me in:<#C0A9BMWC6JU> after:<ISO-date>
```

```
from:me in:<#C0ATH0WH9NW> after:<ISO-date>
```

```
from:me in:<#C05SKMFG69Y> after:<ISO-date>
```

After fetching, filter client-side to messages > 200 characters — those are the ones likely to contain reasoning, not just `:thumbsup:` / `lgtm`.

**Overflow guard:** if a single query's raw payload exceeds ~15k characters, truncate to the first 20 results and note the truncation in the rendered output. Do **not** use `to:me` on a multi-day window — standup-prep's run today returned 68k characters from a single `to:me` query and overflowed the response budget. The `from:me in:<channel>` form is bounded; the broad `to:me` form is not.

### Threads I replied to without follow-up

Tool: `slack_read_channel`. Issue all three channel reads in parallel; channel IDs in SKILL.md "Resolved IDs".

For each message in the channel that:
1. Has `parent_user_id` matching the current user, **or**
2. Is a reply (`thread_ts` set) where one of the replies is from the current user,

collect the `thread_ts`. Then **issue all `slack_read_thread` calls in a single parallel batch** — do not chain follow-ups one at a time. For each fetched thread, flag as Open Question if the user's last reply is also the last reply in the thread.
