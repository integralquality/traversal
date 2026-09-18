---
name: session
description: View and manage session-memory.json — auth context, created resource IDs, and known IDs used for IDOR testing.
tools: [Read, Write, Edit]
---

# Session Memory

Manages `session-memory.json` in the current directory.

## session-memory.json schema

```json
{
  "auth": {
    "header": "Authorization: Bearer <token>",
    "user_id": "<extracted or manually set>",
    "username": "<extracted or manually set>"
  },
  "created_resources": [
    {
      "method": "POST",
      "path": "/users",
      "id": "123",
      "cleanup": "DELETE /users/123",
      "response_summary": "<key fields from the creation response>"
    }
  ],
  "known_ids": {
    "GET /users/{id}": ["42", "123"],
    "GET /posts/{postId}": ["1", "2", "3"]
  },
  "notes": ""
}
```

`known_ids` keys use the path template from the spec (with `{param}` placeholders), not resolved URLs.

## Commands

Called as: `/session [subcommand]`

### `/session` (no args) — show current state
Print a readable summary of `session-memory.json`:
- Auth header (mask the token — show only the first 8 chars + `...`)
- User ID and username if set
- Number of created resources, with cleanup URLs
- All known IDs grouped by endpoint

### `/session set-auth <header>`
Update `auth.header` in `session-memory.json`. Example:
```
/session set-auth "Authorization: Bearer eyJhbG..."
```
Also prompt: "Do you want to set your user ID for IDOR testing? (optional)"

### `/session add-id <path_template> <id>`
Add an ID to `known_ids`. Useful for manually providing IDs from another user's account for IDOR testing. Example:
```
/session add-id "GET /users/{id}" 99
```

### `/session add-note <text>`
Append text to the `notes` field. Useful for recording things like "user 42 is admin, user 123 is regular user".

### `/session clear`
Ask for confirmation, then delete `session-memory.json` and `test-ledger.json`. Issues and coverage report are not deleted.

## How session memory is populated automatically

The **explorer** agent writes to `session-memory.json` during exploration:
- When a POST request returns 201, it extracts the resource ID from the response (looks for `.id`, `.uuid`, `.userId`, or the first numeric/UUID field at the root) and adds an entry to `created_resources` and `known_ids`
- The inferred `user_id` comes from the first GET /me, GET /profile, or similar endpoint that returns a user object

If these fields are wrong or missing, use `/session set-auth` and `/session add-id` to correct them manually before running `/auth-probe`.
