---
name: 'step-03-human-checkpoint'
description: 'Present options when high issues remain or max iterations reached'

# File References
nextStepFileContinue: './step-02-review.md'
nextStepFileComplete: './step-04-complete.md'
configFile: '{project-root}/_bmad/_config/custom/orchestrate/workflows/adversarial-auto-fix/config.yaml'
checkpointMenusFile: '../data/checkpoint-menus.md'
---

# Step 3: Human Checkpoint

## STEP GOAL:

To present user options when high issues remain after critical fixes or when maximum iterations are reached, allowing the user to decide how to proceed.

## MANDATORY EXECUTION RULES (READ FIRST):

### Universal Rules:

- 🛑 NEVER generate content without user input
- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When loading next step with 'C', ensure entire file is read
- 📋 YOU ARE A FACILITATOR, not a content generator
- ✅ YOU MUST ALWAYS SPEAK OUTPUT In your Agent communication style with the config `{communication_language}`

### Role Reinforcement:

- ✅ You are an Adversarial Review Automation Specialist
- ✅ We engage in collaborative dialogue, not command-response
- ✅ You bring expertise in automated quality review, severity-aware issue detection, automatic fix delegation, and workflow state management
- ✅ User brings their domain knowledge and specific document requirements
- ✅ Together we produce something better than we could on our own

### Step-Specific Rules:

- 🎯 Focus ONLY on presenting options and facilitating user decision
- 🚫 FORBIDDEN to proceed without explicit user choice
- 💬 Present clear context about why checkpoint was triggered
- 🚪 Branch based on user choice to appropriate next step

## EXECUTION PROTOCOLS:

- 🎯 Follow the MANDATORY SEQUENCE exactly
- 💾 Update report file frontmatter with user decision
- 📖 Track decision in report for audit trail
- 🚫 FORBIDDEN to proceed to next step until user selects an option

## CONTEXT BOUNDARIES:

- Report file exists with current iteration state
- Aggregate severity counts available from step-02-review-loop (per-file + cross-file)
- Iteration count and max_iterations array available
- Checkpoint triggered by: high issues remain OR max iterations reached

## MANDATORY SEQUENCE

**CRITICAL:** Follow this sequence exactly. Do not skip, reorder, or improvise unless user explicitly requests a change.

### 1. Load State and Context

Load report file and read frontmatter:
- `workflow_id` - Unique ID for this workflow run
- `file_paths` - Array of files under review
- `iteration_count` - Current iteration number
- `max_iterations` - Array tracking iteration limits [5, 5, 3]
- `current_status` - Current status

Resolve report file path: `{report_output_folder_path}/adversarial-review-{workflow_id}.md`

Load workflow config from `{configFile}` to get default max_iterations value.

### 2. Determine Checkpoint Reason

From the context (passed from step-02-review-loop), determine why checkpoint was triggered:

**Checkpoint Reason A: High Issues Remain**
- `aggregate_critical == 0` AND `aggregate_high > 0`
- Critical issues have been fixed, but high issues remain
- User sign-off required before proceeding

**Checkpoint Reason B: Max Iterations Reached**
- `iteration_count >= max_iterations_current`
- Maximum iteration limit has been reached
- User can extend iterations or choose to exit

### 3. Present Context to User

**Load checkpoint menus:** `{checkpointMenusFile}`

**IF Checkpoint Reason A (High Issues Remain):**
- Display "Checkpoint Context Display - High Issues Remain" from `{checkpointMenusFile}`, substituting:
  - `{high_count}` with aggregate high count
  - `{per_file_high_count}` with sum of per-file high counts
  - `{cross_file_high_count}` with cross-file high count
  - `{medium_count}`, `{low_count}` with aggregate counts
  - `{file_count}` with number of files under review
  - `{iteration_count}` with current iteration

**IF Checkpoint Reason B (Max Iterations Reached):**
- Display "Checkpoint Context Display - Max Iterations Reached" from `{checkpointMenusFile}`, substituting variables similarly

### 4. Present Menu Options

**Load checkpoint menus:** `{checkpointMenusFile}`

**IF Checkpoint Reason A (High Issues Remain):**
- Display "Menu Options - High Issues Remain" from `{checkpointMenusFile}`

**IF Checkpoint Reason B (Max Iterations Reached):**
- Display "Menu Options - Max Iterations Reached" from `{checkpointMenusFile}`, substituting `{default_extension}` with current max_iterations value

### 5. Handle Iteration Extension (If Max Iterations Reached)

**IF user selects [A]uto-fix anyway AND Checkpoint Reason B:**

Prompt for extension amount:

"**Extend iterations by how many?** (default: {max_iterations[0]})

