---
name: 'step-03-create-story'
description: 'Invoke create-story sub-workflow to generate story file'

# File References
nextStepFile: './step-04-adversarial-story-review.md'
configFile: '{project-root}/_bmad/_config/custom/orchestrate/workflows/orchestrate-story/config.yaml'
---

# Step 2: Create Story

## STEP GOAL

Invoke the create-story sub-workflow to generate the story file for the current backlog item.

## MANDATORY EXECUTION RULES (READ FIRST)

### Universal Rules

- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When loading next step, ensure entire file is read first
- 🎯 This step invokes an external workflow via delegation block

### Step-Specific Rules

- ✅ Verify story status is `backlog` before proceeding
- 🔧 Invoke create-story workflow in isolated context
- ✔️ Verify story file was created
- 📊 Update sprint-status.yaml to `drafted`

---

## AVAILABLE STATE

From step-01-init:

- `story_key`, `story_status`, `story_file`, `story_branch`, `parent_branch`
- `sprint_status` — Path to sprint-status.yaml
- `sub_workflows.*`, `models.*`, `max_retries` from config

---

## EXECUTION SEQUENCE

### 1. Verify Prerequisites

Check that `story_status == backlog`.

IF story_status != backlog:
- Report: "ℹ️ Story status is {{story_status}}, not backlog. Skipping creation."
- Load, read entire file, then execute {nextStepFile}

### 2. Report Action

```
═══════════════════════════════════════════════════════
PHASE 1: Story Creation
═══════════════════════════════════════════════════════

Story: {{story_key}}
Action: Creating story file via create-story workflow
Model: {{models.create_story}}

Invoking sub-workflow...
═══════════════════════════════════════════════════════
```

### 3. Invoke Create-Story Workflow

Execute create-story workflow via delegation block:

```xml
<delegate model="{{models.create_story}}">
  <system-instructions>
    You are executing a BMAD workflow in isolated context.
    - Follow workflow instructions sequentially
    - Do not spawn additional agents or delegation blocks
    - CRITICAL: Execute in #yolo mode - skip all user prompts and auto-proceed through workflow.xml steps
    - When workflow.xml asks for user input (e.g., "Continue to next step?", template-output prompts), automatically proceed without waiting
    - Treat this as non-interactive execution - simulate expert user responses to proceed automatically
  </system-instructions>

  <context>
    - workflow_path: {{sub_workflows.create_story}}
    - project_root: {{project-root}}
    - story_key: {{story_key}}
    - output_path: {{story_file}}
  </context>

  <prompt>
    Execute the create-story workflow:
    1. Load {{sub_workflows.create_story}}/workflow.yaml
    2. Read and understand workflow configuration
    3. Load {project-root}/_bmad/core/tasks/workflow.xml
    4. Follow workflow.xml instructions to execute the create-story workflow (auto-proceed through all prompts)
    5. The workflow will discover the story from sprint-status.yaml
    6. Generate complete story file with all required sections
    7. Save story file to {{story_file}}
  </prompt>

  <output>
    - Keep response concise (under 500 words)
    - outcome: success | failure
    - story_file_path: path to created file
    - issues: any problems encountered
  </output>
</delegate>
```

### 4. Verify Story Creation

**Check story file exists:**

Verify `{{story_file}}` was created and has content.

IF story file does NOT exist:
- Report: "❌ Story file was not created at {{story_file}}"
- Ask user: "Would you like to:
  - [R] Retry story creation
  - [M] Exit for manual creation"
- IF R: Go back to Step 3
- IF M: Exit workflow (preserve branch)

### 5. Update Sprint Status and Report Success

Update sprint-status.yaml:

1. Load `{{sprint_status}}`
2. Find `{{story_key}}` in development_status section
3. Set status to `drafted`
4. Save file, preserving ALL comments and structure including STATUS DEFINITIONS

Report: "✅ Sprint status updated: {{story_key}} → drafted"

```
✅ Step 2: Story Creation Complete

Story file: {{story_file}}
Status: drafted

Proceeding to validation...
```

### 6. Update Workflow State File

Before proceeding, update `{workflow_state_file}`:
1. Load `{workflow_state_file}`
2. Append 'step-03-create-story' to `workflows.{story_key}.workflow_state.stepsCompleted` array
3. Set `workflows.{story_key}.workflow_state.lastStep = 'step-03-create-story'`
4. Update `workflows.{story_key}.execution_metadata.lastActivity = {current_timestamp}`
5. Update `workflows.{story_key}.domain_reference.storyStatus = 'drafted'`
6. Save file

### 7. Proceed to Next Step

Load, read entire file, then execute {nextStepFile} to begin adversarial story review.

---

## SUCCESS METRICS

- Story file created at expected path
- Story file contains required sections
- Sprint status updated to `drafted`
- Proceeded to adversarial story review step

## FAILURE MODES

- Invoking create-story for non-backlog story
- Not verifying story file exists
- Not updating sprint-status
- Proceeding without successful creation

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
