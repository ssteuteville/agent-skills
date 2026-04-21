---
name: gh-review-pr
description: Multi-persona code review for a GitHub pull request. Posts inline review comments from 7 specialized personas. Adapts to large PRs by splitting files into groups. Use when the user asks to review a PR, review a pull request, or run a code review.
---

# Review PR

Multi-persona code review that posts inline comments directly on a GitHub pull request. Seven specialized agents review the diff in parallel, findings are confidence-scored, and surviving comments are posted as PR reviews — one review per persona.

For large PRs, the diff is split into related file groups and each group is reviewed by a single agent covering all persona perspectives.

No local checkout required — everything runs through `gh api`.

## Prerequisites

`gh` must be installed and authenticated. See [../README.md](../README.md).

Built and tested with `gh version 2.86.0`.

## Phase 1: Gather Context

### Step 1.1 — PR metadata

Fetch PR details and repo identity:

```bash
gh pr view <NUMBER_OR_URL> --json number,title,state,isDraft,headRefName,baseRefName,author,url,additions,deletions
```

```bash
gh repo view --json owner,name
```

Abort if the PR is **closed** or **merged**. If it is a **draft**, warn the user and ask whether to proceed.

Extract and hold: PR number, owner, repo, base branch, total additions + deletions.

### Step 1.2 — Parallel context gathering

Launch 2 **haiku** agents in parallel:

#### Agent A — CLAUDE.md Discovery

Fetch project convention files from the base branch via the GitHub contents API:

```bash
gh api repos/{owner}/{repo}/contents/CLAUDE.md?ref={base} --jq '.content'
```

```bash
gh api repos/{owner}/{repo}/contents/.claude/CLAUDE.md?ref={base} --jq '.content'
```

```bash
gh api repos/{owner}/{repo}/contents/AGENTS.md?ref={base} --jq '.content'
```

The content is base64-encoded — decode it. These requests may 404 (file not found) — that is fine. Return whichever exist.

#### Agent B — PR files and diff

Fetch the list of changed files with their patches:

```bash
gh api repos/{owner}/{repo}/pulls/{number}/files --paginate
```

This returns an array of file objects, each with: `filename`, `status`, `additions`, `deletions`, `patch` (the per-file diff). This is the primary data source for all review agents.

Also fetch the head commit SHA for posting review comments:

```bash
gh pr view <NUMBER> --json headRefOid --jq '.headRefOid'
```

### Step 1.3 — Assess size and route

Compute totals from the files response:
- **Total lines changed** = sum of all file `additions` + `deletions`
- **File count** = number of files

| Condition | Mode |
|-----------|------|
| ≤ 500 lines changed AND ≤ 15 files | **Normal** |
| > 500 lines OR > 15 files | **Large** |

### Step 1.4 — Build PR summary

Produce a brief summary to pass to review agents:
- **What changed**: 1-3 sentence overview
- **Files touched**: grouped by area (backend, frontend, tests, config, etc.)
- **Technologies detected**: languages, frameworks, databases — specifically flag MongoDB presence

MongoDB detection heuristics:
- Paths containing `model`, `schema`, `migration`, `mongo`, `mongoose`, `collection`, `repository`
- Imports of `mongodb`, `mongoose`, `@typegoose` in patches
- Aggregation operators (`$lookup`, `$match`, `$group`, `$unwind`)
- Query methods (`find(`, `findOne(`, `aggregate(`, `createIndex`)

## Phase 2: Review

### Model Selection

| Mode | Complexity | Model |
|------|-----------|-------|
| Normal | ≤ 100 lines, ≤ 3 files | sonnet |
| Normal | everything else | opus |
| Large | always | opus |

If unsure, default to **opus**.

### 2A — Normal Mode (persona-per-agent)

Launch up to 7 agents **in parallel**. Each agent receives:
- The per-file patches from Step 1.2
- The CLAUDE.md / AGENTS.md content (if any)
- The PR summary from Step 1.4
- Their persona instructions from [resources/personas.md](resources/personas.md)

| # | Agent | Condition |
|---|-------|-----------|
| 1 | Simplifier | Always |
| 2 | Staff Engineer | Always |
| 3 | Senior Engineer | Always |
| 4 | MongoDB Expert | Only if MongoDB code detected |
| 5 | Security Engineer | Always |
| 6 | Curious Junior Engineer | Always |
| 7 | DX Engineer | Only if config/script/build files changed |

Each agent returns findings in this structure:

```json
[
  {
    "path": "src/foo.ts",
    "line": 42,
    "side": "RIGHT",
    "body": "suggestion or question text",
    "reasoning": "why this matters",
    "severity": "warning",
    "persona": "Simplifier"
  }
]
```

The `line` must be a line number that appears in the diff hunk for that file. `side` is `RIGHT` (head branch — the new code) in almost all cases. Use `LEFT` only when commenting on a deleted line.

### 2B — Large Mode (file-group-per-agent)

#### Step 2B.1 — Group files

Group changed files into clusters of **3-8 related files**. Grouping heuristics (in priority order):

1. **Test + source**: pair test files with the source files they test (e.g., `foo.ts` + `foo.test.ts`, `foo.spec.ts`)
2. **Same directory**: files in the same directory likely belong together
3. **Same feature area**: group by common path prefix (e.g., `src/auth/` files together)
4. **Config together**: group config/build files (`.env`, `package.json`, `tsconfig.json`, CI files)
5. **Catch-all**: remaining files that don't fit elsewhere

