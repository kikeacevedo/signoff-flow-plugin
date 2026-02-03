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

## Workflow Steps

### Step 0: Check/Bootstrap Governance

First, check if governance exists at `_bmad-output/governance/governance.yaml`.

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
