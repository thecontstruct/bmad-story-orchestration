---
description: 'Execute complete BMAD planning document workflow: create → review → merge with automated adversarial review validation and human checkpoint'
---

Load and execute the workflow at @_bmad-custom-src/workflows/orchestrate-planning-documents/workflow.md

Follow the INITIALIZATION SEQUENCE in workflow.md:

1. Load config from @_bmad-custom-src/workflows/orchestrate-planning-documents/config.yaml
2. Load bmb config from @_bmad/bmb/config.yaml for output_folder, user_name, communication_language
3. Load bmm config from @_bmad/bmm/config.yaml for planning_artifacts
4. Execute @_bmad-custom-src/workflows/orchestrate-planning-documents/steps-c/step-01-init.md to begin

The workflow will:
- Prompt for document type selection (PRD, Architecture, or Epic)
- Route to appropriate step based on document status in tracking files
- Execute sub-workflows (create-prd, create-architecture, create-epic)
- Perform adversarial review with retry loops
- Present human checkpoint before commit
- Commit and merge on approval
