# Signoff Flow Plugin

PR-centric lead-only signoff workflow with Jira visibility for Claude Code.

## Overview

This plugin helps teams manage initiative approvals through a structured workflow:

```
PRD → UX → Architecture → Epics & Stories → Implementation Readiness
```

Each step creates:
- A **GitHub PR** for lead approvals
- **Jira tickets** for visibility (one per required group)
- **State tracking** in the repo

## Installation

### Option 1: Install from Marketplace (Recommended)

```bash
/plugin install signoff-flow
```

### Option 2: Install from Directory

```bash
claude --plugin-dir ./signoff-flow
```

### Option 3: Copy as Standalone Skill

Copy `skills/signoff-flow/SKILL.md` to `~/.claude/skills/signoff-flow/SKILL.md`

## Prerequisites

1. **Atlassian MCP** configured for Jira access
2. **GitHub CLI** (`gh`) installed and authenticated
3. A git repository for your project

## Quick Start

1. Run the workflow:
   ```
   /signoff-flow:signoff-flow new
   ```

2. Follow the prompts to:
   - Configure governance (first run only)
   - Create an initiative
   - Generate PRs and Jira tickets

3. When PRs are approved and merged, run again to advance:
   ```
   /signoff-flow:signoff-flow INIT-001
   ```

## How It Works

### Governance

On first run, you'll configure leads for each group:
- **BA** (Business Analyst)
- **Design**
- **Dev** (Development)

This is stored in `_bmad-output/governance/governance.yaml`.

### Artifacts

Each initiative progresses through 5 artifacts:

| Artifact | Required Groups |
|----------|-----------------|
| PRD | BA, Design, Dev |
| UX | BA, Design |
| Architecture | Dev |
| Epics & Stories | BA, Dev |
| Readiness | BA, Design, Dev |

### Signoff Process

1. Agent creates artifact stub + PR + Jira tickets
2. Leads review and approve the PR
3. PR is merged
4. Run workflow again to advance to next artifact

## File Structure

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

## Commands

| Command | Description |
|---------|-------------|
| `/signoff-flow:signoff-flow new` | Start a new initiative |
| `/signoff-flow:signoff-flow <key>` | Continue existing initiative |
| `/signoff-flow:signoff-flow status` | Show all initiatives status |

## Configuration

### Governance File

Edit `_bmad-output/governance/governance.yaml` to:
- Add/remove leads
- Change Jira project
- Modify signoff rules

### Templates

Templates are in `skills/signoff-flow/templates/`:
- `governance.yaml` - Governance structure
- `state.yaml` - Initiative state structure

## License

MIT
