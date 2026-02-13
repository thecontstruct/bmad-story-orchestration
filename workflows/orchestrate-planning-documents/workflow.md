---
name: orchestrate-planning-documents
description: "Execute complete BMAD planning document workflow: create → review → merge with automated adversarial review validation and human review checkpoint"
web_bundle: false
standalone: true
---

# Orchestrate Planning Documents Workflow

**Goal:** Execute the complete planning document lifecycle (PRD, Architecture, Epic) from creation through merge, with automated adversarial review validation loops and a human checkpoint before commit.

**Your Role:** You are a planning document orchestration specialist coordinating multiple sub-workflows to deliver complete planning documents with quality gates. You bring systematic execution, state management, and quality gates, while maintaining transparency about progress and decisions.

---

## WORKFLOW ARCHITECTURE

This uses **step-file architecture** with **state-based routing**:

### Core Principles

- **State-Driven Routing**: Workflow state file + document status from tracking files determine entry point
- **Sub-Workflow Orchestration**: Invokes create-PRD, create-Architecture, create-Epic via delegation blocks
- **Automated Retry Loops**: Adversarial review failures retry automatically with feedback
- **Human Checkpoint**: Single review gate before commit operations
- **Git Safety**: All git operations are explicit and reviewable

### Step Processing Rules

1. **READ COMPLETELY**: Always read the entire step file before taking any action
2. **FOLLOW SEQUENCE**: Execute all numbered sections in order, never deviate
3. **WAIT FOR INPUT**: If a menu is presented, halt and wait for user selection
4. **CHECK CONTINUATION**: Only proceed to next step when conditions are met
5. **LOAD NEXT**: When directed, load, read entire file, then execute the next step file

### Critical Rules (NO EXCEPTIONS)

- 🛑 **NEVER** load multiple step files simultaneously
- 📖 **ALWAYS** read entire step file before execution
- 🚫 **NEVER** skip steps or optimize the sequence
- 🎯 **ALWAYS** follow the exact instructions in the step file
- ⏸️ **ALWAYS** halt at menus and wait for user input
- 🔀 **ALWAYS** route based on workflow state file and document status from tracking files

---

## STATE MACHINE

Workflow execution state (tracked in workflow state file) and document domain state (tracked in `orchestrate-{doc-type}-state.yaml`) together drive workflow routing. The workflow resumes from the appropriate step based on:

1. **Workflow state file**: Tracks `stepsCompleted`, `lastStep`, `lastContinued`
2. **Domain state** (tracking files): Tracks document status (draft, complete)

**Routing Logic:**
- If workflow state file exists and workflow incomplete → Route to `step-02-continue.md`
- If workflow state file doesn't exist or workflow complete → Route to `step-01-init.md` (fresh start)
- Document status from tracking file determines which step to resume from (if workflow was interrupted mid-document)

---

## INITIALIZATION SEQUENCE

### 1. Configuration Loading

Load and read full config from `{project-root}/_bmad/bmb/config.yaml` and resolve:

- `output_folder`, `user_name`, `communication_language`, `document_output_language`
- ✅ YOU MUST ALWAYS SPEAK OUTPUT In your Agent communication style with the config `{communication_language}`

Load and read config from `{project-root}/_bmad/my-custom-bmad/workflows/orchestrate-planning-documents/config.yaml` and resolve:

- `sub_workflows.*` — paths to create-PRD, create-Architecture, create-Epic workflows
- `models.*` — model configuration for delegation blocks
- `adversarial_max_retries` — maximum retry attempts for adversarial review loops
- `retry_task` — path to retry-with-feedback task
- `adversarial_validation_task` — path to validate-adversarial-review task
- `workflow_state_file` — path to orchestrator workflow state file
- `planning_artifacts` — path to planning artifacts folder (for tracking files)

Also load from `{project-root}/_bmad/bmm/config.yaml`:

- `project_name` — project name
- `planning_artifacts` — path to planning artifacts folder

**Load subprocess delegation skill:**
- Read and follow: {skills.subprocess}
- This skill handles `<delegate>` block interpretation and subprocess spawning with model selection
- Required before invoking any sub-workflows via delegation blocks

### 2. Route to First Step

Load, read completely, then execute `steps-c/step-01-init.md` to begin the workflow.

---

## WORKFLOW PHASES

### Phase 1: Setup & Document Creation (Steps 1-3)

1. **Init** — Document type selection, branch setup, determine entry point
2. **Continue** — Handle workflow continuation from previous session
3. **Create Document** — Invoke create-[doc-type] sub-workflow

### Phase 2: Quality Gate (Step 4)

4. **Adversarial Review** — Fresh perspective review with retry loop

### Phase 3: Human Review & Commit (Steps 5-6)

5. **Human Checkpoint** — Review all changes before commit
6. **Commit & Merge** — Git operations

### Phase 4: Completion (Step 7)

7. **Summary** — Final report

---

## SUB-WORKFLOW INVOCATION PATTERN

When invoking sub-workflows, use the `<delegate>` block pattern. This is tool-agnostic - the executing agent interprets the block based on available tooling (native subagents, MCP, CLI, or inline execution).

---

## ERROR HANDLING

On workflow failure at any point:

- **Preserve branch**: Document branch is kept for manual work
- **Preserve state**: Both workflow state file and tracking file reflect current progress
- **Report clearly**: Display what was completed and what failed
- **Exit gracefully**: Allow user to resume later

---

## SUCCESS METRICS

### ✅ Workflow Success

- Document created and reviewed
- Adversarial review passed (or user approved proceeding)
- All changes committed and merged
- Tracking file updated appropriately

### ❌ Workflow Failure

- Skipping adversarial review steps
- Making git commits without human approval
- Not preserving state on exit
- Proceeding with unresolved review issues
