# BMAD Story Orchestration

A standalone repository containing BMAD workflow orchestration systems and all their dependencies. This repository includes custom workflows, tasks, skills, agents, and commands needed to execute complete workflow automation for stories, planning documents, and adversarial review.

## Repository Structure

```
bmad-story-orchestration/
├── module.yaml                    # BMAD module configuration (required for install)
├── tasks/                         # Custom tasks
├── workflows/                     # Custom workflows
├── .claude/agents/                # Delegate definitions (Claude + subprocess)
├── .claude/skills/                # Subprocess delegation skill (module-bundled)
└── LICENSE
```

Note: BMAD generates commands and installs skills at project root (`.cursor/commands/bmad`, `.claude/commands/bmad`, etc.) from manifests — do not include these in the module.

## What's Included

### Core Content (root)

- **module.yaml**: BMAD module configuration defining workflows and tasks
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

**See**: `workflows/orchestrate-story/README.md` for detailed documentation

### Workflow 2: Orchestrate Planning Documents

PRD, Architecture, and Epic document creation workflow:

- Document type selection (PRD, Architecture, or Epic)
- Document creation via sub-workflows
- Adversarial review with retry loops
- Human checkpoint before commit
- Commit and merge operations
- Summary and tracking updates

**See**: `workflows/orchestrate-planning-documents/README.md` for detailed documentation

### Workflow 3: Adversarial Auto-Fix

Automated adversarial review with severity-aware fixing:

- Two-phase review: per-file + cross-file holistic analysis
- Automatic fixing of critical issues (up to configurable max iterations)
- Human checkpoint for high-severity issues
- Comprehensive validation reports
- Continuation support for interrupted sessions

**See**: `workflows/adversarial-auto-fix/README.md` for detailed documentation

### Delegate Definitions (`.claude/agents/`)

Delegate definitions for subprocess delegation and Claude Code slash commands:

- **delegate-composer-1.md**: Fast execution for simple/repetitive tasks
- **delegate-opus-4.6-thinking.md**: Architecture, planning, complex implementation
- **delegate-gemini-3-pro.md**: Code review
- **delegate-gpt-5.2.md**: Analysis and evaluation

### Commands and Skills

BMAD generates workflow commands at project root. Skills are **module-bundled** at `_bmad/orchestrate/.claude/skills/` — workflows load from there by default. Override in workflow config if using project-level skills.

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
- `{project-root}/_bmad/core/config.yaml` - BMB configuration (for `output_folder`, `user_name`, etc.)
- `{project-root}/_bmad/bmm/config.yaml` - BMM configuration (for `planning_artifacts`)

**For adversarial-auto-fix:**
- `{project-root}/_bmad/core/config.yaml` - BMB configuration (for output paths)
- `{project-root}/_bmad/core/tasks/review-adversarial-general.xml` - Adversarial review task

### Configuration Requirements

The workflows expect:

1. **BMAD Installation**: A full BMAD installation with BMM and BMB modules
2. **Sprint Status File**: `sprint-status.yaml` in the implementation artifacts directory (for orchestrate-story)
3. **Story/Document Directories**: As configured in BMM/BMB config files
4. **Git Repository**: A git repository with proper branch structure

## Usage

### Setup

1. During BMAD install (`npx bmad-method install`), add this as a custom module: path to this repo root (must contain `module.yaml`)
2. Ensure you have BMAD installed with the BMM and BMB modules
3. Configure paths in workflow `config.yaml` files if needed
4. Ensure your project has the required BMAD modules installed

**Config location**: BMAD installs workflow config to `_bmad/_config/custom/orchestrate/workflows/{name}/config.yaml`. Workflows load from that canonical path (not from the workflow folder). The module's `workflows/*/config.yaml` files are the source that gets copied there.

**Skills**: Module-bundled at `_bmad/orchestrate/.claude/skills/`. Workflows load from there by default. No manual copy needed.

### Running Workflows

**Orchestrate Story:**
```
/bmad-orchestrate-story
```

**Orchestrate Planning Documents:**
```
/bmad-orchestrate-planning-documents
```

**Adversarial Auto-Fix:**
```
/adversarial-auto-fix <file-or-directory> [--guidance TEXT] [--max-iterations N] [--auto BOOL]
```
or
```
/bmad-orchestrate-adversarial-auto-fix
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
