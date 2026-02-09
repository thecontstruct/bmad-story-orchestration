# Subprocess Delegation Patterns

Syntax patterns for spawning subprocesses. This skill recognizes and handles these patterns.

## Pattern 1: XML Delegate Block

Structured XML format for explicit delegation:

```xml
<delegate [model="..."]>
  <system-instructions>
    Role, constraints, and behavior instructions
  </system-instructions>
  
  <context>
    File paths, variables, prior state
  </context>
  
  <prompt>
    The specific task to execute
  </prompt>
  
  <output>
    Expected structure/format of result
  </output>
</delegate>
```

### Processing Steps

1. **Extract model** from `model` attribute (if present)
2. **Extract sections**: system-instructions, context, prompt, output
3. **Inject escalation protocol** into system-instructions
4. **Determine subagent** via model mapping or heuristics
5. **Invoke**: `Task(subagent_type: "delegate-{model-id}", prompt: "<full block>")`
6. **Handle response**: Check for `[AWAITING_HUMAN_INPUT]`, format per output spec

### Examples

**Explicit model:**
```xml
<delegate model="opus-4.6-thinking">
  <prompt>Implement payment processing module</prompt>
</delegate>
```

**Using alias:**
```xml
<delegate model="review">
  <prompt>Review src/auth/ for security vulnerabilities</prompt>
</delegate>
```

**Heuristics-based:**
```xml
<delegate>
  <prompt>Analyze database schema for optimization</prompt>
</delegate>
```
→ Routes to `gpt-5.2` (contains "analyze")

## Pattern 2: BMAD Prose Pattern

Natural language instruction pattern used in BMAD workflows:

```markdown
**Try to use Task tool [model: <model>] to spawn a subprocess:**

"The task prompt goes here...

Return structured findings."

**Graceful degradation (if no Task tool):**
- Fallback instruction 1
- Fallback instruction 2
```

### Processing Steps

1. **Detect trigger**: `**Try to use Task tool` followed by `to spawn a subprocess:**`
2. **Extract model** (if present): `(model: <value>)` between "Task tool" and "to spawn"
3. **Extract prompt**: Quoted text block following trigger
4. **Extract graceful degradation**: Content after `**Graceful degradation` marker
5. **Build structured prompt** with escalation protocol
6. **Attempt invocation**: `Task(subagent_type: "delegate-{model-id}", prompt: "<constructed>")`
7. **Handle response or degradation**: If Task fails, execute graceful degradation instructions

### Examples

**No model (heuristics apply):**
```markdown
**Try to use Task tool to spawn a subprocess:**

"Perform information density validation:
1. Scan for anti-patterns
2. Count violations
3. Return structured findings."

**Graceful degradation (if no Task tool):**
- Perform analysis directly in current context
```
→ Routes to `composer-1` (contains "validation", "scan")

**With explicit model:**
```markdown
**Try to use Task tool (model: analysis) to spawn a subprocess:**

"Evaluate PRD from multiple perspectives:
- Document flow & coherence
- Dual audience effectiveness
- Overall quality rating"

**Graceful degradation (if no Task tool):**
- Read complete PRD
- Evaluate directly using criteria above
```
→ Routes to `gpt-5.2` (explicit `model: analysis`)

**Using Advanced Elicitation:**
```markdown
**Try to use Task tool (model: reasoning) to spawn a subprocess using Advanced Elicitation:**

"Perform holistic quality assessment..."

**Graceful degradation (if Advanced Elicitation unavailable):**
- Perform assessment directly
```
→ Routes to `opus-4.6-thinking` (explicit `model: reasoning`)

## Pattern Recognition Notes

- Both patterns inject escalation protocol automatically
- BMAD pattern includes graceful degradation fallback
- XML pattern relies on Task tool availability (no built-in fallback)
- Model can be explicit or determined via heuristics
- Subagents provide automatic context isolation

## Fallback Options

If Task tool fails or is broken:

1. **cursor-agent CLI**: Use `.claude/skills/cursor-agent-cli/SKILL.md` for CLI-based delegation
   - Provides same isolation benefits
   - Requires cursor-agent installation
   - Use same model selection and safeguards

2. **Inline execution**: Execute task directly in current context (loses isolation benefits)
