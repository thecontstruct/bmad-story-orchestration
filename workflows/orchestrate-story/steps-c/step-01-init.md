---
name: 'step-01-init'
description: 'Story identification, branch setup, continuation detection, and state-based routing'

# File References
configFile: '../config.yaml'
continueStepFile: './step-02-continue.md'
routingTable: '../data/routing-table.md'
workflowStateStructure: '../data/workflow-state-structure.md'
---

# Step 1: Initialization

## STEP GOAL

Identify the target story from sprint-status.yaml, detect existing workflow state, ensure proper branch setup, initialize workflow state file if needed, and route to the appropriate step (continuation or fresh start).

## MANDATORY EXECUTION RULES (READ FIRST)

### Universal Rules

- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When routing to next step, ensure entire file is read first
- 🎯 This step is automatic — no user input required unless issues arise

### Step-Specific Rules

- 🔍 Read sprint-status.yaml to find actionable story
- 🚪 DETECT existing workflow state and handle continuation properly
- 🌿 Verify or create story branch
- 💾 Initialize workflow state file if starting fresh
- 🔀 Route to correct step based on workflow state and story status
- ⚠️ Prompt user only if information is missing or conflicts exist

---

## AVAILABLE STATE

This is the entry step. It establishes:

- `story_key` — Story identifier from sprint-status.yaml
- `story_status` — Current status (backlog, drafted, ready-for-dev, etc.)
- `story_file` — Path to story markdown file
- `story_branch` — Git branch for this story
- `parent_branch` — Branch to merge back to
- `sprint_status` — Path to sprint-status.yaml
- `workflow_state_file` — Path to orchestrate-story-state.yaml
- All config variables from workflow config

---

## EXECUTION SEQUENCE

### 1. Load Configuration

Load `{configFile}` to resolve:
- `sub_workflows.*`
- `models.*`
- `max_retries`
- `sprint_status` path
- `story_dir` path
- `workflow_state_file` path

Load `{project-root}/_bmad/bmm/config.yaml` to resolve:
- `implementation_artifacts`
- `user_name`

### 2. Read Sprint Status

Read `{sprint_status}` file completely.

Find the first story with status in priority order:
1. `in-progress` — Resume development
2. `review` — Resume code review
3. `ready-for-dev` — Start development
4. `drafted` — Needs adversarial review
5. `backlog` — Needs creation

Extract from sprint-status.yaml:
- `story_key` — Story identifier (e.g., "PROJ-123-feature-name")
- `story_status` — Current status
- `parent_branch` — Branch to merge back to (if specified)

### 2b. Check for Existing Workflow State

Read `{workflow_state_file}` if it exists.

**IF workflow state file exists AND contains entry for {story_key}:**
- Check `workflows.{story_key}.workflow_state.stepsCompleted` array
- **IF workflow incomplete** (step-11-summary NOT in stepsCompleted):
  - **STOP here** and load `{continueStepFile}` immediately
  - Do not proceed with any initialization tasks
  - Let step-02-continue handle the continuation logic
- **IF workflow complete** (step-11-summary in stepsCompleted):
  - Continue to section 3 (fresh start for this story, or ask user if they want to restart)

**IF workflow state file does NOT exist OR no entry for {story_key}:**
- Continue to section 3 (fresh workflow start)

### 3. Construct Story Variables

```
story_file = {story_dir}/{story_key}.md
story_branch = story/{story_key}
```

### 4. Initialize Workflow State File (Fresh Start)

Load `{workflowStateStructure}` for reference structures.

**IF workflow state file does NOT exist:**
- Create `{workflow_state_file}` using the "Empty State File Structure" from `{workflowStateStructure}`

**IF workflow state file exists but no entry for {story_key}:**
- Add entry for {story_key} using the "New Story Entry Structure" from `{workflowStateStructure}`

### 5. Check Parent Branch

IF `parent_branch` is not specified in sprint-status.yaml:
- Execute: `git branch --show-current`
- Store current branch as `parent_branch`

