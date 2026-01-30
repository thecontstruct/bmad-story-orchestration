# Validate Using Adversarial Review

Execute adversarial review via delegation and return validation outcome compatible with retry-with-feedback task.

**Prerequisites:**
- Load subprocess delegation skill: `{project-root}/.cursor/skills/subprocess-delegation/SKILL.md`
- This task uses `<delegate>` blocks which require the skill for proper interpretation

---

## CONTEXT VALIDATION

Before executing, verify required context variables are available.

### Required Variables

| Variable | Description | Source |
|----------|-------------|--------|
| `document` | Document/diff to validate | From retry-with-feedback task context |
| `project_root` | Project root path | From retry-with-feedback task context |
| `adversarial_review_task` | Path to adversarial review task | `{project-root}/_bmad/core/tasks/review-adversarial-general.xml` |

### Optional Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `also_consider` | Additional areas to consider | Context-specific (story vs code) |
| `content_type` | Type of content | Auto-detect from document |
| `adversarial_model` | Model for adversarial review delegation | "gemini-3-pro" |

---

## EXECUTION FLOW

### Step 1: Load Document

1. Load document from `{{document}}` parameter (provided by retry-with-feedback task)
2. If document is empty or not found, return error:
   ```yaml
   outcome: "ISSUES_FOUND"
   feedback: "Document not found or empty: {{document}}"
   ```
3. Identify content type (story file, code diff, etc.)
4. Capture full content as `{content}`

### Step 2: Invoke Adversarial Review via Delegation

**CRITICAL:** Use delegation to ensure information asymmetry - fresh context with no prior knowledge.

**Model for adversarial review:** 

```xml
<delegate model="gemini-3-pro">
  <system-instructions>
    You are executing an adversarial review task in isolated context.
    - Follow review-adversarial-general.xml instructions precisely
    - Do not spawn additional agents or delegation blocks
    - Review with extreme skepticism - assume problems exist
    - Find at least 10 issues (mandatory minimum)
    - Use professional, precise tone
  </system-instructions>

  <context>
    - adversarial_review_task: {project-root}/_bmad/core/tasks/review-adversarial-general.xml
    - project_root: {project-root}
    - content: {content}
    - also_consider: {also_consider}
  </context>

  <prompt>
    Execute adversarial review:
    1. Load {project-root}/_bmad/core/tasks/review-adversarial-general.xml
    2. Follow task instructions to review the content
    3. Find at least 10 issues to fix or improve
    4. Output findings as a Markdown list
    5. Evaluate severity for each finding (Critical, High, Medium, Low)
  </prompt>

  <output>
    - Keep response concise but complete
    - findings: Markdown list of all issues found (minimum 10)
    - finding_count: Total number of findings
    - severity_breakdown: Count of Critical/High/Medium/Low findings
    - findings_with_severity: List with each finding labeled by severity
  </output>
</delegate>
```

### Step 3: Parse Adversarial Review Results

1. Extract findings list from delegation output
2. Count total findings: `{finding_count}`
3. Parse severity for each finding
4. Count by severity:
   - `{critical_count}` - Critical severity findings
   - `{high_count}` - High severity findings
   - `{medium_count}` - Medium severity findings
   - `{low_count}` - Low severity findings

**IF finding_count == 0:**
- This is suspicious - adversarial review must find at least 10 issues
- Return `ISSUES_FOUND` with feedback: "Adversarial review found zero issues - this is suspicious. Review task may have failed or content may be perfect (rare)."

### Step 4: Determine Validation Outcome

Evaluate findings to determine outcome:

**IF critical_count > 0 OR high_count > 3:**
- Set `outcome = "ISSUES_FOUND"`
- These are significant issues that need fixing

**IF critical_count == 0 AND high_count <= 3:**
- Set `outcome = "PASS"`
- Only minor issues present, acceptable to proceed

### Step 5: Format Feedback Report

Create structured feedback report:

```markdown
# Adversarial Review Validation Report

**Document:** {{document}}
**Content Type:** {{content_type}}
**Date:** {{timestamp}}

## Summary

- **Total Findings:** {{finding_count}}
- **Critical:** {{critical_count}}
- **High:** {{high_count}}
- **Medium:** {{medium_count}}
- **Low:** {{low_count}}
- **Outcome:** {{outcome}}

## Findings

{{for each finding:}}
### F{{finding_number}}: [Severity] - {{finding_description}}

{{finding_details}}

{{end for}}

## Recommendations

### Must Fix (Critical/High)
{{list critical and high findings with actionable fixes}}

### Should Improve (Medium)
{{list medium findings with suggestions}}

### Consider (Low)
{{list low findings}}
```

### Step 6: Save Validation Report

1. Create report file: `{{document}}.adversarial-validation-report-{{timestamp}}.md`
2. Save formatted feedback report to file
3. Store path as `{validation_report_path}`

### Step 7: Return Validation Result

Return structured result compatible with retry-with-feedback:

```yaml
outcome: {{outcome}}  # PASS | ISSUES_FOUND
finding_count: {{finding_count}}
critical_findings: {{critical_count}}
high_findings: {{high_count}}
medium_findings: {{medium_count}}
low_findings: {{low_count}}
feedback: |
  {{formatted_feedback_report}}
validation_report_path: {{validation_report_path}}
```

---

## USAGE NOTES

### Invoked by Retry-with-Feedback

This task is invoked by the retry-with-feedback task via delegation. The retry task provides:
- `document` - Path to document/diff to validate
- `project_root` - Project root path

### Context-Specific Considerations

**For Story Review (Step 2b):**
- `also_consider`: "Story context completeness, developer implementation guidance, missing technical details, vague requirements, architectural compliance"
- `content_type`: "story"

**For Code Review (Step 4b):**
- `also_consider`: "Code quality, security vulnerabilities, performance issues, architectural compliance, test coverage, error handling, edge cases, code smells, missing validations"
- `content_type`: "code_diff"

### Return Format

Must return exactly:
- `outcome`: PASS | ISSUES_FOUND (string)
- `feedback`: Detailed feedback (string, can be multi-line markdown)
- `validation_report_path`: Path to saved report (string)

Additional fields (finding_count, critical_findings, etc.) are optional but helpful for debugging.

---

## ERROR HANDLING

### Document Not Found
- Return `ISSUES_FOUND` with clear error message
- Do not proceed with review

### Adversarial Review Fails
- Return `ISSUES_FOUND` with error details
- Include information about what went wrong

### Zero Findings
- Return `ISSUES_FOUND` (zero findings is itself an issue)
- Include note that this is suspicious

### Delegation Not Available
- Fallback: Load review-adversarial-general.xml directly
- Execute inline (less ideal but functional)
- Still return proper format
