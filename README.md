# Signoff Flow Skill

PR-centric lead-only signoff workflow with Jira visibility for Claude Code.

## Quick Install

```bash
mkdir -p ~/.claude/skills && git clone https://github.com/kikeacevedo/signoff-flow-plugin.git ~/.claude/skills/signoff-flow
```

## Usage

Open Claude Code in any project and run:

```
/signoff-flow new
```

## What It Does

Guides initiatives through a structured approval process:

1. **PRD** → BA, Design, Dev approval
2. **UX** → BA, Design approval
3. **Architecture** → Dev approval
4. **Epics & Stories** → BA, Dev approval
5. **Implementation Readiness** → BA, Design, Dev approval

For each step:
- Creates a GitHub PR with reviewers
- Creates Jira tickets for visibility
- Tracks state in `_bmad-output/`

## Prerequisites

1. **Claude Code** installed
2. **GitHub CLI** (`gh`) installed and authenticated
3. **Atlassian MCP** configured for Jira access

## First Run

On first run in a new project, you'll configure governance:
- BA / Design / Dev lead GitHub usernames
- Jira project key
- (Optional) Jira account IDs for assignment

This creates `_bmad-output/governance/governance.yaml`.

## Commands

| Command | Description |
|---------|-------------|
| `/signoff-flow new` | Start a new initiative |
| `/signoff-flow <key>` | Continue existing initiative |
| `/signoff-flow status` | Show all initiatives status |

## File Structure

After running the workflow:

```
_bmad-output/
├── governance/
│   └── governance.yaml          # Lead configuration
└── initiatives/
    └── <key>/
        ├── state.yaml           # Current state
        ├── timeline.md          # Audit log
        └── artifacts/
            ├── PRD.md
            ├── UX.md
            ├── ARCHITECTURE.md
            ├── EPICS_AND_STORIES.md
            └── IMPLEMENTATION_READINESS.md
```

## Uninstall

```bash
rm -rf ~/.claude/skills/signoff-flow
```

## License

MIT
