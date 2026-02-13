---
name: 'step-01-init'
description: 'Document type selection, branch setup, continuation detection, and state-based routing'

# File References
configFile: '../config.yaml'
continueStepFile: './step-02-continue.md'
routingTable: '../data/routing-table.md'
workflowStateStructure: '../data/workflow-state-structure.md'
---

# Step 1: Initialization

## STEP GOAL

Detect document type (command argument or user selection), check for existing tracking file, auto-detect existing document or prepare for new creation, setup git branch, initialize workflow state file if needed, and route to the appropriate step (continuation or fresh start).

## MANDATORY EXECUTION RULES (READ FIRST)

### Universal Rules

- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When routing to next step, ensure entire file is read first
- 🎯 This step is automatic — no user input required unless issues arise

### Step-Specific Rules

- 🔍 Detect document type from command argument or prompt user
- 🚪 DETECT existing workflow state and handle continuation properly
- 🌿 Verify or create document branch
- 💾 Initialize workflow state file if starting fresh
- 🔀 Route to correct step based on workflow state and document status
- ⚠️ Prompt user only if information is missing or conflicts exist

---

## AVAILABLE STATE

This is the entry step. It establishes:

- `document_type` — Document type ('prd' | 'architecture' | 'epic')
- `document_file` — Path to document markdown file
- `document_branch` — Git branch for this document
- `parent_branch` — Branch to merge back to
- `tracking_file` — Path to orchestrate-{doc-type}-state.yaml
- `workflow_state_file` — Path to orchestrator workflow state file
- All config variables from workflow config

---

## EXECUTION SEQUENCE

### 1. Load Configuration

Load `{configFile}` to resolve:
- `sub_workflows.*` — paths to create-PRD, create-Architecture, create-Epic workflows
- `models.*` — model configuration for delegation blocks
- `adversarial_max_retries`
- `planning_artifacts` path
- `workflow_state_file` path
- `tracking_files.*` — paths to document-type-specific tracking files

Load `{project-root}/_bmad/bmm/config.yaml` to resolve:
- `planning_artifacts`
- `user_name`
- `project_name`

Load `{project-root}/_bmad/bmb/config.yaml` to resolve:
- `user_name`
- `communication_language`

### 2. Detect Document Type

**Check command arguments:**
- If workflow invoked with document type argument (e.g., `/orchestrate-planning-documents PRD`):
  - Extract document type from argument
  - Validate: must be 'prd', 'architecture', or 'epic'
  - Store as `document_type`

**IF document type not provided via command:**
- Ask user: "Which document type would you like to create?
  - [PRD] Product Requirements Document
  - [A] Architecture Document
  - [E] Epic Document"
- Store user selection as `document_type` (normalize to lowercase: 'prd', 'architecture', 'epic')

### 3. Determine Tracking File Path

Based on `document_type`, set `tracking_file`:
- PRD: `{tracking_files.prd}`
- Architecture: `{tracking_files.architecture}`
- Epic: `{tracking_files.epic}`

### 4. Check for Existing Workflow State

Read `{workflow_state_file}` if it exists.

**IF workflow state file exists:**
- Check if any entry exists for current `document_type` or document file path
- **IF workflow incomplete** (step-07-summary NOT in stepsCompleted for this document):
  - **STOP here** and load `{continueStepFile}` immediately
  - Do not proceed with any initialization tasks
  - Let step-02-continue handle the continuation logic
- **IF workflow complete** (step-07-summary in stepsCompleted):
  - Continue to section 5 (fresh start, or ask user if they want to restart)

**IF workflow state file does NOT exist:**
- Continue to section 5 (fresh workflow start)

### 5. Check for Existing Tracking File

Read `{tracking_file}` if it exists.

**IF tracking file exists:**
- Extract `document_file` path from tracking file
- Check if document file exists at that path
- Read document file frontmatter to get `document_status`
- Store `document_file` and `document_status`

**IF tracking file does NOT exist:**
- Set `document_file` = null (will be created by sub-workflow)
- Set `document_status` = null

### 6. Construct Document Variables

**IF document_file exists:**
- Extract document key from file path (e.g., "prd-2026-02-03.md" → "prd-2026-02-03")
- `document_branch` = `{document_type}/{document_key}`

**IF document_file does NOT exist:**
- Generate document key: `{document_type}-{current_date}` (format: YYYY-MM-DD)
- `document_file` = `{planning_artifacts}/{document_type}/{document_key}.md`
- `document_branch` = `{document_type}/{document_key}`

### 7. Initialize Workflow State File (Fresh Start)

Load `{workflowStateStructure}` for reference structures.

**IF workflow state file does NOT exist:**
- Create `{workflow_state_file}` using the "Empty State File Structure" from `{workflowStateStructure}`

**IF workflow state file exists but no entry for {document_file_path}:**
- Add entry for {document_file_path} using the "New Document Entry Structure" from `{workflowStateStructure}`

### 8. Initialize Tracking File (If Needed)

**IF tracking file does NOT exist:**
- Create `{tracking_file}` with structure from `{workflowStateStructure}`:
  - Use `document_file_path` as key (relative path from planning_artifacts)
  - Initialize with default values

**IF tracking file exists:**
- Update `lastActivity` timestamp

### 9. Check Parent Branch

Execute: `git branch --show-current`
Store current branch as `parent_branch`

