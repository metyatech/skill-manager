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
- Avoid interactive git prompts by using --no-edit or setting GIT_EDITOR=true.
- If elevated privileges are required, use sudo directly; do not launch a separate elevated shell (e.g., Start-Process -Verb RunAs). Fall back to run as Administrator only when sudo is unavailable.
- Keep changes scoped to affected repositories; when shared modules change, update consumers and verify at least one.
- If no branch is specified, work on the current branch; direct commits to main/master are allowed.
- Do not assume agent platform capabilities beyond what is available; fail explicitly when unavailable.
- When building a CLI, follow standard conventions: --help/-h, --version/-V, stdin/stdout piping, --json output, --dry-run for mutations, deterministic exit codes, and JSON Schema config validation.
## Codex-only PowerShell safety
- `Remove-Item` (aliases: `rm`, `ri`, `del`, `erase`) ↁEUse: `if ([IO.File]::Exists($p)) { [IO.File]::SetAttributes($p,[IO.FileAttributes]::Normal); [IO.File]::Delete($p) }`
- `Remove-Item -Recurse` (aliases: `rmdir`, `rd`) ↁEUse: `if ([IO.Directory]::Exists($d)) { [IO.File]::SetAttributes($d,[IO.FileAttributes]::Normal); foreach ($e in [IO.Directory]::EnumerateFileSystemEntries($d,'*',[IO.SearchOption]::AllDirectories)) { [IO.File]::SetAttributes($e,[IO.FileAttributes]::Normal) }; [IO.Directory]::Delete($d,$true) }`
- In PowerShell, use `;` for sequential command chaining; never use `&&` or `||` as control-flow operators.
## Post-change deployment
- After modifying code, check whether deployment steps beyond commit/push are needed before concluding.
- If the repo is globally linked (`npm ls -g` shows `->` to local path), rebuild and verify the global binary is functional.
- If the repo powers a running service, daemon, or scheduled task, rebuild, restart, and verify with deterministic evidence.
- Do not claim completion until the running instance reflects the changes.
- Detection and verification procedures are in the `post-deploy` skill.

- **PowerShell native environment**: Always remember that this is a Windows/PowerShell environment. Do not use Unix commands like grep or ls directly; use native PowerShell cmdlets (Select-String, Get-ChildItem) or provided optimized tools like grep_search.

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
6. **IN** final response: AC -> evidence mapping with outcomes and verification commands.

## Quality principles
- Quality (correctness, safety, robustness, verifiability) > speed/convenience.
- CI must run full suite on PRs/pushes; require passing checks for merges; add CI if missing.
- Commit-time hooks must run full verify and block commits; confirm hooks installed.
- Test-first: add/update tests, observe failure, implement fix, observe pass.
- Never swallow errors; fail fast with explicit context.
- Validate config/external inputs at boundaries.
- For user-facing apps, perform deterministic runtime verification before completion.
- Before implementation is considered complete, define the claimed runtime environment matrix and verify every claimed environment directly; anything not directly verified must be reported as unverified or unsupported, never implied as covered.
- Prefer the least costly faithful verification environment: use automated tests, local or production-like environments, emulators, and simulators whenever they provide equivalent coverage; require real devices or live environments only for behaviors that cannot be validated faithfully otherwise.
- For authentication, billing, authorization, or data-persistence changes, completion requires end-to-end verification in a live or production-like deployed environment, including post-deploy smoke coverage of the critical user journey.
- For critical systems, passing unit/integration tests, CI, build, and health checks is necessary but insufficient; do not conclude until runtime user flows succeed in each claimed environment.
- When an intended environment cannot be exercised with available tools or access, stop short of a completion claim, state the exact gap, and treat that environment as out of scope until verified.
- For GUI/UX changes, include a first-use walkthrough against the primary user goal; functional E2E alone is insufficient when clarity/usability is in scope.
- If the user still reports a GUI flow as confusing, treat that as a failed acceptance gate: refine labels/order/flow and add a regression check for that confusion class before concluding.
- For GUI work, do not conclude from functional correctness alone: require screenshot-based review plus automated checks for horizontal overflow, clipping, unintended compact-control wrapping, and clearly visible primary/current state where feasible.
- Never claim bug-free behavior. Report scope, evidence, and residual risk explicitly.
- For AI review bots, follow the re-triggering procedures in the `pr-review-workflow` skill.

Detailed evidence format and procedures are in the quality-workflow skill.

Source: github:metyatech/agent-rules@HEAD/rules/global/release-and-publication.md

# Release and publication

- Include LICENSE in published artifacts (copyright holder: metyatech).
- Do not ship build/test artifacts or local configs; ensure a clean environment can use the product via README steps.
- Define a SemVer policy and document what counts as a breaking change.
- Keep package version and Git tag consistent.
- Run dependency security checks before release.
- Verify published packages resolve and run correctly before reporting done.
- For public repos, set GitHub Description, Topics, and Homepage. Assign topics from the standard set defined in the `release-publish` skill.
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
