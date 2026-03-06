---
name: gh-pr-comment-resolution
description: Triage and resolve open PR review threads — read each comment, verify it against the code, get user approval on what to fix vs dismiss, apply fixes, post replies, and auto-resolve bot threads. Human threads get replies but stay open for the reviewer. Use when the user asks to resolve PR comments, handle review feedback, address PR threads, or clean up pull request reviews.
---

# PR Comment Resolution

Work through every unresolved review thread on a pull request: read the code, decide whether each comment is actionable, get user sign-off, make fixes, and close out bot threads. Human-authored threads are replied to but **left open** — only the reviewer themselves should mark those resolved.

## Prerequisites

`gh` must be installed and authenticated. See [../README.md](../README.md).

Built and tested with `gh version 2.86.0`.

## Workflow

### 1. Find the PR

```bash
gh pr view --json number,headRefName,baseRefName
```

Pull out the PR number, head branch, and base branch. If owner/repo are needed later, grab them with `gh repo view --json owner,name`.

### 2. Pull open review threads

Query the GitHub GraphQL API for all review threads and their comments:

```bash
gh api graphql -f query='{
  repository(owner: "OWNER", name: "REPO") {
    pullRequest(number: PR_NUMBER) {
      reviewThreads(first: 50) {
        nodes {
          id
          isResolved
          path
          line
          comments(first: 5) {
            nodes {
              databaseId
              author { login }
              body
            }
          }
        }
      }
    }
  }
}'
```

Drop any threads that are already resolved. If nothing remains, tell the user there are no open comments and stop.

For each thread, determine whether the author is a **bot** or a **human**:
- Bots: `copilot-pull-request-reviewer`, `github-actions`, `coderabbitai`, or any login containing `[bot]`
- Everyone else is human

Keep track of this — it controls whether the thread gets auto-resolved at the end.

### 3. Assess each comment

For every open thread:

1. Read the file at the referenced path and line.
2. Look at later commits on the branch — the problem may already be addressed.
3. Cross-check the claim against the real code. Bot reviewers (Copilot, Codex, Cursor, etc.) regularly get things wrong about files they can't see — always verify.

Assign one of these categories:

| Category | When to use | Action |
|----------|-------------|--------|
| **Already fixed** | A subsequent commit addresses it | Resolve |
| **Incorrect** | The reviewer's claim doesn't hold up | Resolve |
| **Actionable** | Legitimate issue in production code | Fix, then resolve |
| **Low priority** | Real but in scripts, docs, or non-critical paths | Resolve |
| **Pre-existing** | Not introduced by this PR | Resolve |

### 4. Get user input

Show a numbered list — one entry per thread:

**#N — `path/to/file.ts:LINE`** (author)
Category: **<category>** | Recommended: <action>
> Brief description of the issue

Ask the user which ones to fix and which to dismiss. Follow their decisions exactly.

### 5. Apply fixes

For each comment the user wants fixed, open the file and make the change. If there's nothing to fix (everything was dismissed), jump to step 6.

### 6. Checkpoint — wait for user approval

**Do not commit, push, or post any replies until the user says go.**

Show them a summary of what will happen, split by author type:

**Bot threads** (will get a reply AND be resolved):
- For each: file, one-line summary, planned reply

**Human threads** (will get a reply but stay open):
- For each: file, one-line summary, planned reply, note that it stays open

**Code changes**:
- Each modified file with a one-line description

Ask the user to review the diffs in their editor and confirm. They can:
- Ask for adjustments to code changes
- Edit reply text before it's posted
- Approve and proceed

### 7. Commit, push, and post replies

Once approved:

**Push code changes** (if any):

```bash
git add <files>
git commit -m "<conventional commit message>"
git push
```

**Reply to every thread** — bot and human alike. Every reply must start with `**[Agent]**` to make it clear the response came from an AI.

Use the `databaseId` of the first comment in each thread (from the GraphQL response in step 2):

```bash
gh api repos/OWNER/REPO/pulls/PR_NUMBER/comments/COMMENT_ID/replies -f body="**[Agent]** <reply>"
```

Example replies:

- **Already fixed**: `**[Agent]** This was addressed in <sha> — <what the commit did>.`
- **Incorrect**: `**[Agent]** <Why the reviewer's claim doesn't match the actual code>.`
- **Pre-existing**: `**[Agent]** This predates the PR — not introduced by these changes.`
- **Actionable — fixed**: `**[Agent]** Fixed in <sha>.`
- **Low priority**: `**[Agent]** Acknowledged — low priority for this PR (scripts/docs/non-critical).`

**Resolve bot threads:**

```bash
gh api graphql -f query='mutation { resolveReviewThread(input: {threadId: "THREAD_ID"}) { thread { isResolved } } }'
```

These are independent and can run in parallel. Never resolve human threads.

### 8. Wrap up

Report back with:

- How many bot threads were resolved and a brief reason for each
- How many human threads were addressed, what was replied, and confirmation they were left open
- Which files were changed and why

## Ground Rules

- Always read the referenced code before categorizing a comment.
- Don't take bot reviewers at face value — verify every claim.
- Scripts, test fixtures, and dev tooling are low priority — generally not worth fixing in the PR.
- Code that existed before the PR and wasn't touched is out of scope.
- Never skip user approval — always confirm before committing, pushing, or resolving.
- Bot threads can be auto-resolved. Human threads cannot — they stay open for the reviewer.
