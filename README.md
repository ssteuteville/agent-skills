# Agent Skills

A collection of agent skills organized by vertical. Each vertical provides skills for interacting with a specific CLI tool or service.

## Project Structure

```
agent-skills/
├── README.md                      # This file
└── <vertical>/                    # One folder per tool/service (e.g. jira, github)
    ├── README.md                  # Vertical-level prerequisites and setup
    ├── <skill-name>/              # Individual skill
    │   ├── SKILL.md               # Main skill instructions
    │   └── resources/             # Reference docs the skill can read on demand
    └── skill-maintenance/         # Maintenance skill for this vertical
        ├── SKILL.md               # General maintenance workflow
        └── resources/             # Per-skill maintenance playbooks
            └── <skill-name>.md
```

### Verticals

Each vertical is a top-level folder named after the tool or service it targets. A vertical contains:

- **A README** with prerequisites (installation, authentication) for the underlying CLI or API.
- **One or more skill folders**, each with a `SKILL.md` and optional `resources/` directory.
- **A `skill-maintenance/` skill** (see below).

### Skill Maintenance

Every vertical includes a `skill-maintenance/` skill. Its purpose is to keep the other skills in that vertical up to date as CLI versions change, APIs evolve, or undocumented quirks are discovered.

The maintenance skill provides:

- A general workflow for auditing CLI `--help` output against skill resource files.
- Per-skill playbooks in `skill-maintenance/resources/` with specific commands to run, known quirks to re-validate, and test flows to execute with the user.
- Testing conventions (dedicated test prefix, cleanup requirements, no temp files).

When a CLI is upgraded or a skill starts producing unexpected errors, run the maintenance skill to systematically verify and update the affected skill.

## Verticals Index

| Vertical | CLI | README |
|----------|-----|--------|
| Jira | `acli` (Atlassian CLI) | [jira/README.md](jira/README.md) |
