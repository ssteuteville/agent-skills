---
name: jira-skill-maintenance
description: Guide agents through validating and updating Jira-related skills after CLI updates, API changes, or drift. Use when the user wants to do skill maintenance, verify a Jira skill still works, or update skills after an acli upgrade.
---

# Jira Skill Maintenance

Validate and update Jira agent skills against the current `acli` CLI. Each skill has a dedicated maintenance playbook in [resources/](resources/).

## Getting Started

1. Determine which skill to maintain (e.g. "comment", "workitem").
2. Ask the user for a **Jira issue key** to use as a test target.
3. Read the playbook at `resources/<skill-name>.md` for skill-specific steps.

## General Maintenance Workflow

### 1. Version Check

```bash
acli --version
```

Compare the installed version against the version recorded in the target skill's SKILL.md. If they differ, flag this to the user — behavioral changes are likely.

### 2. Audit CLI Help

Run `--help` on every subcommand the skill uses. Compare the output against the skill's resource files. Look for:
- New flags or subcommands
- Removed or renamed flags
- Changed defaults or descriptions

### 3. Research Changes

If discrepancies are found, search the web for `acli` release notes or changelog entries between the recorded version and the installed version.

### 4. Update Skill Files

- Update resource files to match current `--help` output.
- Update SKILL.md if workflows, quirks, or recommendations have changed.
- Update the recorded CLI version in the skill's SKILL.md.

### 5. Test

Run through the full test flow defined in the skill's playbook. Iterate with the user to confirm things work as expected.

### 6. Clean Up

Delete all test artifacts and verify cleanup. Refer to the playbook for skill-specific cleanup steps.

## Available Playbooks

| Skill | Playbook |
|-------|----------|
| jira-comment | [resources/comment.md](resources/comment.md) |
