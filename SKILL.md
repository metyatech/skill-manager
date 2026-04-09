---
name: sub-agent-dispatch
description: "Reference data and procedures for dispatching sub-agents in parallel: model inventory tables, agents-mcp parameter details, verification prompt templates, routing decision sequence, quota fallback logic, and platform-specific workarounds. Use when actually dispatching sub-agents and you need the full model inventory or detailed procedures. Triggers on: 'dispatch', 'orchestrate', 'sub-agent', 'agents-mcp', 'model inventory', 'route'."
---
<!-- markdownlint-disable MD013 -->

# Sub-agent dispatch reference

> **Source:** [metyatech/skill-sub-agent-dispatch](https://github.com/metyatech/skill-sub-agent-dispatch).
> To update this skill, edit the repository and push. The agent
> MUST NOT edit the installed copy.

This skill contains reference data and detailed procedures for
dispatching sub-agents. The always-loaded invariants for sub-agent
dispatch (agents-mcp tooling, ai-quota, mode, decision framework,
verification deliverable format, cost discipline, execution
discipline, lifecycle) live in the `sub-agent-dispatch` rule
module. Spawning prompt requirements and delegated-agent
obligations live in the `delegation` rule module. Approval rules
live in the `approval-gates` rule module.

## Dispatch workflow

1. Receive the user's request.
2. Decompose into discrete work items. Track them.
3. Classify each item using the Decision framework defined in
   the `sub-agent-dispatch` rule.
4. Dispatch all independent items in parallel.
5. Report to the user: what was dispatched, what is pending.
   Return control immediately.
6. On every subsequent user message, the agent MUST call
   `Status(wait=false)` first and report any state changes
   before addressing the user's new request.
7. When all sub-agents are done and review gates pass, the agent
   MUST summarize results and proceed to next steps.

## Cross-agent invocation (agents-mcp)

`agents-mcp` is the metyatech MCP server that works uniformly
from Claude Code, Codex, Gemini CLI, and other agent CLIs.

### One-time setup

See README.md for platform-specific installation commands.

### Dispatching a task

Use agents-mcp to spawn sub-agents with these parameters:

- `prompt` — the full self-contained task description.
- `agent_type` — target agent (`claude`, `codex`, `copilot`).
  The agent MUST NOT use `gemini` (see Gemini section below).
- `model` — explicit model string from the Model Inventory
  (e.g., `"claude-sonnet-4-6"`).
- `effort` — optional reasoning effort. Set from the Model
  Inventory; omit when not needed.
- `mode: 'edit'` — REQUIRED for implementation agents.

### Monitoring

After spawning agents, return to the user immediately. Use
background or non-blocking monitoring to be notified when agents
finish. `agents-mcp wait` and `Status(wait=true)` both poll until
completion or timeout.

### If agents-mcp is not configured

Stop and report the limitation to the user. The agent MUST NOT
simulate or substitute the work itself.

## Verification prompt templates

### Implementation-agent template

```text
Delegated mode. Implement approved scope only.
Return exactly: (1) Restated AC, (2) AC->evidence with exact commands/manual steps + PASS/FAIL/NOT RUN, (3) Files changed, (4) Assumptions/uncertainties, (5) Risks/rollback notes.
If spec/behavior text changed: cite file + section and summarize behavior delta.
Run applicable tests/verification and include exact commands and outputs summary.
```

### Review-agent template

```text
Delegated mode reviewer. Do not implement; review only.
Original requirements: [paste original AC/spec here]
Implementation output: [paste implementation agent's report here]
Note: tests were written by an agent and may confirm implementation behavior rather than spec intent.
Return: explicit PASS or FAIL, reasons, spec alignment (does the implementation meet the original requirements, not just the tests?), AC coverage gaps, and any ambiguous points.
Reject if evidence format is incomplete, outcomes are unsupported, or implementation deviates from original spec.
```

### Spec-change protocol

- If behavior or spec text changes, the implementation agent MUST
  cite where it changed (file + section heading) and MUST
  summarize the behavior delta.
- The reviewer MUST explicitly confirm spec alignment and list
  any ambiguous points requiring follow-up.

## Reviewer model selection

- For Standard tier reviews when verification fails: use
  `claude-haiku-4-5-20251001`.
- For Heavy tier reviews and release/production changes: use
  `claude-sonnet-4-6` with `medium` effort.

## Error handling

- If a sub-agent fails, call `Status` and treat
  `agents[].errors` and `agents[].diagnostics.tail_errors` as the
  primary failure reason.
- Open raw logs only when needed via
  `agents[].diagnostics.log_paths.stdout` (e.g., tail the file).
  Do not manually hunt logs.

## Model inventory and routing

**Last reviewed: 2026-02-27.** Update this section when models
change.

### Tier definitions

- **Free** — trivial lookups, simple Q&A, straightforward
  single-file edits. Copilot only.
- **Light** — mechanical transforms, formatting, simple
  implementations, quick clarifications.
- **Standard** — general implementation, code review, multi-file
  changes, most development work.
- **Heavy** — architecture decisions, safety-critical code,
  complex multi-step reasoning.
- **Large Context** — tasks requiring more than 200k token input.
- **Bulk/Idle** — automated compliance checks, repetitive
  transforms across repos. Prefer lowest-cost agents.

Classify each task into a tier, then pick an agent with available
quota and select the ★ preferred model for that tier. Fall back
to other models in the same tier when the preferred model's
agent has no quota.

### Claude

| Tier | Model | Effort | Notes |
| --- | --- | --- | --- |
| Light | claude-haiku-4-5-20251001 | — | Effort not supported; SWE-bench 73% |
| Standard | claude-sonnet-4-6 | medium | ★ Default; SWE-bench 80% |
| Heavy | claude-opus-4-6 | high | SWE-bench 81%; `max` effort for hardest tasks |

### Codex

| Tier | Model | Effort | Notes |
| --- | --- | --- | --- |
| Light | gpt-5.1-codex-mini | medium | `medium`/`high` only |
| Standard | gpt-5.3-codex | medium | ★ Latest flagship; SWE-bench Pro 57% |
| Standard | gpt-5.2-codex | medium | Previous gen; SWE-bench Pro 56% |
| Standard | gpt-5.2 | medium | General-purpose; best non-codex reasoning; SWE-bench 80% |
| Heavy | gpt-5.3-codex | xhigh | ★ Best codex at max effort |
| Heavy | gpt-5.1-codex-max | xhigh | Extended reasoning; context compaction |
| Heavy | gpt-5.2-codex | xhigh | Alternative |
| Heavy | gpt-5.2 | xhigh | General reasoning fallback |

### Gemini sub-agent reliability

> **⚠ The `gemini` agent type MUST NOT be used for sub-agent
> delegation.** Gemini CLI agents fail with HTTP 429 "No capacity
> available" server errors too frequently to be reliable, even
> for single unattended tasks. Gemini CLI MAY be used
> interactively when invoked directly by the user. For
> large-context work, use **Copilot with the `gemini-3-pro`
> model** instead.

| Tier | Model | Effort | Notes |
| --- | --- | --- | --- |
| Light | gemini-3-flash-preview | — | SWE-bench 78% |
| Standard | gemini-3-pro-preview | — | ★ 1M token context; SWE-bench 76% (interactive only) |
| Large Context | gemini-3-pro-preview | — | >200k token tasks; 1M context (interactive only) |

### Aider (BYOK)

Aider uses external API keys. Cost is per-token from the
provider (no subscription). Best for bulk/idle tasks with cheap
API models.

| Tier | Model | Provider | Notes |
| --- | --- | --- | --- |
| Bulk/Idle | deepseek/deepseek-chat | DeepSeek | ★ $0.07-$0.27/MTok input; SWE-bench 73% |
| Bulk/Idle | deepseek/deepseek-reasoner | DeepSeek | $1.30/Aider task |
| Light | claude-3-5-haiku-20241022 | Anthropic | Same as Claude Haiku |
| Standard | claude-sonnet-4-6-20261022 | Anthropic | Same as Claude Sonnet |

Configure via `DEEPSEEK_API_KEY` or `ANTHROPIC_API_KEY`. Aider is
not yet integrated into agent-runner (tracked task).

### Copilot

Copilot charges different quota per model. Prefer lower-multiplier
models when task complexity allows. Effort is not configurable
(ignored).

| Tier | Model | Quota | Notes |
| --- | --- | --- | --- |
| Free | gpt-5-mini | 0x | ★ SWE-bench ~70% |
| Free | gpt-4.1 | 0x | 1M context; SWE-bench 55% |
| Light | claude-haiku-4-5 | 0.33x | ★ SWE-bench 73% |
| Light | gpt-5.1-codex-mini | 0.33x | Mechanical transforms |
| Standard | claude-sonnet-4-6 | 1x | ★ Default; SWE-bench 80% |
| Standard | gpt-5.3-codex | 1x | Latest codex flagship |
| Standard | gpt-5.2 | 1x | Best general reasoning |
| Standard | gpt-5.2-codex | 1x | Agentic coding |
| Standard | gpt-5.1-codex-max | 1x | Extended reasoning |
| Standard | claude-sonnet-4-5 | 1x | SWE-bench 77%; prefer 4.6 |
| Standard | gpt-5.1-codex | 1x | SWE-bench 77% |
| Standard | gpt-5.1 | 1x | General purpose; SWE-bench ~76% |
| Standard | gemini-3-pro | 1x | 1M context; SWE-bench 76% |
| Standard | claude-sonnet-4 | 1x | Legacy; last choice |
| Heavy | claude-opus-4-6 | 3x | ★ SWE-bench 81% |
| Heavy | claude-opus-4-5 | 3x | SWE-bench 81%; prefer 4.6 |
| — | claude-opus-4-6 fast | 30x | Avoid; excessive quota cost |

### Routing principles

- Active sub-agent types are `claude`, `codex`, `copilot`. The
  `gemini` agent type MUST NOT be spawned (see Gemini section
  above). All active agent types operate on independent
  flat-rate subscriptions. Route by model quality, quota
  conservation, and quota distribution.
- All agents can execute code, modify files, and perform
  multi-step tasks. Route by model quality and quota, not by
  execution capability.
- Spread work across agents to maximize total throughput.
- For large-context tasks (>200k tokens), prefer Copilot with
  the `gemini-3-pro` model. Do NOT use the `gemini` agent type.
- For trivial tasks, prefer Copilot free-tier models (0x quota)
  before consuming other agents' quota.
- When multiple agents can handle a task equally well, prefer
  the one with the most remaining quota.

### Cost-efficiency priority

- Prefer flat-rate (subscription) agents over pay-per-token
  agents when quota is available.
- Idle/bulk tasks MUST NOT consume premium quota. Route to:
  Copilot 0x → Aider+DeepSeek → Amazon Q.
- Reserve premium quota for interactive use. Usage gates enforce
  minimum 70% remaining for Claude and Codex 5-hour windows.
- **Sonnet 4.6 is the workhorse.** Default to Sonnet; escalate
  to Opus for Heavy tier or when strict rule compliance failures
  occur. Use `medium` effort for both — research shows higher
  effort degrades instruction-following on multi-constraint rule
  sets (arXiv:2505.11423).

### Quota fallback logic

If the primary agent has no remaining quota:

1. Query quota for all agents via `ai-quota`.
2. Select any agent with available quota that has a model at
   the required tier.
3. For Copilot fallback, prefer lower-multiplier models to
   conserve quota.
4. For Standard tier with no subscription quota: fall back to
   Aider + DeepSeek (pay-per-token, ~$1/task).
5. If the fallback model is significantly less capable, note
   the degradation in the dispatch report.
6. If no agent has quota and no pay-per-token fallback is
   available, queue the task and report the block immediately.
   Do not drop silently.

### Routing decision sequence

1. Classify the task tier (Free / Light / Standard / Heavy /
   Large Context / Bulk/Idle).
2. For Free tier: dispatch to Copilot with a 0x model. Skip
   quota check.
3. For Bulk/Idle tier: dispatch to Copilot 0x → Aider+DeepSeek
   → Amazon Q (in that order). Skip premium agents and do not
   use the Gemini agent type.
4. For other tiers: check quota for all agents via `ai-quota`.
5. Pick the agent with available quota at the required tier;
   prefer the agent with the most remaining quota when multiple
   qualify.
6. Set `agent_type`, `model`, and `effort` from the tables above
   (omit `effort` when column shows —).
7. If primary choice has no quota: apply fallback logic.
8. Include the chosen agent, model, tier, and effort in the
   dispatch report.

### Notable models (not in current routing)

These models are not yet accessible via supported agent CLIs but
are worth monitoring:

| Model | Provider | Strength | Access |
| --- | --- | --- | --- |
| Kimi K2.5 | Moonshot AI | IFEval SOTA 94.0 | API only; no CLI |
| Grok 4.20 | xAI | 4-agent council; 2M context | xAI API only |
| Qwen 3.5 | Alibaba | IFEval 92.6 | API only |
| GLM-4.7 | Zhipu AI | Dual-judge highest score | API only |

## Platform-specific workarounds

- On platforms with policy-blocked file deletion commands (e.g.,
  Codex on Windows), see the Codex-specific workarounds in the
  `command-execution` rule module.
