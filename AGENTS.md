<!-- markdownlint-disable MD025 -->
# Tool Rules (compose-agentsmd)

- **Session gate**: before responding to ANY user message, run `compose-agentsmd` from the project root. AGENTS.md contains the rules you operate under; stale rules cause rule violations. If you discover you skipped this step mid-session, stop, run it immediately, re-read the diff, and adjust your behavior before continuing.
- `compose-agentsmd` intentionally regenerates `AGENTS.md`; any resulting `AGENTS.md` diff is expected and must not be treated as an unexpected external change.
- If `compose-agentsmd` is not available, install it via npm: `npm install -g compose-agentsmd`.
- To update shared/global rules, use `compose-agentsmd edit-rules` to locate the writable rules workspace, make changes only in that workspace, then run `compose-agentsmd apply-rules` (do not manually clone or edit the rules source repo outside this workflow).
- If you find an existing clone of the rules source repo elsewhere, do not assume it is the correct rules workspace; always treat `compose-agentsmd edit-rules` output as the source of truth.
- `compose-agentsmd apply-rules` pushes the rules workspace when `source` is GitHub (if the workspace is clean), then regenerates `AGENTS.md` with refreshed rules.
- Do not edit `AGENTS.md` directly; update the source rules and regenerate.
- `tools/tool-rules.md` is the shared rule source for all repositories that use compose-agentsmd.
- Before applying any rule updates, present the planned changes first with an ANSI-colored diff-style preview, ask for explicit approval, then make the edits.
- These tool rules live in tools/tool-rules.md in the compose-agentsmd repository; do not duplicate them in other rule modules.

Source: github:metyatech/agent-rules@HEAD/rules/global/agent-rules-composition.md

# Rule composition and maintenance

- AGENTS.md is self-contained; place at project root. Shared rules centrally; project-local only for truly local policies.
- Before work in a repo with `agent-ruleset.json`, run `compose-agentsmd` to refresh AGENTS.md.
- Pre-commit hooks must run the repo's full verification suite, then `compose-agentsmd --compose`, then `git add AGENTS.md`. Do not fail commits on drift or add freshness checks to CI.

## Update and editing

- Never edit AGENTS.md directly; update source rules and regenerate. "Update rules" = update module/ruleset, then regenerate.
- Persistent user instructions → encode in appropriate module (global vs local) in the same change set.
- New repos must meet all global rules (AGENTS.md, CI, linting, community health, docs, scanning) before reporting complete.
- Update rulesets for missing domain rules before proceeding. Omit AGENTS.md diffs unless asked.
- Treat AGENTS.md diffs produced by compose-agentsmd as intentional updates: do not discard/revert them unless the requester explicitly asks to drop them.
- When the repository is git-managed, stage those intentional AGENTS.md updates normally (git add) unless the requester explicitly says to exclude them.
- Infer core intent; prefer global over project-local. Keep rules MECE, concise, non-redundant, action-oriented ("do X", "never Z"). No hedging or numeric filename prefixes.
- When updating global rules, encode the underlying general principle rather than the incident-specific example; prefer the broadest wording that still gives clear action.
- Placement: based on where needed. Any-workspace → global; domain only for opt-in repos.

## Size budget

- Total ≤350 lines; per-module ≤30 (soft). Overage → extract procedural content to skills.
- **Rules** = invariants (always loaded, concise). **Skills** = procedures (on-demand, detailed).

Source: github:metyatech/agent-rules@HEAD/rules/global/autonomous-operations.md

# Autonomous operations

