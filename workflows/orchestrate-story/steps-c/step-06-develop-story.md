---
name: 'step-06-develop-story'
description: 'Invoke dev-story sub-workflow to implement the story'

# File References
nextStepFile: './step-07-adversarial-code-review.md'
configFile: '{project-root}/_bmad/_config/custom/orchestrate/workflows/orchestrate-story/config.yaml'
---

# Step 4: Develop Story

## STEP GOAL

Invoke the dev-story sub-workflow to implement the story requirements.

## MANDATORY EXECUTION RULES (READ FIRST)

### Universal Rules

- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When loading next step, ensure entire file is read first
- 🎯 This step invokes an external workflow via delegation block

### Step-Specific Rules

- ✅ Skip development if status is already `review` or `done`
- 🔧 Invoke dev-story workflow in isolated context
- ✔️ Verify implementation was created
- 📊 Update sprint-status.yaml to `review`

---

## AVAILABLE STATE

From previous steps:

- `story_key`, `story_status`, `story_file`, `story_branch`, `parent_branch`
- `sprint_status` — Path to sprint-status.yaml
- `sub_workflows.*`, `models.*`, `max_retries` from config

---

## EXECUTION SEQUENCE

### 1. Check if Development Needed

IF `story_status` in [`review`, `done`]:
- Report: "ℹ️ Story already developed (status: {{story_status}}), skipping development"
- Load, read entire file, then execute {nextStepFile}

### 2. Report Action

```
═══════════════════════════════════════════════════════
PHASE 2: Story Development
═══════════════════════════════════════════════════════

Story: {{story_key}}
Status: {{story_status}}
Action: Implementing story via dev-story workflow
Model: {{models.dev_story}}

Invoking sub-workflow...
═══════════════════════════════════════════════════════
```

### 3. Invoke Dev-Story Workflow

Execute dev-story workflow via delegation block:

```xml
<delegate model="{{models.dev_story}}">
  <system-instructions>
    You are executing a BMAD workflow in isolated context.
    - Follow workflow instructions sequentially
    - Do not spawn additional agents or delegation blocks
    - CRITICAL: Execute in #yolo mode - skip all user prompts and auto-proceed through workflow.xml steps
    - When workflow.xml asks for user input (e.g., "Continue to next step?", template-output prompts), automatically proceed without waiting
    - Treat this as non-interactive execution - simulate expert user responses to proceed automatically
  </system-instructions>

  <context>
    - workflow_path: {{sub_workflows.dev_story}}
    - project_root: {{project-root}}
    - story_file: {{story_file}}
  </context>

  <prompt>
    Execute the dev-story workflow:
    1. Load {{sub_workflows.dev_story}}/workflow.yaml
    2. Read and understand workflow configuration
    3. Load {project-root}/_bmad/core/tasks/workflow.xml
    4. Follow workflow.xml instructions to execute the dev-story workflow (auto-proceed through all prompts)
    5. Read story file at {{story_file}} to understand requirements
    6. Implement all story requirements
    7. Update story file with development section
  </prompt>

  <output>
    - Keep response concise (under 500 words)
    - outcome: success | failure
    - files_modified: list of changed files
    - issues: any problems encountered
  </output>
</delegate>
```

### 4. Verify Development

**Check implementation:**

- Verify implementation files were modified (git status shows changes)
- Verify story file has development section added

IF implementation appears incomplete:
- Report: "⚠️ Development may be incomplete"
- Ask user: "Would you like to:
  - [R] Retry development
  - [C] Continue to code review anyway
  - [M] Exit for manual development"
- IF R: Go back to Step 3
- IF C: Continue to Step 5
- IF M: Exit workflow (preserve branch)

**Update sprint-status:**

Update sprint-status.yaml: Set story status to `review`

### 5. Report Success

```
✅ Step 4: Story Development Complete

Story: {{story_key}}
Status: review

Proceeding to code review...
```

### 6. Update Workflow State File

Before proceeding, update `{workflow_state_file}`:
1. Load `{workflow_state_file}`
2. Append 'step-06-develop-story' to `workflows.{story_key}.workflow_state.stepsCompleted` array
3. Set `workflows.{story_key}.workflow_state.lastStep = 'step-06-develop-story'`
4. Update `workflows.{story_key}.execution_metadata.lastActivity = {current_timestamp}`
5. Update `workflows.{story_key}.domain_reference.storyStatus = 'review'`
6. Save file

### 7. Proceed to Next Step

Load, read entire file, then execute {nextStepFile} to begin adversarial code review.

---

## SUCCESS METRICS

- Dev-story workflow executed successfully
- Implementation files modified
- Story file updated with development section
- Sprint status updated to `review`
- Proceeded to adversarial code review step

## FAILURE MODES

- Invoking dev-story for already-reviewed story
- Not verifying implementation was created
- Not updating sprint-status
- Proceeding without successful development

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
