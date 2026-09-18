---
description: View and manage session-memory.json — auth context, created resource IDs, and known IDs for IDOR testing.
argument-hint: [show | set-auth | add-id | add-note | clear]
---

Manages `session-memory.json` in the current directory.

## session-memory.json schema

```json
{
  "auth": {
    "header": "Authorization: Bearer <token>",
    "user_id": "",
    "username": ""
  },
  "created_resources": [
    {
      "method": "POST",
      "path": "/users",
      "id": "123",
      "cleanup": "DELETE /users/123",
      "response_summary": "name: John, email: john@example.com"
    }
  ],
  "known_ids": {
    "GET /users/{id}": ["42", "123"]
  },
  "notes": ""
}
```

## Subcommands

**`/session`** — print a readable summary. Mask the auth token (show first 8 chars + `...`).

**`/session set-auth <header>`** — update `auth.header`. Optionally prompt for user_id.
Example: `/session set-auth "Authorization: Bearer eyJhbG..."`

**`/session add-id <path_template> <id>`** — append an ID to `known_ids`.
Example: `/session add-id "GET /users/{id}" 99`
Use this to manually add a second user's resource ID for IDOR testing.

**`/session add-note <text>`** — append to `notes`.
Example: `/session add-note "user 42 is admin, user 123 is regular"`

**`/session clear`** — confirm, then delete `session-memory.json` and `test-ledger.json`.
Issues and coverage report are not deleted.

## How session-memory.json is populated automatically

The **explorer** agent writes to it during exploration:
- POST → 201: extracts the resource ID (looks for `.id`, `.uuid`, `._id`), adds to `created_resources` and `known_ids`
- First user profile endpoint (GET /me, /profile, /account): extracts `user_id` and `username` if not already set

If the auto-extraction is wrong, use `/session set-auth` and `/session add-id` to correct it.
