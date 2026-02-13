---
name: 'step-05-human-checkpoint'
description: 'Human review checkpoint before commit operations'

# File References
nextStepFile: './step-06-commit-merge.md'
---

# Step 5: Human Checkpoint

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

- `document_type` — Document type ('prd' | 'architecture' | 'epic')
- `document_file` — Path to document markdown file
- `document_branch` — Git branch for this document
- `parent_branch` — Branch to merge back to
- `tracking_file` — Path to orchestrate-{doc-type}-state.yaml
- `workflow_state_file` — Path to orchestrator workflow state file
- `document_status` — Current status from document frontmatter
- `review_outcome` — Result from adversarial review step (PASS or MAX_RETRIES_REACHED)
- `retry_count` — Number of retry attempts from adversarial review
- `sub_workflows.*`, `models.*` from config

---

## EXECUTION SEQUENCE

### 1. Gather Summary Information

Execute: `git diff {{parent_branch}}...{{document_branch}} --stat`

Store diff stats for display.

### 2. Display Phase Summary

```
═══════════════════════════════════════════════════════
HUMAN REVIEW CHECKPOINT: Before Commit
═══════════════════════════════════════════════════════

Document Type: {{document_type}}
Document: {{document_file}}
Branch: {{document_branch}}
Parent: {{parent_branch}}

PHASE 1: Document Creation
✅ {{document_type}} document created
✅ Document status: {{document_status}}

PHASE 2: Quality Gate
✅ Adversarial review: {{review_outcome}}
{{#if retry_count > 0}}Review attempts: {{retry_count}}{{/if}}

FILES MODIFIED:
{{diff_stats}}

━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

### 3. Present MENU OPTIONS

Display: "**Ready to commit and merge?** Review options:

- [C] **Continue** to commit phase
- [V] **View changes** (show full git diff)
- [D] **View document** (view {{document_type}} document)
- [M] **Manual review** (preserve branch and exit)
- [A] **Abort** workflow"

#### Menu Handling Logic:

- IF C: Update workflow state file and tracking file, then load, read entire file, then execute {nextStepFile}
- IF V: Execute `git diff {{parent_branch}}...{{document_branch}}`, display full diff output, then [Redisplay Menu Options](#3-present-menu-options)
- IF D: Read and display `{{document_file}}` contents, then [Redisplay Menu Options](#3-present-menu-options)
- IF M: Report manual review mode message, exit workflow
- IF A: Report workflow aborted message, exit workflow
- IF Any other: help user respond then [Redisplay Menu Options](#3-present-menu-options)

#### EXECUTION RULES:

- ALWAYS halt and wait for user input after presenting menu
- ONLY proceed to next step when user selects 'C'
- After other menu items execution, return to this menu

### 4. Handle User Choice

**IF user chose V (View changes):**

Execute: `git diff {{parent_branch}}...{{document_branch}}`

Display full diff output.

Return to Step 3 (redisplay options).

**IF user chose D (View document):**

Read and display `{{document_file}}` contents.

Return to Step 3 (redisplay options).

**IF user chose M (Manual review):**

```
ℹ️ Manual Review Mode

Branch {{document_branch}} preserved for manual review.
Document file: {{document_file}}

To resume later:
1. Complete manual review
2. Run orchestrate-planning-documents workflow again
3. Workflow will detect document status and resume appropriately

Exiting workflow.
```

Exit workflow.

**IF user chose A (Abort):**

```
⚠️ Workflow Aborted

Branch {{document_branch}} preserved.
No changes committed.

Exiting workflow.
```

Exit workflow.

**IF user chose C (Continue):**

```
✅ Step 5: Human Checkpoint Approved

Proceeding to commit phase...
```

**Before proceeding, update workflow state file:**
1. Load `{workflow_state_file}`
2. Append 'step-05-human-checkpoint' to `workflows.{document_file_path}.workflow_state.stepsCompleted` array
3. Set `workflows.{document_file_path}.workflow_state.lastStep = 'step-05-human-checkpoint'`
4. Update `workflows.{document_file_path}.execution_metadata.lastActivity = {current_timestamp}`
5. Save file

**Update tracking file:**
1. Load `{tracking_file}`
2. Append 'step-05-human-checkpoint' to `workflows.{document_file_path}.workflow_state.stepsCompleted` array
3. Set `workflows.{document_file_path}.workflow_state.lastStep = 'step-05-human-checkpoint'`
4. Update `workflows.{document_file_path}.execution_metadata.lastActivity = {current_timestamp}`
5. Save file

Load, read entire file, then execute {nextStepFile}

---

## SUCCESS METRICS

- Summary displayed clearly
- User given all review options
- User explicitly approved continuation
- Respected user's abort/manual choices
- Workflow state file updated
- Tracking file updated

## FAILURE MODES

- Auto-proceeding without user input
- Not showing diff stats
- Not offering view options
- Committing without explicit approval
- Not updating workflow state file
- Not updating tracking file

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
