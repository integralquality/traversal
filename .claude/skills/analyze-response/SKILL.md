---
description: Analyze an HTTP request/response pair for security issues, schema drift, auth bypass, and unexpected behavior.
argument-hint: <METHOD URL> <status> <response body>
---

Work through the checklist in `checklist.md` (in this skill directory) against the provided request and response.

## Input

The user provides:
- The HTTP method, URL, headers, and body that was sent
- The response status code, headers, and body
- What the endpoint was supposed to do (from spec or ledger)
- Which test charter triggered this request (optional)

## Output

For each confirmed finding:

```
Finding:        <title>
Severity:       CRITICAL | HIGH | MEDIUM | LOW | INFO
Signal:         <exact text or behavior observed>
Why it matters: <one sentence>
Recommendation: <one sentence>
```

If nothing found: `CLEAN — no issues detected.`

Work through every checklist category before concluding CLEAN. Do not skip categories because the status code looks normal.

## Severity guide

- **CRITICAL**: Unauthenticated access to sensitive data, RCE indicator, credential exposure
- **HIGH**: Auth bypass, IDOR, SQL error in response, stack trace, mass assignment confirmed
- **MEDIUM**: Path disclosure, schema drift, verbose error message, wrong Content-Type
- **LOW**: Unexpected 2xx on malformed input, missing 404, minor behavior anomaly
- **INFO**: 500 on bad input (poor error handling but not a leak), timing anomaly
