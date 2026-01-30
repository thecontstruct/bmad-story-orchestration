---
name: 'step-02-continue'
description: 'Handle workflow continuation from previous session'

# File References
nextStepFile: './step-03-create-story.md'
configFile: '../config.yaml'
---

# Step 1B: Workflow Continuation

## STEP GOAL:

To resume the orchestrate-story workflow from where it was left off, ensuring smooth continuation without loss of context or progress.

## MANDATORY EXECUTION RULES (READ FIRST):

### Universal Rules:

- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When loading next step with 'C', ensure entire file is read
- 📋 YOU ARE A FACILITATOR, not an autonomous executor

### Role Reinforcement:

- ✅ You are a story orchestration specialist
- ✅ We engage in collaborative dialogue, not command-response
- ✅ You bring workflow execution expertise, user brings domain knowledge
- ✅ Together we deliver complete story implementation

### Step-Specific Rules:

- 🎯 Focus ONLY on analyzing and resuming workflow state
- 🚫 FORBIDDEN to modify story files or sprint-status.yaml during continuation analysis
- 💬 Maintain continuity with previous sessions
- 🚪 DETECT exact continuation point from workflow state file

## EXECUTION PROTOCOLS:

- 🎯 Show your analysis of current state before taking action
- 💾 Keep existing workflow state file values intact during analysis
- 📖 Review workflow state file and sprint-status.yaml to understand progress
- 🚫 FORBIDDEN to modify state files during continuation analysis
- 📝 Update workflow state file with continuation timestamp when resuming

## CONTEXT BOUNDARIES:

- Workflow state file (`{workflow_state_file}`) is available
- Sprint status file (`{sprint_status}`) is available
- Previous context = workflow state file + sprint-status.yaml
- Last completed step = last value in `stepsCompleted` array from workflow state file

## CONTINUATION SEQUENCE:

### 1. Load Configuration

Load `{configFile}` to resolve:
- `workflow_state_file` — path to orchestrate-story-state.yaml
- `sprint_status` — path to sprint-status.yaml
- `story_dir` — path to stories directory
- All other config variables

### 2. Load Workflow State File

Read `{workflow_state_file}` completely.

**IF workflow state file does NOT exist:**
- This is a fresh workflow (shouldn't have reached this step)
- Report error: "Workflow state file not found. Routing to step-01-init.md"
- Load, read entire file, then execute `./step-01-init.md`

**IF workflow state file exists:**
- Continue to section 3

### 3. Analyze Current Workflow State

Review the workflow state file to understand:

- `workflows.{story_key}.workflow_state.stepsCompleted`: Which steps are already done (the rightmost value is the last step completed)
- `workflows.{story_key}.workflow_state.lastStep`: Name of last completed step (if exists)
- `workflows.{story_key}.workflow_state.lastContinued`: Last continuation timestamp
- `workflows.{story_key}.execution_metadata`: Branch info, timestamps, etc.

**Example:** If `stepsCompleted: ['step-01-init', 'step-03-create-story', 'step-05-validate-story']`, then step-05-validate-story was the last completed step.

### 4. Read Sprint Status

Read `{sprint_status}` file completely to get current story status:

- Find the story identified in workflow state file
- Extract current `story_status` (backlog, drafted, ready-for-dev, review, done)
- Extract `story_key`, `parent_branch` if available

### 5. Determine Story Context

From workflow state file and sprint-status.yaml:

- `story_key` — Story identifier
- `story_status` — Current domain status
- `story_branch` — Git branch for this story (from execution_metadata)
- `parent_branch` — Branch to merge back to (from execution_metadata)
- `story_file` — Path to story markdown file (`{story_dir}/{story_key}.md`)

### 6. Read Completed Step Files (For Context)

For each step in `stepsCompleted` array (excluding step-01-init):

1. **Construct step filename**: `step-[N]-[name].md`
2. **Read the complete step file** to understand:
   - What that step accomplished
   - What the next step should be (from nextStep references)
   - Any specific context or decisions made

**Example:** If `stepsCompleted: ['step-01-init', 'step-03-create-story', 'step-05-validate-story']`:
- Read `step-03-create-story.md`
- Read `step-05-validate-story.md`
- The last file will tell you what step-06 should be

### 7. Determine Next Step

Based on the last completed step and story status:

1. **Find the nextStep reference** in the last completed step file
2. **Validate the file exists** at the referenced path
3. **Confirm the workflow is incomplete** (not all steps finished)
4. **Consider story status** — if story status changed externally, may need to route differently

**Routing Logic (based on last completed step):**
- If last step was step-03-create-story → Next is step-04-adversarial-story-review
- If last step was step-04-adversarial-story-review → Next is step-05-validate-story
- If last step was step-05-validate-story → Next is step-06-develop-story
- If last step was step-06-develop-story → Next is step-07-adversarial-code-review
- If last step was step-07-adversarial-code-review → Next is step-08-review-story
- If last step was step-08-review-story → Next is step-09-human-checkpoint
- If last step was step-09-human-checkpoint → Next is step-10-commit-merge
- If last step was step-10-commit-merge → Next is step-11-summary
- If last step was step-11-summary → Workflow complete

**Status-based routing (if no steps completed or status changed externally):**
- `backlog` → step-03-create-story
- `drafted` → step-04-adversarial-story-review
- `ready-for-dev` → step-06-develop-story
- `in-progress` → step-06-develop-story
- `review` → step-08-review-story
- `done` → step-11-summary

### 8. Welcome Back Dialog

Present a warm, context-aware welcome:

"Welcome back! I see we've resumed the orchestrate-story workflow.

**Current Story:** {story_key}
**Story Status:** {story_status}
**Last Completed Step:** {lastStep}
**Last Continued:** {lastContinued}

We last worked on {brief description of last step}.

Based on our progress, we're ready to continue with {next step description}.

Are you ready to continue where we left off?"

### 9. Validate Continuation Intent

Ask confirmation questions if needed:

"Has anything changed since our last session that might affect our approach?"
"Are you still aligned with the story requirements?"
"Would you like to review what we've accomplished so far?"

### 10. Present MENU OPTIONS

Display: "**Resuming workflow - Select an Option:** [C] Continue to {Next Step Name}"

#### EXECUTION RULES:

- ALWAYS halt and wait for user input after presenting menu
- ONLY proceed to next step when user selects 'C'
- User can chat or ask questions - always respond and then end with display again of the menu options
- Update workflow state file with continuation timestamp when 'C' is selected

#### Menu Handling Logic:

- IF C:
  1. Update workflow state file: add `lastContinued: {current_timestamp}` to `workflows.{story_key}.workflow_state`
  2. Load, read entire file, then execute the appropriate next step file (determined in section 7)
- IF Any other comments or queries: help user respond then [Redisplay Menu Options](#10-present-menu-options)

## CRITICAL STEP COMPLETION NOTE

ONLY WHEN C is selected and continuation analysis is complete, will you then:

1. Update workflow state file with continuation timestamp
2. Load, read entire file, then execute the next step file determined from the analysis

Do NOT modify sprint-status.yaml or story files during this continuation step.

## 🚨 SYSTEM SUCCESS/FAILURE METRICS

### ✅ SUCCESS:

- Correctly identified last completed step from `stepsCompleted` array
- Read and understood all previous step contexts
- User confirmed readiness to continue
- Workflow state file updated with continuation timestamp
- Workflow resumed at appropriate next step

### ❌ SYSTEM FAILURE:

- Skipping analysis of existing state
- Modifying sprint-status.yaml or story files during continuation
- Loading wrong next step file
- Not updating workflow state file with continuation info
- Proceeding without user confirmation

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.

