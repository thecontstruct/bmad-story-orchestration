# Retry With Feedback Task

Execute an action with validation, automatically retry on failure up to max attempts, passing feedback from previous attempts to improve the next attempt.

**Prerequisites:**
- Load subprocess delegation skill: `{project-root}/.claude/skills/subprocess-delegation/SKILL.md`
- This task uses `<delegate>` blocks which require the skill for proper interpretation

---

## CONTEXT VALIDATION

Before executing, verify required context variables are available. Prompt user for any missing required variables.

### Required Variables

| Variable | Description | Prompt if Missing |
|----------|-------------|-------------------|
| `action_workflow` | Workflow to execute | "What workflow should I run to create/fix the artifact?" |
| `validation_task` | Task to validate result | "What validation task should I use?" |
| `target_document` | Document being validated | "What document should I validate?" |
| `max_retries` | Maximum retry attempts | "How many retry attempts? (default: 3)" |

### Optional Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `validation_checklist` | Checklist for validation | `{action_workflow}/checklist.md` |
| `fix_workflow` | Workflow for fixes | Same as `action_workflow` |
| `fix_model` | Model for fix attempts | Current model |

### Validation Process

1. For each required variable:
   - Check if defined in current context
   - IF missing: Display prompt and wait for user input
   - Store user response as variable value

2. For each optional variable:
   - Check if defined in current context
   - IF missing: Apply default value
   - Report: "Using default for {variable}: {default_value}"

3. Confirm configuration before proceeding:

   "**Retry Task Configuration:**
   - Action: {{action_workflow}}
   - Validation: {{validation_task}}
   - Checklist: {{validation_checklist}}
   - Target: {{target_document}}
   - Fix workflow: {{fix_workflow}}
   - Max retries: {{max_retries}}
   
   **Ready to proceed? [Y/n]**"

---

## EXECUTION FLOW

### Initialize

```
retry_count = 0
feedback = null
accumulated_feedback = []
```

### Step 1: Execute Action (if needed)

Check if `{{target_document}}` exists.

IF target document does NOT exist:
- Report: "Target document not found. Executing action workflow..."
- Execute `{{action_workflow}}` via delegation block:

```xml
<delegate model="{{models.action}}">
  <system-instructions>
    You are executing a BMAD workflow in isolated context.
    - Follow workflow instructions sequentially
    - Do not spawn additional agents or delegation blocks
  </system-instructions>

  <context>
    - workflow_path: {{action_workflow}}
    - project_root: {{project-root}}
  </context>

  <prompt>
    Execute the action workflow:
    1. Load {{action_workflow}}/workflow.md completely
    2. Read and understand: Goal, Role, Architecture rules
    3. Load config as specified in INITIALIZATION SEQUENCE
    4. Execute first step file as directed
    5. Follow step file instructions sequentially
  </prompt>

  <output>
    - Keep response concise (under 500 words)
    - outcome: success | failure
    - artifact_path: path to created document
    - issues: any problems encountered
  </output>
</delegate>
```

- Verify target document was created
- IF not created: Report error and return `{ outcome: "ACTION_FAILED", feedback: "..." }`

### Step 2: Execute Validation

Execute `{{validation_task}}` against `{{target_document}}`:

```xml
<delegate model="{{models.validate}}">
  <system-instructions>
    You are executing a validation task in isolated context.
    - Follow validation instructions precisely
    - Do not spawn additional agents or delegation blocks
  </system-instructions>

  <context>
    - validation_task: {{validation_task}}
    - project_root: {{project-root}}
    - checklist: {{validation_checklist}}
    - document: {{target_document}}
  </context>

  <prompt>
    Execute validation:
    1. Load {{validation_task}}
    2. Execute validation against the checklist and document
    3. Report all issues found with specific locations
  </prompt>

  <output>
    - Keep response concise (under 500 words if possible)
    - outcome: PASS | ISSUES_FOUND
    - validation_report_path: path to report
    - feedback: detailed feedback including checklist item that failed, specific location, what was found vs expected, actionable recommendation
  </output>
</delegate>
```

