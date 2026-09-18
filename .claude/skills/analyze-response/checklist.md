# Response Analysis Checklist

## Authentication & Authorization

- [ ] **Auth bypass** (HIGH)
  Was auth omitted or invalid? Did the server return 2xx?
  Expected 401 or 403.

- [ ] **IDOR** (HIGH)
  Does the response body contain data belonging to a different user?
  Look for: user IDs, emails, account numbers, names that don't match the requester.

- [ ] **Privilege escalation** (HIGH)
  Did a low-privilege token succeed on an endpoint that requires higher privileges?

## Injection & Info Leakage

- [ ] **Stack trace** (HIGH)
  Response body contains any of:
  `Traceback`, `Exception in thread`, `at line`, `NullPointerException`,
  `StackOverflowError`, `System.Exception`, `SyntaxError`, `at Object.`

- [ ] **SQL error** (HIGH)
  Response body contains any of:
  `syntax error`, `ORA-`, `MySQL server`, `pg_query`, `SQLSTATE`,
  `Unclosed quotation mark`, `quoted string not properly terminated`

- [ ] **Path disclosure** (MEDIUM)
  Response body contains internal paths:
  `/var/www`, `/home/`, `/app/`, `/usr/local/`, `C:\Users\`, `C:\inetpub\`

- [ ] **Token or credential leak** (HIGH)
  Response body contains strings that look like API keys, JWTs (`eyJ`...), passwords,
  private keys (`-----BEGIN`), or connection strings.

## Mass Assignment

- [ ] **Undocumented field accepted** (HIGH)
  The request included fields like `role`, `is_admin`, `balance`, `permissions`.
  Server returned 2xx without rejecting them.
  Confirm by reading the resource back with GET — did the injected value persist?

## Schema Conformance

- [ ] **Schema drift** (MEDIUM)
  Compare response fields to the documented schema:
  - Required fields missing from the response?
  - Undocumented fields present?
  - Types wrong (string where int expected)?

- [ ] **Wrong Content-Type** (LOW)
  Response header says `application/json` but body is HTML, XML, or plain text.
  Or vice versa.

## Status Code Correctness

- [ ] **Unexpected 2xx on bad input** (LOW–MEDIUM)
  The request was intentionally malformed (missing field, wrong type, injection payload)
  but the server returned 2xx. Should have been 4xx.

- [ ] **Unexpected 5xx** (INFO)
  Got 500 for a request that should be handled gracefully.
  Indicates poor error handling even if no sensitive data leaked.

- [ ] **Missing 404** (LOW)
  Accessing a non-existent resource ID returned 200 with empty data instead of 404.

## Response Quality

- [ ] **Overly verbose error** (MEDIUM)
  4xx/5xx body reveals more than a client needs:
  framework name/version, internal function names, config values, server paths.

- [ ] **Timing anomaly** (INFO)
  Response took significantly longer than baseline requests.
  Could indicate a slow query triggered by an injection payload (blind SQLi signal).
