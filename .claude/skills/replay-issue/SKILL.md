---
description: Re-run a specific issue's curl command and compare the response to the original. Confirms whether the issue is still reproducible.
argument-hint: <ISSUE-NNN>
---

Verifies that a logged issue is still reproducible.

## Steps

### 1. Find the issue

Read `issues.md`. Find `## <ISSUE-NNN>`. Extract:
- The curl command from the bash block under **Curl:**
- Original response status and body
- Endpoint and charter

If the ID isn't found, list available issue IDs and stop.

### 2. Prepare the curl

The logged curl uses `$TOKEN` as a placeholder. Substitute from `session-memory.json` → `auth.header`. If unavailable, ask for the token.

Print the command that will run (mask the real token in output).

### 3. Run it

Execute via Bash. Capture status and body.

### 4. Report

```
ISSUE-NNN — <title>
Endpoint: METHOD /path

Original: <status>
Current:  <status>

Result: REPRODUCIBLE | NOT REPRODUCIBLE | CHANGED

<note key differences: missing fields, different status, error message changed>
```

- **REPRODUCIBLE** — same status and same key indicators in the body
- **NOT REPRODUCIBLE** — problematic behavior is gone
- **CHANGED** — status matches but response differs (different error, fewer fields exposed)

### 5. Offer to annotate

Offer to append a replay note to the issue in `issues.md`:
```markdown
**Replay** (<date>): REPRODUCIBLE — same 200 status, `role` field still accepted.
```
