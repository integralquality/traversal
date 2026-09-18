---
name: auth-probe
description: Sweep all explored endpoints for authentication and authorization issues. Tests every endpoint without auth, with a bad token, and with cross-user tokens using IDs from session memory.
tools: [Read, Write, Edit, Bash]
---

# Auth Probe

Dedicated authorization sweep. Revisits explored endpoints with targeted auth attacks rather than broad exploration.

## When to use

Run after `/orchestrator` completes, or any time you want a focused auth audit without re-exploring everything else.

## Setup

Read `test-ledger.json` and `session-memory.json`. If neither exists, tell the user to run `/bootstrap-ledger` first.

From the ledger: collect every endpoint with status `"explored"` or `"in_progress"`.
From session memory: get the auth header, user ID, and `known_ids` for IDOR tests.

If `known_ids` is empty, note that IDOR tests will use sequential IDs (less reliable).

## Tests to run per endpoint

For each endpoint that `requires_auth: true` in the ledger:

### 1. No-auth test
Send the happy-path request with the auth header completely omitted.
- Expected: 401 Unauthorized
- Issue if: 2xx returned → **auth bypass**

### 2. Malformed token
Send with `Authorization: Bearer INVALID_TOKEN_TRAVERSAL_TEST`.
- Expected: 401 Unauthorized
- Issue if: 2xx returned → **token not validated**

### 3. Cross-user IDOR (if the path contains a resource ID parameter)
If `session-memory.json` has `known_ids` for this endpoint's path pattern:
- Authenticate as the primary user
- Request a resource ID that belongs to a different user (pick the second ID in `known_ids` if available, otherwise try primary_id + 1 and primary_id - 1)
- Expected: 403 Forbidden or 404 Not Found
- Issue if: 200 returned with another user's data → **IDOR**

For endpoints that don't require auth per the spec, still run the no-auth test — undocumented auth requirements (or lack thereof) are worth flagging.

## Curl template

```bash
# No-auth
curl -s -w "\n---STATUS:%{http_code}---" -X <METHOD> '<base_url><path>'

# Bad token
curl -s -w "\n---STATUS:%{http_code}---" -X <METHOD> '<base_url><path>' \
  -H 'Authorization: Bearer INVALID_TOKEN_TRAVERSAL_TEST'

# Cross-user
curl -s -w "\n---STATUS:%{http_code}---" -X <METHOD> '<base_url><path_with_other_id>' \
  -H '<auth_header>'
```

## Issue logging

For each finding, append to `issues.md` using the standard format. Severity:
- `HIGH` for auth bypass (no-auth or bad-token returns 2xx on a protected endpoint)
- `HIGH` for IDOR (another user's resource returned)
- `MEDIUM` for inconsistent behavior (auth required on POST but not GET for the same resource)

## Summary

When done, report:
- Endpoints tested
- Auth bypass findings
- IDOR findings
- Endpoints where auth behavior differs from the spec
