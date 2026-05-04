# Output Template

## Audience

The EOD note is a **personal log** — past-self writing to future-self. Use precise technical language (DB-level, framework-level, internal class names, query plan details) where it aids future recall. The value is capturing what would otherwise be forgotten by tomorrow morning.

Hyperlinks are mandatory — the prose carries the synthesis, the links let future-you re-read the source.

## Format

```
**Shipped**
- [SK-XXXX](https://skyslope.atlassian.net/browse/SK-XXXX) — one sentence; technical specifics OK.
- [<repo> #NNN](https://github.com/skyslope/<repo>/pull/NNN) — merged. (When the PR is the unit and there's no ticket.)

**Learned**
- One-line synthesis of a decision, gotcha, or trade-off — [SK-XXXX](url) / [PR #NNN](url) for context.
- (Each bullet must cite a source link so future-me can re-read the original.)

**Open Questions**
- [SK-XXXX](url) — Blocked on <what / who>.
- [PR #NNN](url) — N unresolved review threads.
- <thread ref> — replied to <person>, no follow-up yet.
```

## Rules

- **Always hyperlink.**
  - Jira tickets: `[SK-XXXX](https://skyslope.atlassian.net/browse/SK-XXXX)`.
  - GitHub PRs: `[<repo> #NNN](https://github.com/skyslope/<repo>/pull/NNN)`.
  - Slack thread refs can be a permalink if available, otherwise `<channel> <timestamp>` without a link.
- **Use precise technical language.** The audience is the engineer themselves.
- **Each Learned bullet must cite a source.** No orphan claims. Either a hyperlink to the originating commit / PR / Jira / Slack thread, or `(self)` for a bullet that came from the "anything else?" prompt at the end of Step 3.
- **Three sections, always.** If a section is empty, render `_None_` — do not omit the heading and do not fabricate to fill.
- **Headings** are bolded inline (`**Shipped**`), not Markdown `#` — paste-friendly for Slack.
- **One-sentence bullets** for Learned. The synthesis discipline is the value. If a bullet wants to be a paragraph, split it into multiple bullets.

## Worked example

```
**Shipped**
- [SK-2064](https://skyslope.atlassian.net/browse/SK-2064) — Hoisted the type-71 TitleCompany contact onto `TitleFeedTransactionEvent` via `tblTransactionContact` JOIN; merged [agent-calculator #516](https://github.com/skyslope/agent-calculator/pull/516).
- [SK-1842](https://skyslope.atlassian.net/browse/SK-1842) — Wired ingest pipeline through the v2 auth middleware; merged [ingest-svc #88](https://github.com/skyslope/ingest-svc/pull/88).

**Learned**
- `gh search commits` is default-branch-only — fall back to `gh search prs` for any feature-branch shipping. Bit me when [agent-calculator #509](https://github.com/skyslope/agent-calculator/pull/509) didn't show up in the cross-repo scan.
- SQL Server's `OUTER APPLY` with `TOP 1` is non-deterministic across plan recompiles — added explicit `ORDER BY ct.id DESC` to fix phantom-modified-event spam ([agent-calculator #516 review](https://github.com/skyslope/agent-calculator/pull/516)).
- Slack `to:me` searches return 60k+ chars on a busy week — use the focused `<@USER>` mention search instead, or paginate. Ran into this in `eod-note` itself today (self).

**Open Questions**
- [agent-calculator #509](https://github.com/skyslope/agent-calculator/pull/509) — 4 business days in review with no reviewer assigned. Bot bumped today.
- [SK-2097](https://skyslope.atlassian.net/browse/SK-2097) — Email updates sub-task; In Progress since 5/1, no movement. Decide tomorrow whether to descope or escalate.
- Slack thread w/ Kyle re: unified integ branch (#skunk-code-review 2026-05-04 09:25) — flagged that a single integ branch loses per-feature deploy isolation when two PRs land in the same window; awaiting his take.
```

## Empty-bucket example

```
**Shipped**
- _None_

**Learned**
- Reviewed [agent-calculator #523](https://github.com/skyslope/agent-calculator/pull/523); the seller-side pest/home inspection fee defaults are state-conditional, which Kyle handled differently than I would have. Worth referencing for SK-2120's UI pass.

**Open Questions**
- _None_
```
