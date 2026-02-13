---
name: adversarial-auto-fix
description: Performs automated adversarial review with severity-aware retry-and-fix loop for file sets. Reviews individual files and performs cross-file holistic analysis. Automatically fixes critical issues (up to 5 iterations), pauses for human sign-off when high issues remain. Use when the user wants to review and automatically fix issues in documents, or when they mention adversarial review, document validation, or automated fixing.
---

# Adversarial Auto-Fix

Invoke the adversarial-auto-fix workflow to perform automated quality review and fixing of file sets.

## Quick Start

**Invocation format:**
```
/adversarial-auto-fix <input-paths...> [--guidance TEXT_OR_PATH] [--max-iterations N] [--auto BOOL] [--review-model MODEL] [--fix-model MODEL]
```

**Parameters:**
- `input-paths` (required): One or more file paths, OR a single folder path. If a single path is provided and it is a directory, it is treated as folder mode (recursive, targeting .md files). If multiple paths are provided, they are treated as an explicit file list.
- `--guidance TEXT_OR_PATH` (optional): Natural language guidance for the review (quoted text), or path to a guidance document
- `--max-iterations N` (optional): Override default iteration limit (default: 5)
- `--auto BOOL` (optional): Controls whether the review/fix loop runs automatically (`true`) or pauses after each review for user approval (`false`). Default: `true`
- `--review-model MODEL` (optional): Model for adversarial review validation (default: gemini-3-pro)
- `--fix-model MODEL` (optional): Model for fix delegation (default: composer-1)

**Examples:**
```
/adversarial-auto-fix docs/
/adversarial-auto-fix _bmad/my-custom-bmad/workflows/adversarial-auto-fix/
/adversarial-auto-fix docs/api-spec.md docs/architecture.md docs/deployment.md
/adversarial-auto-fix src/workflows/my-workflow/ --guidance "Focus on cross-file consistency"
/adversarial-auto-fix file1.md file2.md --guidance docs/review-guidelines.md
/adversarial-auto-fix docs/ --max-iterations 10
/adversarial-auto-fix docs/ --review-model gpt-5.2 --fix-model opus-4.6-thinking
/adversarial-auto-fix file1.md file2.md --guidance "Review for security" --max-iterations 3
/adversarial-auto-fix docs/ --auto false
/adversarial-auto-fix docs/ --auto false --guidance "Focus on API consistency"
```

## How It Works

