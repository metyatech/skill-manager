# skill-manager

<!-- markdownlint-disable MD013 -->

A platform-agnostic agent skill that turns your AI coding assistant into a task orchestrator. Instead of doing work itself, the manager decomposes requests, delegates to background agents, and coordinates their results.

## Installation

```bash
npx skills add metyatech/skill-manager
```

## Usage

Invoke the skill using your platform's skill invocation syntax:

| Platform | Command |
| --- | --- |
| Claude Code | `/manager` |
| Codex | `$manager` |
| Other platforms | See your platform's skill invocation syntax |

## Delegation Backend (Recommended)

This skill is most useful when your client can delegate to multiple coding agents through an agent broker. The recommended setup is an MCP (Model Context Protocol) broker so the manager can:

- list available agents
- ask one agent for a response
- run multi-turn conversations when needed
- query multiple agents (council/compare)

A practical Windows-friendly broker is `mcp-rubber-duck`, which can expose CLI coding agents (Claude Code, Codex, Gemini CLI, etc.) through an MCP server.

### Install mcp-rubber-duck

```bash
npm install -g mcp-rubber-duck
```

### Example MCP Server Config

Most MCP-capable clients support a config similar to:

```json
{
  "mcpServers": {
    "rubber-duck": {
      "command": "mcp-rubber-duck",
      "args": ["--mcp"],
      "env": {
        "MCP_SERVER": "true",
        "CLI_CLAUDE_ENABLED": "true",
        "CLI_CODEX_ENABLED": "true",
        "CLI_GEMINI_ENABLED": "true"
      }
    }
  }
}
```

Notes:

- CLI ducks run as local processes. Ensure the corresponding CLIs (`claude`, `codex`, `gemini`) are installed and available on `PATH`.
- Conversation state is typically stored in the broker process memory. If the MCP server is restarted, conversation history may be lost.

### Windows: Gemini CLI Shim Workaround

On Windows, some CLIs are installed as `*.cmd` shims which may not be spawnable by certain brokers without a shell. If `gemini` fails to launch, configure Gemini as a custom CLI provider that runs `node` with the global Gemini CLI entrypoint.

1. Find your global node_modules directory:

```powershell
npm root -g
```

1. Configure a custom Gemini duck (example in PowerShell):

```powershell
$globalNodeModules = (npm root -g)
$geminiEntrypoint = Join-Path $globalNodeModules "@google\gemini-cli\dist\index.js"

$env:CLI_GEMINI_ENABLED = "false"

$env:CLI_CUSTOM_GEMINIWIN_COMMAND = "node"
$env:CLI_CUSTOM_GEMINIWIN_NICKNAME = "Gemini CLI (Windows)"
$env:CLI_CUSTOM_GEMINIWIN_PROMPT_DELIVERY = "flag"
$env:CLI_CUSTOM_GEMINIWIN_PROMPT_FLAG = "-p"
$env:CLI_CUSTOM_GEMINIWIN_OUTPUT_FORMAT = "json"
$env:CLI_CUSTOM_GEMINIWIN_CLI_ARGS = "$geminiEntrypoint,--output-format,json,--yolo,--model,gemini-2.5-flash"
```

If your Gemini CLI requires a different approval mode or model, adjust the `CLI_ARGS` accordingly.

## What the Manager Does

- Receives user requests and decomposes them into discrete work items
- Delegates independent tasks (and parallelizes when possible)
- Tracks task status and reports progress concisely
- Requests evidence (tests/commands/logs) and reports outcomes
- Handles agent failures by retrying or escalating to the user
- Coordinates dependent tasks in the correct sequence

## Recommended Session Settings

Use a mid-tier model with medium reasoning effort for the manager itself. The manager's job is orchestration and decomposition, not deep implementation. Delegate heavy analysis and implementation work to sub-agents.

## License

MIT
