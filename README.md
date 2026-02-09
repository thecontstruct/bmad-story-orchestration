# BMAD Story Orchestration

A standalone repository containing BMAD workflow orchestration systems and all their dependencies. This repository includes custom workflows, tasks, skills, agents, and commands needed to execute complete workflow automation for stories, planning documents, and adversarial review.

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
│       ├── orchestrate-story/     # Story lifecycle orchestration
│       ├── orchestrate-planning-documents/  # PRD/Architecture/Epic orchestration
│       └── adversarial-auto-fix/  # Automated adversarial review with auto-fix
├── .claude/
│   ├── agents/                    # Delegation agents
│   │   ├── delegate-composer-1.md
│   │   ├── delegate-opus-4.6-thinking.md
│   │   ├── delegate-gemini-3-pro.md
│   │   └── delegate-gpt-5.2.md
│   ├── commands/                  # Claude commands
│   │   ├── orchestrate-bmad-story.md
│   │   └── orchestrate-bmad-planning-documents.md
│   └── skills/                    # Skills for subprocess delegation
│       ├── subprocess-delegation/
│       ├── claude-cli/
│       ├── cursor-agent-cli/
│       ├── cursor-task-tool/
│       └── adversarial-auto-fix/
└── LICENSE                         # MIT License
```

## What's Included

### Core Content (`_bmad-custom-src/`)

- **custom.yaml**: BMAD module configuration defining workflows and tasks
- **tasks/**: Custom tasks for retry logic, validation, and adversarial review
- **workflows/**: Three complete orchestration workflows:
  1. **orchestrate-story/**: Complete story lifecycle automation
  2. **orchestrate-planning-documents/**: PRD/Architecture/Epic document creation workflow
  3. **adversarial-auto-fix/**: Automated adversarial review with severity-aware auto-fix

### Workflow 1: Orchestrate Story

Complete story lifecycle orchestration:

- Story identification from sprint-status.yaml
- Story creation via create-story sub-workflow
- Adversarial story review (optional)
- Story validation with retry loops
- Story development via dev-story sub-workflow
- Adversarial code review (optional)
- Code review with retry loops
- Human checkpoint before commit
- Commit and merge operations
- Summary and next story identification

**See**: `_bmad-custom-src/workflows/orchestrate-story/README.md` for detailed documentation

### Workflow 2: Orchestrate Planning Documents

PRD, Architecture, and Epic document creation workflow:

- Document type selection (PRD, Architecture, or Epic)
- Document creation via sub-workflows
- Adversarial review with retry loops
- Human checkpoint before commit
- Commit and merge operations
- Summary and tracking updates

**See**: `_bmad-custom-src/workflows/orchestrate-planning-documents/README.md` for detailed documentation

### Workflow 3: Adversarial Auto-Fix

Automated adversarial review with severity-aware fixing:

- Two-phase review: per-file + cross-file holistic analysis
- Automatic fixing of critical issues (up to configurable max iterations)
- Human checkpoint for high-severity issues
- Comprehensive validation reports
- Continuation support for interrupted sessions

**See**: `_bmad-custom-src/workflows/adversarial-auto-fix/README.md` for detailed documentation

### Claude Skills (`.claude/skills/`)

Skills for subprocess delegation and agent routing:

- **subprocess-delegation/**: Router skill that detects delegation patterns and routes to appropriate execution tool
- **claude-cli/**: Skill for executing via Anthropic's Claude CLI with session recovery
- **cursor-agent-cli/**: Skill for executing via Cursor Agent CLI with continuation support
- **cursor-task-tool/**: Skill for executing via Cursor's native Task tool
- **adversarial-auto-fix/**: Skill for invoking the adversarial-auto-fix workflow

### Claude Agents (`.claude/agents/`)

Agent definitions for delegated execution:

- **delegate-composer-1.md**: Fast execution agent for simple/repetitive tasks
- **delegate-opus-4.6-thinking.md**: Architecture, planning, complex implementation agent
- **delegate-gemini-3-pro.md**: Code review agent
- **delegate-gpt-5.2.md**: Analysis and evaluation agent

### Claude Commands (`.claude/commands/`)

- **orchestrate-bmad-story.md**: Command that invokes the orchestrate-story workflow
- **orchestrate-bmad-planning-documents.md**: Command that invokes the orchestrate-planning-documents workflow

## External Dependencies

This repository contains the custom orchestration logic but relies on external BMAD modules for core functionality:

### Required BMAD Modules

The workflows reference these paths (which must exist in your project):

**For orchestrate-story:**
- `{project-root}/_bmad/bmm/workflows/4-implementation/create-story` - Story creation sub-workflow
- `{project-root}/_bmad/bmm/workflows/4-implementation/dev-story` - Story development sub-workflow
- `{project-root}/_bmad/bmm/workflows/4-implementation/code-review` - Code review sub-workflow
- `{project-root}/_bmad/core/tasks/review-adversarial-general.xml` - Adversarial review task
- `{project-root}/_bmad/bmm/config.yaml` - BMAD configuration (for `implementation_artifacts` path)

**For orchestrate-planning-documents:**
- `{project-root}/_bmad/bmb/workflows/create-prd` - PRD creation sub-workflow
- `{project-root}/_bmad/bmb/workflows/create-architecture` - Architecture creation sub-workflow
- `{project-root}/_bmad/bmb/workflows/create-epic` - Epic creation sub-workflow
- `{project-root}/_bmad/bmb/config.yaml` - BMB configuration (for `output_folder`, `user_name`, etc.)
- `{project-root}/_bmad/bmm/config.yaml` - BMM configuration (for `planning_artifacts`)

**For adversarial-auto-fix:**
- `{project-root}/_bmad/bmb/config.yaml` - BMB configuration (for output paths)
- `{project-root}/_bmad/core/tasks/review-adversarial-general.xml` - Adversarial review task

### Configuration Requirements

The workflows expect:

1. **BMAD Installation**: A full BMAD installation with BMM and BMB modules
2. **Sprint Status File**: `sprint-status.yaml` in the implementation artifacts directory (for orchestrate-story)
3. **Story/Document Directories**: As configured in BMM/BMB config files
4. **Git Repository**: A git repository with proper branch structure

## Usage

### Setup

1. Clone this repository into your project workspace
2. Ensure you have BMAD installed with the BMM and BMB modules
3. Configure paths in workflow `config.yaml` files if needed
4. Ensure your project has the required BMAD modules installed

### Running Workflows

**Orchestrate Story:**
```
@orchestrate-bmad-story
```
or
```
@_bmad-custom-src/workflows/orchestrate-story/workflow.md
```

**Orchestrate Planning Documents:**
```
@orchestrate-bmad-planning-documents
```
or
```
@_bmad-custom-src/workflows/orchestrate-planning-documents/workflow.md
```

**Adversarial Auto-Fix:**
```
/adversarial-auto-fix <file-or-directory> [--guidance TEXT] [--max-iterations N] [--auto BOOL]
```
or
```
@_bmad-custom-src/workflows/adversarial-auto-fix/workflow.md
```

## Workflow Details

Each workflow includes comprehensive documentation:

- **orchestrate-story/README.md**: Complete flow diagram, step-by-step execution, state machine routing, configuration options
- **orchestrate-planning-documents/README.md**: Document creation flow, routing logic, configuration
- **adversarial-auto-fix/README.md**: Two-phase review process, auto-fix behavior, checkpoint handling

## Key Features

### State-Based Routing

All workflows support resumable execution with state preservation. Workflows can be interrupted and resumed from the appropriate step.

### Automated Retry Loops

Validation and review steps use retry-with-feedback tasks that automatically retry with accumulated feedback up to configurable maximum attempts.

### Adversarial Review Integration

Optional adversarial review steps provide fresh-perspective validation before formal review, catching issues early.

### Human Checkpoints

Single human review gates before commit operations ensure quality and provide control over final changes.

### Subprocess Delegation

All workflows use subprocess delegation for isolated execution, supporting multiple execution backends (Task tool, Claude CLI, Cursor Agent CLI) with automatic fallback.

## License

MIT License - see LICENSE file for details.
