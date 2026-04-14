# Subagent Frontmatter Fields

| Field | Required | Values / Notes |
|---|---|---|
| `name` | Yes | Lowercase letters and hyphens |
| `description` | Yes | When Claude should delegate — primary routing signal |
| `tools` | No | Comma-separated tool names. Omit = inherit all. Use `Agent(name1,name2)` to restrict spawnable subagents |
| `disallowedTools` | No | Tools to remove from inherited/specified list |
| `model` | No | `sonnet` \| `opus` \| `haiku` \| full ID (e.g. `claude-opus-4-6`) \| `inherit` (default) |
| `permissionMode` | No | `default` \| `acceptEdits` \| `auto` \| `dontAsk` \| `bypassPermissions` \| `plan` |
| `maxTurns` | No | Integer — max agentic turns before agent stops |
| `skills` | No | List of skill names — full content injected at startup |
| `mcpServers` | No | List of server names (string refs) or inline server definitions |
| `hooks` | No | Lifecycle hooks scoped to this agent (PreToolUse, PostToolUse, Stop) |
| `memory` | No | `user` \| `project` \| `local` — enables persistent cross-session memory |
| `background` | No | `true` = always run as background task (default: `false`) |
| `effort` | No | `low` \| `medium` \| `high` \| `max` (Opus 4.6 only) — overrides session effort level |
| `isolation` | No | `worktree` = run in temporary git worktree; auto-cleaned if no changes |
| `color` | No | `red` \| `blue` \| `green` \| `yellow` \| `purple` \| `orange` \| `pink` \| `cyan` |
| `initialPrompt` | No | Auto-submitted as first user turn when agent runs as main session (via `--agent`). Skills/commands processed. |

## Model resolution order

1. `CLAUDE_CODE_SUBAGENT_MODEL` env var
2. Per-invocation `model` parameter (set by Claude when spawning)
3. Agent definition's `model` frontmatter
4. Main conversation's model

## Permission mode details

| Mode | Behavior |
|---|---|
| `default` | Standard prompts |
| `acceptEdits` | Auto-accept file edits in working dir / additionalDirectories |
| `auto` | Background classifier reviews commands |
| `dontAsk` | Auto-deny prompts (explicitly allowed tools still work) |
| `bypassPermissions` | Skip all prompts — use with caution |
| `plan` | Read-only exploration |

Note: If parent uses `bypassPermissions` or `auto`, child cannot override.

## Hooks in frontmatter

Supported events: `PreToolUse`, `PostToolUse`, `Stop` (converted to `SubagentStop` at runtime).

`Stop` hooks fire when agent spawned via Agent tool or @-mention. They do NOT fire when running as main session via `--agent`.

## Project-level hooks for subagent lifecycle (settings.json)

```json
{
  "hooks": {
    "SubagentStart": [{ "matcher": "agent-name", "hooks": [{ "type": "command", "command": "..." }] }],
    "SubagentStop":  [{ "hooks": [{ "type": "command", "command": "..." }] }]
  }
}
```
