# Sources Mapping

How each bucket gets populated from the raw signals gathered in Step 2 of [../SKILL.md](../SKILL.md). Read this when synthesizing — it answers "where does this bullet come from?" and "what counts as evidence?"

For literal commands, see [commands.md](commands.md). This file is the *interpretation* layer.

## Shipped

**Pure status filtering — no LLM synthesis required.**

Sources (all hyperlinked):

- **Jira tickets moved to a closing status today** — JQL #1 in [commands.md](commands.md#jql-1-closed-by-me-today-shipped). Each ticket → one bullet keyed by `SK-XXXX` summary.
- **Merged PRs today** — `gh search prs --merged-at=">=<date>"`. Each PR → one bullet, **but** if the PR title starts with `SK-XXXX`, fold it under that ticket's bullet rather than listing separately.
- **Commits not tied to a ticket** — `git log` results where the subject does not start with `SK-XXXX`. Render as `<commit subject> ([<repo>](https://github.com/skyslope/<repo>))`.

Dedup rule: a single piece of work usually leaves traces in *all three* sources (ticket, PR, commits). Surface it once — under the Jira ticket if available, otherwise under the PR, otherwise the commit.

## Learned

**Qualitative LLM extraction from prose. The agent writes one-sentence bullets, not quotes.**

Three input streams, all rationale-bearing:

### Stream A: commit message bodies

`git log` with `%b` (see [commands.md#local-commits-with-bodies-learned-signal](commands.md#local-commits-with-bodies-learned-signal)).

What to look for in `%b`:
- Explicit markers: `Why:`, `Decided:`, `Note:`, `Trade-off:`, `Caveat:`.
- Causal prose: "because…", "to avoid…", "since…", "this fixes…".
- Documented surprises: "turns out…", "discovered that…", "X breaks because…".

Skip:
- Co-author trailers, sign-offs, bot-generated content.
- Bodies that are just a list of changes ("- updated foo, - added bar") with no rationale — that's covered by the PR title in Shipped.

Cite as the commit URL: `https://github.com/skyslope/<repo>/commit/<sha>`. If the commit is part of a PR, prefer the PR URL.

### Stream B: Jira comments authored by the user today

JQL #2 (which is the documented client-side-filter form, since SkySlope doesn't have ScriptRunner) in [commands.md#jql-2-tickets-i-touched-today-learned-signal](commands.md#jql-2-tickets-i-touched-today-learned-signal). The single JQL call returns the candidate ticket set; the per-ticket comment scan then runs against the already-fetched JSON — no follow-up calls.

What to look for:
- Comments where the user explained a design decision, ruled out an option, or recorded a constraint.
- Comments that are responses to questions from PMs / QA — these often crystallize the *why*.

Skip:
- One-line acknowledgements ("done", "merged", "ty").
- Comments under 200 characters (heuristic; tune as needed).

Cite as the ticket URL with the comment anchor if extractable, else the ticket URL.

### Stream C: user Slack messages > 200 chars

`from:me in:<channel> after:<date>` per channel in [commands.md#slack](commands.md#slack).

What to look for:
- Long messages explaining a decision, debugging conclusion, or rejected approach.
- Replies in threads where the user laid out a rationale.

Skip:
- Code snippets pasted without surrounding prose — they're not rationale.
- Bug reports / status updates ("X is broken, fixing").

Cite as the Slack permalink (use `chat.getPermalink` if needed, else `<channel-name> <timestamp>` as a non-link reference).

### Synthesis rules

- **One bullet per learning.** If a single source contains two distinct learnings, write two bullets, each citing the same source.
- **One sentence per bullet.** No paragraphs. If it wants to be longer, split.
- **Past-tense, first-person framing implicit.** "Discovered X." "Decided Y because Z." Avoid passive voice.
- **No quotes from sources.** The agent's job is to compress and synthesize.
- **Always cite.** No orphan claims.

## Open Questions

**Status filtering plus light synthesis for the "why" clause.**

Three input streams:

### Stream D: PRs with unresolved review threads

`gh search prs --state=open` results, then for each PR run the GraphQL `reviewThreads` query in [commands.md#open-prs-by-me-with-unresolved-review-comments-open-questions](commands.md#open-prs-by-me-with-unresolved-review-comments-open-questions). **Issue all per-PR GraphQL calls in a single parallel batch** — one per open PR.

Count `nodes[].isResolved == false`. If count > 0, render as:

```
- [<repo> #NNN](url) — N unresolved review thread(s).
```

For triage detail, defer to the existing `gh-pr-comment-resolution` skill — do not duplicate its logic.

### Stream E: Jira tickets in Blocked / Needs Input

JQL #3 in [commands.md#jql-3-currently-blocked-open-questions](commands.md#jql-3-currently-blocked-open-questions).

Render with the blocking reason if visible in the most recent comment; otherwise render as:

```
- [SK-XXXX](url) — Status: <status>; check the latest comment for context.
```

### Stream F: Jira @-mentions of the user without a reply

For tickets the user is involved in where the most recent comment contains `[~currentUser]` or the user's account ID and was authored by someone else, surface it.

Implementation: scope to tickets `updated >= "<one week ago>"` (single JQL filter to bound the candidate set — without this guard the fetch could fan out to dozens of tickets). Reuse the `comment` field already returned by JQL #2's fetch where possible; for tickets present in JQL #1 / #3 but missing comment data, **issue the supplementary `getJiraIssue` calls in a single parallel batch**, not sequentially. Then scan comments for the user's mention pattern and check if any later comment is from the user. If not, it's an Open Question.

### Stream G: Slack threads the user replied to today, no follow-up

From `slack_read_thread` results: a thread is an Open Question if the user's last reply is also the **last** reply in the thread. Per [commands.md#threads-i-replied-to-without-follow-up](commands.md#threads-i-replied-to-without-follow-up), all `slack_read_thread` fan-out calls run in a single parallel batch. Render as:

```
- <channel-name> <ts> — replied to <person>, no follow-up yet.
```

## "Anything else?" prompt

After auto-extraction completes, ask the user once at the end of Step 3:

> Anything else worth capturing? (one-sentence answer fine, or 'nope')

Treat any substantive answer as one extra bullet in Learned with `(self)` as the source marker (no hyperlink). If the user declines (any short negative or empty reply), move on without prompting again.

This catches decisions that didn't leave a trace in any of streams A–C — typically in-person conversations or DMs this skill doesn't index.

## Anti-pattern: prescriptive bullets

This is a descriptive log, not a TODO list. "Decide tomorrow whether to X" is fine in Open Questions because it describes a state ("undecided"); "you should do X" is not, because it prescribes action. The value of the log is the past tense; tomorrow-Shane will set tomorrow-Shane's priorities.
