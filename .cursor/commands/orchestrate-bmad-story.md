---
description: 'Execute complete BMAD story workflow: create → develop → review → merge with automated validation and human checkpoint'
---

Load and execute the workflow at @_bmad-custom-src/workflows/orchestrate-story/workflow.md

Follow the INITIALIZATION SEQUENCE in workflow.md:

1. Load config from @_bmad-custom-src/workflows/orchestrate-story/config.yaml
2. Load bmm config from @_bmad/bmm/config.yaml for implementation_artifacts
3. Execute @_bmad-custom-src/workflows/orchestrate-story/steps/step-01-init.md to begin

The workflow will:
- Identify the next story from sprint-status.yaml
- Route to appropriate step based on story status
- Execute sub-workflows (create-story, dev-story, code-review)
- Present human checkpoint before commit
- Commit and merge on approval

