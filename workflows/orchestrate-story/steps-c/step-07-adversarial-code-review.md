---
name: 'step-07-adversarial-code-review'
description: 'Adversarial review of code diff using retry-with-feedback task'

# File References
nextStepFile: './step-08-review-story.md'
configFile: '../config.yaml'

# Task References
retryWithFeedbackTask: '{project-root}/_bmad/my-custom-bmad/tasks/retry-with-feedback.md'
---

# Step 4b: Adversarial Code Review

## STEP GOAL

Perform adversarial review of code changes (diff) using retry-with-feedback before the formal code-review workflow. This catches implementation issues early with a fresh, skeptical perspective, with automatic retry and delegation support.

## MANDATORY EXECUTION RULES (READ FIRST)

### Universal Rules

- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When loading next step, ensure entire file is read first
- 🎯 This step uses retry-with-feedback task for automated adversarial review with retry loop

### Step-Specific Rules

- ✅ Construct complete diff from baseline commit
- 🔁 Use retry task for automatic retry with feedback
- 🔍 Adversarial review runs in fresh sub-agent context (delegation)
- 📊 Update workflow state on success
- 🚪 Present user options on max retries reached

---

## AVAILABLE STATE

From step-06-develop-story:

- `story_key`, `story_status`, `story_file`, `story_branch`, `parent_branch`
- `baseline_commit` — Git commit hash at workflow start (from step-01-init)
- `sprint_status` — Path to sprint-status.yaml
- `sub_workflows.*`, `models.*` from config
- `adversarial_validation_task` — Path to adversarial validation wrapper task (from config)
- `adversarial_max_retries` — Max retries for adversarial review loop (from config)
- `models.adversarial_validate` — Model for adversarial validation (gemini-3-pro)

---

## EXECUTION SEQUENCE

### 1. Verify Baseline Commit

Check that `{{baseline_commit}}` is available.

IF baseline_commit is NOT available:
- Report: "⚠️ Baseline commit not available. Skipping adversarial code review."
- Load, read entire file, then execute {nextStepFile}

### 2. Construct Diff

Build complete diff of all changes since workflow started.

**If `{{baseline_commit}}` is a Git commit hash:**

**Tracked File Changes:**
```bash
git diff {{baseline_commit}}
```

**New Untracked Files:**
Only include untracked files that were created during development (step-04).
Do not include pre-existing untracked files.
For each new file created, include its full content as a "new file" addition.

**If `{{baseline_commit}}` is "NO_GIT":**

Use best-effort diff construction:
- List all files modified during step-04
- For each file, show the changes made (before/after if available, or current state)
- Include any new files created with their full content
- Note: This is less precise than Git diff but still enables meaningful review

**Capture as `{diff_output}`**

Save diff to temporary file: `{{story_file}}.adversarial-diff.md`

**Note:** Do NOT `git add` anything - this is read-only inspection.

### 3. Report Action

```
═══════════════════════════════════════════════════════
PHASE 2: Adversarial Code Review
═══════════════════════════════════════════════════════

Story: {{story_key}}
Status: {{story_status}}
Action: Performing adversarial review of code changes
Baseline: {{baseline_commit}}

Initializing retry task...
═══════════════════════════════════════════════════════
```

### 4. Configure Retry Context

Load config file `{{configFile}}` and extract:
- `adversarial_validation_task` path
- `adversarial_max_retries` (defaults to 3 if not set)
- `models.adversarial_validate` (model for adversarial validation)

Set context variables for retry task:

```
action_workflow = {{sub_workflows.dev_story}}
validation_task = {{adversarial_validation_task}}  # From config.yaml
target_document = {{story_file}}.adversarial-diff.md
fix_workflow = {{sub_workflows.dev_story}}
fix_model = {{models.fix_review}}
max_retries = {{adversarial_max_retries}}
models.validate = {{models.adversarial_validate}}
```

**Note:** 
- `adversarial_validation_task` is loaded from `config.yaml` and points to `{project-root}/_bmad/my-custom-bmad/tasks/validate-adversarial-review.md`
- `adversarial_max_retries` is separate from the main `max_retries` setting
- `models.validate` is set to `models.adversarial_validate` (gemini-3-pro) for adversarial review delegation

**Note:** 
- `adversarial_validation_task` is loaded from `config.yaml` and points to `{project-root}/_bmad/my-custom-bmad/tasks/validate-adversarial-review.md`
- `adversarial_max_retries` is separate from the main `max_retries` setting
- `models.validate` is set to `models.adversarial_validate`

### 5. Execute Retry Task

Execute {retryWithFeedbackTask}

The task will:
1. Validate code diff using adversarial review (delegated to fresh context)
2. On issues: automatically retry by invoking dev-story workflow with feedback up to adversarial_max_retries
3. Return outcome: PASS | MAX_RETRIES_REACHED

### 6. Handle Task Outcome

**IF outcome == PASS:**

```
✅ Step 4b: Adversarial Code Review Passed

Story: {{story_key}}
Review attempts: {{retry_count}}
Outcome: PASSED

Proceeding to formal code review...
```

- Update workflow state file: Append 'step-07-adversarial-code-review' to `workflows.{story_key}.workflow_state.stepsCompleted`, set `lastStep = 'step-07-adversarial-code-review'`, set `lastActivity = {current_timestamp}`
- Load, read entire file, then execute {nextStepFile}

**IF outcome == MAX_RETRIES_REACHED:**

```
⚠️ Adversarial Code Review: Maximum Retries Reached

Story: {{story_key}}
Attempts: {{retry_count}}/{{adversarial_max_retries}}

Accumulated feedback:
{{feedback}}
```

Present user options:

"Maximum adversarial review attempts ({{adversarial_max_retries}}) reached. What would you like to do?
- [R] Reset retry count and try again
- [M] Exit for manual fix (branch preserved)
- [S] Skip adversarial review and continue to formal review (not recommended)"

**Handle user choice:**

- IF R: Reset retry_count = 0, go back to Step 5
- IF M: Report "Branch {{story_branch}} preserved for manual fixes", exit workflow
- IF S:
  - Warn: "⚠️ Proceeding with unresolved adversarial review issues"
  - Update workflow state file: Append 'step-07-adversarial-code-review' to `workflows.{story_key}.workflow_state.stepsCompleted`, set `lastStep = 'step-07-adversarial-code-review'`, set `lastActivity = {current_timestamp}`
  - Load, read entire file, then execute {nextStepFile}

---

## SUCCESS METRICS

- Diff constructed successfully
- Adversarial review executed via retry-with-feedback
- Retry loop handled correctly
- User decisions honored
- Workflow state updated appropriately
- Proceeded to formal code review step

## FAILURE MODES

- Missing baseline_commit (can't construct accurate diff)
- Not including new untracked files in diff
- Not using retry task
- Proceeding without handling max retries outcome
- Not updating workflow state

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
