---
name: 'step-03-create-document'
description: 'Invoke create-[doc-type] sub-workflow to generate document file'

# File References
nextStepFile: './step-04-adversarial-review.md'
delegationBlocks: '../data/delegation-blocks.md'
---

# Step 3: Create Document

## STEP GOAL

Invoke the create-[doc-type] sub-workflow to generate the document file (PRD, Architecture, or Epic) based on the document type selected in step-01-init.

## MANDATORY EXECUTION RULES (READ FIRST)

### Universal Rules

- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When loading next step, ensure entire file is read first
- 🎯 This step invokes an external workflow via delegation block

### Step-Specific Rules

- ✅ Verify document doesn't exist or is incomplete before proceeding
- 🔧 Invoke create-[doc-type] workflow in isolated context
- ✔️ Verify document file was created
- 📊 Update tracking file with document status

---

## AVAILABLE STATE

From step-01-init:

- `document_type` — Document type ('prd' | 'architecture' | 'epic')
- `document_file` — Path to document markdown file
- `document_branch` — Git branch for this document
- `parent_branch` — Branch to merge back to
- `tracking_file` — Path to orchestrate-{doc-type}-state.yaml
- `workflow_state_file` — Path to orchestrator workflow state file
- `document_status` — Current status (draft, complete, or null)
- `sub_workflows.*`, `models.*` from config

---

## EXECUTION SEQUENCE

### 1. Verify Prerequisites

Check that `document_status` is null OR `document_status == 'draft'` OR document is incomplete.

IF document_status == 'complete' AND document file exists:
- Report: "ℹ️ Document already exists and is complete at {{document_file}}. Skipping creation."
- Load, read entire file, then execute {nextStepFile}

### 2. Determine Sub-Workflow Path

Based on `document_type`, set sub-workflow path:
- PRD: `{sub_workflows.create_prd}`
- Architecture: `{sub_workflows.create_architecture}`
- Epic: `{sub_workflows.create_epic}`

Set model based on `document_type`:
- PRD: `{models.create_prd}`
- Architecture: `{models.create_architecture}`
- Epic: `{models.create_epic}`

### 3. Report Action

```
═══════════════════════════════════════════════════════
PHASE 2: Document Creation
═══════════════════════════════════════════════════════

Document Type: {{document_type}}
Action: Creating {{document_type}} document via create-{{document_type}} workflow
Model: {{models.create_[doc_type]}}
Output: {{document_file}}

Invoking sub-workflow...
═══════════════════════════════════════════════════════
```

### 4. Invoke Create-[Doc-Type] Workflow

Load `{delegationBlocks}` for delegation block pattern.

Execute create-[doc-type] workflow via delegation block:

```xml
<delegate model="{{models.create_[doc_type]}}">
  <system-instructions>
    You are executing a BMAD workflow in isolated context.
    - Follow workflow instructions sequentially
    - Do not spawn additional agents or delegation blocks
    - CRITICAL: Execute in #yolo mode - skip all user prompts and auto-proceed through workflow steps
    - When workflow asks for user input (e.g., "Continue to next step?", menu prompts), automatically proceed without waiting
    - Treat this as non-interactive execution - simulate expert user responses to proceed automatically
    - If document file path is provided, use it; otherwise, let workflow determine output path
  </system-instructions>

  <context>
    - workflow_path: {{sub_workflows.create_[doc_type]}}
    - project_root: {{project-root}}
    - document_type: {{document_type}}
    - output_path: {{document_file}}
    - document_branch: {{document_branch}}
  </context>

  <prompt>
    Execute the create-{{document_type}} workflow:
    1. Load {{sub_workflows.create_[doc_type]}}/workflow.md
    2. Read and understand workflow configuration
    3. Follow workflow instructions to execute the create-{{document_type}} workflow (auto-proceed through all prompts)
    4. Generate complete {{document_type}} document with all required sections
    5. Save document file to {{document_file}} (or let workflow determine path if not specified)
    6. Complete the workflow fully - do not stop until document is created
  </prompt>

  <output>
    - Keep response concise (under 500 words)
    - outcome: success | failure
    - document_file_path: path to created file
    - issues: any problems encountered
  </output>
</delegate>
```

**Note:** Replace `[doc_type]` with actual document type (prd, architecture, epic) when executing.

### 5. Verify Document Creation

**Check document file exists:**

Verify `{{document_file}}` was created and has content.

IF document file does NOT exist:
- Check if workflow created document at a different path (workflow may determine its own path)
- Search for recently created files matching document type pattern in planning_artifacts folder
- IF found: Update `document_file` to actual path
- IF not found:
  - Report: "❌ Document file was not created at {{document_file}}"
  - Ask user: "Would you like to:
    - [R] Retry document creation
    - [M] Exit for manual creation"
  - IF R: Go back to Step 4
  - IF M: Exit workflow (preserve branch)

### 6. Read Document Status

Read `{{document_file}}` frontmatter to extract:
- `document_status` — Current status (draft, complete, etc.)
- `stepsCompleted` — Steps completed in document creation workflow

Update `document_status` variable with value from document frontmatter.

### 7. Update Tracking File

Update tracking file:

1. Load `{{tracking_file}}`
2. Find entry for `{{document_file_path}}` (or create if doesn't exist)
3. Update `workflows.{document_file_path}.execution_metadata.documentFile = {document_file}`
4. Update `workflows.{document_file_path}.domain_reference.documentStatus = {document_status}`
5. Update `workflows.{document_file_path}.execution_metadata.lastActivity = {current_timestamp}`
6. Save file

Report: "✅ Tracking file updated: {{document_type}} document status → {{document_status}}"

### 8. Update Workflow State File

Before proceeding, update `{workflow_state_file}`:
1. Load `{workflow_state_file}`
2. Append 'step-03-create-document' to `workflows.{document_file_path}.workflow_state.stepsCompleted` array
3. Set `workflows.{document_file_path}.workflow_state.lastStep = 'step-03-create-document'`
4. Update `workflows.{document_file_path}.execution_metadata.lastActivity = {current_timestamp}`
5. Update `workflows.{document_file_path}.execution_metadata.documentFile = {document_file}`
6. Update `workflows.{document_file_path}.domain_reference.documentStatus = {document_status}`
7. Save file

### 9. Report Success

```
✅ Step 3: Document Creation Complete

Document Type: {{document_type}}
Document file: {{document_file}}
Status: {{document_status}}

Proceeding to adversarial review...
```

### 10. Proceed to Next Step

Load, read entire file, then execute {nextStepFile} to begin adversarial review.

---

## SUCCESS METRICS

- Document file created at expected path (or workflow-determined path)
- Document file contains required sections
- Document status extracted from frontmatter
- Tracking file updated with document status
- Workflow state file updated with step completion
- Proceeded to adversarial review step

## FAILURE MODES

- Invoking create-[doc-type] for already-complete document
- Not verifying document file exists
- Not reading document status from frontmatter
- Not updating tracking file
- Not updating workflow state file
- Proceeding without successful creation

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
