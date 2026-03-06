# PR Comment Resolution Skill Maintenance Playbook

Maintenance guide for the `gh-pr-comment-resolution` skill located at `../../pr-comment-resolution/`.

## Skill Files

| File | Purpose |
|------|---------|
| `../../pr-comment-resolution/SKILL.md` | Main skill instructions and workflows |

## Testing Conventions

- Use a real PR with existing review comments for testing — do not create synthetic reviews (the GitHub API does not support creating review comments outside of a real review).
- Prefix all test replies with `**[Agent testing]**` (not `[Agent]`) so they are distinguishable from real agent replies.
- Always clean up test replies when done.
- Iterate with the user — ask them to verify thread state in the GitHub UI after each test action.

## Step 1: Version Check

```bash
gh --version
```

Compare against the version in `../../pr-comment-resolution/SKILL.md` (look for the "Built and tested with" line). If they differ, expect potential behavioral changes.

## Step 2: Audit CLI Help

Run each of these and compare against the skill's SKILL.md:

```bash
gh pr view --help
gh api --help
```

For each command:
1. Read the skill file at `../../pr-comment-resolution/SKILL.md`.
2. Run the `--help` command.
3. Compare flags and syntax against what the skill uses.
4. Pay special attention to `--json` fields on `gh pr view` and the `gh api graphql -f query=` syntax.

## Step 3: Verify GraphQL Schema

The skill relies on the GitHub GraphQL API for fetching review threads and resolving them. Verify the schema still works.

### 3a. Test Review Threads Query

Ask the user for a PR number with review comments, then run:

```bash
gh api graphql -f query='{
  repository(owner: "OWNER", name: "REPO") {
    pullRequest(number: PR_NUMBER) {
      reviewThreads(first: 5) {
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

Verify:
- The query succeeds without errors.
- `reviewThreads.nodes` contains thread objects with `id`, `isResolved`, `path`, `line`.
- `comments.nodes` contains comment objects with `databaseId`, `author.login`, `body`.

If any fields are missing or renamed, update the skill's SKILL.md.

### 3b. Test Resolve Mutation (on a bot thread only)

If a resolvable bot thread is available, test the mutation:

```bash
gh api graphql -f query='mutation { resolveReviewThread(input: {threadId: "THREAD_ID"}) { thread { isResolved } } }'
```

Then unresolve it to restore the original state:

```bash
gh api graphql -f query='mutation { unresolveReviewThread(input: {threadId: "THREAD_ID"}) { thread { isResolved } } }'
```

Ask the user to verify the thread toggled correctly in the GitHub UI.

## Step 4: Test Reply Posting

### 4a. Post a Test Reply

Use the `databaseId` of a comment from Step 3a:

```bash
gh api repos/OWNER/REPO/pulls/PR_NUMBER/comments/COMMENT_ID/replies -f body="**[Agent testing]** Maintenance check — this reply will be deleted."
```

Ask the user to verify the reply appears in the thread.

### 4b. Delete the Test Reply

Get the reply's comment ID from the API response, then delete it:

```bash
gh api repos/OWNER/REPO/pulls/comments/REPLY_ID -X DELETE
```

Ask the user to confirm the reply was removed.

## Step 5: Validate Bot Detection

Review the bot-detection logic in the skill. The current known bot authors are:
- `copilot-pull-request-reviewer`
- `github-actions`
- `coderabbitai`
- Any login containing `[bot]`

Check if new common bot reviewers have emerged (e.g., new AI code review tools). If so, add them to the list in the skill's SKILL.md.

## Step 6: Update Skill Files

Update `../../pr-comment-resolution/SKILL.md`:
- Update the CLI version line.
- Update GraphQL queries if the schema changed.
- Update bot-detection list if new bots were identified.
- Update any workflow steps affected by CLI or API changes.

## Step 7: Cleanup Checklist

1. Verify no test replies remain on the PR — search for comments containing `[Agent testing]`.
2. Verify any threads that were resolved during testing have been unresolved back to their original state.
3. Update the CLI version in `../../pr-comment-resolution/SKILL.md` to the current installed version.