- Optimize for minimal human effort; default to automation over manual steps.
- Drive work from the desired outcome: choose the highest-quality safe path and execute end-to-end.
- Correctness, safety, robustness, verifiability > speed unless requester explicitly approves the tradeoff.
- Default to long-term maintainability over short-term optimization.
- End-to-end repo autonomy (issues, PRs, pushes, merges, releases, admin) within user-controlled repos; third-party repos require explicit request.
- No backward compatibility unless requested; no legacy aliases, shims, or temporary fallback behavior.
- Proactively fix rule gaps, redundancy, or misplacement; regenerate AGENTS.md without waiting.
- Self-evaluate continuously; fix rule/skill gaps immediately on discovery. In delegated mode, include improvement suggestions in the task result.
- On user-reported failures: treat as systemic - fix, update rules, check for same pattern elsewhere, in one action.
- Session memory resets; use rule files as persistent memory. Never write to platform-specific local memory files; all persistent behavioral knowledge MUST live in agent rules.
- Rules are source of truth; update conflicting repos to comply or encode the exception.
- Investigate unclear items before proceeding; no assumptions without approval. Make scope/risk/cost/irreversibility decisions explicit.
- In direct mode, treat any normal user instruction as approval for the full in-scope follow-up work needed to satisfy that request, unless the user explicitly narrows scope or higher-priority rules require additional approval.
- After any user instruction, infer and execute the natural delivery chain by default: implementation, testing, debugging, runtime verification, deployment/release when applicable, documentation updates, follow-on defect cleanup, and residual-risk reduction, until the strongest justified terminal state is reached or an irreducible blocker remains.
- When the user has requested multiple tasks, keep the conversation focused on the current in-progress task until it reaches a clear terminal state or blocker. Do not shift discussion to later requested tasks early unless the user explicitly asks to switch now.
- When asked to handle PR review feedback, keep running the full PR review loop across successive review rounds without waiting for another user prompt: address feedback, verify, commit/push, re-request the relevant reviewer(s), monitor for new feedback, and repeat until no actionable review feedback remains or a blocker requires user input.
- Do not stop at intermediate milestones or pause for optional reassurance, optional next-step confirmation, or convenience check-ins while actionable in-scope work remains; interrupt only for blockers, mandatory approvals imposed by higher-priority rules, explicit stop/pause instructions, or material scope/risk changes that require user input.
- Do not lower the requested outcome on your own. If the intended end state is not yet fully met, continue working or explicitly return the remaining gap to the user; never treat partial satisfaction as completion by your own judgment.
- Actively consider whether user input carries intent beyond its literal wording, and when it does, state that inferred intent and propose the matching next step. For requests to design or build systems jointly managed by humans and AI agents, make actors, canonical store, human surface, AI surface, sync, conflict policy, validation surfaces, generated artifacts, and human startup path explicit before implementation, and use the `human-ai-system-builder` skill.
- If a problem can be fundamentally solved by modifying global rules, solve it by modifying global rules.
- When modifying global rules, choose the shortest rule change that is still sufficient to solve the problem.
- When multiple viable approaches exist, default to the highest-standard option that maximizes long-term quality, maintainability, and durability; lower-standard tradeoffs require explicit user request.

## Autonomous task resolution

- If a verification step (e.g., `npm run verify`, `npm audit`) fails due to known security vulnerabilities, attempt to fix them automatically (e.g., `npm audit fix`). If the fix is successful and verification passes, commit and push the changes to the PR branch.
- If a task is stuck or constantly failing due to quota limits (429 errors), ensure the task state is correctly updated in `task-tracker` so it can resume from the last successful stage in the next execution cycle.
- Do not remain idle on a failing PR if a known automated fix exists.

Source: github:metyatech/agent-rules@HEAD/rules/global/command-execution.md