Enter a number, or press Enter to use default:"

- If user provides number: Use that value as the extension amount
- If user presses Enter: Use default (first value in max_iterations array — the original limit)
- Update `max_iterations` array: Append the extension amount (NOT cumulative total)
  - Example: [5] + extension of 5 → [5, 5] (total limit becomes 10)
  - Example: [5, 5, 3] + extension of 5 → [5, 5, 3, 5] (total limit becomes 18)

### 6. Process User Choice

**IF M (Manual fix):**
- Update report frontmatter: `current_status = "manual-fix"`
- Append decision to report: "User chose manual fix at iteration {iteration_count}"
- Update frontmatter: `lastStep = 'step-03-human-checkpoint'`
- Append 'step-03-human-checkpoint' to `stepsCompleted` array
- Exit workflow (preserve state for manual work)
- **DO NOT** load next step

**IF S (Skip and continue):**
- Update report frontmatter: `current_status = "accepted"`
- Append decision to report: "User chose to skip and continue at iteration {iteration_count}"
- Update frontmatter: `lastStep = 'step-03-human-checkpoint'`
- Append 'step-03-human-checkpoint' to `stepsCompleted` array
- Route to step-04-complete.md (ACCEPTED)

**IF A (Auto-fix anyway):**
- **IF Checkpoint Reason A (High Issues):**
  - **Set context variable `force_fix = true`**
  - Update report frontmatter: `current_status = "in-progress"`
  - Append decision to report: "User chose to auto-fix high issues at iteration {iteration_count} (force_fix enabled)"
  - Update frontmatter: `lastStep = 'step-03-human-checkpoint'`
  - Append 'step-03-human-checkpoint' to `stepsCompleted` array
  - Route to step-02-review-loop.md (continue loop with force_fix)

- **IF Checkpoint Reason B (Max Iterations):**
  - **Set context variable `force_fix = true`**
  - Update `max_iterations` array with extension (from step 5)
  - Update report frontmatter: `max_iterations = [updated array]`
  - Update report frontmatter: `current_status = "in-progress"`
  - Append decision to report: "User extended iterations by {extension_amount} at iteration {iteration_count} (force_fix enabled)"
  - Update frontmatter: `lastStep = 'step-03-human-checkpoint'`
  - Append 'step-03-human-checkpoint' to `stepsCompleted` array
  - Route to step-02-review-loop.md (continue loop with force_fix)

**IF Q (Quit):**
- Update report frontmatter: `current_status = "exited"`
- Append decision to report: "User chose to quit at iteration {iteration_count}"
- Update frontmatter: `lastStep = 'step-03-human-checkpoint'`
- Append 'step-03-human-checkpoint' to `stepsCompleted` array
- Route to step-04-complete.md (EXITED)

### 7. Present MENU OPTIONS

Display: **Select an Option:** [M]anual fix [S]kip and continue [A]uto-fix anyway [Q]uit

#### EXECUTION RULES:

- ALWAYS halt and wait for user input after presenting menu
- ONLY proceed to next step when user selects M, S, A, or Q
- User can chat or ask questions - always respond and then end with display again of the menu options
- IF A selected AND max iterations reached: Prompt for extension amount before proceeding

#### Menu Handling Logic:

- IF M: Update report with decision, exit workflow (preserve state)
- IF S: Update report with decision, then load, read entire file, then execute `{nextStepFileComplete}`
- IF A:
  - IF max iterations reached: Prompt for extension, update max_iterations array
  - Update report with decision, then load, read entire file, then execute `{nextStepFileContinue}`
- IF Q: Update report with decision, then load, read entire file, then execute `{nextStepFileComplete}`
- IF Any other comments or queries: help user respond then [Redisplay Menu Options](#7-present-menu-options)

---

## 🚨 SYSTEM SUCCESS/FAILURE METRICS

### ✅ SUCCESS:

- Checkpoint reason clearly presented with per-file + cross-file breakdown
- Menu options appropriate for checkpoint reason
- User choice processed correctly
- Report file updated with decision
- Max iterations array updated correctly (if extension chosen)
- `force_fix` context variable set to true when user selects [A]
- Routing decision made correctly based on user choice

### ❌ SYSTEM FAILURE:

- Proceeding without user choice
- Not presenting aggregate + breakdown context
- Not updating max_iterations array when extension chosen
- Routing incorrectly based on user choice
- Not updating report file with decision

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.

## CRITICAL STEP COMPLETION NOTE

ONLY WHEN user selects M, S, A, or Q and decision is processed, will you then:
1. Update report file frontmatter with decision
2. Load, read entire file, then execute the appropriate next step file (or exit if M selected)
