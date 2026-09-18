---
description: Generate a coverage report from test-ledger.json and issues.md. Writes coverage-report.md and prints a summary.
---

Read `test-ledger.json` and `issues.md` from the current directory and produce a structured report.

## Steps

### 1. Read and compute

From `test-ledger.json`:
- Total endpoints, count by status (`unexplored` / `in_progress` / `explored`)
- Total requests, total issues, charters run
- Endpoints sorted by `issue_count` descending

From `issues.md`:
- Parse all `## ISSUE-NNN` entries
- Count by severity: CRITICAL / HIGH / MEDIUM / LOW / INFO
- Group by endpoint

If either file is missing, note it and work with what's available.

### 2. Write coverage-report.md

```markdown
# API Exploration Coverage Report

**API:** <api_title>  
**Base URL:** <base_url>  
**Generated:** <timestamp>

## Summary

| Metric | Value |
|---|---|
| Total endpoints | N |
| Explored | N (X%) |
| In progress | N |
| Unexplored | N |
| Total requests | N |
| Charters run | N |
| Issues found | N |

## Issues by Severity

| Severity | Count |
|---|---|
| CRITICAL | N |
| HIGH | N |
| MEDIUM | N |
| LOW | N |
| INFO | N |

## Top Findings

<5 highest-severity issues: ISSUE-NNN · SEVERITY · title · endpoint>

## Endpoints with Issues

<endpoint — N issues, for each endpoint with issue_count > 0>

## Coverage Gaps

### Unexplored (by priority)

<unexplored endpoints: high first, then medium, then low>

## Recommendations

<3–5 specific, actionable recommendations based on what was found.
Reference specific issue IDs and endpoints. Not generic advice.>
```

### 3. Print the summary section and tell the user where the full report was written.