# Workflow and command execution
## MCP server setup verification
- After adding or modifying an MCP server configuration, immediately verify connectivity using the platform's MCP health check and confirm the server is connected.
- If a configured MCP server fails to connect, diagnose and fix before proceeding. Do not silently fall back to alternative tools without reporting the degradation.
- At session start, if expected MCP tools are absent from the available tool set, verify MCP server health and report/fix connection failures before continuing.
- Do not add wrappers or pipes to commands unless the user explicitly asks.
- Prefer repository-standard scripts/commands (package.json scripts, README instructions).
- Reproduce reported command issues by running the same command (or closest equivalent) before proposing fixes.
- When a Windows process is intended to run headlessly, every non-interactive child console process in that spawn chain must also be launched headlessly (`windowsHide: true`, `CreateNoWindow`, or `Start-Process -WindowStyle Hidden` as appropriate).
- From a windowless PowerShell parent, do not spawn non-interactive console children with the `&` call operator; use an explicit hidden process launch that preserves output/exit-code handling.
- When diagnosing third-party tool failures, check the latest stable release first; if the latest still reproduces the failure, treat upgrade as insufficient, record the verified limitation, and use a deterministic workaround when available.
- Avoid interactive git prompts by using --no-edit or setting GIT_EDITOR=true.
- If elevated privileges are required, use sudo directly; do not launch a separate elevated shell (e.g., Start-Process -Verb RunAs). Fall back to run as Administrator only when sudo is unavailable.
- Keep changes scoped to affected repositories; when shared modules change, update consumers and verify at least one.
- If no branch is specified, work on the current branch; direct commits to main/master are allowed.
- Do not assume agent platform capabilities beyond what is available; fail explicitly when unavailable.
- When building a CLI, follow standard conventions: --help/-h, --version/-V, stdin/stdout piping, --json output, --dry-run for mutations, deterministic exit codes, and JSON Schema config validation.
- Treat `agent-browser` sessions as temporary resources: close them immediately when they are no longer needed, and before concluding a task verify that no sessions spawned for that task remain open.
- When launching `agent-browser` on Windows, do not rely on the implicit default session; always provide an explicit `--session <name>` or set `AGENT_BROWSER_SESSION` to a chosen session name before use, and if a localhost bind/access error occurs (for example `os error 10013`), retry with a different explicit session name instead of reusing the blocked default session, then report the working session name or the remaining failure.
- For federated identity flows (Google, Apple, Microsoft, GitHub, etc.), when automation-launched browser contexts are blocked, degraded, or risky, hand off only the IdP step to a real browser/session via CDP or explicit user interaction, then resume automation after the redirect; do not try to bypass provider anti-automation or embedded-browser restrictions.
## Codex-only PowerShell safety
- `Remove-Item` (aliases: `rm`, `ri`, `del`, `erase`) ↁEUse: `if ([IO.File]::Exists($p)) { [IO.File]::SetAttributes($p,[IO.FileAttributes]::Normal); [IO.File]::Delete($p) }`
- `Remove-Item -Recurse` (aliases: `rmdir`, `rd`) ↁEUse: `if ([IO.Directory]::Exists($d)) { [IO.File]::SetAttributes($d,[IO.FileAttributes]::Normal); foreach ($e in [IO.Directory]::EnumerateFileSystemEntries($d,'*',[IO.SearchOption]::AllDirectories)) { [IO.File]::SetAttributes($e,[IO.FileAttributes]::Normal) }; [IO.Directory]::Delete($d,$true) }`
- In PowerShell, use `;` for sequential command chaining; never use `&&` or `||` as control-flow operators.
## Post-change deployment
- After modifying code, check whether deployment steps beyond commit/push are needed before concluding; if the repo is globally linked (`npm ls -g` shows `->` to local path), rebuild and verify the global binary is functional.
- If the repo powers a running service, daemon, or scheduled task, rebuild, restart, and verify with deterministic evidence; do not claim completion until the running instance reflects the changes. Detection and verification procedures are in the `post-deploy` skill.
- For tests or helper flows that spawn background services, daemons, browsers, or other long-lived processes, verify teardown and absence of agent-owned orphaned temp/process artifacts before concluding; if teardown fails, fix the harness instead of leaving residue.
- **PowerShell native environment**: This is a Windows/PowerShell environment. Do not use Unix commands directly. On Windows, any Bash-tool command containing `pwsh` or `powershell` is invalid; rewrite it to `pwsh`/`powershell -File` with a `.ps1` file before execution. Do not use `-Command`, stdin, heredoc, or `-EncodedCommand` for PowerShell scripts.

Source: github:metyatech/agent-rules@HEAD/rules/global/implementation-and-coding-standards.md

# Engineering and implementation standards

