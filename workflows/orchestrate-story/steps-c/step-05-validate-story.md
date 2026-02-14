---
name: 'step-05-validate-story'
description: 'Validate story file using retry-with-feedback task'

# File References
nextStepFile: './step-06-develop-story.md'
configFile: '{project-root}/_bmad/_config/custom/orchestrate/workflows/orchestrate-story/config.yaml'

# Task References
retryWithFeedbackTask: '{project-root}/_bmad/orchestrate/tasks/retry-with-feedback.md'
---

# Step 3: Validate Story

## STEP GOAL

Validate the story file against the create-story checklist, automatically retrying with feedback on failures.

## MANDATORY EXECUTION RULES (READ FIRST)

### Universal Rules

- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When loading next step, ensure entire file is read first
- 🎯 This step uses the retry-with-feedback task for automated retry

### Step-Specific Rules

- ✅ Skip validation if status is already `ready-for-dev` or later
- 🔁 Use retry task for automatic retry with feedback
- 📊 Update sprint-status to `ready-for-dev` on success
- 🚪 Present user options on max retries reached

---

## AVAILABLE STATE

From previous steps:

- `story_key`, `story_status`, `story_file`, `story_branch`, `parent_branch`
- `sprint_status` — Path to sprint-status.yaml
- `sub_workflows.*`, `models.*`, `max_retries` from config

---

## EXECUTION SEQUENCE

### 1. Check if Validation Needed

IF `story_status` in [`ready-for-dev`, `in-progress`, `review`, `done`]:
- Report: "ℹ️ Story already validated (status: {{story_status}}), skipping validation"
- Load, read entire file, then execute {nextStepFile}

### 2. Report Action

```
═══════════════════════════════════════════════════════
PHASE 1: Story Validation
═══════════════════════════════════════════════════════

Story: {{story_key}}
Status: {{story_status}}
Action: Validating story file against checklist

Initializing retry task...
═══════════════════════════════════════════════════════
```

### 3. Configure Retry Context

Set context variables for retry task:

```
action_workflow = {{sub_workflows.create_story}}
validation_task = {{validation_task}}
validation_checklist = {{sub_workflows.create_story}}/checklist.md
target_document = {{story_file}}
fix_workflow = {{sub_workflows.create_story}}
fix_model = {{models.fix_validation}}
max_retries = {{max_retries}}
```

### 4. Execute Retry Task

Execute {retryWithFeedbackTask}

The task will:
1. Validate story file against checklist
2. On issues: automatically retry with feedback up to max_retries
3. Return outcome: PASS | MAX_RETRIES_REACHED

### 5. Handle Task Outcome

**IF outcome == PASS:**

```
✅ Step 3: Story Validation Passed

Story: {{story_key}}
Validation attempts: {{retry_count}}

Updating status to ready-for-dev...
```

- Update sprint-status.yaml: Set story status to `ready-for-dev`
- Update workflow state file: Append 'step-05-validate-story' to `workflows.{story_key}.workflow_state.stepsCompleted`, set `lastStep = 'step-05-validate-story'`, set `lastActivity = {current_timestamp}`
- Load, read entire file, then execute {nextStepFile}

**IF outcome == MAX_RETRIES_REACHED:**

```
⚠️ Story Validation: Maximum Retries Reached

Story: {{story_key}}
Attempts: {{retry_count}}/{{max_retries}}

Accumulated feedback:
{{feedback}}
```

Present user options:

"Maximum validation attempts ({{max_retries}}) reached. What would you like to do?
- [R] Reset retry count and try again
- [M] Exit for manual fix (branch preserved)
- [S] Skip validation and continue anyway (not recommended)"

**Handle user choice:**

- IF R: Reset retry_count = 0, go back to Step 4
- IF M: Report "Branch {{story_branch}} preserved for manual fixes", exit workflow
- IF S: 
  - Warn: "⚠️ Proceeding with unresolved validation issues"
  - Update sprint-status.yaml: Set story status to `ready-for-dev`
  - Update workflow state file: Append 'step-05-validate-story' to `workflows.{story_key}.workflow_state.stepsCompleted`, set `lastStep = 'step-05-validate-story'`, set `lastActivity = {current_timestamp}`
  - Load, read entire file, then execute {nextStepFile}

---

## SUCCESS METRICS

- Validation executed against checklist
- Retry loop handled correctly
- User decisions honored
- Sprint status updated appropriately
- Proceeded to development step

## FAILURE MODES

- Skipping validation for `drafted` status
- Not using retry task
- Proceeding without handling max retries outcome
- Not updating sprint-status

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
