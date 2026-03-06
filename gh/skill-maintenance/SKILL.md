---
name: gh-skill-maintenance
description: Guide agents through validating and updating GitHub-related skills after CLI updates, API changes, or drift. Use when the user wants to do skill maintenance, verify a GitHub skill still works, or update skills after a gh CLI upgrade.
---

# GitHub Skill Maintenance

Validate and update GitHub agent skills against the current `gh` CLI. Each skill has a dedicated maintenance playbook in [resources/](resources/).

## Getting Started

1. Determine which skill to maintain (e.g. "create-pr", "pr-comment-resolution").
2. Ask the user for a **GitHub repo** to use as a test target (or use the current repo).
3. Read the playbook at `resources/<skill-name>.md` for skill-specific steps.

## General Maintenance Workflow

### 1. Version Check

```bash
gh --version
```

Compare the installed version against the version recorded in the target skill's SKILL.md (look for the "Built and tested with" line). If they differ, flag this to the user — behavioral changes are likely.

### 2. Audit CLI Help

Run `--help` on every subcommand the skill uses. Compare the output against the skill's SKILL.md and any resource files. Look for:
- New flags or subcommands
- Removed or renamed flags
- Changed defaults or descriptions

### 3. Research Changes

If discrepancies are found, search the web for `gh` CLI release notes or changelog entries between the recorded version and the installed version.

### 4. Update Skill Files

- Update SKILL.md if workflows, flags, or recommendations have changed.
- Update any resource files to match current `--help` output.
- Update the recorded CLI version in the skill's SKILL.md.

### 5. Test

Run through the full test flow defined in the skill's playbook. Iterate with the user to confirm things work as expected.

### 6. Clean Up

Delete all test artifacts and verify cleanup. Refer to the playbook for skill-specific cleanup steps.

## Available Playbooks

| Skill | Playbook |
|-------|----------|
| gh-create-pr | [resources/create-pr.md](resources/create-pr.md) |
| gh-pr-comment-resolution | [resources/pr-comment-resolution.md](resources/pr-comment-resolution.md) |
