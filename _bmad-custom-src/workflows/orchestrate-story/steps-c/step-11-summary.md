---
name: 'step-11-summary'
description: 'Display final workflow summary and identify next story'

# File References
configFile: '../config.yaml'
---

# Step 8: Summary

## STEP GOAL

Display comprehensive workflow summary and identify the next story in the backlog.

## MANDATORY EXECUTION RULES (READ FIRST)

### Universal Rules

- 📖 CRITICAL: Read the complete step file before taking any action
- 🎯 This is the final step — workflow ends after summary

### Step-Specific Rules

- 📊 Display complete summary of all phases
- 🔍 Identify next backlog story
- 🏁 End workflow gracefully
- 📝 Note: Workflow state file was already updated in step-07 (before merge) - this step is read-only

---

## AVAILABLE STATE

From previous steps:

- `story_key`, `story_status`, `story_file`, `story_branch`, `parent_branch`
- `sprint_status` — Path to sprint-status.yaml
- `sub_workflows.*`, `models.*`, `max_retries` from config
- `review_outcome` — Result from code review step

---

## EXECUTION SEQUENCE

### 1. Gather Summary Data

Read `{{sprint_status}}` to:
- Confirm story status is `done`
- Find next story in backlog

Execute: `git diff {{parent_branch}}~1...{{parent_branch}} --stat` to get merged changes summary.

### 2. Display Final Summary

```
═══════════════════════════════════════════════════════
🎉 BMAD Story Workflow Complete
═══════════════════════════════════════════════════════

Story: {{story_key}}
Status: done ✅

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

PHASE 1: Story Creation & Validation
✅ Story file created
✅ Validation passed

PHASE 2: Development & Review
✅ Implementation complete
✅ Code review: {{review_outcome}}

PHASE 3: Commit & Merge
✅ Story status updated to done
✅ All changes committed
✅ Merged to {{parent_branch}}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

FILES MODIFIED:
{{diff_stats}}

STORY ARTIFACTS:
- Story file: {{story_file}}

BRANCHES:
- Story branch: {{story_branch}} (preserved locally)
- Current branch: {{parent_branch}}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━

NEXT STORY: {{next_story_key || "No stories in backlog"}}

═══════════════════════════════════════════════════════
```

**Note:** The workflow state file was already updated in step-07 (before merge) to mark both step-07 and step-08 as complete. This ensures all state changes are included in the commit.

### 3. Workflow Complete

Report: "Story orchestration workflow complete. Run again to process the next story."

---

## SUCCESS METRICS

- Complete summary displayed
- All phases listed with outcomes
- Files modified listed
- Next story identified
- Workflow ended gracefully

## FAILURE MODES

- Incomplete summary
- Not identifying next story
- Not displaying artifacts

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
