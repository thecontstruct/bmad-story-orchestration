---
name: 'step-09-human-checkpoint'
description: 'Human review checkpoint before commit operations'

# File References
nextStepFile: './step-10-commit-merge.md'
configFile: '../config.yaml'
---

# Step 6: Human Checkpoint

## STEP GOAL

Present a human review checkpoint before committing changes. This is the single gate where a human reviews all changes before git operations.

## MANDATORY EXECUTION RULES (READ FIRST)

### Universal Rules

- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When loading next step, ensure entire file is read first
- ⏸️ CRITICAL: This step REQUIRES explicit user approval to proceed

### Step-Specific Rules

- 🛑 HALT and wait for user input — no auto-proceed
- 👀 Show summary of all changes
- 📋 Allow user to view details before deciding
- ✅ Only proceed with explicit user approval

---

## AVAILABLE STATE

From previous steps:

- `story_key`, `story_status`, `story_file`, `story_branch`, `parent_branch`
- `sprint_status` — Path to sprint-status.yaml
- `sub_workflows.*`, `models.*`, `max_retries` from config
- `review_outcome` — Result from code review step

---

## EXECUTION SEQUENCE

### 1. Gather Summary Information

Execute: `git diff {{parent_branch}}...{{story_branch}} --stat`

Store diff stats for display.

### 2. Display Phase Summary

```
═══════════════════════════════════════════════════════
HUMAN REVIEW CHECKPOINT: Before Commit
═══════════════════════════════════════════════════════

Story: {{story_key}}
Branch: {{story_branch}}
Parent: {{parent_branch}}

PHASE 1: Story Creation & Validation
✅ Story file created
✅ Story validated

PHASE 2: Development & Review  
✅ Implementation complete
✅ Code review: {{review_outcome}}

FILES MODIFIED:
{{diff_stats}}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 3. Present MENU OPTIONS

Display: "**Ready to commit and merge?** Review options:

- [C] **Continue** to commit phase
- [V] **View changes** (show full git diff)
- [S] **Story file** (view story document)
- [M] **Manual review** (preserve branch and exit)
- [A] **Abort** workflow"

#### Menu Handling Logic:

- IF C: Update workflow state file, then load, read entire file, then execute {nextStepFile}
- IF V: Execute `git diff {{parent_branch}}...{{story_branch}}`, display full diff output, then [Redisplay Menu Options](#3-present-menu-options)
- IF S: Read and display `{{story_file}}` contents, then [Redisplay Menu Options](#3-present-menu-options)
- IF M: Report manual review mode message, exit workflow
- IF A: Report workflow aborted message, exit workflow
- IF Any other: help user respond then [Redisplay Menu Options](#3-present-menu-options)

#### EXECUTION RULES:

- ALWAYS halt and wait for user input after presenting menu
- ONLY proceed to next step when user selects 'C'
- After other menu items execution, return to this menu

### 4. Handle User Choice

**IF user chose V (View changes):**

Execute: `git diff {{parent_branch}}...{{story_branch}}`

Display full diff output.

Return to Step 3 (redisplay options).

**IF user chose S (Story file):**

Read and display `{{story_file}}` contents.

Return to Step 3 (redisplay options).

**IF user chose M (Manual review):**

```
ℹ️ Manual Review Mode

Branch {{story_branch}} preserved for manual review.
Story file: {{story_file}}

To resume later:
1. Complete manual review
2. Run orchestrate-story workflow again
3. Workflow will detect story status and resume appropriately

Exiting workflow.
```

Exit workflow.

**IF user chose A (Abort):**

```
⚠️ Workflow Aborted

Branch {{story_branch}} preserved.
No changes committed.

Exiting workflow.
```

Exit workflow.

**IF user chose C (Continue):**

```
✅ Step 6: Human Checkpoint Approved

Proceeding to commit phase...
```

**Before proceeding, update workflow state file:**
1. Load `{workflow_state_file}`
2. Append 'step-09-human-checkpoint' to `workflows.{story_key}.workflow_state.stepsCompleted` array
3. Set `workflows.{story_key}.workflow_state.lastStep = 'step-09-human-checkpoint'`
4. Update `workflows.{story_key}.execution_metadata.lastActivity = {current_timestamp}`
5. Save file

Load, read entire file, then execute {nextStepFile}

---

## SUCCESS METRICS

- Summary displayed clearly
- User given all review options
- User explicitly approved continuation
- Respected user's abort/manual choices

## FAILURE MODES

- Auto-proceeding without user input
- Not showing diff stats
- Not offering view options
- Committing without explicit approval

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
