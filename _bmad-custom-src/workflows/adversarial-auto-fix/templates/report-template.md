---
workflow_id: "{{workflow_id}}"
workflow_name: "adversarial-auto-fix"
input_type: "{{input_type}}"
input_source: "{{input_source}}"
file_paths: []
review_guidance: "{{review_guidance}}"
stepsCompleted: []
lastStep: ''
iteration_count: 0
max_iterations: [5]
auto_fix: true
current_status: "in-progress"
created: "{{date}}"
user_name: "{{user_name}}"
---

# Adversarial Review Report

**Workflow ID:** {{workflow_id}}  
**Input:** {{input_source}} ({{file_count}} files)  
**Created:** {{date}}  
**User:** {{user_name}}

{% if review_guidance %}
**Review Guidance:** {{review_guidance}}
{% endif %}

## Files Under Review

| # | File | Type |
|---|------|------|
{{file_table_rows}}

## Iteration History

[Iteration summaries will be appended here progressively]

## Final Status

[Final status will be determined and documented here]

## Summary

- **Total Iterations:** {{iteration_count}}
- **Final Status:** {{current_status}}
- **Aggregate Issue Counts:**
  - Critical: {{critical_count}}
  - High: {{high_count}}
  - Medium: {{medium_count}}
  - Low: {{low_count}}
