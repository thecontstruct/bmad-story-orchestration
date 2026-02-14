# Adversarial Auto-Fix Workflow

Automated adversarial review with severity-aware retry-and-fix loop for file sets. Reviews individual files and performs cross-file holistic analysis. Automatically fixes critical issues, pauses for human sign-off when high issues remain.

## Invocation

Via the Claude skill at `.claude/skills/adversarial-auto-fix/SKILL.md`:

```
/adversarial-auto-fix <input-paths...> [--guidance TEXT_OR_PATH] [--max-iterations N] [--auto BOOL] [--review-model MODEL] [--fix-model MODEL]
```

**Input:** One or more file paths, or a single folder path (recursive, targets `.md` files).

**Examples:**
```
/adversarial-auto-fix docs/
/adversarial-auto-fix file1.md file2.md file3.md
/adversarial-auto-fix docs/ --auto false --guidance "Focus on consistency"
/adversarial-auto-fix docs/ --max-iterations 10 --review-model gpt-5.2
```

## How It Works

### Two-Phase Review

Each iteration runs two review phases:

1. **Per-file review** — Delegated individually for each target file. Catches issues within a single document: factual errors, logical inconsistencies, clarity, style.
2. **Cross-file holistic review** — Delegated with all files in context. Catches inter-file issues: contradictions, missing references, structural gaps, redundancy. Automatically skipped when reviewing a single file.

Findings from both phases are aggregated into unified severity counts.

### Severity Levels

| Severity | Description |
|----------|-------------|
| Critical | Factual errors, broken functionality, security issues |
| High | Logical inconsistencies, missing essential information |
| Medium | Clarity improvements, best practice deviations |
| Low | Style nits, minor wording, cosmetic |

### Review/Fix Cycle

The workflow alternates between two steps until exit conditions are met:

```
step-02-review → step-02b-fix → step-02-review → step-02b-fix → ...
```

- **Auto mode** (`--auto true`, default): Cycle runs silently when critical issues exist
- **Manual mode** (`--auto false`): Pauses after each review for user approval before fixing

### Fix Routing

Fixes are context-aware:
- Per-file issues → single-file fix delegation (one per affected file)
- Cross-file issues → cross-file fix delegation (can modify multiple files)
- Single-file fixes run first, then cross-file fixes

### Exit Conditions

| Condition | Route |
|-----------|-------|
| All issues fixed (PASS) | Complete (PASSED) |
| Critical fixed, high remain | Human checkpoint |
| Max iterations reached | Human checkpoint |

### Human Checkpoints

User options: **[M]anual fix**, **[S]kip and continue**, **[A]uto-fix anyway**, **[Q]uit**

### Iteration Limits

Default: 5 iterations. Stored as an array of extension amounts (e.g., `[5, 5, 3]` = 13 total). Users can extend at checkpoints.

## Workflow Structure

```
adversarial-auto-fix/
├── workflow.md                  # Entry point
├── (config in _bmad/_config/custom/orchestrate/workflows/adversarial-auto-fix/)
├── README.md                    # This file
├── steps-c/
│   ├── step-01-init.md          # Resolve file set, create report
│   ├── step-01b-continue.md     # Resume interrupted workflow
│   ├── step-02-review.md        # Two-phase review + routing
│   ├── step-02b-fix.md          # Context-aware fix delegation
│   ├── step-03-human-checkpoint.md  # User decision point
│   └── step-04-complete.md      # Final report + completion
├── data/
│   ├── aggregation-rules.md     # How to combine per-file + cross-file findings
│   ├── checkpoint-menus.md      # Menu templates for human checkpoints
│   ├── delegation-templates.md  # XML templates for review/fix delegations
│   ├── exit-conditions.md       # When to exit the review/fix cycle
│   └── file-resolution.md       # How to resolve folder/file list input
└── templates/
    └── report-template.md       # Report document template
```

## Configuration

Defined in `_bmad/_config/custom/orchestrate/workflows/adversarial-auto-fix/config.yaml`:

| Setting | Default | Description |
|---------|---------|-------------|
| `max_iterations` | 5 | Max review/fix cycles before checkpoint |
| `auto_fix` | true | Run cycle automatically or pause for approval |
| `models.review` | gemini-3-pro | Model for adversarial review |
| `models.fix` | composer-1 | Model for fix delegation |
| `file_extensions.target` | [".md"] | File types to review |
| `file_extensions.supporting` | [".yaml", ".yml", ".xml", ".json"] | Supporting files read if referenced |

All settings can be overridden per-invocation via command flags.

## Output

- Fixed files (modifications applied by fix delegations)
- Consolidated report at `_bmad-output/validation-reports/adversarial-review-{workflow_id}.md`
  - Per-file findings, cross-file findings, iteration history, aggregate counts

## Continuation

Workflows can be resumed after interruption. State is preserved in report frontmatter. Invoke the skill again with the same input paths to resume.

## Phase 2 (Future)

Batched parallel coordination for large file sets — overlapping subset delegations with result aggregation. Deferred until Phase 1 proves out on real file sets.
