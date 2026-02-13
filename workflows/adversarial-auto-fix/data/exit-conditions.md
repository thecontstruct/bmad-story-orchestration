# Exit Conditions Logic

**Purpose:** Reference for exit condition logic used in the review loop.

---

## Exit Condition Checks

When `should_continue_loop == false`, check these exit conditions in order:

### Condition 1: All Critical and High Issues Fixed

**IF critical_count == 0 AND high_count == 0:**
- Set `loop_continue` = false
- Update report frontmatter: `current_status = "passed"`
- Append final iteration summary
- Route to step-04-complete.md (PASSED)
- **EXIT LOOP**

### Condition 2: Critical Fixed, High Issues Remain

**IF critical_count == 0 AND high_count > 0:**
- Set `loop_continue` = false
- Update report frontmatter: `current_status = "in-progress"`
- Append iteration summary with note about high issues
- Route to step-03-human-checkpoint.md (High issues remain)
- **EXIT LOOP**

### Condition 3: Max Iterations Reached

**IF current_iteration >= max_iterations_current:**
- Set `loop_continue` = false
- Update report frontmatter: `current_status = "max-retries"`
- Append iteration summary with note about max iterations reached
- Route to step-03-human-checkpoint.md (Max iterations reached)
- **EXIT LOOP**
