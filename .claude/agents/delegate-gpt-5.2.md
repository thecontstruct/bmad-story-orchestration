---
name: delegate-gpt-5.2
description: Execute delegated work with gpt-5.2. Invoke via /delegate-gpt-5.2 for analysis tasks.
model: gpt-5.2
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
