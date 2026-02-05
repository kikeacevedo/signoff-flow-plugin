---
name: signoff-flow
description: PR-centric lead-only signoff workflow with Jira visibility. Use when managing initiative approvals, creating signoff PRs, or tracking artifact progression through PRD → UX → Architecture → Epics → Readiness.
argument-hint: "[initiative-key] or 'new' or 'status'"
---

# Signoff Flow Workflow

Guide initiatives through a structured approval process using GitHub PRs as gates and Jira tickets for visibility.

## Overview

This workflow manages the progression of initiatives through planning artifacts:
1. **PRD** (requires: BA, Design, Dev leads)
2. **UX** (requires: BA, Design leads)
3. **Architecture** (requires: Dev lead)
4. **Epics & Stories** (requires: BA, Dev leads)
5. **Implementation Readiness** (requires: BA, Design, Dev leads)

**Key principles:**
- Repo is source of truth (state files tracked in git)
- PR-centric signoff (approval = PR approval + merge)
- Lead-only signoff (only configured leads count)
- Jira is visibility only (tickets mirror signoffs, not canonical)

## IMPORTANT: Pre-flight Check

**Before doing anything else, perform these checks in order:**

### 1. Check if project has governance

Look for `_bmad-output/governance/governance.yaml` in the current directory.

### 2. If NO governance found, help user find a project

When no governance exists, DON'T immediately ask to set it up. Instead:

1. **Check if gh CLI is installed**: Run `which gh`
2. **Check if user is authenticated**: Run `gh auth status`

**If gh is installed and authenticated:**

Show the user their available options:

```
## No signoff workflow found in this directory

Let me help you find a project to work on.

### Your GitHub Organizations
[Run: gh api user/orgs -q '.[].login' and list results]

### Your Recent Repositories  
[Run: gh repo list --limit 5 --json name,owner and list results]

### Options:
1. **Clone a project**: Tell me which repo you want to clone (e.g., "HALO/my-project")
2. **Set up signoff here**: I'll configure governance in the current directory
3. **List more repos**: Tell me an organization name to see its repositories

Which would you like to do?
```

**If gh is NOT installed:**
```
## No signoff workflow found

The gh CLI is not installed. To browse and clone projects from GitHub, please install it:

macOS: brew install gh
Windows: winget install --id GitHub.cli

After installing, run: gh auth login

Alternatively, I can set up the signoff workflow in the current directory.
Would you like to set up governance here?
```

**If gh is installed but NOT authenticated:**
```
## No signoff workflow found

You're not logged into GitHub. To browse and clone projects, please authenticate:

Run: gh auth login

Follow the prompts to log in via browser.

Alternatively, I can set up the signoff workflow in the current directory.
Would you like to set up governance here?
```

### 3. Clone project if requested

When user wants to clone a project:

```bash
# Clone to ~/signoff-projects/<repo-name>
mkdir -p ~/signoff-projects
gh repo clone <owner>/<repo> ~/signoff-projects/<repo>
cd ~/signoff-projects/<repo>
```

Then check if governance exists in the cloned repo.

## Workflow Steps (after project is selected)

### Step 0: Check/Bootstrap Governance

If governance exists at `_bmad-output/governance/governance.yaml`, show current status.

If missing, ask the user for:
- BA lead GitHub username(s)
- Design lead GitHub username(s)  
- Dev lead GitHub username(s)
- Jira project key
- (Optional) GitHub team slugs per group
- (Optional) Jira account IDs for assignment

Create the governance file with this structure:

```yaml
version: 1
people:
  <person_key>:
    github_username: "<github_username>"
    jira:
      account_id: "<optional>"
      display_name: "<optional>"

groups:
  ba:
    leads:
      github_users: ["<lead1>"]
      jira_account_ids: ["<optional>"]
    github:
      team_slug: "<optional>"
  design:
    leads:
      github_users: ["<lead1>"]
      jira_account_ids: ["<optional>"]
    github:
      team_slug: "<optional>"
  dev:
    leads:
      github_users: ["<lead1>"]
      jira_account_ids: ["<optional>"]
    github:
      team_slug: "<optional>"

jira:
  project_key: "<PROJECT_KEY>"
  issue_types:
    signoff_request: "Task"

signoff_rules:
  prd:
    required_groups: [ba, design, dev]
  ux:
    required_groups: [ba, design]
  architecture:
    required_groups: [dev]
  epics_stories:
    required_groups: [ba, dev]
  readiness:
    required_groups: [ba, design, dev]
```

