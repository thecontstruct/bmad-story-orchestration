---
name: 'step-02-continue'
description: 'Handle workflow continuation from previous session'

# File References
configFile: '../config.yaml'
---

# Step 2: Workflow Continuation

## STEP GOAL:

To resume the orchestrate-planning-documents workflow from where it was left off, ensuring smooth continuation without loss of context or progress.

## MANDATORY EXECUTION RULES (READ FIRST):

### Universal Rules:

- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When loading next step with 'C', ensure entire file is read
- 📋 YOU ARE A FACILITATOR, not an autonomous executor

### Role Reinforcement:

- ✅ You are a planning document orchestration specialist
- ✅ We engage in collaborative dialogue, not command-response
- ✅ You bring workflow execution expertise, user brings domain knowledge
- ✅ Together we deliver complete planning documents with quality gates

### Step-Specific Rules:

- 🎯 Focus ONLY on analyzing and resuming workflow state
- 🚫 FORBIDDEN to modify document files or tracking files during continuation analysis (tracking file is domain state, only updated by workflow steps that modify the document)
- 💬 Maintain continuity with previous sessions
- 🚪 DETECT exact continuation point from workflow state file

## EXECUTION PROTOCOLS:

- 🎯 Show your analysis of current state before taking action
- 💾 Keep existing workflow state file values intact during analysis
- 📖 Review workflow state file and tracking file to understand progress
- 🚫 FORBIDDEN to modify state files during continuation analysis
- 📝 Update workflow state file with continuation timestamp when resuming

## CONTEXT BOUNDARIES:

- Workflow state file (`{workflow_state_file}`) is available
- Tracking file (`{tracking_file}`) is available
- Previous context = workflow state file + tracking file
- Last completed step = last value in `stepsCompleted` array from workflow state file

## CONTINUATION SEQUENCE:

### 1. Load Configuration

Load `{configFile}` to resolve:
- `workflow_state_file` — path to orchestrator workflow state file
- `planning_artifacts` — path to planning artifacts folder
- `tracking_files.*` — paths to document-type-specific tracking files
- All other config variables

### 2. Load Workflow State File

Read `{workflow_state_file}` completely.

**IF workflow state file does NOT exist:**
- This is a fresh workflow (shouldn't have reached this step)
- Report error: "Workflow state file not found. Routing to step-01-init.md"
- Load, read entire file, then execute `./step-01-init.md`

**IF workflow state file exists:**
- Continue to section 3

### 3. Analyze Current Workflow State

Review the workflow state file to understand:

- Find the most recent entry (by `lastActivity` timestamp or by checking all entries)
- Extract `document_file_path` from the entry key
- Extract `workflows.{document_file_path}.workflow_state.stepsCompleted`: Which steps are already done (the rightmost value is the last step completed)
- Extract `workflows.{document_file_path}.workflow_state.lastStep`: Name of last completed step (if exists)
- Extract `workflows.{document_file_path}.workflow_state.lastContinued`: Last continuation timestamp
- Extract `workflows.{document_file_path}.execution_metadata`: Branch info, document type, timestamps, etc.

**Example:** If `stepsCompleted: ['step-01-init', 'step-03-create-document', 'step-04-adversarial-review']`, then step-04-adversarial-review was the last completed step.

### 4. Determine Document Context

From workflow state file entry:

- `document_type` — Document type ('prd' | 'architecture' | 'epic') from execution_metadata
- `document_file` — Path to document markdown file from execution_metadata
- `document_branch` — Git branch for this document from execution_metadata
- `parent_branch` — Branch to merge back to from execution_metadata
- `tracking_file` — Path to tracking file based on document_type

### 5. Load Tracking File

Read `{tracking_file}` completely to get current document status:

- Find the entry matching `document_file_path`
- Extract current `document_status` (draft, complete)
- Extract `reviewStatus` if available

### 6. Read Completed Step Files (For Context)

For each step in `stepsCompleted` array (excluding step-01-init):

1. **Construct step filename**: `step-[N]-[name].md`
2. **Read the complete step file** to understand:
   - What that step accomplished
   - What the next step should be (from nextStep references)
   - Any specific context or decisions made

**Example:** If `stepsCompleted: ['step-01-init', 'step-03-create-document', 'step-04-adversarial-review']`:
- Read `step-03-create-document.md`
- Read `step-04-adversarial-review.md`
- The last file will tell you what step-05 should be

### 7. Determine Next Step

Based on the last completed step and document status:

1. **Find the nextStep reference** in the last completed step file
2. **Validate the file exists** at the referenced path
3. **Confirm the workflow is incomplete** (not all steps finished)
4. **Consider document status** — if document status changed externally, may need to route differently

**Routing Logic (based on last completed step):**
- If last step was step-03-create-document → Next is step-04-adversarial-review
- If last step was step-04-adversarial-review → Next is step-05-human-checkpoint
- If last step was step-05-human-checkpoint → Next is step-06-commit-merge
- If last step was step-06-commit-merge → Next is step-07-summary
- If last step was step-07-summary → Workflow complete

**Status-based routing (if no steps completed or status changed externally):**
- `draft` or incomplete → step-03-create-document
- `complete` → step-04-adversarial-review

### 8. Welcome Back Dialog

Present a warm, context-aware welcome:

"Welcome back! I see we've resumed the orchestrate-planning-documents workflow.

**Current Document:** {document_file}
**Document Type:** {document_type}
**Document Status:** {document_status}
**Last Completed Step:** {lastStep}
**Last Continued:** {lastContinued}

We last worked on {brief description of last step}.

Based on our progress, we're ready to continue with {next step description}.

Are you ready to continue where we left off?"

### 9. Validate Continuation Intent

Ask confirmation questions if needed:

"Has anything changed since our last session that might affect our approach?"
"Are you still aligned with the document requirements?"
"Would you like to review what we've accomplished so far?"

### 10. Present MENU OPTIONS

Display: "**Resuming workflow - Select an Option:** [C] Continue to {Next Step Name}"

#### Menu Handling Logic:

- IF C:
  1. Update workflow state file: add `lastContinued: {current_timestamp}` to `workflows.{document_file_path}.workflow_state`
  2. Load, read entire file, then execute the appropriate next step file (determined in section 7)
- IF Any other comments or queries: help user respond then [Redisplay Menu Options](#10-present-menu-options)

#### EXECUTION RULES:

- ALWAYS halt and wait for user input after presenting menu
- ONLY proceed to next step when user selects 'C'
- User can chat or ask questions - always respond and then end with display again of the menu options
- Update workflow state file with continuation timestamp when 'C' is selected

## CRITICAL STEP COMPLETION NOTE

ONLY WHEN C is selected and continuation analysis is complete, will you then:

1. Update workflow state file with continuation timestamp
2. Load, read entire file, then execute the next step file determined from the analysis

Do NOT modify document files or tracking files during this continuation step. The tracking file is domain state and should only be updated by workflow steps that modify the document itself.

## 🚨 SYSTEM SUCCESS/FAILURE METRICS

### ✅ SUCCESS:

- Correctly identified last completed step from `stepsCompleted` array
- Read and understood all previous step contexts
- User confirmed readiness to continue
- Workflow state file updated with continuation timestamp
- Workflow resumed at appropriate next step

### ❌ SYSTEM FAILURE:

- Skipping analysis of existing state
- Modifying document files or tracking files during continuation
- Loading wrong next step file
- Not updating workflow state file with continuation info
- Proceeding without user confirmation

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
