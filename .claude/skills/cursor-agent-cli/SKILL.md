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

**Continue most recent session** (on timeout):
```bash
cursor-agent --print --force --continue "Please continue where you left off. If you were in the middle of a task, complete it."
```

**Flags**:
- `--print` / `-p`: Non-interactive headless mode (required)
- `--force` / `-f`: Bypass approval prompts (required)
- `--model`: Model selection
- `--continue` / `-c`: Continue most recent conversation
- `--resume`: Resume by chatId/session ID
- `--output-format json`: For structured output (if available)

## Execution

1. Add safeguards to the normalized prompt:
   ```
   {normalizedPrompt}
   
   CONSTRAINTS:
   - DO NOT spawn additional agents or call cursor-agent CLI
   - Keep response under 500 words
   - Summarize actions, don't narrate step-by-step
   ```

2. Build and run command with `required_permissions: ['all']`:
   ```bash
   cursor-agent --print --force --model {model} "{safeguardedPrompt}"
   ```
   
   **CRITICAL**: Always use `required_permissions: ['all']` when executing `cursor-agent` commands via `run_terminal_cmd`. The tool needs:
   - Keychain access for authentication
   - Write access to config directories
   
   If `--output-format json` is available, use it to capture session/chat IDs:
   ```bash
   cursor-agent --print --force --model {model} --output-format json "{safeguardedPrompt}"
   ```

3. **Handle execution result**:
   - **Success**: Parse response and extract `chatId`/`session_id` if available - store it for potential resume
   - **Timeout/Interruption**: 
     * Try to extract `chatId`/`session_id` from any partial output
     * Immediately run continuation command: `cursor-agent --print --force --continue "Please continue where you left off. If you were in the middle of a task, complete it."`
     * If `chatId`/`session_id` was captured, prefer: `cursor-agent --print --force --resume {chatId} "Please continue where you left off. If you were in the middle of a task, complete it."`
     * Parse continuation response
     * If continuation also times out, attempt once more, then return error

4. Check response for `[AWAITING_HUMAN_INPUT]`
   - If present → Return chatId/session ID to router for user input

5. Return result to router

## Session Tracking & Timeout Recovery

**If `--output-format json` is available**, use it to capture session/chat IDs. The JSON response may include:
```json
{
  "chatId": "abc123-...",
  "result": "...",
  "session_id": "xyz789-..."
}
```

**CRITICAL: On timeout, automatically continue the session**

If the command times out or is interrupted:
1. **First attempt**: Try to extract `chatId` or `session_id` from any partial output before timeout
   - If `chatId`/`session_id` found → Use `cursor-agent --print --force --resume "{chatId}"` to resume
   - If not found → Use `cursor-agent --print --force --continue` to continue most recent session

2. **Continue command** (use this when timeout detected):
   ```bash
   cursor-agent --print --force --continue \
     "Please continue where you left off. If you were in the middle of a task, complete it."
   ```

3. **If chatId/session ID was captured** (preferred method):
   ```bash
   cursor-agent --print --force --resume "{chatId}" \
     "Please continue where you left off. If you were in the middle of a task, complete it."
   ```

4. **Parse the continuation response** and return to router
   - If it times out again, repeat the continue attempt once more
   - After 2 timeout attempts, return error to router

**Implementation flow**:
- Store `chatId`/`session_id` from initial command output (even if partial)
- On timeout/interruption: Immediately attempt `--continue` continuation
- Parse continuation response for completion or further timeout
- Return final result or error to router

## Error Handling

On error, return to router for graceful degradation:
- "command not found" → CLI not installed
- Authentication error → Not logged in
- Model unavailable → Try fallback model

**On timeout/interruption** (CRITICAL):
1. **DO NOT immediately fallback** - the subagent may have made progress
2. **Automatically attempt continuation**:
   - Extract `chatId`/`session_id` from partial output if available
   - Run `cursor-agent --print --force --continue` (or `--resume "{chatId}"` if available) with continuation prompt
   - Parse continuation response
   - If continuation succeeds → Return result to router
   - If continuation also times out → Attempt once more, then return error to router
3. Only return error to router after 2 failed continuation attempts

**Hanging/timeout causes**:
- Long-running task → Use continuation to resume

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
