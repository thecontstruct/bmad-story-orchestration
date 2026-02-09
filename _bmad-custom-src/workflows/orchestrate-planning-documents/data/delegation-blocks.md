# Sub-Workflow Delegation Blocks

**Purpose:** Reference for invoking sub-workflows via delegation blocks.

## Pattern

When invoking sub-workflows, use the `<delegate>` block pattern:

```xml
<delegate>
  <workflow>{sub_workflow_path}</workflow>
  <model>{model_name}</model>
  <context>
    {context_variables}
  </context>
</delegate>
```

## Examples

### Create PRD Workflow

```xml
<delegate>
  <workflow>{sub_workflows.create_prd}</workflow>
  <model>{models.create_prd}</model>
  <context>
    document_type: prd
    output_path: {document_file}
  </context>
</delegate>
```

### Create Architecture Workflow

```xml
<delegate>
  <workflow>{sub_workflows.create_architecture}</workflow>
  <model>{models.create_architecture}</model>
  <context>
    document_type: architecture
    output_path: {document_file}
  </context>
</delegate>
```

### Create Epic Workflow

```xml
<delegate>
  <workflow>{sub_workflows.create_epic}</workflow>
  <model>{models.create_epic}</model>
  <context>
    document_type: epic
    output_path: {document_file}
  </context>
</delegate>
```
