# Workflow State File Structure

**Purpose:** Reference for workflow state file initialization structure.

## Empty State File Structure

```yaml
# Orchestrate Planning Documents Workflow State
workflows: {}
```

## New Document Entry Structure

```yaml
workflows:
  {document_file_path}:  # e.g., "planning/prd-2026-02-03.md" (full relative path)
    workflow_state:
      stepsCompleted: []
      lastStep: ''
      lastContinued: ''
      workflowVersion: 'v6'
      retryCount: 0  # Track retry attempts during fixing state
    execution_metadata:
      parentBranch: ''
      documentBranch: '{doc-type}/{document-key}'  # e.g., "prd/prd-2026-02-03"
      documentFile: '{full_path_to_document}'
      documentType: 'prd' | 'architecture' | 'epic'
      startedAt: '{current_timestamp}'  # ISO 8601 format (e.g., 2026-02-03T02:47:37Z)
      lastActivity: '{current_timestamp}'
    domain_reference:
      documentStatus: 'draft' | 'complete'  # from document frontmatter
      reviewStatus: 'pending' | 'in-fix' | 'passed' | 'max-retries'
```
