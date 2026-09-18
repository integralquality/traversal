---
description: Initialize test-ledger.json from an OpenAPI spec file. Run once before starting exploration.
argument-hint: <spec_path> [base_url] [auth_header]
---

Initialize `test-ledger.json` and `session-memory.json` in the current directory.

## Arguments

- `spec_path` — path to an OpenAPI 2.x or 3.x spec (JSON or YAML) — required
- `base_url` — override the server URL from the spec — optional
- `auth_header` — full header string e.g. `"Authorization: Bearer mytoken"` — optional

## Steps

### 1. Parse the spec

Read the spec file. If YAML, convert:
```bash
python3 -c "import yaml,json,sys; print(json.dumps(yaml.safe_load(sys.stdin)))" < <spec_path>
```

### 2. Extract endpoints

For each `paths` entry × HTTP method (GET, POST, PUT, PATCH, DELETE, HEAD, OPTIONS):
- summary, parameters (name/in/required/schema), requestBody schema, response codes, security, tags

Resolve base URL: use argument if provided, else `servers[0].url` (OpenAPI 3.x) or `host + basePath` (2.x). Strip trailing slash.

### 3. Assign priorities

- **high** — POST, PUT, PATCH, DELETE; paths containing `admin`, `auth`, `login`, `token`, `password`, `reset`, `upload`, `import`
- **low** — GET with no path parameters and no query parameters
- **medium** — everything else

### 4. Write test-ledger.json

```json
{
  "api_title": "<info.title>",
  "base_url": "<base_url>",
  "spec_path": "<spec_path>",
  "started_at": "<ISO 8601 timestamp>",
  "session": {
    "auth_header": "<auth_header or empty>",
    "total_requests": 0,
    "total_issues": 0
  },
  "endpoints": {
    "<METHOD> <path>": {
      "status": "unexplored",
      "priority": "<high|medium|low>",
      "summary": "<summary>",
      "tags": [],
      "requires_auth": <bool>,
      "parameters": [{"name":"","in":"","required":false,"type":""}],
      "request_body_schema": null,
      "response_codes": ["200"],
      "charters_run": [],
      "issue_count": 0
    }
  }
}
```

If `test-ledger.json` already exists, ask: overwrite or merge (add only new endpoints)?

### 5. Write session-memory.json (if missing)

```json
{
  "auth": { "header": "<auth_header>", "user_id": "", "username": "" },
  "created_resources": [],
  "known_ids": {},
  "notes": ""
}
```

Preserve existing file if it exists.

### 6. Confirm

Report: endpoint count by priority, base URL, auth set (yes/no — don't print the token), fresh vs. merge.
