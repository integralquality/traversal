---
name: cleanup
description: Delete resources created during testing. Reads created_resources from session-memory.json and runs the corresponding DELETE requests.
tools: [Read, Write, Edit, Bash]
---

# Cleanup

Deletes test resources created during exploration.

## Setup

Read `session-memory.json`. If it doesn't exist or `created_resources` is empty, report nothing to clean up.

Get the auth header from `session-memory.json`. If auth is missing, ask the user before proceeding — deletes without auth will likely fail.

## Steps

### 1. Preview

Before deleting anything, print the list of resources to be deleted:

```
Resources to delete (N total):
  DELETE /users/123    (created via POST /users)
  DELETE /posts/456    (created via POST /posts)
  ...
```

Ask: "Delete all N resources? (yes/no)"

### 2. Execute deletions

For each entry in `created_resources`, run:
```bash
curl -s -w "\n---STATUS:%{http_code}---" -X DELETE '<base_url><cleanup_path>' \
  -H '<auth_header>'
```

Track: succeeded (2xx or 404), failed (other status), errored (connection error).

404 counts as success — the resource is already gone.

### 3. Update session memory

Remove successfully deleted resources from `created_resources` in `session-memory.json`. Leave failed ones so they can be retried or handled manually.

### 4. Report

```
Cleanup complete:
  Deleted:  N
  Already gone (404): N
  Failed:   N  ← list these with their status codes
```

If any deletions failed, suggest running `/session` to see what remains and offer to retry.

## Manual cleanup

If `session-memory.json` is missing but the user knows what was created, they can add entries manually:
```
/session add-resource "POST /users" "DELETE /users/99"
```
Then run `/cleanup` again.
