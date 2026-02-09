---
name: 'step-04-complete'
description: 'Generate final status report and complete workflow'

# File References (No nextStepFile - this is the final step)
---

# Step 4: Complete

## STEP GOAL:

To generate the final status report, mark the workflow as complete, and provide a summary of the multi-file adversarial review process.

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

- 🎯 Focus ONLY on finalizing the report and completing the workflow
- 🚫 FORBIDDEN to proceed to any next step (this is the final step)
- 💬 Provide clear summary of what was accomplished
- 🚪 Mark workflow as complete in frontmatter

## EXECUTION PROTOCOLS:

- 🎯 Follow the MANDATORY SEQUENCE exactly
- 💾 Update report file with final status and summary
- 📖 Mark workflow complete in frontmatter
- 🚫 This is the final step - workflow ends here

## CONTEXT BOUNDARIES:

- Report file exists with all iteration summaries
- Current status available from frontmatter
- Iteration count and aggregate severity counts available
- All previous steps completed
- file_paths array tracks all files reviewed

## MANDATORY SEQUENCE

**CRITICAL:** Follow this sequence exactly. Do not skip, reorder, or improvise unless user explicitly requests a change.

### 1. Load State and Context

Load report file and read frontmatter:
- `workflow_id` - Unique ID for this workflow run
- `input_type` - How files were provided
- `input_source` - Folder path or "explicit-list"
- `file_paths` - Array of files reviewed
- `review_guidance` - Optional guidance provided
- `iteration_count` - Total iterations completed
- `max_iterations` - Array tracking iteration limits
- `current_status` - Current status (passed/accepted/exited/manual-fix)
- `stepsCompleted` - Array of completed steps

Resolve report file path: `{report_output_folder_path}/adversarial-review-{workflow_id}.md`

### 2. Determine Final Status

Based on `current_status` from frontmatter:

**IF current_status == "passed":**
- Final status: **PASSED**
- Status description: "All critical and high issues have been fixed across all files."

**IF current_status == "accepted":**
- Final status: **ACCEPTED**
- Status description: "Critical issues fixed. High-severity issues remain, but user chose to proceed."

**IF current_status == "exited":**
- Final status: **EXITED**
- Status description: "Maximum iterations reached. Workflow exited per your request."

**IF current_status == "manual-fix":**
- Final status: **MANUAL_FIX**
- Status description: "Workflow paused for manual fixes. State preserved for resumption."

### 3. Get Final Issue Counts

From the last iteration summary in the report, extract final aggregate counts:
- `final_critical_count` - Final aggregate critical count
- `final_high_count` - Final aggregate high count (per-file + cross-file)
- `final_medium_count` - Final aggregate medium count
- `final_low_count` - Final aggregate low count

If no iteration summaries exist, set all counts to 0.

### 4. Generate Final Summary Section

Append final summary section to report file:

```markdown
## Final Status

**Workflow Status:** {final_status}  
**Status Description:** {status_description}

**Files Reviewed:** {file_count}
**Input:** {input_source}

**Summary:**
- **Total Iterations:** {iteration_count}
- **Max Iterations:** {max_iterations_current} (extended: {max_iterations_array})
- **Aggregate Issue Counts:**
  - Critical: {final_critical_count}
  - High: {final_high_count}
  - Medium: {final_medium_count}
  - Low: {final_low_count}

**Files:**
{numbered list of file_paths}

**Report File:** {report_file_path}
**Completed:** {current_date}
```

### 5. Update Report Frontmatter

Update report file frontmatter:

```yaml
---
workflow_id: "{workflow_id}"
workflow_name: "adversarial-auto-fix"
input_type: "{input_type}"
input_source: "{input_source}"
file_paths: [{file_paths_array}]
review_guidance: "{review_guidance}"
stepsCompleted: ['step-01-init', 'step-02-review-loop', 'step-03-human-checkpoint', 'step-04-complete']
lastStep: 'step-04-complete'
iteration_count: {iteration_count}
max_iterations: {max_iterations_array}
current_status: "{final_status}"
created: "{created_date}"
completed: "{current_date}"
user_name: "{user_name}"
---
```

### 6. Display Final Summary

Present final summary to user:

```
═══════════════════════════════════════════════════════
Adversarial Review Workflow Complete
═══════════════════════════════════════════════════════

**Workflow ID:** {workflow_id}
**Input:** {input_source} ({file_count} files)
**Status:** {final_status}

**Summary:**
- Total Iterations: {iteration_count}
- Final Status: {final_status}
- Aggregate Issue Counts:
  - Critical: {final_critical_count}
  - High: {final_high_count}
  - Medium: {final_medium_count}
  - Low: {final_low_count}

**Files Reviewed:**
{numbered list of file_paths}

**Report File:** {report_file_path}

{status_description}

═══════════════════════════════════════════════════════
```

### 7. Workflow Complete

**This workflow is now complete.**

No further steps. The workflow has finished successfully.

---

## 🚨 SYSTEM SUCCESS/FAILURE METRICS

### ✅ SUCCESS:

- Final status determined correctly
- Report file updated with multi-file final summary
- Frontmatter marked as complete with file_paths preserved
- Final summary displayed with file list and aggregate counts
- Workflow ends cleanly

### ❌ SYSTEM FAILURE:

- Not determining final status correctly
- Not including file list in final summary
- Not updating report file with final summary
- Not marking workflow as complete
- Attempting to load a next step (this is final)
- Not displaying final summary

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.

## CRITICAL STEP COMPLETION NOTE

This is the FINAL step. The workflow ends here. Do NOT attempt to load any next step file.