Parse validation response:
- Extract `outcome`: PASS | ISSUES_FOUND
- Extract `feedback`: detailed validation feedback
- Extract `validation_report_path`: path to validation report

### Step 3: Handle PASS Outcome

IF `outcome == PASS`:
- Report: "✅ Validation passed!"
- Return:
  ```yaml
  outcome: "PASS"
  retry_count: {{retry_count}}
  feedback: null
  validation_report_path: {{validation_report_path}}
  ```

### Step 4: Handle ISSUES_FOUND Outcome

IF `outcome == ISSUES_FOUND`:

1. Increment retry count:
   ```
   retry_count = retry_count + 1
   accumulated_feedback.push(feedback)
   ```

2. Report issues:
   "⚠️ Validation found issues (attempt {{retry_count}}/{{max_retries}}):
   {{feedback}}"

3. Check retry limit:

   IF `retry_count < max_retries`:
   - Report: "Automatically fixing issues..."
   - Execute fix workflow with feedback context:

   ```xml
   <delegate model="{{fix_model}}">
     <system-instructions>
       You are executing a fix workflow in isolated context.
       - CRITICAL: Read and address validation feedback before proceeding
       - Follow workflow instructions sequentially
       - Do not spawn additional agents or delegation blocks
     </system-instructions>

     <context>
       - workflow_path: {{fix_workflow}}
       - project_root: {{project-root}}
       - validation_feedback: {{feedback}}
       - validation_report_path: {{validation_report_path}}
     </context>

     <prompt>
       Execute fix workflow:
       1. Review: {{validation_report_path}}
       2. Fix all issues mentioned in feedback
       3. Load {{fix_workflow}}/workflow.md
       4. Execute workflow to regenerate/fix the artifact
     </prompt>

     <output>
       - Keep response concise (under 500 words)
       - outcome: success | failure
       - fixes_applied: list of changes made
       - issues: any problems encountered
     </output>
   </delegate>
   ```

   - GOTO Step 2 (re-validate)

   IF `retry_count >= max_retries`:
   - Report: "❌ Maximum retry attempts ({{max_retries}}) reached."
   - Display accumulated feedback from all attempts
   - Return:
     ```yaml
     outcome: "MAX_RETRIES_REACHED"
     retry_count: {{retry_count}}
     feedback: {{accumulated_feedback}}
     validation_report_path: {{validation_report_path}}
     ```

---

## OUTPUT STRUCTURE

The task returns a structured result:

| Field | Type | Description |
|-------|------|-------------|
| `outcome` | string | `PASS` \| `ISSUES_FOUND` \| `MAX_RETRIES_REACHED` \| `ACTION_FAILED` |
| `retry_count` | number | Number of attempts made |
| `feedback` | string/array | Accumulated feedback from validation attempts |
| `validation_report_path` | string | Path to last validation report |

---

## USAGE NOTES

### Calling from Steps

The calling step should set context variables before invoking this task:

```markdown
### 1. Configure Retry Context

action_workflow = {{sub_workflows.create_story}}
validation_task = {project-root}/_bmad/orchestrate/tasks/validate-workflow.xml
validation_checklist = {{sub_workflows.create_story}}/checklist.md
target_document = {{story_file}}
fix_model = {{models.fix_validation}}
max_retries = {{max_retries}}

### 2. Execute Retry Loop

Execute {retryWithFeedbackTask}

### 3. Handle Result

Based on task outcome:
- IF outcome == "PASS": Proceed to next step
- IF outcome == "MAX_RETRIES_REACHED": Present user options
- IF outcome == "ACTION_FAILED": Report error and halt
```

### User Options on MAX_RETRIES_REACHED

The calling step should handle this outcome by presenting options:

```markdown
"Maximum retry attempts reached. What would you like to do?
- [R] Reset retry count and try again
- [M] Exit for manual fix (preserve current state)
- [S] Skip validation and continue anyway (not recommended)"
```
