---
name: 'step-02b-fix'
description: 'Delegate context-aware fixes for per-file and cross-file issues, then route back to review'

# File References
nextStepFileReview: './step-02-review.md'
configFile: '../config.yaml'
delegationTemplatesFile: '../data/delegation-templates.md'
aggregationRulesFile: '../data/aggregation-rules.md'

# Task References
subprocessDelegationSkill: '{project-root}/.claude/skills/subprocess-delegation/SKILL.md'
---

# Step 2b: Fix Delegation

## STEP GOAL:

To delegate context-aware fixes (single-file for per-file issues, cross-file for cross-file issues), verify fixes were applied, and route back to review for re-validation.

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

- 🎯 This step runs ONE fix pass — no internal loops
- 🚫 FORBIDDEN to loop within this step — routing handles iteration
- 💬 Operate silently during fixes — only report when routing back to review
- 📋 Do NOT append to `stepsCompleted` — this step is part of the review/fix cycle
- ⚙️ If subprocess unavailable: Apply fixes directly in main thread

## EXECUTION PROTOCOLS:

- 🎯 Follow the MANDATORY SEQUENCE exactly
- 💾 Update report file frontmatter with fix results
- 📖 Append fix summary to current iteration in report
- 🚫 After completing fixes, immediately load next step — do NOT halt for user input

## CONTEXT BOUNDARIES:

- Review results from step-02-review are available in context (per-file findings, cross-file findings)
- Report file exists with current iteration state
- File paths and review guidance available
- Configuration loaded (models, validation_task)
- `force_fix` context variable may be set (cleared after this step)

## MANDATORY SEQUENCE

**CRITICAL:** Follow this sequence exactly. Do not skip, reorder, or improvise.

### 1. Load Configuration and Context

Load workflow config from `{configFile}` and resolve fix model value (with context overrides).
Load report file frontmatter for context: `workflow_id`, `file_paths`.
Load aggregation rules from `{aggregationRulesFile}` for fix routing rules.

Resolve report file path: `{report_output_folder_path}/adversarial-review-{workflow_id}.md`

### 2. Identify Fixes Needed

From the review results (carried in context from step-02-review):

**Per-file issues:** For each file with findings, collect:
- File path
- Per-file feedback (all findings for that file)

**Cross-file issues:** If cross-file findings exist, collect:
- Affected file paths
- Cross-file feedback

### 3. Delegate Single-File Fixes

**Load subprocess delegation skill:** `{subprocessDelegationSkill}`
**Load delegation templates:** `{delegationTemplatesFile}`

Follow fix routing rules from `{aggregationRulesFile}`:

For each file with per-file issues:
- Delegate using "Single-File Fix Delegation Template"
- Substitute: `{models.fix}`, `{file_path}`, `{feedback}`, `{validation_report_path}`, `{project-root}`
- Collect result: outcome, fixes_applied, issues

**If subprocess unavailable:** Apply fixes directly in main thread.

### 4. Delegate Cross-File Fixes

**IF cross-file issues exist:**

Delegate using "Cross-File Fix Delegation Template":
- Substitute: `{models.fix}`, `{file_paths}`, `{cross_file_feedback}`, `{validation_report_path}`, `{project-root}`
- Collect result: outcome, files_modified, fixes_applied, issues

**If subprocess unavailable:** Apply cross-file fixes directly in main thread.

**IF no cross-file issues:** Skip this section.

### 5. Verify and Record Fixes

- Verify files were updated (check modification or delegation success)
- Append fix summary to report body under current iteration:
  - Single-file fixes applied (per file)
  - Cross-file fixes applied (if any)
  - Any issues encountered during fixing

### 6. Clear Force Fix and Route to Review

- **Clear `force_fix`**: Set `force_fix = false` (revert to standard critical-only logic for next review pass)
- Update report frontmatter: `lastStep = 'step-02b-fix'`
- Load, read entire file, then execute `{nextStepFileReview}` for re-validation

---

## 🚨 SYSTEM SUCCESS/FAILURE METRICS

**✅ SUCCESS:** Single-file fixes delegated for per-file issues, cross-file fixes delegated for cross-file issues (in correct order), fixes verified, force_fix cleared, routed back to review immediately.

**❌ SYSTEM FAILURE:** Halting for user input, looping internally, not delegating fixes, not clearing force_fix, not routing back to review, applying cross-file fixes before single-file fixes.

**Master Rule:** This step runs ONE fix pass and routes back to review. No internal loops. No halting for user input. Execute and route.