IF `parent_branch` could not be determined:
- Ask user: "Could not determine parent branch. Please specify the parent branch name:"
- Store user response as `parent_branch`
- Update workflow state file: `workflows.{document_file_path}.execution_metadata.parentBranch = {parent_branch}`

### 10. Check Branch State

Execute: `git branch --show-current`

**IF current branch == document_branch:**
- Report: "ℹ️ Already on branch {{document_branch}}, resuming workflow"
- Skip to Step 11 (Routing)

**IF current branch != document_branch:**
- Check if document_branch exists: `git branch --list {{document_branch}}`
- IF exists: `git checkout {{document_branch}}`
- IF not exists: Continue to Step 11

### 11. Create Document Branch (if needed)

**Check working tree status:**

Execute: `git status --porcelain`

IF working tree is NOT clean:
- Ask user: "Working tree has uncommitted changes. How should I proceed?
  - [C] Commit changes with message 'WIP: Pre-document orchestration checkpoint'
  - [S] Stash changes
  - [A] Abort workflow"

- IF user chose C: Execute `git add -A && git commit -m "WIP: Pre-document orchestration checkpoint"`
- IF user chose S: Execute `git stash push -m "Pre-document orchestration checkpoint"`
- IF user chose A: Exit workflow

**Create branch:**

Execute: `git checkout -b {{document_branch}}`

Verify: `git branch --show-current`

Report: "✅ Created branch {{document_branch}} from {{parent_branch}}"

### 12. Update Workflow State File

Update `{workflow_state_file}` with initialization info:
- `workflows.{document_file_path}.execution_metadata.documentBranch = {document_branch}`
- `workflows.{document_file_path}.execution_metadata.parentBranch = {parent_branch}`
- `workflows.{document_file_path}.execution_metadata.documentFile = {document_file}`
- `workflows.{document_file_path}.execution_metadata.documentType = {document_type}`
- `workflows.{document_file_path}.execution_metadata.lastActivity = {current_timestamp}`
- `workflows.{document_file_path}.domain_reference.documentStatus = {document_status}`

### 13. Update Tracking File

Update `{tracking_file}` with initialization info:
- `workflows.{document_file_path}.execution_metadata.documentBranch = {document_branch}`
- `workflows.{document_file_path}.execution_metadata.parentBranch = {parent_branch}`
- `workflows.{document_file_path}.execution_metadata.documentFile = {document_file}`
- `workflows.{document_file_path}.execution_metadata.documentType = {document_type}`
- `workflows.{document_file_path}.execution_metadata.lastActivity = {current_timestamp}`
- `workflows.{document_file_path}.domain_reference.documentStatus = {document_status}`

### 14. Route Based on Document Status

Report current state:
```
═══════════════════════════════════════════════════════
ORCHESTRATE PLANNING DOCUMENTS: Initialization Complete
═══════════════════════════════════════════════════════

Document Type: {document_type}
Status: {document_status}
Branch: {document_branch}
Parent: {parent_branch}
Document file: {document_file}

Routing to appropriate step...
═══════════════════════════════════════════════════════
```

**Route based on status:**

Load `{routingTable}` and use the routing table to determine the next step based on `{document_status}`.

**Routing logic:**
- IF `document_status` == null OR `document_status` == 'draft' OR document incomplete:
  - Next step: `./step-03-create-document.md`
- IF `document_status` == 'complete':
  - Next step: `./step-04-adversarial-review.md`

**Before loading next step:**
- Update workflow state file: Append 'step-01-init' to `workflows.{document_file_path}.workflow_state.stepsCompleted`
- Update workflow state file: Set `workflows.{document_file_path}.workflow_state.lastStep = 'step-01-init'`
- Update tracking file: Append 'step-01-init' to `workflows.{document_file_path}.workflow_state.stepsCompleted`
- Update tracking file: Set `workflows.{document_file_path}.workflow_state.lastStep = 'step-01-init'`

Load, read entire file, then execute the appropriate next step file.

---

## CONTEXT PASSED TO NEXT STEPS

The following variables are established and available for subsequent steps:

- `document_type` — Document type ('prd' | 'architecture' | 'epic')
- `document_file` — Path to document markdown file
- `document_branch` — Git branch for this document
- `parent_branch` — Branch to merge back to
- `tracking_file` — Path to orchestrate-{doc-type}-state.yaml
- `workflow_state_file` — Path to orchestrator workflow state file
- `document_status` — Current status from document frontmatter or tracking file
- All config variables from workflow config

---

## SUCCESS METRICS

- Document type detected (command argument or user selection)
- Workflow state file checked for continuation (routed to step-02-continue if needed)
- Tracking file checked for existing document
- Workflow state file initialized (if fresh start)
- Tracking file initialized (if fresh start)
- Branch state verified or created
- Variables established for subsequent steps
- Routed to correct step based on document status
- Workflow state file updated with step-01-init completion
- Tracking file updated with step-01-init completion

## FAILURE MODES

- Not checking workflow state file for continuation
- Not detecting document type properly
- Not checking tracking file for existing documents
- Not initializing workflow state file for fresh starts
- Not initializing tracking file for fresh starts
- Creating branch without checking working tree
- Routing to wrong step for document status
- Not preserving parent_branch information
- Not updating workflow state file with step completion
- Not updating tracking file with step completion

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
