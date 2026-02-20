# skill-manager

A platform-agnostic agent skill that turns your AI coding assistant into a task orchestrator. Instead of doing work itself, the manager decomposes requests, dispatches background agents in parallel, and coordinates their results.

## Installation

```bash
npx skills add metyatech/skill-manager
```

## Usage

Invoke the skill using your platform's slash command syntax:

| Platform | Command |
|---|---|
| Claude Code | `/manager` |
| Codex | `$manager` |
| Other platforms | See your platform's skill invocation syntax |


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
