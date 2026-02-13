---
name: 'step-04-adversarial-review'
description: 'Adversarial review of document file using retry-with-feedback task'

# File References
nextStepFile: './step-05-human-checkpoint.md'
configFile: '../config.yaml'

# Task References
retryWithFeedbackTask: '{project-root}/_bmad/my-custom-bmad/tasks/retry-with-feedback.md'
---

# Step 4: Adversarial Review

## STEP GOAL

Perform adversarial review of the created document file (PRD, Architecture, or Epic) using retry-with-feedback to catch issues before completion. This provides a fresh, skeptical perspective that complements structured validation, with automatic retry and delegation support.

## MANDATORY EXECUTION RULES (READ FIRST)

### Universal Rules

- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When loading next step, ensure entire file is read first
- 🎯 This step uses retry-with-feedback task for automated adversarial review with retry loop

### Step-Specific Rules

- ✅ Skip if document file doesn't exist (shouldn't happen, but handle gracefully)
- 🔁 Use retry task for automatic retry with feedback
- 🔍 Adversarial review runs in fresh sub-agent context (delegation)
- 📊 Update workflow state and tracking file on success
- 🚪 Present user options on max retries reached
- 🔢 Track retryCount in workflow state during fixing state

---

## AVAILABLE STATE

From step-03-create-document:

- `document_type` — Document type ('prd' | 'architecture' | 'epic')
- `document_file` — Path to document markdown file
- `document_branch` — Git branch for this document
- `parent_branch` — Branch to merge back to
- `tracking_file` — Path to orchestrate-{doc-type}-state.yaml
- `workflow_state_file` — Path to orchestrator workflow state file
- `document_status` — Current status from document frontmatter
- `sub_workflows.*`, `models.*` from config
- `adversarial_validation_task` — Path to adversarial validation task (from config)
- `adversarial_max_retries` — Max retries for adversarial review loop (from config, default: 1)
- `models.adversarial_validate` — Model for adversarial validation (from config)

---

## EXECUTION SEQUENCE

### 1. Verify Document File Exists

Check that `{{document_file}}` exists and has content.

IF document file does NOT exist:
- Report: "❌ Document file not found at {{document_file}}. Skipping adversarial review."
- Load, read entire file, then execute {nextStepFile}

### 2. Report Action

```
═══════════════════════════════════════════════════════
PHASE 2: Adversarial Document Review
═══════════════════════════════════════════════════════

Document Type: {{document_type}}
Document: {{document_file}}
Action: Performing adversarial review of {{document_type}} document
Purpose: Catch issues before completion

Initializing retry task...
═══════════════════════════════════════════════════════
```

### 3. Configure Retry Context

Load config file `{{configFile}}` and extract:
- `adversarial_validation_task` path (default: `{project-root}/_bmad/my-custom-bmad/tasks/validate-adversarial-review.md`)
- `adversarial_max_retries` (defaults to 1 if not set)
- `models.adversarial_validate` (default: "gemini-3-pro")

Set context variables for retry task:

```
action_workflow = {{sub_workflows.create_[doc_type]}}  # Path to create workflow based on document_type
validation_task = {{adversarial_validation_task}}  # From config.yaml or default
target_document = {{document_file}}
fix_workflow = {{sub_workflows.create_[doc_type]}}  # Same as action_workflow
fix_model = {{models.fix_validation}}  # From config or default
max_retries = {{adversarial_max_retries}}  # From config or default (1)
models.validate = {{models.adversarial_validate}}  # From config or default
```

**Document-Type-Specific `also_consider`:**

Set `also_consider` based on `document_type`:

- **PRD:** "Requirements completeness, stakeholder needs alignment, success criteria clarity, scope boundaries, technical feasibility, epic coverage, UX alignment"
- **Architecture:** "Architectural consistency, design patterns, scalability, security, performance, integration points, technology choices, implementation feasibility"
- **Epic:** "Epic scope clarity, story breakdown completeness, dependencies, acceptance criteria, technical approach, resource requirements"

**Note:** 
- Replace `[doc_type]` with actual document type (prd, architecture, epic) when executing
- `adversarial_validation_task` is loaded from `config.yaml` and points to `{project-root}/_bmad/my-custom-bmad/tasks/validate-adversarial-review.md`
- `adversarial_max_retries` is separate from other retry settings
- `models.validate` is set to `models.adversarial_validate` for adversarial review delegation

### 4. Update Workflow State to Fixing (Before Retry Loop)

Before executing retry task, update workflow state to indicate we're entering fixing state:

1. Load `{workflow_state_file}`
2. Update `workflows.{document_file_path}.workflow_state.retryCount = 0` (initialize)
3. Update `workflows.{document_file_path}.domain_reference.reviewStatus = 'in-fix'`
4. Save file

