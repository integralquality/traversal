---
description: Generate test charters for a single API endpoint. Provide the method, path, parameters, and schema.
argument-hint: <METHOD /path> [spec details or ledger path]
---

Given an API endpoint, produce a numbered list of test charters using the techniques in `techniques.md` (in this skill directory).

## Input

The user provides one or more of:
- HTTP method + path (e.g. `POST /users/{id}`)
- Parameter names, types, and constraints
- Request body schema
- Documented response codes and whether auth is required
- Related endpoints (e.g. the GET that reads what this POST creates)
- Path to `test-ledger.json` (read it for the full endpoint spec if available)

## Output format

Number each charter. For each:

```
### Charter N — <technique>: <short description>

Goal:     what this tests and what "wrong" looks like
Method:   <HTTP method>
Path:     <resolved path with example values>
Headers:  <Content-Type + auth if needed>
Body:     <JSON body or "(none)">
Expected: <status code and why>
Signal:   <what in the response indicates a bug>
```

## Charter selection

Always include:
1. Happy path — valid, realistic values
2. No-auth — omit the Authorization header (if endpoint uses auth)
3. At least one boundary test per numeric or string parameter

Add where relevant:
- Missing required field (one per required field)
- Wrong type for a key field
- Null for a required field
- Extra undocumented fields: `"role": "admin"`, `"is_admin": true` (mass assignment)
- SQL injection on a string field: `' OR '1'='1`
- XSS on a string field: `<script>alert(1)</script>`
- IDOR: path ID ± 1, or a known ID from `session-memory.json`'s `known_ids`
- State chain: for CRUD sets, create → verify → update → delete → verify gone

Order: auth tests first, happy path second, then security probes, then boundary/type tests.

Use realistic test data (`john.doe@example.com`, `"John Doe"`) not `test@test.com` or `"test"`.

See `techniques.md` for full rules per technique.
