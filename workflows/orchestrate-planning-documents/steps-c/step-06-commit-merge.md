---
name: 'step-06-commit-merge'
description: 'Execute git commit and merge operations'

# File References
nextStepFile: './step-07-summary.md'
---

# Step 6: Commit & Merge

## STEP GOAL

Execute git operations: update document status, commit all changes, and merge to parent branch.

## MANDATORY EXECUTION RULES (READ FIRST)

### Universal Rules

- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When loading next step, ensure entire file is read first
- 🎯 This step executes git operations — be precise

### Step-Specific Rules

- 📝 Update document file frontmatter to `complete` (before staging)
- 📊 Update workflow state file BEFORE any git operations (commit/merge)
- 📊 Update tracking file BEFORE any git operations
- 💾 Commit ALL changes including document files
- 🔀 Merge document branch to parent branch
- ✅ Verify each operation succeeded

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

### 1. Update Document Status

Update the document file:

a. Read `{{document_file}}` completely
b. Find the line starting with `status:` or `Status:` (case-insensitive) in frontmatter and update to `status: complete` (or `Status: complete`)
c. If status field doesn't exist, add it to frontmatter
d. Save the updated document file
e. Verify the change was made successfully

Report: "✅ Document file updated to complete"

### 2. Update Workflow State File (Mark Workflow Complete)

Update `{workflow_state_file}` BEFORE any git operations to mark both step-06 and step-07 as complete (since step-07 is final summary):
1. Load `{workflow_state_file}`
2. Append 'step-06-commit-merge' to `workflows.{document_file_path}.workflow_state.stepsCompleted` array
3. Append 'step-07-summary' to `workflows.{document_file_path}.workflow_state.stepsCompleted` array
4. Set `workflows.{document_file_path}.workflow_state.lastStep = 'step-07-summary'` (final step)
5. Update `workflows.{document_file_path}.execution_metadata.lastActivity = {current_timestamp}`
6. Update `workflows.{document_file_path}.domain_reference.documentStatus = 'complete'`
7. Update `workflows.{document_file_path}.domain_reference.reviewStatus = 'passed'`
8. Save file

Report: "✅ Workflow state file updated (workflow marked complete)"

### 3. Update Tracking File

Update `{tracking_file}` BEFORE any git operations:
1. Load `{tracking_file}`
2. Append 'step-06-commit-merge' to `workflows.{document_file_path}.workflow_state.stepsCompleted` array
3. Append 'step-07-summary' to `workflows.{document_file_path}.workflow_state.stepsCompleted` array
4. Set `workflows.{document_file_path}.workflow_state.lastStep = 'step-07-summary'` (final step)
5. Update `workflows.{document_file_path}.execution_metadata.lastActivity = {current_timestamp}`
6. Update `workflows.{document_file_path}.domain_reference.documentStatus = 'complete'`
7. Update `workflows.{document_file_path}.domain_reference.reviewStatus = 'passed'`
8. Save file

Report: "✅ Tracking file updated (workflow marked complete)"

### 4. Stage All Changes

Execute: `git add -A`

Verify staging: `git status`

Report files staged.

### 5. Generate Commit Message

Read `{{document_file}}` to extract:
- Document title (from frontmatter or first heading)
- Document type and key information
- Review outcome

Construct comprehensive commit message:

```
Complete {{document_type}} Document: [Title]

Document Type: {{document_type}}
Document: {{document_file}}

Adversarial Review: {{review_outcome}} ✅
{{#if retry_count > 0}}Review attempts: {{retry_count}}{{/if}}

Document artifacts:
- {{document_file}}
{{#if tracking_file}}- Tracking file: {{tracking_file}}{{/if}}

🤖 Generated with BMAD Orchestration Workflow
```

### 6. Execute Commit

Execute commit with heredoc for message:

```bash
git commit -m "$(cat <<'EOF'
Complete {{document_type}} Document: [Title]

Document Type: {{document_type}}
Document: {{document_file}}

Adversarial Review: {{review_outcome}} ✅

Document artifacts:
- {{document_file}}

🤖 Generated with BMAD Orchestration Workflow
EOF
)"
```

Verify commit succeeded: `git log -1 --oneline`

Report: "✅ Changes committed to {{document_branch}}"

### 7. Checkout Parent Branch

Execute: `git checkout {{parent_branch}}`

Verify: `git branch --show-current`

IF checkout failed:
- Report error
- Ask user how to proceed

### 8. Merge Document Branch

Execute: `git merge {{document_branch}} -m "Merge {{document_type}} document: [title]"`

Verify merge succeeded.

IF merge conflict:
- Report: "⚠️ Merge conflict detected"
- Display conflict information
- Ask user: "Would you like to:
  - [R] Resolve conflicts manually (exit workflow)
  - [A] Abort merge and return to document branch"
- Handle user choice appropriately

Report: "✅ Document branch merged to {{parent_branch}}"

### 9. Report Success

```
✅ Step 6: Commit & Merge Complete

Document Type: {{document_type}}
Document: {{document_file}}
Committed to: {{document_branch}}
Merged to: {{parent_branch}}
Status: complete

Proceeding to summary...
```

### 10. Proceed to Summary

Load, read entire file, then execute {nextStepFile}

---

## SUCCESS METRICS

- Document status updated to `complete`
- All changes committed
- Document branch merged to parent
- Workflow state file updated
- Tracking file updated
- Proceeded to summary step

## FAILURE MODES

- Not updating document status before commit
- Incomplete commit (missing files)
- Merge without verification
- Not handling merge conflicts
- Not updating workflow state file
- Not updating tracking file

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
