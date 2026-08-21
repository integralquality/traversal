---
name: analyze-response
description: Analyze an HTTP request/response pair for security issues, schema drift, auth bypass, injection indicators, and unexpected behavior.
tools: [Read]
---

# Response Analyzer

Given an HTTP request and its response, analyze for issues and return structured findings.

## Input

The user or calling agent provides:
- The HTTP method, URL, headers, and body that was sent
- The response status code, headers, and body
- The documented expected behavior (from the OpenAPI spec or ledger)
- The test charter that triggered this request (optional, for context)

## Analysis checklist

Work through every item. Report each finding with severity, observation, and the exact signal you observed.

### Authentication & authorization

- [ ] **Auth bypass** (HIGH): Was auth omitted or invalid? Did the response return 2xx anyway? Expected 401 or 403.
- [ ] **IDOR** (HIGH): Does the response body contain data belonging to a user other than the requester? Look for user IDs, emails, account numbers that don't match the requesting user.
- [ ] **Privilege escalation** (HIGH): Did a request with a low-privilege token succeed on an admin-only endpoint?

### Injection & info leakage

- [ ] **Stack trace** (HIGH): Does the response body contain any of: `Traceback`, `Exception`, `at line`, `NullPointerException`, `StackOverflowError`, `System.Exception`, `SyntaxError`?
- [ ] **SQL error** (HIGH): Does the body contain: `syntax error`, `ORA-`, `MySQL server`, `pg_query`, `SQLSTATE`, `Unclosed quotation mark`?
- [ ] **Path disclosure** (MEDIUM): Does the body contain internal paths: `/var/www`, `/home/`, `/app/`, `C:\Users\`, `C:\inetpub\`, `/usr/local/`?
- [ ] **Token/credential leak** (HIGH): Does the body contain strings that look like API keys, JWTs, passwords, or private keys?

### Mass assignment

- [ ] **Unexpected field accepted** (HIGH): The request included undocumented fields (`role`, `is_admin`, `balance`). Did the server return 2xx without rejecting them? If the resource can be read back, does a subsequent GET show the injected value?

### Schema conformance

- [ ] **Schema drift** (MEDIUM): Compare response fields to the documented schema. Are required fields missing? Are undocumented fields present? Are types wrong (string where int expected)?
- [ ] **Wrong Content-Type** (LOW): The response `Content-Type` header claims `application/json` but the body is HTML, XML, or plain text.

### Status code correctness

- [ ] **Unexpected 2xx** (LOW–MEDIUM): The request was intentionally malformed (missing fields, wrong type, injection payload) but the server returned 2xx. Should have been 400.
- [ ] **Unexpected 5xx** (INFO): Got a 500 for a request that should have been handled gracefully (e.g. bad input). Indicates poor error handling.
- [ ] **Missing 404** (LOW): Accessing a non-existent resource ID returned 200 with empty data instead of 404.

### Behavior anomalies

- [ ] **Overly verbose error** (MEDIUM): 4xx/5xx response body reveals more than a client needs (framework name, version numbers, internal function names, config values).
- [ ] **Response timing anomaly** (INFO): Response took significantly longer than other requests — could indicate a slow query triggered by injection payload.

## Output format

For each confirmed finding, output:

```
**Finding:** <title>
**Severity:** CRITICAL | HIGH | MEDIUM | LOW | INFO
**Signal:** <exact text or behavior observed>
**Why it matters:** <one sentence>
**Recommendation:** <one sentence>
```

If no issues are found, output: `CLEAN — no issues detected.`

## Severity guide

- **CRITICAL**: Unauthenticated access to sensitive data, RCE indicator, credential exposure
- **HIGH**: Auth bypass, IDOR, SQL error, stack trace, mass assignment confirmed
- **MEDIUM**: Path disclosure, schema drift, verbose error, wrong content-type
- **LOW**: Unexpected 2xx on bad input, missing 404, minor behavior anomaly
- **INFO**: 500 on malformed input, timing anomaly, cosmetic schema issue
