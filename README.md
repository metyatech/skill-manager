# skill-manager

A platform-agnostic agent skill that turns your AI coding assistant into a task orchestrator. Instead of doing work itself, the manager decomposes requests, dispatches background agents in parallel, and coordinates their results.

## Installation

```bash
npx skills add metyatech/skill-manager
```

## Usage

Invoke the skill using your platform's slash command syntax:

| Platform | Command |
| --- | --- |
| Claude Code | `/manager` |
| Codex | `$manager` |
| Other platforms | See your platform's skill invocation syntax |

## agents-mcp Setup (One-Time Per Platform)

The manager dispatches sub-agents via [agents-mcp](https://github.com/metyatech/agents-mcp). Install the MCP server once per platform:

```bash
# Claude Code
claude mcp add --scope user Swarm -- npx -y --package git+https://github.com/metyatech/agents-mcp.git#main agents-mcp

# Codex
codex mcp add swarm -- npx -y --package git+https://github.com/metyatech/agents-mcp.git#main agents-mcp

# Gemini CLI
gemini mcp add Swarm -- npx -y --package git+https://github.com/metyatech/agents-mcp.git#main agents-mcp
```

### Platform-specific monitoring

- **Claude Code**: Use `Bash(run_in_background=true, command="agents-mcp wait --task <name>")` for background monitoring, and `Status(wait=false)` for non-blocking checks.
- **Codex**: Use `Status(wait=false)` for polling. `Status(wait=true)` blocks and should be avoided during interactive sessions.
- **All platforms**: Use `Stop` to cancel agents and `Tasks` to list active tasks.

### Codex: PowerShell commands blocked by policy

When running under Codex on Windows, these commands are blocked by policy:

- `Remove-Item` (aliases: `rm`, `ri`, `del`, `erase`) — Use: `if ([IO.File]::Exists($p)) { [IO.File]::SetAttributes($p,[IO.FileAttributes]::Normal); [IO.File]::Delete($p) }`
- `Remove-Item -Recurse` (aliases: `rmdir`, `rd`) — Use: `if ([IO.Directory]::Exists($d)) { [IO.File]::SetAttributes($d,[IO.FileAttributes]::Normal); foreach ($e in [IO.Directory]::EnumerateFileSystemEntries($d,'*',[IO.SearchOption]::AllDirectories)) { [IO.File]::SetAttributes($e,[IO.FileAttributes]::Normal) }; [IO.Directory]::Delete($d,$true) }`

## What the Manager Does

- Receives user requests and decomposes them into discrete work items
- Classifies each item by complexity and resource requirements
- Dispatches independent tasks in parallel to background agents
- Selects the most cost-effective model tier for each task
- Tracks task status and reports progress concisely
- Handles agent failures by retrying or escalating to the user
- Coordinates dependent tasks in the correct sequence

## Recommended Session Settings

Use a **mid-tier model** with **medium reasoning effort** for the manager itself. The manager's job is orchestration and decomposition — it does not need maximum capability. Heavy analytical and implementation work is delegated to sub-agents, where you can apply appropriate model selection per task.

## License

MIT