- Prefer official frameworks and well-maintained dependencies.
- Use latest stable versions of packages/tools; document blockers if not.
- Prefer OSS/free-tier services; call out tradeoffs.
- PowerShell: \ is literal; avoid shadowing auto-vars; prefer single quotes.
- Assess reuse first; prefer remote dependencies over local paths.
- Single responsibility; composition over inheritance; clean dependency direction.
- Avoid deep nesting; guard clauses; small functions; intention-revealing names.
- Prefer config/constants over hardcoding; consolidate change points.
- GUI: prioritize ergonomics, in-app guidance, and a natural top-to-bottom/left-to-right flow from current context to next action to result to optional detail.
- Optimize for first-use clarity: use ordinary task language, avoid internal jargon, and make current selection, source choices, and result targets visually obvious in the UI.
- In GUI interactions, follow established expectations for common controls (for example info icons, disclosure toggles, close buttons, tabs, row selection); only deviate when the UX gain clearly outweighs the surprise cost.
- Prefer modern, visually rich UI and purposeful motion when they improve comprehension; avoid horizontal scrolling in primary application UI unless explicitly justified by the task.
- In interactive selection flows, make the current item, the choice list, and the destination of the chosen result visually and spatially explicit. Prefer anchored drawers, callouts, overlays, or similar patterns over detached panels when they improve comprehension.
- Do not treat stylistic richness or "game-like" presentation as success if users cannot immediately tell what is selected now, what they are choosing from, and where the change will apply. Comprehension wins over style.
- Keep DRY across code/docs/tests/config; refactor repeated procedures.
- Fix root causes; remove obsolete code; repair tools at source.
- Ensure failure paths tear down resources; no partial state.
- Do not block async APIs; avoid sync I/O where responsiveness is expected.
- Avoid external command execution; prefer native SDKs.
- Prefer stable public APIs; isolate/document unavoidable internal API usage.
- Externalize large embedded strings/templates/rules; do not commit build artifacts and keep naming aligned.
- No machine-specific environments; use relative paths and explicit config.
- Agent temp files MUST stay under OS temp unless approved.
- Design tools/services for agent-compatibility via standard interfaces (CLI).
- Lifecycle hooks must succeed on clean machines; use npm exec.
- Regenerate and commit lock files after manifest changes.
- **Robust editing**: Run formatter (e.g. clang-format) IMMEDIATELY BEFORE replace to normalize disk state; do not re-read file unless changed externally.

Source: github:metyatech/agent-rules@HEAD/rules/global/linting-formatting-and-static-analysis.md

# Linters, formatters, and static analysis

- Every code repo must have a formatter and a linter/static analyzer for its primary languages.
- Prefer one formatter and one linter per language; avoid overlapping tools.
- Enforce in CI: run formatting checks (verify-no-changes) and linting on pull requests and require them for merges.
- Treat warnings as errors in CI.
- Do not disable rules globally; keep suppressions narrow, justified, and time-bounded.
- Pin tool versions (lockfiles/manifests) for reproducible CI.
- Repositories with GitHub Actions must configure Dependabot version updates for applicable package ecosystems and for `github-actions`, unless the repository has no external dependency/update surface.
- For web UI projects, enforce automated visual accessibility checks in CI.
- Require dependency vulnerability scanning, secret scanning, and CodeQL for supported languages.

Source: github:metyatech/agent-rules@HEAD/rules/global/model-inventory.md

# Model inventory and routing

- Classify tasks into tiers: Free (trivial, Copilot 0x only), Light, Standard, Heavy, Large Context (>200k tokens, prefer Gemini 1M context).
- Before spawning sub-agents, run `ai-quota` to check availability.
- Always explicitly specify `model` and `effort` from the model inventory when spawning agents; never rely on defaults.
- The full model inventory with agent tables, routing principles, and quota fallback logic is maintained in the `manager` skill.
- **Orchestrator model**: When spawning an orchestrator (manager/autonomous-orchestrator role), default to `claude-sonnet-4-6` with `medium` effort; use `claude-opus-4-6` with `medium` effort when strict rule compliance is required. Research shows higher effort degrades instruction-following on multi-constraint rule sets (arXiv:2505.11423). Use `high`/`max` effort only for complex reasoning tasks, not for rule compliance.
- **Gemini sub-agent reliability**: Do NOT use Gemini (`gemini` agent type) for sub-agent delegation. Even single Gemini agents hit 429 "No capacity available" server errors frequently, making them unreliable for unattended tasks. Use Claude or Copilot instead. Gemini CLI may be used interactively by the user but not as a spawned sub-agent.

Source: github:metyatech/agent-rules@HEAD/rules/global/multi-agent-delegation.md

# Multi-agent delegation

