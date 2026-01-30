# Orchestrate Story - Validation Checklist

## Step 1: Initialization

### Pre-Flight Validation
- [ ] Implementation artifacts directory ({{implementation_artifacts}}) exists
- [ ] Sprint status file ({{sprint_status}}) exists and is readable
- [ ] Story identified from {{sprint_status}}
- [ ] Story key extracted correctly
- [ ] File paths constructed correctly (story_dir: {{story_dir}}, story_file: {{story_file}})
- [ ] Parent branch ({{parent_branch}}) identified and stored

### Branch Setup
- [ ] Working tree clean (or user resolved conflicts)
- [ ] Story branch ({{story_branch}}) created or switched to successfully
- [ ] Branch name follows format: story/{{story_key}}
- [ ] Currently on story branch ({{story_branch}})
- [ ] Parent branch ({{parent_branch}}) stored for later merge

### State Routing
- [ ] Story status ({{story_status}}) correctly identified
- [ ] Routed to appropriate step based on {{story_status}}

## Step 2: Story Creation

### Create-Story Workflow
- [ ] Create-story workflow ({{sub_workflows.create_story}}) invoked successfully
- [ ] Model used: {{models.create_story}}
- [ ] Story file ({{story_file}}) exists at expected path
- [ ] Story file ({{story_file}}) contains all required sections
- [ ] Sprint status ({{sprint_status}}) updated to "drafted" for story {{story_key}}

## Step 3: Story Validation

### Validation Execution
- [ ] Validation task ({{validation_task}}) executed against checklist
- [ ] Validation checklist ({{sub_workflows.create_story}}/checklist.md) used
- [ ] Retry task ({{retry_task}}) used for automated retry loop
- [ ] Fix model used: {{models.fix_validation}}
- [ ] Validation issues addressed (if any)
- [ ] Retry loop handled correctly (attempt {{retry_count}}/{{max_retries}})
- [ ] Validation feedback ({{feedback}}) captured and used for fixes when retrying
- [ ] User decisions honored (reset/manual/skip)
- [ ] Validation outcome recorded
- [ ] Sprint status ({{sprint_status}}) updated to "ready-for-dev" on pass for story {{story_key}}

## Step 4: Development

### Dev-Story Workflow
- [ ] Dev-story workflow ({{sub_workflows.dev_story}}) invoked successfully
- [ ] Model used: {{models.dev_story}}
- [ ] Implementation files modified
- [ ] Story file ({{story_file}}) updated with development section
- [ ] Sprint status ({{sprint_status}}) updated to "review" for story {{story_key}}

## Step 5: Code Review

### Review Execution
- [ ] Code-review workflow ({{sub_workflows.code_review}}) invoked successfully
- [ ] Review model used: {{models.code_review}}
- [ ] Retry task ({{retry_task}}) used for automated retry loop
- [ ] Fix model used: {{models.fix_review}}
- [ ] Review section appended to story file ({{story_file}})
- [ ] Review outcome ({{review_outcome}}) parsed correctly (APPROVE/CHANGES_REQUESTED/BLOCKED)
- [ ] Retry loop handled correctly (attempt {{review_retry_count}}/{{max_retries}})
- [ ] Review feedback ({{review_feedback}}) captured and used for fixes when retrying
- [ ] User decisions honored (reset/manual/continue)
- [ ] Review outcome ({{review_outcome}}) recorded

## Step 6: Human Checkpoint

### Review Gate
- [ ] Phase summary displayed to user for story {{story_key}}
- [ ] Diff stats shown to user
- [ ] Changes available for review (git diff {{parent_branch}}...{{story_branch}})
- [ ] Story file ({{story_file}}) available for review
- [ ] Review outcome ({{review_outcome}}) displayed
- [ ] User explicitly approved to proceed to commit
- [ ] Abort/Manual options respected if chosen

## Step 7: Commit & Merge

### Update Status
- [ ] Story file ({{story_file}}) status updated to "done" for story {{story_key}}
- [ ] Status change verified in {{story_file}}
- [ ] Sprint status ({{sprint_status}}) updated to "done" for story {{story_key}}

### Commit
- [ ] All changes staged (git add -A)
- [ ] Story file ({{story_file}}) included in commit
- [ ] Comprehensive commit message generated for story {{story_key}}
- [ ] Commit created successfully on branch {{story_branch}}

### Merge
- [ ] Checked out parent branch ({{parent_branch}}) successfully
- [ ] Verified on correct parent branch ({{parent_branch}})
- [ ] Merge executed successfully: {{story_branch}} → {{parent_branch}}
- [ ] Merge completed without conflicts
- [ ] Currently on parent branch ({{parent_branch}}) after merge
- [ ] Sprint status ({{sprint_status}}) updated to "done" for story {{story_key}}

## Step 8: Summary

### Final Summary
- [ ] Complete summary displayed for story {{story_key}}
- [ ] All phases listed with outcomes
- [ ] Review outcome ({{review_outcome}}) included in summary
- [ ] Files modified listed
- [ ] Story artifacts listed ({{story_file}}, {{story_dir}})
- [ ] Branch status reported ({{story_branch}} → {{parent_branch}})
- [ ] Next story identified from {{sprint_status}}
- [ ] Next story key ({{next_story_key}}) displayed or "No stories in backlog" message shown

## Error Handling

- [ ] Git conflicts handled with user input
- [ ] Validation retry logic works correctly
- [ ] Review retry logic works correctly
- [ ] State preserved on abort at any step ({{workflow_state_file}})
- [ ] User can exit for manual work at checkpoints
- [ ] Story branch ({{story_branch}}) preserved locally on exit

## State Preservation

- [ ] Sprint status file ({{sprint_status}}) updated correctly at each step
- [ ] Story file ({{story_file}}) preserved throughout workflow
- [ ] Story branch ({{story_branch}}) preserved locally
- [ ] Parent branch ({{parent_branch}}) tracked correctly
- [ ] Workflow state file ({{workflow_state_file}}) updated correctly
- [ ] All validation/review outcomes recorded