#### Step 2B.2 — Launch agents

One agent per file group. Each agent reviews from **all persona perspectives** — Simplifier, Staff Engineer, Senior Engineer, Security, Junior Engineer, DX, and MongoDB Expert (if relevant to that group's files).

Each agent receives:
- Only the patches for files in its group
- CLAUDE.md content
- PR summary (full, for overall context)
- All persona instructions from [resources/personas.md](resources/personas.md)

Same output structure as Normal mode. The `persona` field indicates which perspective each finding comes from.

### False Positive Awareness

Instruct every agent (both modes) to avoid:
- Pre-existing issues not changed in this PR
- Issues a linter, typechecker, or compiler would catch
- General code quality concerns unless explicitly required in CLAUDE.md
- Issues silenced by lint-ignore comments
- Intentional functionality changes related to the PR's purpose
- Style preferences not backed by CLAUDE.md

## Phase 3: Confidence Scoring

After all review agents complete, score every finding using **haiku** agents.

- If findings are **20 or fewer**: one haiku agent per finding, in parallel
- If findings **exceed 20**: batch into groups of 10 per haiku agent

Each scoring agent receives:
- The finding(s) to score
- The relevant file patch(es)
- The CLAUDE.md content (if any)

### Scoring Rubric

| Score | Meaning |
|-------|---------|
| 0 | False positive. Does not hold up to scrutiny, or is a pre-existing issue. |
| 25 | Might be real, but unverified. If stylistic, not called out in CLAUDE.md. |
| 50 | Verified real, but a nitpick or unlikely to matter in practice. |
| 75 | Verified, likely hit in practice. Important — directly impacts functionality or mentioned in CLAUDE.md. |
| 100 | Confirmed. Will happen frequently. Evidence directly supports this. |

For CLAUDE.md-flagged issues: double-check the CLAUDE.md actually calls it out.

### Scoring Questions (Junior Engineer)

Different criteria:
- Would a new team member genuinely struggle with this code?
- Is the answer already documented elsewhere?
- Is the question substantive, not trivially answered by surrounding code?

Same 0-100 scale.

### Filtering

- Discard findings with confidence **below 80**
- De-duplicate: if multiple agents/personas flagged the same issue (same file, overlapping concern), keep the highest-scored version and note which other personas also flagged it
- Carry the `severity` field through filtering for the Phase 5 summary breakdown

## Phase 4: Post Reviews

Post findings as GitHub PR reviews using the REST API. Format each finding's comment body before posting.

### Comment body format

Standard findings:
```
**[{Persona}]** {suggestion}

**Reasoning**: {reasoning}
```

Junior Engineer questions:
```
**[Junior Engineer]** {question}

**Context**: {reasoning}
```

### Normal mode — one review per persona

For each persona that has surviving findings, submit one review:

```bash
gh api repos/{owner}/{repo}/pulls/{number}/reviews \
  --method POST \
  --input - <<'EOF'
{
  "commit_id": "{head_sha}",
  "event": "COMMENT",
  "body": "**[Agent — Simplifier]** Reviewed the PR from a simplicity perspective. 2 comments.",
  "comments": [
    {
      "path": "src/foo.ts",
      "line": 42,
      "side": "RIGHT",
      "body": "**[Simplifier]** suggestion text\n\n**Reasoning**: why"
    }
  ]
}
EOF
```

If a persona has zero findings after filtering, do not post a review for that persona.

### Large mode — one review per file group

For each file group that has surviving findings, submit one review using the same `gh api` command structure as Normal mode. The review body lists which personas contributed:

```bash
gh api repos/{owner}/{repo}/pulls/{number}/reviews \
  --method POST \
  --input - <<'EOF'
{
  "commit_id": "{head_sha}",
  "event": "COMMENT",
  "body": "**[Agent — Group Review]** Files: `src/auth/login.ts`, `src/auth/session.ts`. Perspectives: Simplifier, Senior Engineer, Security. 3 comments.",
  "comments": [ ... ]
}
EOF
```

### Posting constraints

- Always use `event: 'COMMENT'` — never `APPROVE` or `REQUEST_CHANGES`
- The `line` in each comment must correspond to a line in the diff. If a finding references a line not in the diff, drop it.
- The `commit_id` must be the head SHA from Step 1.2
- If the reviews API returns an error for a specific comment (e.g., invalid line number), drop that comment and continue with the rest

## Phase 5: Summary

After posting, report to the user in conversation:

```
## Review Posted: <PR title> (#<number>)

**Reviews posted**: <count> (<persona names or group names>)
**Total comments**: <count>
**Breakdown**: <N> critical, <N> warning, <N> info, <N> questions
**Filtered out**: <N> findings below confidence threshold
**Skipped personas**: <list and reasons, or "None">

<PR URL>
```

## Ground Rules

- **All comments are prefixed** with `**[{Persona}]**` so PR participants know these are agent-generated.
- **Comment only** — use `event: 'COMMENT'`. Never approve or request changes.
- **Confidence threshold is firm.** Do not post findings below 80. The user can ask to see filtered-out findings.
- **Skip irrelevant personas.** MongoDB Expert if no Mongo code. DX Engineer if no config/script/build changes.
- **No false urgency.** A naming suggestion is `info`, not `warning`.
- **De-duplicate before posting.** The PR should not have the same issue from multiple personas.
- **No local git operations.** All data comes from `gh api` or `gh pr view`.
- **Line numbers must be in the diff.** Drop any finding that references a line outside the diff hunks.
