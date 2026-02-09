# Model Selection Reference

Detailed guidance for selecting the appropriate model when spawning subprocesses.

## Canonical Model Names

The router normalizes aliases to these canonical model names. CLI skills handle any further mapping.

| Canonical Model | Provider | CLI Skill | Aliases |
|-----------------|----------|-----------|---------|
| `opus-4.6-thinking` | Anthropic | claude-cli | `reasoning`, `planning` |
| `sonnet-4.5` | Anthropic | claude-cli | `sonnet` |
| `sonnet-4.5-thinking` | Anthropic | claude-cli | `sonnet-thinking` |
| `gemini-3-pro` | Google | cursor-agent-cli | `review` |
| `gpt-5.2` | OpenAI | cursor-agent-cli | `analysis` |
| `composer-1` | Cursor | cursor-agent-cli | `fast` |

## Model to Subagent Mapping

| Canonical Model | Subagent Path | Use Case |
|-----------------|---------------|----------|
| `composer-1` | `delegate-composer-1` | Simple tasks, validation, repetitive work |
| `gemini-3-pro` | `delegate-gemini-3-pro` | Code reviews, security audits, quality checks |
| `gpt-5.2` | `delegate-gpt-5.2` | Analysis, evaluation, assessment tasks |
| `opus-4.6-thinking` | `delegate-opus-4.6-thinking` | Complex implementation, architecture, planning |

## Heuristic-Based Selection

When no model is explicitly specified, analyze task content for keywords:

### Code Review → `gemini-3-pro`
- Keywords: "code-review", "code-audit", "code-check", "security review", "review code"
- Best for: Security audits, code quality checks, PR reviews

### Analysis → `gpt-5.2`
- Keywords: "analyze", "examine", "evaluate", "assess", "investigate"
- Best for: Data analysis, requirements evaluation, document assessment

### Architecture/Planning → `opus-4.6-thinking`
- Keywords: "architecture", "design", "plan", "implement", "build", "create complex", "debug", "troubleshoot"
- Best for: System design, complex implementations, debugging, multi-step planning

### Validation/Simple → `composer-1`
- Keywords: "validate", "check", "verify", "scan", "simple", "repetitive"
- Best for: Validation tasks, simple checks, repetitive operations
- **Default fallback** when task type is unclear

## BMAD Workflow Model Patterns

BMAD workflows often use these model assignments:

- **create_story**: `opus-4.6-thinking` (complex creation)
- **validate_story**: `gpt-5.2` (analysis/evaluation)
- **dev_story**: `opus-4.6-thinking` (implementation)
- **code_review**: `gemini-3-pro` (review)
- **fix_validation**, **fix_review**: `composer-1` (simple fixes)

## Model Selection Best Practices

1. **Explicit over implicit**: When model is specified in config or pattern, use it
2. **Match task complexity**: Complex tasks need reasoning models, simple tasks need fast models
3. **Consider context isolation**: Some tasks benefit from isolation regardless of complexity
4. **Default conservatively**: When uncertain, default to `composer-1` (fastest, least resource-intensive)
