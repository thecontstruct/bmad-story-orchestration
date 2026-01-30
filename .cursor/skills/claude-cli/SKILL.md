---
name: claude-cli
description: Execute subprocess delegation via Claude CLI (Anthropic's claude command). Alternative to cursor-agent for Claude models. Invoked by router with model and normalized prompt.
---

# Claude CLI

Execute delegated tasks using Anthropic's `claude` CLI for isolated subprocess execution with Claude models.

## Invocation

**CRITICAL**: 
- Always close stdin with `</dev/null` to prevent hanging
- Always use `--dangerously-skip-permissions` for non-interactive execution

**New task**:
```bash
claude -p --dangerously-skip-permissions --model {model} --append-system-prompt "{systemPrompt}" "{task}" </dev/null
```

**Resume by session ID**:
```bash
claude -r "{sessionId}" -p --dangerously-skip-permissions "{userAnswer}" </dev/null
```

**Continue most recent**:
```bash
claude -c -p --dangerously-skip-permissions "{followUp}" </dev/null
```

## Key Flags

| Flag | Purpose |
|------|---------|
| `-p` / `--print` | Non-interactive mode (required for delegation) |
| `--dangerously-skip-permissions` | Skip permission prompts (required for `-p` mode) |
| `--model` | Model selection |
| `-c` / `--continue` | Continue most recent conversation |
| `-r` / `--resume` | Resume by session ID |
| `--append-system-prompt` | Add to system prompt |
| `--output-format json` | Structured output for parsing |
| `--allowedTools` | Restrict available tools |

## Model Mapping

The router may pass canonical model names. Map them to Claude CLI aliases before execution:

| Router Passes | Use With CLI |
|---------------|--------------|
| `opus-4.5-thinking`, `opus` | `opus` |
| `sonnet-4.5-thinking`, `sonnet-4.5`, `sonnet` | `sonnet` |
| `haiku` | `haiku` |

Claude CLI also accepts full model strings: `--model claude-sonnet-4-5-20250929`

## Execution

1. **Map model to CLI alias** (see table above)

2. Extract system instructions and task from normalized prompt

3. Add safeguards to system prompt:
   ```
   {extractedSystemInstructions}
   
   CONSTRAINTS:
   - DO NOT spawn additional agents or CLI tools
   - Keep response under 500 words
   - Summarize actions, don't narrate step-by-step
   ```

4. Build and run command with JSON output for session tracking:
   
   ```bash
   claude -p --dangerously-skip-permissions --model {model} --output-format json \
     --append-system-prompt "{safeguardedSystemPrompt}" \
     "{taskContent}" </dev/null
   ```

5. Parse JSON response and extract `session_id` - store it for potential resume

6. Check response for `[AWAITING_HUMAN_INPUT]`
   - If present → Return session ID to router for user input

7. Return result to router

## Session Tracking & Timeout Recovery

**Always use `--output-format json`** to capture session IDs. The JSON response includes:
```json
{
  "session_id": "abc123-...",
  "result": "...",
  "cost_usd": 0.05
}
```

**If command times out**:
1. Check the terminal output for partial JSON or session info
2. Use `claude -c -p` to continue the most recent session in that directory
3. Or use `claude -r "{session_id}" -p "continue"` if you have the session ID

**Resume a timed-out session**:
```bash
claude -r "{session_id}" -p --dangerously-skip-permissions "Please continue where you left off" </dev/null
```

**Continue most recent session** (same directory):
```bash
claude -c -p --dangerously-skip-permissions "Please continue where you left off" </dev/null
```

## Error Handling

On error, return to router for fallback:
- "command not found" → CLI not installed
- Authentication error → Not logged in (`claude login` or set `ANTHROPIC_API_KEY`)
- Model unavailable → Try fallback model
- Permission error → Subagent hit sandbox restrictions (expected in some contexts)
- Hanging/timeout → Missing `</dev/null` OR long-running task (try resume)

**On timeout**: Don't immediately fallback. Try to resume the session first using `-c` or `-r`. The subagent may have made progress.

Do not attempt to install or authenticate - router handles fallback.

## Examples

### Standard Model

**Router provides**: model=`opus`, prompt with system instructions and task

```bash
claude -p --dangerously-skip-permissions --model opus \
  --append-system-prompt "---
ESCALATION PROTOCOL:
If you need human clarification:
1. State your question clearly
2. Explain why you need this
3. End with exactly: [AWAITING_HUMAN_INPUT]
---

CONSTRAINTS:
- DO NOT spawn additional agents or CLI tools
- Keep response under 500 words" \
  "Implement the payment processing module for the checkout flow." </dev/null
```

### With JSON Output (for session tracking)

```bash
claude -p --dangerously-skip-permissions --model opus --output-format json \
  --append-system-prompt "..." \
  "task here" </dev/null
```

Response includes `session_id` for resuming.
