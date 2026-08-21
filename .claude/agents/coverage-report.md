---
name: coverage-report
description: Generate a coverage report from test-ledger.json and issues.md. Shows explored endpoints, issue breakdown by severity, and gaps.
tools: [Read, Write]
---

# Coverage Report Generator

Read `test-ledger.json` and `issues.md` from the current directory and produce a structured report.

## Steps

### 1. Read inputs

- Read `test-ledger.json` — endpoint statuses, charter counts, issue counts, session totals
- Read `issues.md` — parse all ISSUE-NNN entries to count by severity and group by endpoint

If either file is missing, note it and work with what's available.

### 2. Compute statistics

- **Coverage %**: `(explored + in_progress) / total * 100`
- **Issue breakdown**: count CRITICAL, HIGH, MEDIUM, LOW, INFO
- **Top endpoints by issue count**: sort endpoints by `issue_count` descending
- **Unexplored endpoints**: list all with status `"unexplored"`
- **Charters run**: total across all endpoints

### 3. Output the report

Write the report to `coverage-report.md` and also print it:

```markdown
# API Exploration Coverage Report

**API:** <api_title>
**Base URL:** <base_url>
**Generated:** <current timestamp>

## Summary

| Metric | Value |
|---|---|
| Total endpoints | N |
| Explored | N (X%) |
| In progress | N |
| Unexplored | N |
| Total requests made | N |
| Total issues found | N |
| Charters run | N |

## Issues by Severity

| Severity | Count |
|---|---|
| CRITICAL | N |
| HIGH | N |
| MEDIUM | N |
| LOW | N |
| INFO | N |

## Top Issues

<list the 5 highest-severity issues with their ISSUE-NNN ID, severity, title, and endpoint>

## Endpoints with Issues

<for each endpoint that has issue_count > 0, list: METHOD /path — N issues>

## Coverage Gaps

### Unexplored (prioritized)

<list unexplored endpoints sorted by priority: high first, then medium, then low>

### Partially explored

<list endpoints with status "in_progress">

## Recommendations

<Based on what was found, provide 3–5 specific recommendations. Examples:
- "ISSUE-003 (mass assignment on PATCH /users) should be verified manually with a real admin account"
- "POST /payments was not explored — it is high priority and should be tested next"
- "All auth bypass findings cluster on unauthenticated GET endpoints — consider a global middleware audit">
```

### 4. Done

Inform the user where `coverage-report.md` was written.
