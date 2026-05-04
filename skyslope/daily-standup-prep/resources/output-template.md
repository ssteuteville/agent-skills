# Output Template

## Audience

Standup is read by a cross-functional team — engineers, product, QA, scrum, sometimes leadership. Write so a non-engineer can follow what shipped and why it mattered. Drop deep tech jargon (DB join strategies, internal class names, framework-specific terms). Translate engineering work into user-visible outcomes. Keep the friendly conversational tone — full sentences, not telegraphic notes.

The hyperlinks let curious readers drill in; the prose carries the meaning.

## Format

```
**Yesterday**
- [SK-1234](https://skyslope.atlassian.net/browse/SK-1234) — <plain-language summary> (<status>)
- <commit subject if not tied to a ticket> ([repo](https://github.com/skyslope/<repo>))

**Today**
- [SK-5678](https://skyslope.atlassian.net/browse/SK-5678) — <plain-language summary> (In Progress)
- <notable meeting>

**Blockers**
- [SK-1234](url) / [PR #412](url) / <thread ref> — <one-line reason in plain language>
```

## Rules

- **Always hyperlink.**
  - Jira tickets: `[SK-1234](https://skyslope.atlassian.net/browse/SK-1234)`.
  - GitHub PRs: `[<repo>#<number>](https://github.com/skyslope/<repo>/pull/<number>)` — short label, full URL.
  - Use Markdown link syntax in chat output. When sending to Slack, the Slack tool renders Markdown links natively, so the same syntax works for both surfaces.
- **Plain-language summaries.** Translate engineering work into the user-visible effect. Avoid: SQL terms, query plan details, internal type/class names, framework jargon, refactor mechanics. Prefer: what the user or downstream system can now do, or what bug stopped happening.
- **Three sections, always.** If a section is empty, render `_None_` rather than omitting the heading.
- **Headings** are bolded inline (`**Yesterday**`), not Markdown `#` — paste-friendly for Slack and Jira.
- **Tickets first, untied commits second** within Yesterday. A commit whose subject starts with a ticket key folds under that ticket and does not get its own bullet.
- **Repo annotation** with a hyperlink for any commit/PR that didn't come from the current working directory.
- **Status** in parens (Done / In Review / In Progress) only when it adds signal. Skip if obvious from context.
- **Blockers must be specific.** Each line names the artifact (linked ticket key, linked PR, Slack thread reference) and gives one short plain-language reason. No vague "waiting on review" without a target.

## Worked example

```
**Yesterday**
- [SK-1842](https://skyslope.atlassian.net/browse/SK-1842) — Connected the ingest pipeline to the new sign-in system so users get logged in correctly when data flows through. (Done)
- [SK-1855](https://skyslope.atlassian.net/browse/SK-1855) — Moved the reports tool onto the new data layout — should fix the missing-fields complaints. (In Review)
- Bumped shared config to 4.2.0 ([shared-config](https://github.com/skyslope/shared-config))

**Today**
- [SK-1860](https://skyslope.atlassian.net/browse/SK-1860) — Speeding up the report-loading query that's been timing out for big customers. (In Progress)
- 1:1 w/ Ryan @ 2pm

**Blockers**
- [SK-1855](https://skyslope.atlassian.net/browse/SK-1855) — In code review since Friday, no reviewer assigned yet.
- [billing-api #412](https://github.com/skyslope/billing-api/pull/412) — Three review threads still open; need to triage today.
```

## Empty-bucket example

```
**Yesterday**
- _None_

**Today**
- [SK-1860](https://skyslope.atlassian.net/browse/SK-1860) — Speeding up the report-loading query. (In Progress)

**Blockers**
- _None_
```