### Step 1: Initialize or Load Initiative

Ask for initiative key (prefer Jira key like `ABC-123`, or use `INIT-001` style).

If new initiative, create:
- `_bmad-output/initiatives/<key>/state.yaml`
- `_bmad-output/initiatives/<key>/timeline.md`
- `_bmad-output/initiatives/<key>/artifacts/` directory

State file tracks current step, PR info, and Jira ticket references.

### Step 2: Advance to Next Artifact

For the current step (prd → ux → architecture → epics_stories → readiness):

1. **Create stub artifact** at `_bmad-output/initiatives/<key>/artifacts/<ARTIFACT>.md`
2. **Create git branch**: `bmad/<key>/<artifact>`
3. **Commit and push** the changes
4. **Create GitHub PR** with title `[BMAD][<key>] <Artifact>`
5. **Request reviewers** from required group leads
6. **Create Jira tickets** (one per required group) with:
   - Summary: `[BMAD][<key>][<artifact>] Signoff required — <GROUP>`
   - Labels: `bmad`, `initiative-<key>`, `artifact-<artifact>`, `group-<group>`
   - Description linking to PR
7. **Update state.yaml** with PR URL and Jira ticket keys
8. **Append to timeline.md** with timestamp

### Step 3: Check Status / Continue

When user runs `/signoff-flow <key>` or `/signoff-flow status`:
- Read the initiative's `state.yaml`
- Report current step and status
- If PR is merged, advance to next step
- If PR is open, show pending approvals

## MCP Tools Required

This workflow uses:
- **Atlassian MCP** for Jira ticket creation (`createJiraIssue`, `editJiraIssue`, `searchJiraIssuesUsingJql`)
- **GitHub CLI** (`gh`) for PR creation and reviewer requests

## File Locations

```
_bmad-output/
├── governance/
│   └── governance.yaml          # Lead/group configuration
└── initiatives/
    └── <initiative-key>/
        ├── state.yaml           # Current state tracking
        ├── timeline.md          # Append-only audit log
        └── artifacts/
            ├── PRD.md
            ├── UX.md
            ├── ARCHITECTURE.md
            ├── EPICS_AND_STORIES.md
            └── IMPLEMENTATION_READINESS.md
```

## Example Usage

### When no project is set up:
```
User: /signoff-flow status
Agent: ## No signoff workflow found in this directory

       Let me help you find a project to work on.

       ### Your GitHub Organizations
       - HALO
       - another-org

       ### Your Recent Repositories
       - kikeacevedo/signoff-flow-mcp-v2
       - kikeacevedo/my-app

       ### Options:
       1. Clone a project (e.g., "HALO/my-project")
       2. Set up signoff in current directory
       3. List repos from a specific organization

       Which would you like?

User: Clone HALO/feature-payments
Agent: ✅ Cloned to ~/signoff-projects/feature-payments
       ✅ Found existing governance configuration
       
       Current status: No active initiatives
       
       Would you like to create a new initiative?
```

### When project has governance:
```
User: /signoff-flow new
Agent: What's the initiative key? (Prefer Jira key like ABC-123)
User: FEAT-100
Agent: Title for this initiative?
User: New checkout flow
Agent: ✅ Created initiative FEAT-100
       ✅ Created PRD artifact
       ✅ Created PR #5 with reviewers
       ✅ Created Jira tickets: STD-15 (BA), STD-16 (Design), STD-17 (Dev)
       
       Next: Wait for PR approvals, then run /signoff-flow FEAT-100 to advance.
```

## Artifact Stub Template

Mock artifacts contain:

```markdown
# <ARTIFACT_NAME> (Mock)

**Initiative:** `<INITIATIVE_KEY>`  
**Current step:** `<STEP_NAME>`  
**Generated at:** `<TIMESTAMP_ISO>`

---

This is a **stub artifact** for the signoff workflow.
Signoff happens via PR approval — repo/PR is source of truth.
```

## Jira Ticket Template

```
BMAD signoff requested (lead-only).

Initiative: <INITIATIVE_KEY>
Artifact: <ARTIFACT>
Group: <GROUP>

PR: <PR_URL>
Artifact path: <ARTIFACT_PATH>

Action: Approve the PR to sign off. Repo/PR is source of truth.
```