- Every agent runs in direct mode (human requester) or delegated mode (spawned by another agent); default to direct mode.
- In delegated mode, delegation is plan approval: do not re-request human approval, respond in English, emit no notification sounds, and report AC/verification concisely. If scope must expand, fail back to the delegator.
- Delegation prompts MUST state delegated mode and approval state, include AC/verification requirements, and focus on task context (agents read repo AGENTS.md automatically).
- If a delegated agent reports read-only/no-write constraints, it MUST run a minimal reversible OS-temp probe and report the exact failure verbatim.
- Restricted operations require explicit delegation: modifying rules/rulesets, merging/closing PRs, creating/deleting repos, releasing/deploying, and force-pushing/rewriting published history.
- Delegated agents must not modify rules directly; submit rule-gap suggestions in results for delegator review.
- Delegated agents inherit delegator repository scope but must not expand it; fail explicitly if unavailable.
- Do not run concurrent agents that modify the same repository/files; different repositories may run in parallel. When conflict risk is unclear, run sequentially.
- Do not stop delegated agents merely because they are slow, retrying, or producing weak intermediate output while still making progress.
- Stop a delegated agent only for a concrete reason: repo/file conflict risk, clear divergence from the approved direction, user-owned data risk, or a better-scoped replacement.
- If a delegated agent creates clearly agent-owned temp, plan, or memory files outside the target repo, assess and clean them up automatically; escalate only when ownership or value is genuinely ambiguous.
- When delegated agents are running and there is no other meaningful in-scope work to do locally, prefer waiting for their completion or next material state change rather than repeatedly returning early and polling without progress.

Execution discipline, agents-mcp dispatch configuration, and cost optimization details are in the `manager` skill.

Source: github:metyatech/agent-rules@HEAD/rules/global/planning-and-approval-gate.md

# Planning and approval gate

- **Default approval model**: in user-controlled repositories and for in-scope work, the user's request is plan approval. Do not require a separate explicit "yes" for normal implementation, testing, commits, pushes, releases, or deploys that are part of completing the requested task.
- **Still ask first**: destructive or hard-to-reverse actions, force-push/history rewrite, deleting repos/data, third-party account side effects, billing/permissions changes, or scope expansion beyond the user's request.
- **Uncertain impact**: request approval.
- **Delegated mode**: delegation itself is plan approval; fail back on scope expansion.
- **Blanket approval**: broad directives (e.g., "fix everything") cover all in-scope follow-up; re-request only for out-of-scope expansion.
- PR review feedback handling is always pre-approved; do not ask for approval before addressing PR comments.
- For user-owned publishable packages, explicit requests such as "commit & push" or "complete this fix" include approval for the release/publish chain when release is the normal completion path, unless the user explicitly limits scope.

Reviewer proxy approval procedures are in the `autonomous-orchestrator` skill.

Source: github:metyatech/agent-rules@HEAD/rules/global/quality-and-delivery.md

# Quality and delivery gates
Non-negotiable gates for any state-changing work or any claim of "done", "fixed", "working", or "passing".
1. **BEFORE** state-changing work: list AC as binary, testable statements.
2. **BEFORE** each git commit: repo's full verification suite must pass.
3. **WITH** each AC: define verification evidence.
4. **FOR** code/runtime changes: automated tests required. Bugfixes MUST include a regression test.
5. **ALWAYS**: run repo-standard verify command; if missing, add it.
6. **IN** final response: keep completion reporting concise by default. Maintain AC/evidence internally, and surface explicit AC/evidence sections only when the user requests them or when higher-priority rules require path-by-path accounting.

