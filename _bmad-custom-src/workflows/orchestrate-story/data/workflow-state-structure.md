# Workflow State File Structure

**Purpose:** Reference for workflow state file initialization structure.

## Empty State File Structure

```yaml
# Orchestrate Story Workflow State
workflows: {}
```

## New Story Entry Structure

```yaml
workflows:
  {story_key}:
    workflow_state:
      stepsCompleted: []
      lastStep: ''
      lastContinued: ''
      workflowVersion: 'v6'
    execution_metadata:
      parentBranch: ''
      storyBranch: ''
      startedAt: '{current_timestamp}'  # ISO 8601 format (e.g., 2026-01-26T22:46:28Z)
      lastActivity: '{current_timestamp}'
    domain_reference:
      storyStatus: '{story_status}'
      storyFile: '{story_file}'
```
