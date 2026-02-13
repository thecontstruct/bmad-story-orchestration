# Aggregation Rules

**Purpose:** Rules for aggregating findings from per-file and cross-file review phases into unified severity counts.

---

## Aggregation Process

After both review phases complete, aggregate findings:

### 1. Collect Per-File Results

For each file reviewed:
- Extract `outcome`, `finding_count`, severity counts (`critical_count`, `high_count`, `medium_count`, `low_count`)
- Extract `feedback` (detailed findings)
- Extract `cross_file_refs` (noted cross-file references)

### 2. Collect Cross-File Results

From the cross-file review:
- Extract `outcome`, `finding_count`, severity counts
- Extract `feedback` (cross-file findings with affected files per finding)

### 3. Compute Aggregate Counts

Sum severity counts across ALL phases:

```
aggregate_critical = sum(per_file_critical for each file) + cross_file_critical
aggregate_high = sum(per_file_high for each file) + cross_file_high
aggregate_medium = sum(per_file_medium for each file) + cross_file_medium
aggregate_low = sum(per_file_low for each file) + cross_file_low
```

### 4. Determine Aggregate Outcome

- If ALL per-file outcomes are PASS AND cross-file outcome is PASS → `aggregate_outcome = PASS`
- Otherwise → `aggregate_outcome = ISSUES_FOUND`

### 5. Build Per-File Summary for Cross-File Review

Before running the cross-file review, summarize per-file findings:

```
Per-file findings summary:
- {file_path_1}: {outcome} (C:{critical} H:{high} M:{medium} L:{low})
  Key findings: {brief list}
- {file_path_2}: {outcome} (C:{critical} H:{high} M:{medium} L:{low})
  Key findings: {brief list}
```

This summary is passed to the cross-file review delegation as `{per_file_summary}` so it can avoid duplicating per-file issues.

---

## Fix Routing Rules

Based on aggregated findings, route fixes by scope:

### Per-File Issues → Single-File Fix Delegations

For each file with issues:
- Collect all per-file findings for that file
- Delegate a single-file fix using the Single-File Fix Delegation Template
- Pass that file's specific feedback

### Cross-File Issues → Cross-File Fix Delegation

If cross-file issues exist:
- Collect all cross-file findings
- Delegate a cross-file fix using the Cross-File Fix Delegation Template
- Pass all affected file paths and cross-file feedback

### Fix Ordering

1. Run single-file fix delegations first (can be sequential)
2. Run cross-file fix delegation after (needs to see single-file fixes applied)

This ordering ensures cross-file fixes operate on already-patched individual files.
