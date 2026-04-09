# skill-sub-agent-dispatch

A platform-agnostic agent skill containing reference data and detailed procedures for dispatching sub-agents in parallel: model inventory tables (Claude / Codex / Gemini / Aider / Copilot), agents-mcp parameter details, verification prompt templates, routing decision sequence, quota fallback logic, and platform-specific workarounds.

The always-loaded **invariants** for sub-agent dispatch (agents-mcp tooling, ai-quota check, mode selection, decision framework, verification deliverable format, cost discipline, execution discipline, lifecycle) live in the `sub-agent-dispatch` rule module of `metyatech/agent-rules`. This skill provides the on-demand reference data the rule points to.

## Installation

```bash
npx skills add metyatech/skill-sub-agent-dispatch
```

## Usage

Invoke the skill using your platform's slash command syntax:

| Platform | Command |
| --- | --- |
| Claude Code | `/sub-agent-dispatch` |
| Codex | `$sub-agent-dispatch` |
| Other platforms | See your platform's skill invocation syntax |

## agents-mcp Setup (One-Time Per Platform)

This skill dispatches sub-agents via [agents-mcp](https://github.com/metyatech/agents-mcp). Install the MCP server once per platform:

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

## What this skill provides

- The complete model inventory tables for Claude, Codex, Gemini, Aider, and Copilot, with tier classifications and SWE-bench benchmarks.
- Detailed dispatch workflow (8 steps) and agents-mcp parameter reference.
- Verification prompt templates for implementation and reviewer agents.
- Routing principles, cost-efficiency priority, quota fallback logic, and the routing decision sequence.
- Spec-change protocol for behavior or spec text changes.
- Reviewer model selection guidance.
- Platform-specific workarounds (Codex on Windows, etc.).

## What lives elsewhere

| Concern | Location |
| --- | --- |
| Always-loaded dispatch invariants (agents-mcp mandatory, ai-quota check, mode='edit', decision framework, verification deliverable format, cost/execution discipline, lifecycle) | `sub-agent-dispatch` rule module in `metyatech/agent-rules` |
| Spawning prompt requirements and delegated-agent obligations | `delegation` rule module in `metyatech/agent-rules` |
| Approval rules for sub-agent dispatch and operations | `approval-gates` rule module in `metyatech/agent-rules` |
| Tier and effort selection cross-cutting rules | `model-routing` rule module in `metyatech/agent-rules` |

## License

MIT
