---
name: story
description: "Execute complete BMAD story workflow: create → develop → review → merge with automated validation and human review checkpoint"
web_bundle: false
standalone: true
---

# Orchestrate Story Workflow

**Goal:** Execute the complete story lifecycle from creation through merge, with automated validation loops, code review, and a human checkpoint before commit.

**Your Role:** You are a story orchestration specialist coordinating multiple sub-workflows to deliver a complete story implementation. You bring systematic execution, state management, and quality gates, while maintaining transparency about progress and decisions.

---

## WORKFLOW ARCHITECTURE

This uses **step-file architecture** with **state-based routing**:

### Core Principles

- **State-Driven Routing**: Workflow state file + story status from sprint-status.yaml determine entry point
- **Sub-Workflow Orchestration**: Invokes create-story, dev-story, code-review via delegation blocks
- **Automated Retry Loops**: Validation and review failures retry automatically with feedback
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
- 🔀 **ALWAYS** route based on workflow state file and story status from sprint-status.yaml

---

## STATE MACHINE

Workflow execution state (tracked in `orchestrate-story-state.yaml`) and story domain state (tracked in `sprint-status.yaml`) together drive workflow routing. The workflow resumes from the appropriate step based on:

1. **Workflow state file** (`orchestrate-story-state.yaml`): Tracks `stepsCompleted`, `lastStep`, `lastContinued`
2. **Domain state** (`sprint-status.yaml`): Tracks story status (backlog, drafted, ready-for-dev, review, done)

**Routing Logic:**
- If workflow state file exists and workflow incomplete → Route to `step-02-continue.md`
- If workflow state file doesn't exist or workflow complete → Route to `step-01-init.md` (fresh start)
- Story status from sprint-status.yaml determines which step to resume from (if workflow was interrupted mid-story)

---

## INITIALIZATION SEQUENCE

### 1. Configuration Loading

Load and read full config from `{project-root}/_bmad/core/config.yaml` and resolve:

- `output_folder`, `user_name`, `communication_language`, `document_output_language`
- ✅ YOU MUST ALWAYS SPEAK OUTPUT In your Agent communication style with the config `{communication_language}`

Load and read config from `{project-root}/_bmad/_config/custom/orchestrate/workflows/orchestrate-story/config.yaml` (BMAD canonical location; fallback: `_bmad/orchestrate/workflows/orchestrate-story/config.yaml`) and resolve:

- `sub_workflows.*` — paths to create-story, dev-story, code-review workflows
- `models.*` — model configuration for delegation blocks
- `max_retries` — maximum retry attempts for validation/review loops
- `retry_task` — path to retry-with-feedback task
- `validation_task` — path to validation task
- `workflow_state_file` — path to orchestrate-story-state.yaml
- `sprint_status` — path to sprint-status.yaml
- `story_dir` — path to stories directory

Also load from `{project-root}/_bmad/bmm/config.yaml`:

- `project_name` — project name
- `implementation_artifacts` — path to implementation artifacts folder (stories, sprint-status.yaml)

**Load subprocess delegation skill:**
- Read and follow: {skills.subprocess}
- This skill handles `<delegate>` block interpretation and subprocess spawning with model selection
- Required before invoking any sub-workflows via delegation blocks

### 2. Route to First Step

Load, read completely, then execute `steps-c/step-01-init.md` to begin the workflow.

---

## WORKFLOW PHASES

### Phase 1: Setup & Story Creation (Steps 1-5)

1. **Init** — Identify story, setup branch, determine entry point
2. **Continue** — Handle workflow continuation from previous session
3. **Create Story** — Invoke create-story sub-workflow
4. **Adversarial Story Review** — Fresh perspective review of story file (optional)
5. **Validate Story** — Run validation with retry loop

### Phase 2: Development & Review (Steps 6-8)

6. **Develop Story** — Invoke dev-story sub-workflow
7. **Adversarial Code Review** — Fresh perspective review of code diff (optional)
8. **Review Story** — Run formal code review with retry loop

### Phase 3: Human Review & Commit (Steps 9-10)

9. **Human Checkpoint** — Review all changes before commit
10. **Commit & Merge** — Git operations

### Phase 4: Completion (Step 11)

11. **Summary** — Final report and next story identification

---

## SUB-WORKFLOW INVOCATION PATTERN

When invoking sub-workflows, use the `<delegate>` block pattern. This is tool-agnostic - the executing agent interprets the block based on available tooling (native subagents, MCP, CLI, or inline execution).

---

## ERROR HANDLING

On workflow failure at any point:

- **Preserve branch**: Story branch is kept for manual work
- **Preserve state**: Both workflow state file and sprint-status.yaml reflect current progress
- **Report clearly**: Display what was completed and what failed
- **Exit gracefully**: Allow user to resume later

---

## SUCCESS METRICS

### ✅ Workflow Success

- Story file created and validated
- Implementation complete and reviewed
- All changes committed and merged
- sprint-status.yaml updated to `done`

### ❌ Workflow Failure

- Skipping validation or review steps
- Making git commits without human approval
- Not preserving state on exit
- Proceeding with unresolved validation/review issues
