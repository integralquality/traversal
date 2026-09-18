---
name: replay-issue
description: Re-run a specific issue's curl command and compare the response to the original. Confirms whether the issue is still reproducible.
tools: [Read, Bash]
---

# Replay Issue

Verifies that a logged issue is still reproducible by re-running its curl command.

## Invocation

```
/replay-issue <ISSUE-NNN>
```

Example: `/replay-issue ISSUE-003`

## Steps

### 1. Read the issue

Read `issues.md`. Find the section starting with `## ISSUE-NNN`. Extract:
- The curl command (the bash block under **Curl:**)
- The original response status and body
- The endpoint and charter

If the issue ID isn't found, list available issue IDs and stop.

### 2. Prepare the curl

The logged curl may use `$TOKEN` as a placeholder. If so, read the current auth header from `session-memory.json` and substitute the token. If `session-memory.json` doesn't exist or has no auth header, ask the user to provide the token.

Print the curl command that will be run (with the real token masked).

### 3. Run it

Execute the curl via Bash. Capture status code and body.

### 4. Compare and report

```
ISSUE-NNN — <title>
Endpoint: METHOD /path

Original response: <status>
Current response:  <status>

Status: REPRODUCIBLE | NOT REPRODUCIBLE | CHANGED

Original body (first 500 chars):
<...>

Current body (first 500 chars):
<...>

Diff:
<note key differences — field names present/absent, status code change, error message change>
```

**REPRODUCIBLE** — same status code and same key indicators in the body.
**NOT REPRODUCIBLE** — status code changed to something non-indicative of the issue, or the problematic behavior is gone.
**CHANGED** — status code matches but response body differs in a meaningful way (e.g., different error message, fewer fields exposed).

### 5. Optional: update issues.md

Offer to append a replay note to the issue entry:
```markdown
**Replay** (2026-09-18): REPRODUCIBLE — same 200 status, `role` field still accepted.
```
