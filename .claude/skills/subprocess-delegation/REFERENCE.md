# Subprocess Delegation Reference

Reference material for edge cases and detailed specifications. Consult when needed, not required for standard execution.

## Model Fallback Chain

When a model fails, try the next in chain:
- `opus-4.6-thinking` → `sonnet-4.5-thinking` → `sonnet-4.5`
- `gpt-5.2` → `composer-1`
- `gemini-3.0-pro` → `composer-1`
- Unknown → `composer-1`

## Session Context Tracking

Track Task tool failures in session:
- Increment `task_tool_failures` on each failure
- If failures >= 2 → Skip Task tool in future attempts this session
- Reset on new session

## User Request Pattern Detection (Detailed)

Valid user request requires BOTH:
1. **Delegation keyword** (case-insensitive):
   - "spawn a subprocess", "spawn subprocess"
   - "delegate this", "delegate task", "delegate to"
   - "run in isolation", "isolated execution"
   - "use a subagent", "use subagent"
   - "run in subprocess", "execute in subprocess"

2. **Imperative structure** (one of):
   - Starts with keyword
   - Contains "please {keyword}"
   - Contains "can you {keyword}" / "could you {keyword}"
   - Contains "I want to {keyword}" / "I need to {keyword}"

Without both → Not a user request pattern (avoids false positives when just discussing delegation).

## Prompt Extraction Details

**XML_DELEGATE_BLOCK**:
- Extract content between `<delegate>` and `</delegate>`
- Look for optional tags: `<system-instructions>`, `<context>`, `<prompt>`, `<output>`
- If no `<prompt>` tag, use all content as task

**BMAD_PROSE_PATTERN**:
- Trigger: `**Try to use Task tool` ... `to spawn a subprocess:**`
- Model: `(model: X)` between trigger phrases (optional)
- Prompt: Quoted text block following trigger line
- Graceful degradation: Content after `**Graceful degradation` marker (historical, now always available)

**USER_REQUEST_PATTERN**:
- Remove delegation keywords from extracted task
- Remove model mentions
- If vague ("delegate this"), use surrounding context
- If still unclear, use entire content as prompt

## Context Extraction Heuristics

Look for in raw prompt:
- Variable references: `{variable}` patterns
- File paths: `/path/to/file`, `src/...`, etc.
- Phrases: "using", "with", "from" followed by context

## Output Format Inference

Look for in raw prompt:
- "Return" / "Output" followed by format description
- "as JSON" / "as markdown" / "as YAML"
- "Format:" / "Structure:" specifications

## Edge Cases

1. **Missing model**: Use heuristics, default to composer-1
2. **Unknown model alias**: Use as-is, handle failure in error recovery
3. **Multiple escalations**: Loop until resolved
4. **Agent ID missing**: Handle gracefully, some tools may not support resume
5. **All tools fail**: Always fall back to inline execution
6. **Partial XML**: Handle missing optional tags gracefully
7. **Vague user request**: Use context or prompt for clarification
