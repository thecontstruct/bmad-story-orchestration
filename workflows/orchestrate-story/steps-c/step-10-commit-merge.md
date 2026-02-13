---
name: 'step-10-commit-merge'
description: 'Execute git commit and merge operations'

# File References
nextStepFile: './step-11-summary.md'
configFile: '../config.yaml'
---

# Step 7: Commit & Merge

## STEP GOAL

Execute git operations: update story status, commit all changes, and merge to parent branch.

## MANDATORY EXECUTION RULES (READ FIRST)

### Universal Rules

- 📖 CRITICAL: Read the complete step file before taking any action
- 🔄 CRITICAL: When loading next step, ensure entire file is read first
- 🎯 This step executes git operations — be precise

### Step-Specific Rules

- 📝 Update story file AND sprint-status.yaml to `done` (before staging)
- 📊 Update workflow state file BEFORE any git operations (commit/merge)
- 💾 Commit ALL changes including story docs
- 🔀 Merge story branch to parent branch
- ✅ Verify each operation succeeded

---

## AVAILABLE STATE

From previous steps:

- `story_key`, `story_status`, `story_file`, `story_branch`, `parent_branch`
- `sprint_status` — Path to sprint-status.yaml
- `sub_workflows.*`, `models.*`, `max_retries` from config
- `review_outcome` — Result from code review step

---

## EXECUTION SEQUENCE

### 1. Update Story and Sprint Status

Update the story file:

a. Read `{{story_file}}` completely
b. Find the line starting with `Status:` and update to `Status: done`
c. Save the updated story file
d. Verify the change was made successfully

Update sprint-status.yaml:

a. Read `{{sprint_status}}` completely
b. Find the `development_status` key matching `{{story_key}}` and update its value to `done`
c. Save the updated sprint-status.yaml file, preserving ALL comments and structure including STATUS DEFINITIONS
d. Verify the change was made successfully

Report: "✅ Story file and sprint-status.yaml updated to done"

### 2. Update Workflow State File (Mark Workflow Complete)

Update `{workflow_state_file}` BEFORE any git operations to mark both step-07 and step-08 as complete (since step-08 is purely informational):
1. Load `{workflow_state_file}`
2. Append 'step-10-commit-merge' to `workflows.{story_key}.workflow_state.stepsCompleted` array
3. Append 'step-11-summary' to `workflows.{story_key}.workflow_state.stepsCompleted` array
4. Set `workflows.{story_key}.workflow_state.lastStep = 'step-11-summary'` (final step)
5. Update `workflows.{story_key}.execution_metadata.lastActivity = {current_timestamp}`
6. Update `workflows.{story_key}.domain_reference.storyStatus = 'done'`
7. Save file

Report: "✅ Workflow state file updated (workflow marked complete)"

### 3. Stage All Changes

Execute: `git add -A`

Verify staging: `git status`

Report files staged.

### 4. Generate Commit Message

Read `{{story_file}}` to extract:
- Story title
- Implementation summary from development section
- Review outcome

Construct comprehensive commit message:

```
Complete Story {{story_key}}: [Title]

[Implementation summary from story file]

Code Review: {{review_outcome}} ✅
- [Key accomplishments from story]
- [Files modified summary]

Story artifacts:
- {{story_file}}

🤖 Generated with BMAD Orchestration Workflow
```

### 5. Execute Commit

Execute commit with heredoc for message:

```bash
git commit -m "$(cat <<'EOF'
Complete Story {{story_key}}: [Title]

[Implementation summary]

Code Review: {{review_outcome}} ✅

Story artifacts:
- {{story_file}}

🤖 Generated with BMAD Orchestration Workflow
EOF
)"
```

Verify commit succeeded: `git log -1 --oneline`

Report: "✅ Changes committed to {{story_branch}}"

### 6. Checkout Parent Branch

Execute: `git checkout {{parent_branch}}`

Verify: `git branch --show-current`

IF checkout failed:
- Report error
- Ask user how to proceed

### 7. Merge Story Branch

Execute: `git merge {{story_branch}} -m "Merge story {{story_key}}: [title]"`

Verify merge succeeded.

IF merge conflict:
- Report: "⚠️ Merge conflict detected"
- Display conflict information
- Ask user: "Would you like to:
  - [R] Resolve conflicts manually (exit workflow)
  - [A] Abort merge and return to story branch"
- Handle user choice appropriately

Report: "✅ Story branch merged to {{parent_branch}}"

### 8. Report Success

```
✅ Step 7: Commit & Merge Complete

Story: {{story_key}}
Committed to: {{story_branch}}
Merged to: {{parent_branch}}
Status: done

Proceeding to summary...
```

### 9. Proceed to Summary

Load, read entire file, then execute {nextStepFile}

---

## SUCCESS METRICS

- Story status updated to `done`
- All changes committed
- Story branch merged to parent
- Sprint status updated
- Proceeded to summary step

## FAILURE MODES

- Not updating story status before commit
- Incomplete commit (missing files)
- Merge without verification
- Not handling merge conflicts
- Not updating sprint-status

**Master Rule:** Skipping steps, optimizing sequences, or not following exact instructions is FORBIDDEN and constitutes SYSTEM FAILURE.
