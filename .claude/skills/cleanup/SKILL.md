---
description: Delete resources created during testing. Reads created_resources from session-memory.json and runs the corresponding DELETE requests.
---

Cleans up test resources created during exploration.

## Steps

### 1. Read session-memory.json

If missing or `created_resources` is empty, report nothing to clean up and stop.

Get `auth.header`. If empty, warn before proceeding — deletes without auth will likely fail.

### 2. Preview

Print the list before touching anything:

```
Resources to delete (N total):
  DELETE /users/123      ← created via POST /users
  DELETE /posts/456      ← created via POST /posts
```

Confirm: "Delete all N? (yes/no)"

### 3. Execute

For each `created_resources` entry:
```bash
curl -s -w "\n---STATUS:%{http_code}---" -X DELETE '<base_url><cleanup_path>' \
  -H '<auth_header>'
```

Track: succeeded (2xx), already gone (404), failed (other).
404 counts as success.

### 4. Update session-memory.json

Remove successfully deleted entries from `created_resources`. Leave failures so they can be retried or handled manually.

### 5. Report

```
Cleanup complete:
  Deleted:      N
  Already gone: N
  Failed:       N  ← list with status codes
```

If any failed, offer to retry or show the curl commands for manual deletion.