IF `parent_branch` could not be determined:
- Ask user: "Could not determine parent branch. Please specify the parent branch name:"
- Store user response as `parent_branch`
- Update sprint-status.yaml with parent_branch
- Update workflow state file: `workflows.{story_key}.execution_metadata.parentBranch = {parent_branch}`

### 6. Check Branch State

Execute: `git branch --show-current`

**IF current branch == story_branch:**
- Report: "ℹ️ Already on branch {{story_branch}}, resuming workflow"
- Skip to Step 7 (Routing)

**IF current branch != story_branch:**
- Check if story_branch exists: `git branch --list {{story_branch}}`
- IF exists: `git checkout {{story_branch}}`
- IF not exists: Continue to Step 6

### 7. Create Story Branch (if needed)

**Check working tree status:**

Execute: `git status --porcelain`

IF working tree is NOT clean:
- Ask user: "Working tree has uncommitted changes. How should I proceed?
  - [C] Commit changes with message 'WIP: Pre-story orchestration checkpoint'
  - [S] Stash changes
  - [A] Abort workflow"

- IF user chose C: Execute `git add -A && git commit -m "WIP: Pre-story orchestration checkpoint"`
- IF user chose S: Execute `git stash push -m "Pre-story orchestration checkpoint"`
- IF user chose A: Exit workflow

**Create branch:**

Execute: `git checkout -b {{story_branch}}`

Verify: `git branch --show-current`

Report: "✅ Created branch {{story_branch}} from {{parent_branch}}"

### 8. Update Workflow State File

Update `{workflow_state_file}` with initialization info:
- `workflows.{story_key}.execution_metadata.storyBranch = {story_branch}`
- `workflows.{story_key}.execution_metadata.parentBranch = {parent_branch}`
- `workflows.{story_key}.execution_metadata.lastActivity = {current_timestamp}`
- `workflows.{story_key}.domain_reference.storyStatus = {story_status}`
- `workflows.{story_key}.domain_reference.storyFile = {story_file}`

### 9. Route Based on Story Status

Report current state:
```
═══════════════════════════════════════════════════════
ORCHESTRATE STORY: Initialization Complete
═══════════════════════════════════════════════════════

Story: {story_key}
Status: {story_status}
Branch: {story_branch}
Parent: {parent_branch}
Story file: {story_file}

Routing to appropriate step...
═══════════════════════════════════════════════════════
```

**Route based on status:**

Load `{routingTable}` and use the routing table to determine the next step based on `{story_status}`.

**Before loading next step:**
- Update workflow state file: Append 'step-01-init' to `workflows.{story_key}.workflow_state.stepsCompleted`
- Update workflow state file: Set `workflows.{story_key}.workflow_state.lastStep = 'step-01-init'`

Load, read entire file, then execute the appropriate next step file.

---

## CONTEXT PASSED TO NEXT STEPS

The following variables are established and available for subsequent steps:

- `story_key` — Story identifier
- `story_status` — Current status from sprint-status.yaml
- `story_file` — Path to story markdown file
- `story_branch` — Git branch for this story
- `parent_branch` — Branch to merge back to
- `sprint_status` — Path to sprint-status.yaml
- `workflow_state_file` — Path to orchestrate-story-state.yaml
- All config variables from workflow config

---

## SUCCESS METRICS

- Workflow state file checked for continuation (routed to step-02-continue if needed)
- Story identified from sprint-status.yaml
- Workflow state file initialized (if fresh start)
- Branch state verified or created
- Variables established for subsequent steps
- Routed to correct step based on status
- Workflow state file updated with step-01-init completion

## FAILURE MODES

- Not checking workflow state file for continuation
- Not reading sprint-status.yaml
- Not initializing workflow state file for fresh starts
- Creating branch without checking working tree
- Routing to wrong step for status
- Not preserving parent_branch information
- Not updating workflow state file with step completion

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