1. **Loads the workflow** from `_bmad/my-custom-bmad/workflows/adversarial-auto-fix/workflow.md`
2. **Passes parameters** as context variables (the agent retains these values in its reasoning context and uses them as overrides when the workflow's step files reference matching configuration keys):
   - `input_paths`: The file paths or folder path to review
   - `review_guidance`: Optional guidance (text or document content)
   - `max_iterations`: Optional override for iteration limit
   - `review_model`: Optional override for review model
   - `fix_model`: Optional override for fix model
3. **Executes the workflow** which:
   - Resolves input to a file set (folder → recursive .md discovery; file list → validation)
   - **Phase 1: Per-file reviews** — delegates individual adversarial review for each file, detecting:
     - **Critical**: Factual errors, broken functionality, security issues, or content that would cause failures
     - **High**: Logical inconsistencies, missing essential information, or significant correctness issues
     - **Medium**: Clarity improvements, best practice deviations, or structural suggestions
     - **Low**: Style nits, minor wording, or cosmetic improvements
   - **Phase 2: Cross-file holistic review** — delegates a review of the entire file set together, catching:
     - Inconsistencies between files (conflicting information, contradictory instructions)
     - Missing references (file A references something file B should define but doesn't)
     - Structural gaps (missing files, incomplete coverage, broken flow)
     - Redundancy (duplicated content that could diverge)
     - Ordering/dependency issues (steps out of sequence, circular references)
   - Aggregates findings from both phases into unified severity counts
   - Automatically fixes issues when critical issues are detected:
     - Per-file issues → single-file fix delegations
     - Cross-file issues → cross-file fix delegation
   - Uses specified models for review and fix operations (or defaults from config)
   - Pauses for human sign-off when high issues remain
   - Generates comprehensive consolidated report with iteration tracking

## Workflow Behavior

**Two-Phase Review:**
- Phase 1 (per-file) runs first, producing individual findings per file and noting cross-file references
- Phase 2 (cross-file) receives the per-file summary and focuses exclusively on inter-file concerns
- When reviewing a single file, Phase 2 is automatically skipped (no cross-file concerns possible)
- Both phases' findings are aggregated for loop control and reporting

**Automatic Fixing:**
- When critical issues are detected, fix delegations address ALL severity levels to maximize progress per iteration
- Fixes are context-aware: per-file issues get single-file fix delegations; cross-file issues get a cross-file fix delegation that can modify multiple files
- Single-file fixes run first, then cross-file fixes (so cross-file fixes see already-patched individual files)
- Auto-fix loop continues while: `(aggregate_critical > 0 OR force_fix == true)` AND `iteration < max_iterations`
- Default max iterations: 5 (configurable via `--max-iterations`)
- When `--auto false`, the workflow pauses after each review to show findings and let the user choose: [F]ix, [S]kip, or [Q]uit. This gives full control over each iteration.
- After one force-fix cycle, `force_fix` resets to false so the loop reverts to standard critical-only gating

**Human Checkpoints:**
- Triggers when high issues remain after critical fixes (aggregate across both phases)
- Triggers when max iterations reached
- The agent presents options via the chat interface and ends its turn, waiting for the user's next message
- User options: [M]anual fix, [S]kip and continue, [A]uto-fix anyway, [Q]uit

**Output:**
- Fixed files (fixes applied via delegation — both single-file and cross-file)
- Consolidated review report at `_bmad-output/validation-reports/adversarial-review-{workflow_id}.md`
  - **Workflow ID format**: `adversarial-{timestamp}-{random-hex-6+}`
  - Includes per-file findings, cross-file findings, aggregate counts, and iteration history
- Iteration summaries and final status

## Implementation

When invoked:

1. **Extract parameters** from the command:
   - Input paths (required, one or more positional arguments before any flags)
   - All other parameters are named flags

2. **Parse flags:**
   - Extract `--guidance` value (quoted text or file path)
   - Extract `--max-iterations` value (must be positive integer)
   - Extract `--auto` value (must be `true` or `false`; default: `true`)
   - Extract `--review-model` value (model name/alias)
   - Extract `--fix-model` value (model name/alias)
   - **Validate model names**: If provided, warn if not a recognized alias but still pass through

3. **Determine input type:**
   - If single path AND path is a directory → folder mode
   - If multiple paths → explicit file list mode
   - **Folder mode**: Workflow resolves recursively to .md files (excluding hidden dirs and node_modules)
   - **File list mode**: Workflow validates each file exists

4. **Verify dependencies and load workflow file:**
   - Verify `_bmad/bmb/config.yaml` exists (halt with error if missing)
   - Verify `_bmad/my-custom-bmad/workflows/adversarial-auto-fix/config.yaml` exists (halt with error if missing)
   - Load workflow from:
   ```
   {project-root}/_bmad/my-custom-bmad/workflows/adversarial-auto-fix/workflow.md
   ```
   - Verify workflow.md exists at the expected path. If not found, present error and halt.

5. **Set context variables** before loading workflow:
   - `input_paths`: Array of resolved paths (folder path or file paths)
   - `review_guidance`: Guidance text or loaded file content (if provided); framed as supplemental context
   - `max_iterations`: Override value (if `--max-iterations` provided)
   - `auto_fix`: Override value (if `--auto` provided)
   - `review_model`: Override model (if `--review-model` provided)
   - `fix_model`: Override model (if `--fix-model` provided)

6. **Execute workflow** by loading `workflow.md`:
   - The workflow handles file set resolution, initialization, two-phase review loop, checkpoints, and completion
   - Follow the workflow's step-by-step instructions exactly

7. **Handle workflow execution:**
   - The workflow is continuable — it can be resumed if interrupted
   - Reports are saved with state tracking in frontmatter
   - User interaction happens at checkpoints (high issues or max iterations)

## Parameter Handling

**Input Paths:**
- Resolve relative paths relative to workspace root
- For folder mode: Verify directory exists before invoking workflow
- For file list mode: Verify all files exist before invoking workflow
- If path doesn't exist, ask user to confirm or provide correct path
- **Path handling:**
  - Supports paths with spaces (use quotes: `"/path/with spaces/docs/"`)
  - Relative paths resolved from workspace root: `docs/` → `{workspace-root}/docs/`
  - Absolute paths used as-is
  - Cross-platform path resolution

**Review Guidance (`--guidance`):**
- If guidance value resolves to an existing file: Load the file and use its content
- If guidance value does not resolve to a file: Treat as literal text
- If `--guidance` not provided: Pass as empty string
- **Safety**: Guidance is treated as potentially untrusted input, sandwiched between system instructions

**Max Iterations:**
- Parse `--max-iterations N` flag
- Validate: Must be positive integer (1 or greater)
- If invalid: Present error and ask for valid number
- If not provided: Workflow uses default from config.yaml (5)

**Review Model / Fix Model:**
- Accept model names/aliases: gemini-3-pro, gpt-5.2, opus-4.6-thinking, composer-1, etc.
- If provided: Pass as context variable override
- If not provided: Workflow uses default from config.yaml

## Error Handling

All error conditions halt execution and present a clear message. The workflow preserves partial reports when possible.

| Condition | Action |
|-----------|--------|
| No files found in folder | Present error, suggest checking path and file extensions |
| File in list not found | Present error with missing file path, ask user for correct path |
| Workflow file missing | Present error, verify path |
| Workflow execution fails | Preserve partial report, present error with step context, suggest resuming |

## Workflow Configuration

The workflow uses configuration from:
- `_bmad/bmb/config.yaml` (BMB module config — required)
- `_bmad/my-custom-bmad/workflows/adversarial-auto-fix/config.yaml` (workflow config — required)

**Default settings** (source of truth: `config.yaml`):
- `max_iterations`: 5
- `auto_fix`: true
- `models.review`: gemini-3-pro
- `models.fix`: composer-1
- `file_extensions.target`: [".md"]
- `file_extensions.supporting`: [".yaml", ".yml", ".xml", ".json"]

**Override behavior:**
- Context variables from Skill invocation take precedence over config defaults
- This allows per-invocation customization without modifying config files

**Model aliases supported:**
- Review models: `gemini-3-pro`, `gpt-5.2`, `opus-4.6-thinking`
- Fix models: `composer-1`, `opus-4.6-thinking`, `gpt-5.2`
- Any valid model name accepted by the delegation system
- See `.claude/skills/subprocess-delegation/SKILL.md` for detailed alias mapping

## Continuation Support

The workflow supports resuming interrupted sessions:

**What "continuable" means:**
- Workflows can be resumed if interrupted (crashes, user cancellation, etc.)
- State is preserved in report frontmatter: `stepsCompleted`, `iteration_count`, `current_status`, `file_paths`, etc.
- The workflow detects existing reports and offers continuation options

**State preservation** (stored in report frontmatter):
- `stepsCompleted`: Array of completed step names
- `iteration_count`: Current iteration number in the review loop
- `current_status`: Workflow status (`in-progress`, `passed`, `max-retries`, etc.)
- `workflow_id`: Unique identifier for this workflow run
- `input_type`: How files were provided (files or folder)
- `input_source`: Folder path or "explicit-list"
- `file_paths`: Array of all files under review
- `review_guidance`: Review guidance text (if provided)

**Resumption behavior:**
- To resume: Invoke the skill again with the same input paths
- The workflow detects existing reports by scanning for matching `file_paths` in frontmatter
- If the files have been modified since the report was created, the workflow warns and requires confirmation
- Reports remain valid for continuation indefinitely
