# Agent Skills

Reusable agent skills for automating workflows with external tools and services. Each skill teaches an AI coding agent how to interact with a specific CLI or API — handling authentication quirks, rich formatting, undocumented behaviors, and other details that would otherwise require trial and error.

## Available Skills

### [GitHub](gh/)

Interact with GitHub via the `gh` CLI. Requires `gh` to be installed and authenticated.

| Skill | What it does |
|-------|--------------|
| [create-pr](gh/create-pr/) | Create a PR for the current branch with an auto-generated description based on the diff against main |
| [pr-comment-resolution](gh/pr-comment-resolution/) | Evaluate and resolve unresolved PR review comments, with bot/human thread handling |
| [review-pr](gh/review-pr/) | Multi-persona PR review — posts inline comments from 7 specialized agents, confidence-scored, adapts to large PRs |

See the [GitHub README](gh/README.md) for setup instructions and full details.

### [Jira](jira/)

Interact with Jira via the Atlassian CLI (`acli`). Requires `acli` to be installed and authenticated.

| Skill | What it does |
|-------|--------------|
| [comment](jira/comment/) | Create, read, update, and delete Jira issue comments with rich text formatting (ADF) |
| [start-task](jira/start-task/) | Browse the active sprint, pick a task, create a branch, assign yourself, and transition to in-progress |

See the [Jira README](jira/README.md) for setup instructions and full details.

### [General](general/)

Domain-agnostic skills for structured thinking and problem-solving workflows. No external tools required.

| Skill | What it does |
|-------|--------------|
| [how-might-we](general/how-might-we/) | Facilitate "How Might We?" brainstorming sessions — reframe a software engineering challenge into HMW questions, brainstorm solutions, and produce a prioritized summary |

## How It Works

Each top-level folder targets a tool or service. Inside, you'll find:

- **Skill folders** with a `SKILL.md` (instructions the agent follows) and a `resources/` directory (reference docs the agent reads on demand).
- **A `skill-maintenance/` skill** that helps keep the other skills current when CLIs are upgraded or APIs change. It walks the agent through auditing `--help` output, re-testing workflows, and updating docs.

```
<vertical>/
├── README.md                  # Prerequisites and skill summaries
├── <skill-name>/
│   ├── SKILL.md               # Agent instructions
│   └── resources/             # Command references, format guides
└── skill-maintenance/
    ├── SKILL.md               # Maintenance workflow
    └── resources/             # Per-skill maintenance playbooks
```

## Usage

Copy or symlink a skill folder into your Cursor skills directory:

```bash
# Personal (available across all projects)
cp -r jira/comment ~/.cursor/skills/jira-comment

# Project-level (shared via repo)
cp -r jira/comment .cursor/skills/jira-comment
```

The agent will automatically discover the skill based on its description and apply it when relevant.
