---
name: subprocess-delegation
description: Router for subprocess delegation. Detects delegation patterns, selects models, routes to implementation skills. Use when you see <delegate> blocks, BMAD subprocess patterns, or user requests to delegate/spawn.
---

# Subprocess Delegation Router

Detect delegation patterns, select models, route to appropriate tool for isolated execution.

## Boundaries

**Always**:
- Detect patterns before attempting delegation
- Normalize prompts to standard structure before execution
- Include escalation protocol in all delegated prompts
- Try tools in priority order, fall back on failure

**Ask First**:
- When user request is ambiguous about what to delegate
- When model selection is unclear and task is complex

**Never**:
- Delegate without detecting a valid pattern first
- Skip the escalation protocol
- Give up without trying graceful degradation

## Pattern Detection

Check content for these patterns (in order):

1. **XML**: Contains `<delegate` → Extract content between tags, model from `model="..."` attribute
2. **BMAD**: Contains `**Try to use Task tool` AND `to spawn a subprocess:**` → Extract quoted text, model from `(model: X)`
3. **User Request**: Contains delegation keywords + imperative language → Extract task description
   - Keywords: "spawn subprocess", "delegate this", "run in isolation", "use subagent"
   - Imperatives: "please", "can you", "I want to"

No pattern match → Not a delegation request, skip.

## Model Selection

1. **Explicit model found** → Normalize alias:
   - "fast" → `composer-1`
   - "review" → `gemini-3-pro`  
   - "analysis" → `gpt-5.2`
   - "reasoning"/"planning" → `opus-4.6-thinking`

2. **No model specified** → Apply heuristics to task content:
   - "code-review", "security" → `gemini-3-pro`
   - "analyze", "evaluate" → `gpt-5.2`
   - "architecture", "implement", "debug" → `opus-4.6-thinking`
   - Default → `composer-1`

## Prompt Normalization

Build this structure:

```
## System Instructions
{any extracted system instructions}

---
ESCALATION PROTOCOL:
If you need human clarification:
1. State your question clearly
2. Explain why you need this
3. End with exactly: [AWAITING_HUMAN_INPUT]
---

## Task
{the extracted task}

## Context
{any relevant context - files, variables}

## Expected Output
{any output format requirements}
```

## Tool Selection & Execution

**Check if model is Anthropic** (contains "sonnet", "opus", "haiku", or "claude"):

**For Anthropic models** (priority order):
1. `claude-cli` - Claude CLI for native Anthropic execution
2. `cursor-task-tool` - Native Task tool (skip if failed 2+ times)
3. `cursor-agent-cli` - Cursor CLI fallback

**For non-Anthropic models** (priority order):
1. `cursor-task-tool` - Native Task tool (skip if failed 2+ times)
2. `cursor-agent-cli` - Cursor CLI fallback

**Execution** (MANDATORY sequence - do NOT skip steps):

```
□ Step 1: Check if Task tool is available in current context
  - Look for "Task" in available tools list
  - If available → Load .claude/skills/cursor-task-tool/SKILL.md and execute
  - If NOT available → proceed to Step 2 (do NOT go to Step 3)

□ Step 2: Execute via CLI shell command
  - This is a SHELL COMMAND, not a Cursor tool - it is ALWAYS available
  - **CRITICAL**: Use `required_permissions: ['all']` when executing CLI commands
    - Both `cursor-agent` and `claude` CLI need full filesystem access for:
      - Keychain access (cursor-agent authentication)
      - Config directory writes (`~/.claude/`, `~/.cursor/`)
  - **Determine CLI based on model type**:
    
    **If Anthropic model** (name contains "sonnet", "opus", "haiku", or "claude"):
    - Load `.claude/skills/claude-cli/SKILL.md`
    - Pass model as-is (claude-cli handles mapping to CLI aliases)
    
    **If non-Anthropic model** (gemini, gpt, composer, or unknown):
    - Load `.claude/skills/cursor-agent-cli/SKILL.md`
    - Pass model as-is
  
  - Execute the CLI command per the loaded skill with `required_permissions: ['all']`
  - If success → done
  - If CLI not installed → continue to Step 3

□ Step 3: Inline execution (LAST RESORT ONLY)
  - Only if CLI command fails (e.g., cursor-agent not installed)
  - Execute prompt directly without isolation
```

**IMPORTANT**: Step 2 uses shell commands (`cursor-agent` or `claude`), NOT Cursor tools. You can ALWAYS run shell commands. If Task tool is unavailable, go to Step 2 (CLI), NOT Step 3.

**FORBIDDEN**: Skipping from Step 1 directly to Step 3. You MUST attempt Step 2 (CLI shell command) before inline execution.

**On escalation** (`[AWAITING_HUMAN_INPUT]` in response):
1. Present the question to user
2. Resume with user's answer

## Example Walkthroughs

### Example 1: XML Delegate Block

**Input**:
```xml
<delegate model="review">
  <prompt>Review src/auth/ for security issues</prompt>
</delegate>
```

**Flow**:
1. Pattern: XML_DELEGATE_BLOCK
2. Model: "review" → `gemini-3-pro`
3. Normalize prompt with escalation protocol
4. Load `cursor-task-tool/SKILL.md`
5. Execute: `Task(subagent_type: "delegate-gemini-3-pro", prompt: "...")`

### Example 2: Non-Anthropic Model Uses Cursor CLI

**Input**: BMAD pattern requesting code review with gemini

**Flow**:
1. Pattern: BMAD_PROSE_PATTERN  
2. Model: explicit `gemini-3-pro` (non-Anthropic model)
3. Step 1: Task tool unavailable
4. Step 2: Check model → "gemini-3-pro" does NOT contain opus/sonnet/haiku/claude
5. Load `cursor-agent-cli/SKILL.md` (NOT claude-cli)
6. Execute: `cursor-agent -p -f --model gemini-3-pro "..."`

### Example 3: Claude CLI Fails, Fallback Chain

**Input**: Same as above, but Claude CLI fails

**Flow**:
1. Claude CLI fails (not installed)
2. Load `cursor-task-tool/SKILL.md`, try Task tool
3. If that fails → Load `cursor-agent-cli/SKILL.md`
4. Execute: `cursor-agent -p -f --model opus "..."`

### Example 4: Escalation Handling

**Flow**:
1. Execute delegation → Response contains `[AWAITING_HUMAN_INPUT]`
2. Extract question: "Should I use REST or GraphQL for the API?"
3. Present to user, wait for answer
4. Resume: `Task(resume: "{agent-id}", prompt: "Human response: Use REST")`

## Reference

For detailed specifications, see:
- [REFERENCE.md](REFERENCE.md) - Edge cases, model fallback chains, extraction details
- [patterns.md](patterns.md) - Pattern syntax examples
- [model-reference.md](model-reference.md) - Full model guidance