## Quality principles
- Quality (correctness, safety, robustness, verifiability) > speed/convenience.
- CI must run full suite on PRs/pushes; require passing checks for merges; add CI if missing.
- Commit-time hooks must run full verify and block commits; confirm hooks installed.
- Test-first: add/update tests, observe failure, implement fix, observe pass.
- Never swallow errors; fail fast with explicit context and validate config/external inputs at boundaries.
- For user-facing apps, perform deterministic runtime verification before completion, define the claimed runtime environment matrix, and prefer the least costly faithful verification environment that covers each claimed behavior.
- For systems whose primary behavior depends on multiple clients, environments, or handoff paths, define the claimed primary path matrix up front and make automated verification of each claimed primary path a completion gate, using the least-cost faithful layer that exercises the boundary.
- For systems whose behavior depends on persisted or carried state across runs or handoffs (for example local storage, cookies, caches, saved sessions, URL tokens, reconnect state, or server restarts), define the claimed primary state matrix up front and make automated verification of fresh-state, resumed-state, and stale-state transitions a completion gate for each affected primary path.
- When evidence differs by client, environment, or path, report each claimed client/environment/path separately as verified, reproduced-as-limitation, or unverified; never generalize evidence across them.
- Anything not directly verified must be reported as unverified or unsupported; use real devices or live environments only when lower-cost faithful environments cannot validate the behavior.
- For authentication, billing, authorization, data persistence, and other critical systems, passing unit/integration tests, CI, build, and health checks is necessary but insufficient; completion requires live or production-like end-to-end verification of the critical user journey.
- If an intended environment cannot be exercised with available tools or access, stop short of a completion claim, state the exact gap, and leave that environment unclaimed until verified.
- For GUI work, include a first-use walkthrough against the primary user goal, and do not conclude from functional correctness alone: require screenshot-based review plus automated checks for overflow, clipping, wrapping, and clearly visible primary/current state where feasible; if the user still reports confusion, treat that as a failed acceptance gate and add a regression check for that confusion class before concluding.
- For GUI work that affects first-use flow, information hierarchy, navigation, or user guidance, require a separate agent review before concluding; the reviewer must assess the rendered UI from the primary user goal and report whether the next action and result location are immediately understandable.
- For GUI work, perform a whole-screen plausibility review: if the result would look obviously wrong, broken, or visually incoherent to a reasonable user at a glance, treat it as unfinished even when tests pass.
- On every user-reported bug, identify the earliest deterministic gate that should have caught it and add or strengthen that gate in the same change set before concluding; a fix without a new catching gate is incomplete.
- For GUI states that intentionally block progress (for example auth walls, modal dialogs, overlays, loading covers, or permission prompts), treat each blocking state as a primary UI state and verify at every primary viewport that it visually dominates, suppresses background interaction, and keeps the next action obvious.
- Never claim bug-free behavior. Report scope, evidence, and residual risk explicitly, and do not let external checks or reviews justify concluding while a known gap against the requested outcome remains.
- For AI review bots, follow the re-triggering procedures in the `pr-review-workflow` skill. Detailed evidence format and procedures are in the quality-workflow skill.

Source: github:metyatech/agent-rules@HEAD/rules/global/release-and-publication.md

# Release and publication

- Include LICENSE in published artifacts (copyright holder: metyatech).
- Do not ship build/test artifacts or local configs; ensure a clean environment can use the product via README steps.
- Define a SemVer policy and document what counts as a breaking change.
- Keep package version and Git tag consistent.
- Run dependency security checks before release.
- Verify published packages resolve and run correctly before reporting done.
- For new user-owned repositories created during a task, create the GitHub remote and push the initial history before reporting complete unless the user explicitly opts out.
- For public repos, set GitHub Description, Topics, and Homepage. Assign topics from the standard set defined in the `release-publish` skill.
- For public npm packages published from GitHub Actions, use npm trusted publishing (OIDC) instead of long-lived npm tokens whenever supported, and keep the publish workflow on a Node/npm runtime that satisfies trusted-publishing requirements.
- Before reporting a publishable-package change as complete, verify the full delivery chain (commit → push → version bump → release → publish → install verify). Procedures in the `release-publish` skill.
- For user-owned publishable packages, when the user asks to commit/push or finalize a fix, treat release/publish as in-scope follow-up by default and execute the full delivery chain unless the user explicitly opts out.

Source: github:metyatech/agent-rules@HEAD/rules/global/skill-authoring.md

# Skill authoring standards

- Follow the Agent Skills open standard (agentskills.io/specification).
- SKILL.md frontmatter: only `name` (lowercase alphanumeric + hyphens, max 64 chars) and `description` (explain trigger conditions).
- SKILL.md body must be platform-agnostic: no platform-specific tool names. Write in terms of intent.
- Platform-specific examples belong in README.md, not SKILL.md.
- Each skill lives in its own repository with SKILL.md at root.
- Install/manage via `npx skills add <owner>/<repo> --yes --global`.
- Default to public repositories; include a LICENSE file (prefer MIT).
- Write SKILL.md and README.md in English; keep instructions concise and action-oriented.
- Do not duplicate rules already covered by AGENTS.md global rules; reference them instead.

Source: github:metyatech/agent-rules@HEAD/rules/global/task-lifecycle-tracking.md

# Task lifecycle tracking

