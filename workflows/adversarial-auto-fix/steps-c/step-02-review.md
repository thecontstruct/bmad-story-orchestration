---
name: 'step-02-review'
description: 'Execute two-phase adversarial review, aggregate findings, and route based on severity'

# File References
nextStepFileFix: './step-02b-fix.md'
nextStepFileHumanCheckpoint: './step-03-human-checkpoint.md'
nextStepFileComplete: './step-04-complete.md'
configFile: '../config.yaml'
validationTask: '{project-root}/_bmad/my-custom-bmad/tasks/validate-adversarial-review.md'
delegationTemplatesFile: '../data/delegation-templates.md'
exitConditionsFile: '../data/exit-conditions.md'
aggregationRulesFile: '../data/aggregation-rules.md'

# Task References
subprocessDelegationSkill: '{project-root}/.claude/skills/subprocess-delegation/SKILL.md'
---

# Step 2: Review

## STEP GOAL:

To execute a single pass of the two-phase adversarial review (per-file + cross-file holistic), aggregate findings, and route based on severity: to fixes, human checkpoint, or completion.

## MANDATORY EXECUTION RULES (READ FIRST):

### Universal Rules:

- 📖 CRITICAL: Read the complete step file before taking any action
- ✅ YOU MUST ALWAYS SPEAK OUTPUT In your Agent communication style with the config `{communication_language}`
- ⚙️ TOOL/SUBPROCESS FALLBACK: If any instruction references a subprocess, subagent, or tool you do not have access to, you MUST still achieve the outcome in your main context thread

### Role Reinforcement:

- ✅ You are an Adversarial Review Automation Specialist
- ✅ We engage in collaborative dialogue, not command-response
- ✅ You bring expertise in automated quality review, severity-aware issue detection, automatic fix delegation, and workflow state management
- ✅ User brings their domain knowledge and specific document requirements
- ✅ Together we produce something better than we could on our own

### Step-Specific Rules:

- 🎯 This step runs ONE review pass — no internal loops
- 🚫 FORBIDDEN to loop within this step — routing handles iteration
- 💬 Operate silently during review — only report when routing decision is made
- 📋 Do NOT append to `stepsCompleted` during review/fix cycle — only when exiting to step-03 or step-04
- ⚙️ If subprocess unavailable: Execute tasks directly in main thread

## EXECUTION PROTOCOLS:

- 🎯 Follow the MANDATORY SEQUENCE exactly
- 💾 Update report file frontmatter with iteration results
- 📖 Append iteration summary to report after review
- 🚫 After completing review and routing, immediately load next step — do NOT halt for user input

## CONTEXT BOUNDARIES:

- Report file exists with frontmatter state tracking including `file_paths` array
- File paths available from step-01-init
- Review guidance available (optional, from step-01-init)
- Configuration loaded (max_iterations, models, validation_task)
- Previous iterations tracked in report frontmatter
- `force_fix` context variable may be set by step-03

## MANDATORY SEQUENCE

**CRITICAL:** Follow this sequence exactly. Do not skip, reorder, or improvise.

### 1. Load Configuration and State

Load workflow config from `{configFile}` and resolve model values (with context overrides).
Load report file frontmatter: `workflow_id`, `file_paths`, `review_guidance`, `iteration_count`, `max_iterations`, `current_status`.
Load aggregation rules from `{aggregationRulesFile}`.

Resolve report file path: `{report_output_folder_path}/adversarial-review-{workflow_id}.md`

Set state variables:
- `current_iteration` = `iteration_count` from frontmatter (or 0 if first run)
- `max_iterations_current` = sum of all values in `max_iterations` array (e.g., [5, 5, 3] → total limit is 13)
- `force_fix` = check context for `force_fix` flag (set by step-03; defaults to false)
- `auto_fix` = from report frontmatter (default: true)

### 2. Phase 1: Per-File Reviews

**Load subprocess delegation skill:** `{subprocessDelegationSkill}`
**Load delegation templates:** `{delegationTemplatesFile}`

For each file in `file_paths`:
- Execute per-file review using the "Per-File Review Delegation Template"
- Substitute: `{models.review}`, `{validationTask}`, `{file_path}`, `{review_guidance}`, `{project-root}`
- Collect result: outcome, severity counts, feedback, cross_file_refs

**If subprocess unavailable:** Load `{validationTask}` directly and review each file in main thread.

