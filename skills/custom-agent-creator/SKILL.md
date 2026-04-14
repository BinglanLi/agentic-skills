---
name: custom-agent-creator
description: Create and configure custom Claude Code subagents. Use when asked to create, write, define, or configure a subagent (also called an "agent" or "sub-agent") for Claude Code — including writing agent .md files, choosing tool access, setting models, configuring permissions, hooks, memory, or MCP servers. Also use when asked to design a workflow that delegates tasks to specialized agents.
---

# Custom Agent Creator

## File format

Subagents = Markdown files with YAML frontmatter. Body = system prompt.

```markdown
---
name: agent-name          # lowercase, hyphens only (required)
description: When Claude should delegate to this agent (required)
tools: Read, Grep, Glob   # omit = inherit all tools
model: sonnet             # sonnet | opus | haiku | inherit (default)
---

System prompt goes here.
```

## Placement (priority order, highest first)

| Location | Scope |
|---|---|
| Managed settings | Org-wide |
| `--agents` CLI flag | Session only |
| `.claude/agents/` | Project |
| `~/.claude/agents/` | All projects (user) |
| Plugin `agents/` | Where plugin enabled |

Use `.claude/agents/` for project-specific; `~/.claude/agents/` for personal reuse.

## All frontmatter fields

See [references/fields.md](references/fields.md) for the complete field reference.

## Tool control

```yaml
tools: Read, Grep, Glob, Bash        # allowlist — only these
disallowedTools: Write, Edit         # denylist — inherit minus these
```

`disallowedTools` applied first; a tool in both is removed.

To restrict which subagents this agent can spawn (only relevant when running as main thread via `--agent`):
```yaml
tools: Agent(worker, researcher), Read, Bash   # allowlist of spawnable agents
```

## Key patterns

**Read-only researcher** — isolates high-volume output:
```yaml
tools: Read, Grep, Glob, Bash
model: haiku
```

**Safe reviewer** — no writes:
```yaml
disallowedTools: Write, Edit
```

**Parallel workers** — spawn multiple, Claude synthesizes:
```yaml
# In your prompt: "Research auth, db, and API modules in parallel using separate subagents"
```

**Chained pipeline**:
```yaml
# "Use code-reviewer to find issues, then optimizer to fix them"
```

## Persistent memory

```yaml
memory: project   # user | project | local
```

- `user` → `~/.claude/agent-memory/<name>/` (cross-project)
- `project` → `.claude/agent-memory/<name>/` (version-controllable, recommended default)
- `local` → `.claude/agent-memory-local/<name>/` (project, not committed)

Include in system prompt: `"Update your agent memory with patterns and decisions you discover."`

## Hooks (frontmatter-scoped, fire only while agent active)

```yaml
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate.sh"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/lint.sh"
```

Hook receives JSON via stdin; exit code 2 blocks the tool call.

## MCP servers

```yaml
mcpServers:
  - playwright:                # inline: scoped to this agent only
      type: stdio
      command: npx
      args: ["-y", "@playwright/mcp@latest"]
  - github                     # string ref: reuses parent session's connection
```

## Preloaded skills

```yaml
skills:
  - api-conventions
  - error-handling-patterns
```

Full skill content injected at startup. Agents don't inherit parent's skills.

## Permission modes

```yaml
permissionMode: acceptEdits   # default | acceptEdits | auto | dontAsk | bypassPermissions | plan
```

## Writing effective descriptions

The `description` field is how Claude decides when to delegate. Be specific:
- Include trigger phrases ("Use proactively after code changes", "Use when encountering errors")
- Name the domain ("SQL queries, BigQuery operations")
- Describe the output ("returns only failing tests with error messages")

## Foreground vs background

- Default: Claude decides
- Force background: ask "run this in the background" or press Ctrl+B
- `background: true` in frontmatter always runs as background task

Disable all background tasks: `CLAUDE_CODE_DISABLE_BACKGROUND_TASKS=1`

## Full examples

See [references/examples.md](references/examples.md) for complete ready-to-use agent files.
