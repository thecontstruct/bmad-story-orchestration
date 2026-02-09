---
name: 'step-07-summary'
description: 'Display final workflow summary'
---

# Step 7: Summary

## STEP GOAL

Display comprehensive workflow summary showing completion of the planning document workflow.

## MANDATORY EXECUTION RULES (READ FIRST)

### Universal Rules

- 📖 CRITICAL: Read the complete step file before taking any action
- 🎯 This is the final step — workflow ends after summary

### Step-Specific Rules

- 📊 Display complete summary of all phases
- 🏁 End workflow gracefully
- 📝 Note: Workflow state file and tracking file were already updated in step-06 (before merge) - this step is read-only

---

## AVAILABLE STATE

From previous steps:

- `document_type` — Document type ('prd' | 'architecture' | 'epic')
- `document_file` — Path to document markdown file
- `document_branch` — Git branch for this document
- `parent_branch` — Branch to merge back to
- `tracking_file` — Path to orchestrate-{doc-type}-state.yaml
- `workflow_state_file` — Path to orchestrator workflow state file
- `document_status` — Current status (should be 'complete')
- `review_outcome` — Result from adversarial review step (PASS or MAX_RETRIES_REACHED)
- `retry_count` — Number of retry attempts from adversarial review
- `sub_workflows.*`, `models.*` from config

---

## EXECUTION SEQUENCE

### 1. Gather Summary Data

Execute: `git diff {{parent_branch}}~1...{{parent_branch}} --stat` to get merged changes summary.

Read `{{document_file}}` to confirm status is `complete`.

### 2. Display Final Summary

```
═══════════════════════════════════════════════════════
🎉 BMAD Planning Document Workflow Complete
═══════════════════════════════════════════════════════

Document Type: {{document_type}}
Document: {{document_file}}
Status: complete ✅

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PHASE 1: Document Creation
✅ {{document_type}} document created
✅ Document status: {{document_status}}

PHASE 2: Quality Gate
✅ Adversarial review: {{review_outcome}}
{{#if retry_count > 0}}Review attempts: {{retry_count}}{{/if}}

PHASE 3: Commit & Merge
✅ Document status updated to complete
✅ All changes committed
✅ Merged to {{parent_branch}}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FILES MODIFIED:
{{diff_stats}}

DOCUMENT ARTIFACTS:
- Document file: {{document_file}}
{{#if tracking_file}}- Tracking file: {{tracking_file}}{{/if}}

BRANCHES:
- Document branch: {{document_branch}} (preserved locally)
- Current branch: {{parent_branch}}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

NEXT STEPS:
- Document is ready for use
- Run workflow again to create another document type
- Tracking file preserved for audit purposes

═══════════════════════════════════════════════════════
```

**Note:** The workflow state file and tracking file were already updated in step-06 (before merge) to mark both step-06 and step-07 as complete. This ensures all state changes are included in the commit.

### 3. Workflow Complete

Report: "Planning document orchestration workflow complete. Document {{document_file}} is ready for use. Run the workflow again to create another document type (PRD, Architecture, or Epic)."

---

## SUCCESS METRICS

- Complete summary displayed
- All phases listed with outcomes
- Files modified listed
- Document artifacts displayed
- Workflow ended gracefully

## FAILURE MODES

- Incomplete summary
- Not displaying artifacts
- Not showing all phases

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
