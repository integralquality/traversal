# traversal

A Claude Code plugin for autonomous REST API exploratory testing. No runtime code — the plugin is entirely `.claude/` skill and agent definitions.

## Structure

```
.claude/
├── settings.json          # allow curl and python3
├── agents/                # true subagents (own context window, spawnable)
│   ├── orchestrator.md    # parses spec, spawns explorer forks
│   ├── explorer.md        # explores endpoint group, logs issues
│   └── auth-probe.md      # dedicated auth/IDOR sweep
├── skills/                # user-invokable prompts (/skill-name)
│   ├── bootstrap-ledger/
│   ├── design-charters/   # bundles techniques.md
│   ├── analyze-response/  # bundles checklist.md
│   ├── session/
│   ├── cleanup/
│   ├── replay-issue/
│   └── coverage-report/
└── agent-memory/
    └── orchestrator/      # cross-session memory (committed)
```

## Agents vs. Skills

**Agents** run in their own context window and can be spawned in parallel. Use for work that is long-running, stateful, or benefits from isolation.

**Skills** run in the current context. Use for focused, user-invokable tasks with a clear input/output.

## Adding a skill

1. Create `.claude/skills/<name>/SKILL.md` with frontmatter:
   ```yaml
   ---
   description: One line — shown in skill listings and used for auto-selection.
   argument-hint: <hint shown in the UI>  # optional
   ---
   ```
2. Put reference material (checklists, technique guides, payload lists) as separate `.md` files in the same directory. Reference them by filename in SKILL.md.
3. Update README.md skills table.

## Adding an agent

1. Create `.claude/agents/<name>.md` with frontmatter:
   ```yaml
   ---
   name: agent-name
   description: One line.
   tools: [Read, Write, Edit, Bash]  # restrict to what it actually needs
   memory: project  # if it should have persistent memory
   ---
   ```
2. If `memory: project`, an empty `MEMORY.md` is auto-created at `.claude/agent-memory/<name>/MEMORY.md` on first use. Seed it with structure if useful.
3. Update README.md.

## State files (generated at runtime, not committed)

| File | Written by | Purpose |
|---|---|---|
| `test-ledger.json` | orchestrator, explorer | endpoint coverage tracking |
| `session-memory.json` | orchestrator, explorer | auth, created IDs, known IDs for IDOR |
| `issues.md` | explorer, auth-probe | append-only issue log |
| `coverage-report.md` | coverage-report skill | generated summary |