- When an actionable task emerges during a session, immediately record it with `task-tracker add` so it persists on disk regardless of session termination.
- `task-tracker` is the persistent cross-session tracker; session-scoped task tools (e.g., TaskCreate) are supplementary. Always use `task-tracker add` first; session-scoped tools may be used in addition but never as a replacement.
- At the start of any session that may involve state-changing work, run `task-tracker check` and report findings before starting new work.
- When reporting a task as complete, state the lifecycle stage explicitly (committed/pushed/released/etc.); never claim "done" when downstream stages remain incomplete.
- If `task-tracker` is not installed, install it via `npm install -g @metyatech/task-tracker` before proceeding.
- CLI: `task-tracker add "desc"` / `check` / `list` / `done <id>` / `remove <id>` / `update <id> --stage <stage>` — use `--stage`, NOT `--status`.
- Persistent stages: `pending`, `in-progress`, `committed`, `released`, `done`.
- Derived display stage: `pushed`. Do not create tracker-only follow-up commits just to record `pushed`; record `committed` in the closing code commit and let `task-tracker` derive `pushed` from upstream reachability of that committed event.
- The task-tracker state file (`.tasks.jsonl`) must be committed to version control; do not add it to `.gitignore`.

Source: github:metyatech/agent-rules@HEAD/rules/global/thread-inbox.md

# Thread inbox

- `thread-inbox` is the persistent cross-session conversation context tracker. Use it to preserve discussion topics, decisions, and context that span sessions.
- If `thread-inbox` is not installed, install it via `npm install -g @metyatech/thread-inbox` before proceeding.
- Store `.threads.jsonl` in the workspace root directory (use `--dir <workspace-root>`). Do not commit it to version control.
- Add `.threads.jsonl` to the repository `.gitignore` explicitly so it stays untracked by default.
- At session start, run `thread-inbox inbox` and `thread-inbox list --status waiting` to find threads needing attention; report findings before starting new work.
- Do not create threads for tasks already tracked by `task-tracker`; threads are for context and decisions, not work items.
- CLI: `thread-inbox new "title" --dir <dir>` (must create before adding messages) / `add <id> --from user|ai "msg" --dir <dir>` / `inbox --dir <dir>` / `list --status <status> --dir <dir>`.
- If a thread captures a persistent behavioral preference, encode it as a rule and resolve the thread.
- Detailed usage procedures (status model, when to create/add messages, lifecycle) are in the `manager` skill.

Source: github:metyatech/agent-rules@HEAD/rules/global/user-identity-and-accounts.md

# User identity and accounts

- The user's name is "metyatech".
- Any external reference using "metyatech" (GitHub org/user, npm scope, repos) is under the user's control.
- The user has GitHub and npm accounts.
- Use the gh CLI to verify GitHub details when needed.
- When publishing, cloning, adding submodules, or splitting repos, prefer the user's "metyatech" ownership unless explicitly instructed otherwise.

Source: github:metyatech/agent-rules@HEAD/rules/global/writing-and-documentation.md

# Writing and documentation
## User responses
- Respond in Japanese unless the user requests otherwise.
- Report commit/push status only when the turn changed files; keep it simple unless the user asks for details.
- In direct mode, emit the Windows SystemSounds.Asterisk sound after completing a response; delegated agents never emit sounds, and managers emit at most once for the overall task.
- When delivering a new tool, feature, or artifact, explain what it is, how to use it, and its key capabilities.
- Prefer short, user-centric progress reports. Explain what the user can now do, not implementation details, unless internals are requested.
- In direct mode, keep intermediary progress reports to the minimum required by higher-priority rules; prefer long uninterrupted execution blocks over frequent narration, and interrupt mid-task only for blockers, mandatory approvals, material risk/scope changes, or final completion.
- Do not include AC/evidence sections or command transcripts in normal user reports unless explicitly requested.
- At the end of a session or task, report any lingering unresolved technical friction or environment issues.
## Developer-facing writing
- Write developer documentation, code comments, commit messages, and rule modules in English.
## README and docs
- Every repository must include README.md covering overview/purpose, supported environments/compatibility, install/setup, usage examples, dev commands, required env/config, release/deploy steps if applicable, and links to SECURITY.md / CONTRIBUTING.md / LICENSE / CHANGELOG.md when they exist.
- For any change, assess documentation impact and update affected docs in the same change set so docs match behavior (README, docs/, examples, comments, templates, ADRs/specs, diagrams).
- If no documentation updates are needed, explain why in the final response.
- For CLIs, document every parameter with a description and at least one example, plus one end-to-end example command.
- Do not include user-specific local paths, fixed workspace directories, drive letters, or personal data in doc examples; prefer repo-relative paths and placeholders.
## Markdown linking
- When a Markdown document links to a local file, use a path relative to the Markdown file.
