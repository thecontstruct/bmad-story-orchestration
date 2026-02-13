# Delegation Templates

**Purpose:** XML templates and patterns for subprocess delegation used in the review loop.

---

## Per-File Review Delegation Template

```xml
<delegate model="{models.review}">
  <system-instructions>
    You are executing a validation task in isolated context.
    - Follow validation instructions precisely
    - Do not spawn additional agents or delegation blocks
    - Review with extreme skepticism - assume problems exist
    - Focus ONLY on the single file provided
    - Note any references to other files that may need cross-file review
  </system-instructions>

  <context>
    - validation_task: {validationTask}
    - project_root: {project-root}
    - document: {file_path}
    - also_consider: {review_guidance}
    - review_scope: single-file
  </context>

  <prompt>
    Execute adversarial review validation on a SINGLE FILE:
    1. Load {validationTask}
    2. Execute validation against the document at {file_path}
    3. Report all issues found with severity breakdown
    4. Flag any cross-file references or dependencies you notice
    5. Return structured result with severity counts
  </prompt>

  <output>
    - Keep response concise but complete
    - outcome: PASS | ISSUES_FOUND
    - finding_count: Total number of findings
    - critical_count: Count of Critical findings
    - high_count: Count of High findings
    - medium_count: Count of Medium findings
    - low_count: Count of Low findings
    - feedback: Detailed feedback report
    - cross_file_refs: List of cross-file references noted
  </output>
</delegate>
```

---

## Cross-File Review Delegation Template

```xml
<delegate model="{models.review}">
  <system-instructions>
    You are executing a CROSS-FILE validation task in isolated context.
    - Review the entire file set as a coherent system
    - Focus on inter-file issues: inconsistencies, contradictions, missing references, structural gaps
    - Do not duplicate per-file issues already found - focus on CROSS-FILE concerns only
    - Do not spawn additional agents or delegation blocks
    - Review with extreme skepticism - assume problems exist
  </system-instructions>

  <context>
    - validation_task: {validationTask}
    - project_root: {project-root}
    - file_set: {file_paths}
    - per_file_findings_summary: {per_file_summary}
    - also_consider: {review_guidance}
    - review_scope: cross-file
  </context>

  <prompt>
    Execute CROSS-FILE adversarial review:
    1. Load {validationTask}
    2. Read ALL files in the file set
    3. Also read any supporting files (yaml, config, etc.) referenced by the target files
    4. Review for cross-file issues:
       - Inconsistencies between files (conflicting information, contradictory instructions)
       - Missing references (file A references something file B should define but doesn't)
       - Structural gaps (missing files, incomplete coverage, broken flow)
       - Redundancy (duplicated content that could diverge)
       - Ordering/dependency issues (steps out of sequence, circular references)
    5. Consider per-file findings summary for context (avoid duplicating those issues)
    6. Return structured result with severity counts for CROSS-FILE issues only
  </prompt>

  <output>
    - Keep response concise but complete
    - outcome: PASS | ISSUES_FOUND
    - finding_count: Total number of cross-file findings
    - critical_count: Count of Critical findings
    - high_count: Count of High findings
    - medium_count: Count of Medium findings
    - low_count: Count of Low findings
    - feedback: Detailed feedback report with affected files listed per finding
  </output>
</delegate>
```

---

## Single-File Fix Delegation Template

```xml
<delegate model="{models.fix}">
  <system-instructions>
    You are executing a fix operation in isolated context.
    - CRITICAL: Read and address ALL validation feedback before proceeding
    - Fix ALL issues mentioned in feedback (critical, high, medium, low)
    - Apply fixes directly to the document file
    - Do not spawn additional agents or delegation blocks
  </system-instructions>

  <context>
    - file_path: {file_path}
    - validation_feedback: {feedback}
    - validation_report_path: {validation_report_path}
    - project_root: {project-root}
    - fix_scope: single-file
  </context>

  <prompt>
    Apply fixes to a SINGLE document:
    1. Load document from {file_path}
    2. Review validation feedback: {validation_report_path}
    3. Fix ALL issues mentioned in feedback:
       - Critical issues (must fix)
       - High issues (fix)
       - Medium issues (fix)
       - Low issues (fix)
    4. Apply fixes directly to the document file
    5. Save the fixed document
  </prompt>

  <output>
    - Keep response concise (under 500 words)
    - outcome: success | failure
    - fixes_applied: List of changes made
    - issues: Any problems encountered
  </output>
</delegate>
```

---

## Cross-File Fix Delegation Template

```xml
<delegate model="{models.fix}">
  <system-instructions>
    You are executing a CROSS-FILE fix operation in isolated context.
    - CRITICAL: Read and address ALL cross-file validation feedback
    - Fixes may require changes to MULTIPLE files to resolve a single issue
    - Ensure consistency across all affected files after fixes
    - Do not spawn additional agents or delegation blocks
  </system-instructions>

  <context>
    - file_paths: {file_paths}
    - cross_file_feedback: {cross_file_feedback}
    - validation_report_path: {validation_report_path}
    - project_root: {project-root}
    - fix_scope: cross-file
  </context>

  <prompt>
    Apply CROSS-FILE fixes:
    1. Load ALL affected files from {file_paths}
    2. Review cross-file validation feedback: {validation_report_path}
    3. Fix ALL cross-file issues mentioned in feedback:
       - Resolve inconsistencies between files
       - Add missing references
       - Fix structural gaps
       - Eliminate problematic redundancy
    4. Apply fixes directly to the affected files
    5. Save all modified files
    6. Verify cross-file consistency after fixes
  </prompt>

  <output>
    - Keep response concise (under 500 words)
    - outcome: success | failure
    - files_modified: List of files changed
    - fixes_applied: List of changes made (grouped by file)
    - issues: Any problems encountered
  </output>
</delegate>
```
