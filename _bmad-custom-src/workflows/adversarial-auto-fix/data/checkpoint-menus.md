# Checkpoint Menu Templates

**Purpose:** Menu display templates for human checkpoint step.

---

## Checkpoint Context Display - High Issues Remain

```
⚠️ Human Checkpoint: High Issues Remain

**Status:**
- Critical issues: All fixed ✅
- High issues: {high_count} remaining ⚠️
  - Per-file: {per_file_high_count}
  - Cross-file: {cross_file_high_count}
- Medium issues: {medium_count}
- Low issues: {low_count}

**Files Reviewed:** {file_count}
**Iterations Completed:** {iteration_count}

Critical issues have been automatically fixed. However, {high_count} high-severity issues remain that require your attention.

**What would you like to do?**
```

---

## Checkpoint Context Display - Max Iterations Reached

```
⚠️ Human Checkpoint: Maximum Iterations Reached

**Status:**
- Iterations completed: {iteration_count}/{max_iterations_current}
- Critical issues: {critical_count} remaining
- High issues: {high_count} remaining
  - Per-file: {per_file_high_count}
  - Cross-file: {cross_file_high_count}
- Medium issues: {medium_count}
- Low issues: {low_count}

**Files Reviewed:** {file_count}

The maximum iteration limit ({max_iterations_current}) has been reached. Some issues may still remain.

**What would you like to do?**
```

---

## Menu Options - High Issues Remain

```
**Select an Option:**

[M]anual fix - Exit for manual fixes (preserve current state)
[S]kip and continue - Proceed despite high issues (mark as ACCEPTED)
[A]uto-fix anyway - Continue auto-fixing high issues (not recommended)
[Q]uit - Abort workflow (mark as EXITED)
```

---

## Menu Options - Max Iterations Reached

```
**Select an Option:**

[M]anual fix - Exit for manual fixes (preserve current state)
[S]kip and continue - Proceed despite remaining issues (mark as ACCEPTED)
[A]uto-fix anyway - Extend iterations and continue (extend by {default_extension})
[Q]uit - Abort workflow (mark as EXITED)
```
