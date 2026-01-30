# Delegation Block Templates

**Purpose:** Reusable delegation block templates for workflow steps.

## Code Review Delegation Block

```xml
<delegate model="{{models.code_review}}">
  <system-instructions>
    You are executing a BMAD workflow in isolated context.
    - Follow workflow instructions sequentially
    - Do not spawn additional agents or delegation blocks
    - CRITICAL: Execute in #yolo mode - skip all user prompts and auto-proceed through workflow.xml steps
    - When workflow.xml asks for user input (e.g., "Continue to next step?", template-output prompts), automatically proceed without waiting
    - Treat this as non-interactive execution - simulate expert user responses to proceed automatically
  </system-instructions>

  <context>
    - workflow_path: {{sub_workflows.code_review}}
    - project_root: {{project-root}}
    - story_file: {{story_file}}
  </context>

  <prompt>
    Execute the code-review workflow:
    1. Load {{sub_workflows.code_review}}/workflow.yaml
    2. Read and understand workflow configuration
    3. Load {project-root}/_bmad/core/tasks/workflow.xml
    4. Follow workflow.xml instructions to execute the code-review workflow (auto-proceed through all prompts)
    5. The workflow will discover the story from sprint-status.yaml
    6. Review implementation against story requirements
    7. Append review section to story file
  </prompt>

  <output>
    - Keep response concise (under 500 words)
    - outcome: APPROVE | CHANGES_REQUESTED | BLOCKED
    - feedback: detailed review feedback
    - files_reviewed: list of files reviewed
  </output>
</delegate>
```

## Fix Review Delegation Block

```xml
<delegate model="{{models.fix_review}}">
  <system-instructions>
    You are executing a BMAD workflow in isolated context.
    - CRITICAL: Read and address review feedback before proceeding
    - Follow workflow instructions sequentially
    - Do not spawn additional agents or delegation blocks
    - CRITICAL: Execute in #yolo mode - skip all user prompts and auto-proceed through workflow.xml steps
    - When workflow.xml asks for user input (e.g., "Continue to next step?", template-output prompts), automatically proceed without waiting
    - Treat this as non-interactive execution - simulate expert user responses to proceed automatically
  </system-instructions>

  <context>
    - workflow_path: {{sub_workflows.dev_story}}
    - project_root: {{project-root}}
    - story_file: {{story_file}}
    - review_feedback: {{review_feedback}}
  </context>

  <prompt>
    Execute development workflow to fix review issues:
    1. Review feedback and identify required changes
    2. Load {{sub_workflows.dev_story}}/workflow.yaml
    3. Follow workflow.xml instructions to execute the dev-story workflow (auto-proceed through all prompts)
    4. Implement fixes for all review issues
    5. Update story file with changes made
  </prompt>

  <output>
    - Keep response concise (under 500 words)
    - outcome: success | failure
    - fixes_applied: list of changes made
    - issues: any problems encountered
  </output>
</delegate>
```
