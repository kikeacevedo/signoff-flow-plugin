# Signoff Flow - Claude Code Skill

A skill for managing signoff workflows with GitHub and Jira integration.

## Quick Install

**Copy and paste this command in Terminal:**

```bash
mkdir -p ~/.claude/skills/signoff-flow && curl -sL https://raw.githubusercontent.com/kikeacevedo/signoff-flow-plugin/main/SKILL.md -o ~/.claude/skills/signoff-flow/SKILL.md && echo "✅ Installed!"
```

Then **restart Claude Desktop** and use `/signoff-flow` in the Code tab.

## What it does

- Guides initiatives through approval process (PRD → UX → Architecture → Epics → Readiness)
- Creates GitHub PRs for each artifact
- Creates Jira tickets for visibility
- Tracks progress in repo (source of truth)

## Usage

In Claude Desktop (Code tab) or Claude Code CLI:

```
/signoff-flow
```

The skill will:
1. Check if you have a project with governance configured
2. If not, help you find and clone a project from GitHub
3. Guide you through creating initiatives and getting signoffs

## Requirements

- GitHub CLI (`gh`) - will guide you to install if missing
- Atlassian MCP connection (for Jira tickets)

## Manual Installation

```bash
mkdir -p ~/.claude/skills/signoff-flow
# Download SKILL.md to that folder
```

## Uninstall

```bash
rm -rf ~/.claude/skills/signoff-flow
```
