---
name: bootstrap-ledger
description: Initialize test-ledger.json from an OpenAPI spec file. Run this once before starting exploration.
tools: [Read, Write, Bash]
---

# Bootstrap Ledger

Initialize `test-ledger.json` in the current directory from an OpenAPI spec file.

## Input

Called as: `/bootstrap-ledger <spec_path> [base_url] [auth_header]`

- `spec_path` — path to an OpenAPI 2.x or 3.x spec (JSON or YAML)
- `base_url` — override the server URL from the spec (optional)
- `auth_header` — full header string e.g. `"Authorization: Bearer mytoken"` (optional)

## Steps

### 1. Read and parse the spec

Read the spec file. If it is YAML, convert it:
```bash
python3 -c "import yaml,json,sys; print(json.dumps(yaml.safe_load(sys.stdin)))" < <spec_path>
```

### 2. Extract endpoints

For each path + method combination in `paths`:
- Method must be one of: GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS
- Extract: summary, parameters (name, in, required, schema), requestBody, response codes, security requirements, tags

Resolve the base URL:
- Use `base_url` argument if provided
- Otherwise use `servers[0].url` (OpenAPI 3.x) or `host` + `basePath` (OpenAPI 2.x)
- Strip trailing slash from base URL

### 3. Assign priorities

- **high**: POST, PUT, PATCH, DELETE methods; paths containing "admin", "auth", "login", "token", "password", "reset", "upload"
- **low**: GET with no path parameters and no query parameters
- **medium**: everything else

### 4. Write test-ledger.json

```json
{
  "api_title": "<info.title>",
  "base_url": "<resolved base URL>",
  "spec_path": "<spec_path>",
  "started_at": "<current ISO 8601 timestamp>",
  "session": {
    "auth_header": "<auth_header or empty string>",
    "total_requests": 0,
    "total_issues": 0
  },
  "endpoints": {
    "<METHOD> <path>": {
      "status": "unexplored",
      "priority": "<high|medium|low>",
      "summary": "<summary or empty string>",
      "tags": ["<tag>"],
      "requires_auth": <true if security is non-empty on the operation or globally>,
      "parameters": [
        {"name": "<name>", "in": "<path|query|header>", "required": <bool>, "type": "<type>"}
      ],
      "request_body_schema": <schema object or null>,
      "response_codes": ["200", "400", "404"],
      "charters_run": [],
      "issue_count": 0
    }
  }
}
```

If `test-ledger.json` already exists, ask the user whether to overwrite it or merge (add only new endpoints without touching existing ones).

### 5. Confirm

Report to the user:
- Total endpoints loaded
- Count by priority (high / medium / low)
- Base URL being used
- Auth header set (yes/no, do not print the actual token)
- Whether this is a fresh ledger or a merge
