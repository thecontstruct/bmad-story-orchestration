---
name: cursor-agent-cli
description: Execute subprocess delegation via cursor-agent CLI. Fallback when Task tool fails. Invoked by router with model and normalized prompt.
---

# Cursor Agent CLI

Execute delegated tasks using cursor-agent CLI for isolated subprocess execution. Always available as fallback.

## Invocation

```bash
cursor-agent --print --force --model {model} "{prompt}"
```

**Resume after escalation** (if chatId available):
```bash
cursor-agent --print --force --resume {chatId} "Human response: {userAnswer}"
```

**Flags**:
- `--print` / `-p`: Non-interactive headless mode (required)
- `--force` / `-f`: Bypass approval prompts (required)
- `--model`: Model selection
- `--output-format json`: For structured output

## Execution

1. Add safeguards to the normalized prompt:
   ```
   {normalizedPrompt}
   
   CONSTRAINTS:
   - DO NOT spawn additional agents or call cursor-agent CLI
   - Keep response under 500 words
   - Summarize actions, don't narrate step-by-step
   ```

2. Run synchronously (NOT background) - output must be immediately available

3. Check response for `[AWAITING_HUMAN_INPUT]`
   - If present → Return to router with question for user interaction

4. Return result to router

## Error Handling

On error, return to router for graceful degradation:
- "command not found" → CLI not installed
- Authentication error → Not logged in
- Model unavailable → Try fallback model

Do not attempt to install or authenticate - router handles fallback.

## Critical Safeguards

**Anti-recursion**: Always include in prompt:
```
DO NOT spawn additional agents or call cursor-agent CLI.
```

**Output control**: Always include:
```
Keep response under 500 words. Summarize actions.
```

## Example

**Router provides**: model=`composer-1`, prompt=normalized task

**Build command**:
```bash
cursor-agent -p -f --model composer-1 "## System Instructions

---
ESCALATION PROTOCOL:
If you need human clarification:
1. State your question clearly
2. Explain why you need this
3. End with exactly: [AWAITING_HUMAN_INPUT]
---

## Task
Add error handling to src/api/users.ts

CONSTRAINTS:
- DO NOT spawn additional agents or call cursor-agent CLI
- Keep response under 500 words
- Summarize actions, don't narrate step-by-step"
```
