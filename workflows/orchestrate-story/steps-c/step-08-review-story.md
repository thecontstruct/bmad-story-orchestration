---
name: 'step-08-review-story'
description: 'Execute code review with retry-with-feedback task'

# File References
nextStepFile: './step-09-human-checkpoint.md'
configFile: '{project-root}/_bmad/_config/custom/orchestrate/workflows/orchestrate-story/config.yaml'
delegationBlocks: '../data/delegation-blocks.md'

# Task References
retryWithFeedbackTask: '{project-root}/_bmad/orchestrate/tasks/retry-with-feedback.md'
---

# Step 5: Review Story

## STEP GOAL

Execute code review via the code-review workflow, automatically retrying development on review failures.

## MANDATORY EXECUTION RULES (READ FIRST)

### Universal Rules

- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When loading next step, ensure entire file is read first
- 🎯 This step uses retry logic for automated dev-review cycles

### Step-Specific Rules

- ✅ Skip review if status is already `done`
- 🔁 Retry development on review failures (up to max_retries)
- 📊 Proceed to human checkpoint on approval
- 🚪 Present user options on max retries reached

---

## AVAILABLE STATE

From previous steps:

- `story_key`, `story_status`, `story_file`, `story_branch`, `parent_branch`
- `sprint_status` — Path to sprint-status.yaml
- `sub_workflows.*`, `models.*`, `max_retries` from config

Step-local state:

- `review_retry_count` — Initialized to 0 in this step
- `review_feedback` — Accumulated review feedback

---

## EXECUTION SEQUENCE

### 1. Check if Review Needed

IF `story_status == done`:
- Report: "ℹ️ Story already reviewed (status: done), skipping code review"
- Load, read entire file, then execute {nextStepFile}

### 2. Report Action

```
═══════════════════════════════════════════════════════
PHASE 2: Code Review
═══════════════════════════════════════════════════════

Story: {{story_key}}
Status: {{story_status}}
Action: Executing code review

Invoking review workflow...
═══════════════════════════════════════════════════════
```

### 3. Initialize Review Loop

```
review_retry_count = 0
review_feedback = null
```

### 4. Execute Code Review

Load `{delegationBlocks}` and use the "Code Review Delegation Block" template to invoke code-review workflow via delegation block.

### 5. Parse Review Outcome

Read `{{story_file}}` to extract review outcome from "Senior Developer Review (AI)" section.

Parse for:
- `review_outcome`: APPROVE | CHANGES_REQUESTED | BLOCKED
- `review_feedback`: Detailed feedback content

### 6. Handle Review Outcome

**IF review_outcome == APPROVE:**

```
✅ Step 5: Code Review Approved

Story: {{story_key}}
Review attempts: {{review_retry_count + 1}}
Outcome: APPROVED

Proceeding to human checkpoint...
```

**Before proceeding, update workflow state file:**
1. Load `{workflow_state_file}`
2. Append 'step-08-review-story' to `workflows.{story_key}.workflow_state.stepsCompleted` array
3. Set `workflows.{story_key}.workflow_state.lastStep = 'step-08-review-story'`
4. Update `workflows.{story_key}.execution_metadata.lastActivity = {current_timestamp}`
5. Save file

Load, read entire file, then execute {nextStepFile}

**IF review_outcome == CHANGES_REQUESTED or BLOCKED:**

```
review_retry_count = review_retry_count + 1
```

Report:
```
⚠️ Code Review: Changes Requested

Story: {{story_key}}
Attempt: {{review_retry_count}}/{{max_retries}}

Feedback:
{{review_feedback}}
```

**IF review_retry_count < max_retries:**

Report: "Automatically retrying development with feedback..."

Update sprint-status.yaml: Set story status to `ready-for-dev`

Load `{delegationBlocks}` and use the "Fix Review Delegation Block" template to execute fix via dev-story workflow with feedback.

Update sprint-status.yaml: Set story status to `review`

Go back to Step 4 (re-execute code review)

**IF review_retry_count >= max_retries:**

```
❌ Code Review: Maximum Retries Reached

Story: {{story_key}}
Attempts: {{review_retry_count}}/{{max_retries}}

Accumulated feedback:
{{review_feedback}}
```

Present user options:

"Maximum review attempts ({{max_retries}}) reached. What would you like to do?
- [R] Reset retry count and try again
- [M] Exit for manual fix (branch preserved)
- [C] Continue anyway (not recommended)"

**Handle user choice:**

- IF R: Reset review_retry_count = 0, update status to `ready-for-dev`, go back to Step 4
- IF M: Report "Branch {{story_branch}} preserved for manual fixes", exit workflow
- IF C:
  - Warn: "⚠️ Proceeding with unaddressed review feedback"
  - Update workflow state file: Append 'step-08-review-story' to `workflows.{story_key}.workflow_state.stepsCompleted`, set `lastStep = 'step-08-review-story'`, update `lastActivity`
  - Load, read entire file, then execute {nextStepFile}

---

## SUCCESS METRICS

- Code review executed
- Review outcome parsed correctly
- Retry loop handled correctly
- User decisions honored
- Proceeded to human checkpoint on approval

## FAILURE MODES

- Skipping code review
- Not handling review feedback
- Proceeding without resolving review issues (unless user chose to)
- Not tracking retry count

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