Store all per-file results for aggregation.

### 3. Phase 2: Cross-File Holistic Review

**IF file_paths contains only 1 file:** Skip Phase 2 entirely. Cross-file review is not applicable for single-file input. Set all cross-file counts to 0. Proceed directly to aggregation.

**IF file_paths contains 2+ files:**

Build per-file findings summary following rules in `{aggregationRulesFile}`.

Execute cross-file review using the "Cross-File Review Delegation Template":
- Substitute: `{models.review}`, `{validationTask}`, `{file_paths}`, `{per_file_summary}`, `{review_guidance}`, `{project-root}`
- Collect result: outcome, severity counts, feedback

**If subprocess unavailable:** Review all files together in main thread, focusing on cross-file concerns.

### 4. Aggregate Results

Follow aggregation rules from `{aggregationRulesFile}`:
- Compute `aggregate_critical`, `aggregate_high`, `aggregate_medium`, `aggregate_low`
- Determine `aggregate_outcome` (PASS or ISSUES_FOUND)

### 5. Append Iteration Summary to Report

Increment `current_iteration`. Append iteration summary to report body with:
- Per-file findings breakdown (file name, outcome, severity counts, key findings)
- Cross-file findings (affected files, severity counts, key findings) — if applicable
- Aggregate severity counts
- Update frontmatter: `iteration_count = current_iteration`, `lastStep = 'step-02-review'`

### 6. Route Based on Results

**Load exit conditions:** `{exitConditionsFile}`

**IF aggregate_outcome == PASS:**
- Update report frontmatter: `current_status = "passed"`
- Append 'step-02-review' to `stepsCompleted` array
- Load, read entire file, then execute `{nextStepFileComplete}`

**IF aggregate_outcome == ISSUES_FOUND:**

Evaluate:
- `has_critical_issues` = (aggregate_critical > 0)
- `has_fixable_issues` = (aggregate_critical > 0) OR (force_fix == true AND (aggregate_high > 0 OR aggregate_medium > 0 OR aggregate_low > 0))
- `within_iteration_limit` = (current_iteration < max_iterations_current)

**IF has_fixable_issues AND within_iteration_limit:**

- **IF auto_fix == true:**
  - Append action to iteration summary: "Routing to fix delegation (auto)"
  - Load, read entire file, then execute `{nextStepFileFix}`

- **IF auto_fix == false:**
  - Present findings summary to user:

    "**Review Complete (Iteration {current_iteration})**

    **Aggregate Severity:** Critical: {aggregate_critical} | High: {aggregate_high} | Medium: {aggregate_medium} | Low: {aggregate_low}

    {Brief summary of key findings}

    **Select an Option:**
    [F]ix issues - Proceed with fix delegation
    [S]kip - Accept current state and complete
    [Q]uit - Exit workflow"

  - **IF F:** Append action "User approved fix delegation", load and execute `{nextStepFileFix}`
  - **IF S:** Update `current_status = "accepted"`, append 'step-02-review' to `stepsCompleted`, load and execute `{nextStepFileComplete}`
  - **IF Q:** Update `current_status = "exited"`, append 'step-02-review' to `stepsCompleted`, load and execute `{nextStepFileComplete}`
  - **IF Any other:** Help user, redisplay menu

**IF NOT has_fixable_issues OR NOT within_iteration_limit:**

Check exit conditions in order:
1. aggregate_critical == 0 AND aggregate_high == 0 → Route to `{nextStepFileComplete}` (PASSED). Append 'step-02-review' to `stepsCompleted`.
2. aggregate_critical == 0 AND aggregate_high > 0 → Route to `{nextStepFileHumanCheckpoint}`. Append 'step-02-review' to `stepsCompleted`.
3. current_iteration >= max_iterations_current → Route to `{nextStepFileHumanCheckpoint}`. Append 'step-02-review' to `stepsCompleted`.

---

## 🚨 SYSTEM SUCCESS/FAILURE METRICS

**✅ SUCCESS:** Both review phases executed (or Phase 2 skipped for single file), findings aggregated, iteration summary appended, routing decision correct, next step loaded immediately.

**❌ SYSTEM FAILURE:** Halting for user input during review, looping internally, skipping a review phase, not aggregating, incorrect routing, not loading next step immediately.

**Master Rule:** This step runs ONE review pass and routes. No internal loops. No halting for user input. Execute and route.
