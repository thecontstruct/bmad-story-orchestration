# BMAD Story Orchestration

A standalone repository containing the BMAD story orchestration workflow and all its dependencies. This repository includes custom workflows, tasks, skills, agents, and commands needed to execute the complete story lifecycle automation.

## Repository Structure

```
bmad-story-orchestration/
├── _bmad-custom-src/              # Core BMAD custom content
│   ├── custom.yaml                # BMAD module configuration
│   ├── tasks/                     # Custom tasks
│   │   ├── retry-with-feedback.md
│   │   ├── validate-adversarial-review.md
│   │   └── validate-workflow.xml
│   └── workflows/
│       └── orchestrate-story/     # Main orchestration workflow
│           ├── config.yaml
│           ├── workflow.md
│           ├── checklist.md
│           ├── README.md
│           ├── steps-c/           # Workflow steps
│           └── data/              # Workflow data files
├── .cursor/
│   ├── skills/                    # Cursor skills for subprocess delegation
│   │   ├── subprocess-delegation/
│   │   ├── claude-cli/
│   │   ├── cursor-agent-cli/
│   │   └── cursor-task-tool/
│   └── agents/                    # Cursor agents for delegation
│       ├── delegate-composer-1.md
│       ├── delegate-opus-4.5-thinking.md
│       ├── delegate-gemini-3-pro.md
│       ├── delegate-gpt-5.2.md
│       └── code-simplifier.md
└── .claude/
    └── commands/                  # Claude commands
        └── orchestrate-bmad-story.md
```

## What's Included

### Core Content (`_bmad-custom-src/`)

- **custom.yaml**: BMAD module configuration defining workflows and tasks
- **tasks/**: Custom tasks for retry logic, validation, and adversarial review
- **workflows/orchestrate-story/**: Complete workflow orchestrating story lifecycle:
  - Story creation
  - Adversarial story review
  - Story validation
  - Story development
  - Adversarial code review
  - Code review
  - Human checkpoint
  - Commit and merge

### Cursor Skills (`.cursor/skills/`)

Skills for subprocess delegation and agent routing:

- **subprocess-delegation/**: Router skill that detects delegation patterns and routes to appropriate execution tool
- **claude-cli/**: Skill for executing via Anthropic's Claude CLI
- **cursor-agent-cli/**: Skill for executing via Cursor Agent CLI
- **cursor-task-tool/**: Skill for executing via Cursor's native Task tool

### Cursor Agents (`.cursor/agents/`)

Agent definitions for delegated execution:

- **delegate-composer-1.md**: Fast execution agent for simple/repetitive tasks
- **delegate-opus-4.5-thinking.md**: Architecture, planning, complex implementation agent
- **delegate-gemini-3-pro.md**: Code review agent
- **delegate-gpt-5.2.md**: Analysis and evaluation agent
- **code-simplifier.md**: Code simplification and refinement agent

### Claude Commands (`.claude/commands/`)

- **orchestrate-bmad-story.md**: Command that invokes the orchestrate-story workflow

## External Dependencies

This repository contains the custom orchestration logic but relies on external BMAD modules for core functionality:

### Required BMAD Modules

The workflow references these paths (which must exist in your project):

- `{project-root}/_bmad/bmm/workflows/4-implementation/create-story` - Story creation sub-workflow
- `{project-root}/_bmad/bmm/workflows/4-implementation/dev-story` - Story development sub-workflow
- `{project-root}/_bmad/bmm/workflows/4-implementation/code-review` - Code review sub-workflow
- `{project-root}/_bmad/core/tasks/review-adversarial-general.xml` - Adversarial review task
- `{project-root}/_bmad/bmm/config.yaml` - BMAD configuration (for `implementation_artifacts` path)

### Configuration Requirements

The workflow expects:

1. **BMAD Installation**: A full BMAD installation with BMM module
2. **Sprint Status File**: `sprint-status.yaml` in the implementation artifacts directory
3. **Story Directory**: Stories directory as configured in BMM config
4. **Git Repository**: A git repository with proper branch structure

## Usage

### Setup

1. Clone this repository into your project workspace
2. Ensure you have BMAD installed with the BMM module
3. Configure paths in `_bmad-custom-src/workflows/orchestrate-story/config.yaml` if needed
4. Ensure your project has the required BMAD modules installed

### Running the Workflow

Invoke the workflow using the Claude command:

```
@orchestrate-bmad-story
```

Or directly via the workflow file:

```
@_bmad-custom-src/workflows/orchestrate-story/workflow.md
```

The workflow will:
1. Identify the next story from `sprint-status.yaml`
2. Route to the appropriate step based on story status
3. Execute sub-workflows as needed
4. Present a human checkpoint before commit
5. Commit and merge on approval

## Workflow Details

See `_bmad-custom-src/workflows/orchestrate-story/README.md` for detailed workflow documentation, including:
- Complete flow diagram
- Step-by-step execution sequence
- State machine routing logic
- Configuration options
- Retry and validation mechanisms

## License

[Add your license information here]

## Contributing

[Add contribution guidelines if applicable]