### 5. Execute Retry Task

Execute {retryWithFeedbackTask}

The task will:
1. Validate document using adversarial review (delegated to fresh context)
2. On issues: automatically retry by invoking create-[doc-type] workflow with feedback up to adversarial_max_retries
3. During retry loop: Workflow state will be in 'fixing' state
4. Return outcome: PASS | MAX_RETRIES_REACHED

**During retry loop (fixing state):**
- Retry task invokes create-[doc-type] workflow with feedback
- Orchestrator waits for completion (passive monitoring)
- After each retry attempt, update `retryCount` in workflow state
- User can skip review even during fixing state (if they choose)

### 6. Handle Task Outcome

**IF outcome == PASS:**

```
✅ Step 4: Adversarial Document Review Passed

Document Type: {{document_type}}
Document: {{document_file}}
Review attempts: {{retry_count}}
Outcome: PASSED

Proceeding to human checkpoint...
```

- Update workflow state file: 
  - Append 'step-04-adversarial-review' to `workflows.{document_file_path}.workflow_state.stepsCompleted`
  - Set `workflows.{document_file_path}.workflow_state.lastStep = 'step-04-adversarial-review'`
  - Set `workflows.{document_file_path}.workflow_state.retryCount = {retry_count}`
  - Update `workflows.{document_file_path}.execution_metadata.lastActivity = {current_timestamp}`
  - Update `workflows.{document_file_path}.domain_reference.reviewStatus = 'passed'`
- Update tracking file:
  - Append 'step-04-adversarial-review' to `workflows.{document_file_path}.workflow_state.stepsCompleted`
  - Set `workflows.{document_file_path}.workflow_state.lastStep = 'step-04-adversarial-review'`
  - Set `workflows.{document_file_path}.workflow_state.retryCount = {retry_count}`
  - Update `workflows.{document_file_path}.domain_reference.reviewStatus = 'passed'`
- Load, read entire file, then execute {nextStepFile}

**IF outcome == MAX_RETRIES_REACHED:**

```
⚠️ Adversarial Document Review: Maximum Retries Reached

Document Type: {{document_type}}
Document: {{document_file}}
Attempts: {{retry_count}}/{{adversarial_max_retries}}

Accumulated feedback:
{{feedback}}
```

Present user options:

"Maximum adversarial review attempts ({{adversarial_max_retries}}) reached. What would you like to do?
- [R] Reset retry count and try again
- [M] Exit for manual fix (document preserved)
- [S] Skip adversarial review and continue to human checkpoint (not recommended)"

#### Menu Handling Logic:

- IF R: 
  - Reset retry_count = 0 in workflow state
  - Update `workflows.{document_file_path}.workflow_state.retryCount = 0`
  - Update `workflows.{document_file_path}.domain_reference.reviewStatus = 'pending'`
  - Go back to Step 5 (Execute Retry Task)
- IF M: 
  - Report "Document {{document_file}} preserved for manual fixes"
  - Update `workflows.{document_file_path}.domain_reference.reviewStatus = 'max-retries'`
  - Exit workflow (preserve branch)
- IF S:
  - Warn: "⚠️ Proceeding with unresolved adversarial review issues"
  - Update workflow state file: 
    - Append 'step-04-adversarial-review' to `workflows.{document_file_path}.workflow_state.stepsCompleted`
    - Set `workflows.{document_file_path}.workflow_state.lastStep = 'step-04-adversarial-review'`
    - Set `workflows.{document_file_path}.workflow_state.retryCount = {retry_count}`
    - Update `workflows.{document_file_path}.execution_metadata.lastActivity = {current_timestamp}`
    - Update `workflows.{document_file_path}.domain_reference.reviewStatus = 'max-retries'`
  - Update tracking file similarly
  - Load, read entire file, then execute {nextStepFile}
- IF Any other comments or queries: help user respond then redisplay menu options

#### EXECUTION RULES:

- ALWAYS halt and wait for user input after presenting menu
- ONLY proceed when user selects R, M, or S
- User can chat or ask questions - always respond and then end with display again of the menu options

---

## SUCCESS METRICS

- Adversarial review executed via retry-with-feedback
- Retry loop handled correctly with fixing state tracking
- RetryCount tracked in workflow state
- User decisions honored
- Workflow state updated appropriately
- Tracking file updated appropriately
- Proceeded to human checkpoint step

## FAILURE MODES

- Not using retry task
- Proceeding without handling max retries outcome
- Not updating workflow state
- Not updating tracking file
- Not tracking retryCount during fixing state
- Skipping adversarial review without user choice

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
