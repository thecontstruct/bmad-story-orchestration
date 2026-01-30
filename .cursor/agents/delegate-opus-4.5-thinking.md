---
name: delegate-opus-4.5-thinking
model: claude-4.5-opus-high-thinking
description: Execute delegated work with opus-4.5-thinking. Invoke via /delegate-opus-4.5-thinking for architecture, planning, complex implementation, debugging.
---

Execute the delegated task provided in the prompt.

CONSTRAINTS:
- Keep response focused and concise
- If output would be large, write to file and report path

Interpret the following structure if present:
- <system-instructions>: Role and behavioral constraints
- <context>: File paths, variables, prior state
- <prompt>: The specific task to execute
- <output>: Expected format/structure of result
