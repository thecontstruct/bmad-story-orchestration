---
name: 'step-04-adversarial-story-review'
description: 'Adversarial review of story file using retry-with-feedback task'

# File References
nextStepFile: './step-05-validate-story.md'
configFile: '../config.yaml'

# Task References
retryWithFeedbackTask: '{project-root}/_bmad/my-custom-bmad/tasks/retry-with-feedback.md'
---

# Step 2b: Adversarial Story Review

## STEP GOAL

Perform adversarial review of the created story file using retry-with-feedback to catch issues before checklist validation. This provides a fresh, skeptical perspective that complements the structured checklist validation, with automatic retry and delegation support.

## MANDATORY EXECUTION RULES (READ FIRST)

### Universal Rules

- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When loading next step, ensure entire file is read first
- 🎯 This step uses retry-with-feedback task for automated adversarial review with retry loop

### Step-Specific Rules

- ✅ Skip if story file doesn't exist (shouldn't happen, but handle gracefully)
- 🔁 Use retry task for automatic retry with feedback
- 🔍 Adversarial review runs in fresh sub-agent context (delegation)
- 📊 Update workflow state on success
- 🚪 Present user options on max retries reached

---

## AVAILABLE STATE

From step-03-create-story:

- `story_key`, `story_status`, `story_file`, `story_branch`, `parent_branch`
- `sprint_status` — Path to sprint-status.yaml
- `sub_workflows.*`, `models.*` from config
- `adversarial_validation_task` — Path to adversarial validation wrapper task (from config)
- `adversarial_max_retries` — Max retries for adversarial review loop (from config, default: 3)
- `models.adversarial_validate` — Model for adversarial validation (gemini-3-pro)

---

## EXECUTION SEQUENCE

### 1. Verify Story File Exists

Check that `{{story_file}}` exists and has content.

IF story file does NOT exist:
- Report: "❌ Story file not found at {{story_file}}. Skipping adversarial review."
- Load, read entire file, then execute {nextStepFile}

### 2. Report Action

```
═══════════════════════════════════════════════════════
PHASE 1: Adversarial Story Review
═══════════════════════════════════════════════════════

Story: {{story_key}}
Status: {{story_status}}
Action: Performing adversarial review of story file
Purpose: Catch issues before checklist validation

Initializing retry task...
═══════════════════════════════════════════════════════
```

### 3. Configure Retry Context

Load config file `{{configFile}}` and extract:
- `adversarial_validation_task` path
- `adversarial_max_retries` (defaults to 3 if not set)
- `models.adversarial_validate` (model for adversarial validation)

Set context variables for retry task:

```
action_workflow = {{sub_workflows.create_story}}
validation_task = {{adversarial_validation_task}}  # From config.yaml
target_document = {{story_file}}
fix_workflow = {{sub_workflows.create_story}}
fix_model = {{models.fix_validation}}
max_retries = {{adversarial_max_retries}}
models.validate = {{models.adversarial_validate}}
```

**Note:** 
- `adversarial_validation_task` is loaded from `config.yaml` and points to `{project-root}/_bmad/my-custom-bmad/tasks/validate-adversarial-review.md`
- `adversarial_max_retries` is separate from the main `max_retries` setting
- `models.validate` is set to `models.adversarial_validate` (gemini-3-pro) for adversarial review delegation

### 4. Execute Retry Task

Execute {retryWithFeedbackTask}

The task will:
1. Validate story file using adversarial review (delegated to fresh context)
2. On issues: automatically retry by invoking create-story workflow with feedback up to adversarial_max_retries
3. Return outcome: PASS | MAX_RETRIES_REACHED

### 5. Handle Task Outcome

**IF outcome == PASS:**

```
✅ Step 2b: Adversarial Story Review Passed

Story: {{story_key}}
Review attempts: {{retry_count}}
Outcome: PASSED

Proceeding to checklist validation...
```

- Update workflow state file: Append 'step-04-adversarial-story-review' to `workflows.{story_key}.workflow_state.stepsCompleted`, set `lastStep = 'step-04-adversarial-story-review'`, set `lastActivity = {current_timestamp}`
- Load, read entire file, then execute {nextStepFile}

**IF outcome == MAX_RETRIES_REACHED:**

```
⚠️ Adversarial Story Review: Maximum Retries Reached

Story: {{story_key}}
Attempts: {{retry_count}}/{{adversarial_max_retries}}

Accumulated feedback:
{{feedback}}
```

Present user options:

"Maximum adversarial review attempts ({{adversarial_max_retries}}) reached. What would you like to do?
- [R] Reset retry count and try again
- [M] Exit for manual fix (branch preserved)
- [S] Skip adversarial review and continue to validation (not recommended)"

**Handle user choice:**

- IF R: Reset retry_count = 0, go back to Step 4
- IF M: Report "Branch {{story_branch}} preserved for manual fixes", exit workflow
- IF S:
  - Warn: "⚠️ Proceeding with unresolved adversarial review issues"
  - Update workflow state file: Append 'step-04-adversarial-story-review' to `workflows.{story_key}.workflow_state.stepsCompleted`, set `lastStep = 'step-04-adversarial-story-review'`, set `lastActivity = {current_timestamp}`
  - Load, read entire file, then execute {nextStepFile}

---

## SUCCESS METRICS

- Adversarial review executed via retry-with-feedback
- Retry loop handled correctly
- User decisions honored
- Workflow state updated appropriately
- Proceeded to validation step

## FAILURE MODES

- Not using retry task
- Proceeding without handling max retries outcome
- Not updating workflow state
- Skipping adversarial review

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
