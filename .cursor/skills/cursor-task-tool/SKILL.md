---
name: cursor-task-tool
description: Execute subprocess delegation via Cursor's native Task tool. Invoked by router with model and normalized prompt.
---

# Cursor Task Tool

Execute delegated tasks using Cursor's native Task tool for isolated subprocess execution.

## Invocation

**New task**:
```
Task(subagent_type: "delegate-{model}", prompt: "{normalizedPrompt}")
```

**Resume after escalation**:
```
Task(resume: "{agentId}", prompt: "Human response: {userAnswer}")
```

## Model to Subagent Mapping

| Model | Subagent |
|-------|----------|
| `composer-1` | `delegate-composer-1` |
| `gemini-3-pro` | `delegate-gemini-3-pro` |
| `gpt-5.2` | `delegate-gpt-5.2` |
| `opus-4.5-thinking` | `delegate-opus-4.5-thinking` |
| `sonnet-4.5` | `delegate-sonnet-4.5` |
| `sonnet-4.5-thinking` | `delegate-sonnet-4.5-thinking` |

Unknown model → Use as subagent name directly.

## Execution

1. Map model to subagent path
2. Invoke Task tool with normalized prompt (already includes escalation protocol)
3. Check response for `[AWAITING_HUMAN_INPUT]`
   - If present → Return to router with question and agent ID for user interaction
4. Return result to router

## Error Handling

On any error, return to router for fallback handling:
- Task tool unavailable → Router tries CLI
- Model/subagent not found → Router tries fallback model
- Other errors → Router tries next tool or graceful degradation

## Example

**Router provides**: model=`opus-4.5-thinking`, prompt=normalized task

**Execute**:
```
Task(subagent_type: "delegate-opus-4.5-thinking", prompt: "## System Instructions

---
ESCALATION PROTOCOL:
If you need human clarification:
1. State your question clearly
2. Explain why you need this
3. End with exactly: [AWAITING_HUMAN_INPUT]
---

## Task
Implement the payment processing module for the checkout flow.

## Context
Files: src/checkout/, src/api/payments.ts
")
```
