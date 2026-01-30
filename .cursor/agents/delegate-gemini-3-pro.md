---
name: delegate-gemini-3-pro
model: gemini-3-pro
description: Execute delegated work with gemini-3-pro. Invoke via /delegate-gemini-3-pro for code review.
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
