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

## Scope and composition

- AGENTS.md is self-contained; do not rely on parent/child AGENTS for inheritance or precedence.
- Maintain shared rules centrally and compose per project; use project-local rules only for truly local policies.
- Place AGENTS.md at the project root; only add another AGENTS.md for nested independent projects.
- Before doing any work in a repository that contains `agent-ruleset.json`, run `compose-agentsmd` in that repository to refresh its AGENTS.md and ensure rules are current.

## AGENTS.md freshness

- Pre-commit hooks should run `compose-agentsmd --compose` and stage any AGENTS.md changes automatically. Do not fail the commit on AGENTS.md drift; let the updated file be included in the commit.
- Do not add AGENTS.md freshness checks to CI. AGENTS.md is generated from external rule sources; checking it in CI creates cross-repo coupling. Pre-commit hooks are sufficient.

## Update policy

- Never edit AGENTS.md directly; update source rules and regenerate AGENTS.md.
- A request to "update rules" means: update the appropriate rule module and ruleset, then regenerate AGENTS.md.
- If the user gives a persistent instruction (e.g., "always", "must"), encode it in the appropriate module (global vs local).
- When acknowledging a new persistent instruction, update the rule module in the same change set and regenerate AGENTS.md.
- When creating a new repository, verify that it meets all applicable global rules before reporting completion: rule files and AGENTS.md, CI workflow, linting/formatting, community health files, documentation, and dependency scanning. Do not treat repository creation as complete until full compliance is verified.
- When updating rules, infer the core intent; if it is a global policy, record it in global rules rather than project-local rules.
- If a task requires domain rules not listed in agent-ruleset.json, update the ruleset to include them and regenerate AGENTS.md before proceeding.
- Do not include composed `AGENTS.md` diffs in the final response unless the user explicitly asks for them.

## Editing standards

- Keep rules MECE, concise, and non-redundant.
- Use short, action-oriented bullets; avoid numbered lists unless order matters.
- Prefer the most general applicable rule to avoid duplication.
- Write rules as clear directives that prescribe specific behavior ("do X", "always Y", "never Z"). Do not use hedging language ("may", "might", "could", "consider") — if a behavior is required, state it as a requirement; if it is not required, omit it.
- Do not use numeric filename prefixes (e.g., `00-...`) to impose ordering; treat rule modules as a flat set. If ordering matters, encode it explicitly in composition/tooling rather than filenames.

## Rule placement (global vs domain)

- Decide rule placement based on **where the rule is needed**, not what topic it covers.
- If the rule could be needed from any workspace or repository, make it global.
- Only use domain rules when the rule is strictly relevant inside repositories that opt in to that domain.
- Before choosing domain, verify: "Will this rule ever be needed when working from a workspace that does not include this domain?" If yes, make it global.

## Rules vs skills

- **Rules** = invariants/constraints (always loaded, concise). **Skills** = procedures/workflows (on-demand, detailed). When a rule grows with procedural content, extract to a skill.

Source: github:metyatech/agent-rules@HEAD/rules/global/autonomous-operations.md

# Autonomous operations

- Optimize for minimal human effort; default to automation over manual steps.
- Drive work from the desired outcome: choose the highest-quality safe path that satisfies the requested quality/ideal bar, and execute end-to-end.
- Treat speed as a secondary optimization; never trade down correctness, safety, robustness, or verifiability unless the requester explicitly approves that tradeoff.
- Assume end-to-end autonomy for repository operations (issue triage, PRs, direct pushes to main/master, merges, releases, repo admin) only within repositories under the user's control (e.g., owned by metyatech or where the user has explicit maintainer/push authority), unless the user restricts scope; for third-party repos, require explicit user request before any of these operations.
- Do not preserve backward compatibility unless explicitly requested; avoid legacy aliases and compatibility shims by default.
- When work reveals rule gaps, redundancy, or misplacement, proactively update rule modules/rulesets (including moves/renames) and regenerate AGENTS.md without waiting for explicit user requests.
- Continuously evaluate your own behavior, rules, and skills during operation. When you identify a gap, ambiguity, inefficiency, or missing guidance — whether through self-observation, task friction, or comparison with ideal behavior — update the appropriate rule or skill immediately without waiting for the user to notice or point out the issue. After each task, assess whether avoidable mistakes occurred and apply corrections in the same task. In delegated mode, include improvement suggestions in the task result.
- When the user points out a behavior failure, treat it as a systemic gap: fix the immediate issue, update rules to prevent recurrence, and identify whether the same gap pattern applies elsewhere — all in a single action. Do not wait for the user to enumerate each corrective step; a single observation implies all necessary corrections.
- If you state a persistent workflow change (e.g., `from now on`, `I'll always`), immediately propose the corresponding rule update and request approval in the same task; do not leave it as an unrecorded promise. This is a blocking gate: do not proceed to the next task or close the response until the rule update is committed or explicitly deferred by the requester. When operating under a multi-agent-delegation model, follow that rule module's guidance on restricted operations before proposing changes.
- Because session memory resets between tasks, treat rule files as persistent memory; when any issue or avoidable mistake occurs, update rules in the same task to prevent recurrence.
- Never apply rules from memory of previous sessions; always reference the current AGENTS.md. If unsure whether a rule still applies, re-read it.
- Treat these rules as the source of truth; do not override them with repository conventions. If a repo conflicts, update the repo to comply or update the rules to encode the exception; do not make undocumented exceptions.

## Skill role persistence

- When the `manager` skill is invoked in a session, treat its role as session-scoped and continue operating as a manager/orchestrator for the remainder of the session.
- Do not revert to a direct-implementation posture mid-session unless the user explicitly asks to stop using the manager role/skill or selects a different role.

- When something is unclear, investigate to resolve it; do not proceed with unresolved material uncertainty. If still unclear, ask and include what you checked.
- Do not proceed based on assumptions or guesses without explicit user approval; hypotheses may be discussed but must not drive action.
- Make decisions explicit when they affect scope, risk, cost, or irreversibility.
- Prefer asynchronous, low-friction control channels (GitHub Issues/PR comments) unless a repository mandates another.
- Design autonomous workflows for high volume: queue requests, set concurrency limits, and auto-throttle to prevent overload.

## GitHub notifications

- After addressing a GitHub notification (CI failure fixed, PR reviewed, issue resolved), mark it as done using the GraphQL `markNotificationsAsDone` mutation.
- Detailed notification management procedures (API commands, pagination) are in the `manager` skill.

Source: github:metyatech/agent-rules@HEAD/rules/global/cli-standards.md

# CLI standards

- When building a CLI, follow standard conventions: --help/-h, --version/-V, stdin/stdout piping, --json output, --dry-run for mutations, deterministic exit codes, and JSON Schema config validation.

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

Source: github:metyatech/agent-rules@HEAD/rules/global/delivery-hard-gates.md

# Delivery hard gates

These are non-negotiable completion gates for any state-changing work and for any response that claims "done", "fixed", "working", or "passing".

## Acceptance criteria (AC)

- Before state-changing work, list Acceptance Criteria (AC) as binary, testable statements.
- For read-only tasks, AC may be deliverables/questions answered; keep them checkable.
- If AC are ambiguous or not testable, ask blocking questions before proceeding.
- Keep AC compact by default (aim: 1-3 items). Expand only when risk/complexity demands it or when the requester asks.

## Evidence and verification

- Do not run `git commit` until the repo's full verification command has passed in the current working tree. This applies to every commit, not only the final delivery.
- For each AC, define verification evidence (automated test preferred; otherwise a deterministic manual procedure).
- Maintain an explicit mapping: `AC -> evidence (tests/commands/manual steps)`.
- The mapping may be presented in a compact per-item form (one line per AC including evidence + outcome) to reduce verbosity.
- For code or runtime-behavior changes, automated tests are required unless the requester explicitly approves skipping.
- Bugfixes MUST include a regression test that fails before the fix and passes after.
- Run the repo's full verification suite (lint/format/typecheck/test/build) using a single repo-standard `verify` command when available; if missing, add it.
- Enforce verification locally via commit-time hooks (pre-commit or repo-native) and in CI; skipping requires explicit requester approval.
- For non-code changes, run the relevant subset and justify.
- If required checks cannot be run, stop and ask for explicit approval to proceed with partial verification, and provide an exact manual verification plan.

## Final response (MUST include)

- A compact goal+verification report. Labels may be `Goal`/`Verification` instead of `AC` as long as it is equivalent.
- `AC -> evidence` mapping with outcomes (PASS/FAIL/NOT RUN/N/A), possibly in compact per-item form.
- The exact verification commands executed and their outcomes.

Source: github:metyatech/agent-rules@HEAD/rules/global/implementation-and-coding-standards.md

# Engineering and implementation standards

- Prefer official/standard approaches recommended by the framework or tooling.
- Prefer well-maintained external dependencies; build in-house only when no suitable option exists.
- Prefer third-party tools/services over custom implementations when they can meet the requirements; prefer free options (OSS/free-tier) when feasible and call out limitations/tradeoffs.
- PowerShell: `\` is a literal character (not an escape). Do not cargo-cult `\\` escaping patterns from other languages; validate APIs that require names like `Local\...` (e.g., named mutexes).
- PowerShell: avoid assigning to or shadowing automatic/read-only variables (e.g., `$args`, `$PID`); use different names for locals.
- PowerShell: when invoking PowerShell from PowerShell, avoid double-quoted `-Command` strings that allow the outer shell to expand `$...`; prefer `-File`, single quotes, or here-strings to control expansion.
- If functionality appears reusable, assess reuse first and propose a shared module/repo; prefer remote dependencies (never local filesystem paths).
- Maintainability > testability > extensibility > readability.
- Single responsibility; keep modules narrowly scoped and prefer composition over inheritance.
- Keep dependency direction clean and swappable; avoid global mutable state.
- Avoid deep nesting; use guard clauses and small functions.
- Use clear, intention-revealing naming; avoid "Utils" dumping grounds.
- Prefer configuration/constants over hardcoding; consolidate change points.
- For GUI changes, prioritize ergonomics and discoverability so first-time users can complete core flows without external documents.
- Every user-facing GUI component (inputs, actions, status indicators, lists, and dialog controls) must include an in-app explanation (for example tooltip, context help panel, or equivalent).
- Do not rely on README-only guidance for GUI operation; critical usage guidance must be available inside the GUI itself.
- For GUI styling, prefer frameworks and component libraries that provide a modern, polished appearance out of the box (e.g., Material Design, shadcn/ui, Fluent); avoid hand-crafting extensive custom styles when an established design system can achieve the same result with less effort.
- When selecting a UI framework, prioritize built-in component quality and default aesthetics over raw flexibility; the goal is a standard, modern-looking UI with minimal custom styling code.
- Keep everything DRY across code, specs, docs, tests, configs, and scripts; proactively refactor repeated procedures into shared configs/scripts with small, local overrides.
- Persist durable runtime/domain data in a database with a fully normalized schema (3NF/BCNF target): store each fact once with keys/constraints, and compute derived statuses/views at read time instead of duplicating them.
- Fix root causes; remove obsolete/unused code, branches, comments, and helpers. When a tool, dependency, or service under user control malfunctions, investigate and fix the source rather than building workarounds. User-owned repositories are fixable code, not external constraints.
- Avoid leaving half-created state on failure paths. Any code that allocates/registers/starts resources must have a shared teardown that runs on all failure and cancellation paths.
- Do not block inside async APIs or async-looking code paths; avoid synchronous I/O and synchronous process execution where responsiveness is expected.
- Avoid external command execution (PATH-dependent tools, stringly-typed argument concatenation). Prefer native libraries/SDKs. If unavoidable: use absolute paths, safe argument handling, and strict input validation.
- Prefer stable public APIs over internal/private APIs. If internal/private APIs are unavoidable, isolate them and document the reason and the expected break risk.
- Externalize large embedded strings/templates/rules when possible.
- Do not commit build artifacts (follow the repo's .gitignore).
- Align file/folder names with their contents and keep naming conventions consistent.
- Do not assume machine-specific environments (fixed workspace directories, drive letters, per-PC paths). Prefer repo-relative paths and explicit configuration so workflows work in arbitrary clone locations.
- Temporary files/directories created by the agent MUST be placed only under the OS temp directory (e.g., `%TEMP%` / `$env:TEMP`). Do not create ad-hoc temp folders in repos/workspaces unless the requester explicitly approves.
- When building tools, CLIs, or services intended for agent use, design for cross-agent compatibility. Do not rely on features specific to a single agent platform (Claude Code, Codex, Gemini CLI, Copilot). Use standard interfaces (CLI, HTTP, stdin/stdout, MCP) that any agent can invoke.
- After modifying dependency manifests (package.json, pyproject.toml, Cargo.toml, etc.), regenerate lock files (run `npm install`, `pip freeze`, `cargo generate-lockfile`, etc.) and include the updated lock file in the same commit. Never commit a manifest change without the corresponding lock file update.

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

Source: github:metyatech/agent-rules@HEAD/rules/global/multi-agent-delegation.md

# Multi-agent delegation

## Execution context

- Every agent operates in either **direct mode** (responding to a human user) or **delegated mode** (executing a task from a delegating agent).
- In direct mode, the "requester" is the human user. In delegated mode, the "requester" is the delegating agent.
- Default to direct mode. Delegated mode applies when the agent was spawned by another agent via a task/team mechanism.

## Delegated mode overrides

When operating in delegated mode:

- The delegation constitutes plan approval; do not re-request approval from the human user.
- Respond in English, not the user-facing language.
- Do not emit notification sounds.
- Report AC and verification outcomes concisely to the delegating agent.
- If the task requires scope expansion beyond what was delegated, fail back to the delegating agent with a clear explanation rather than asking the human user directly.

## Delegation prompt hygiene

- Delegated agents MUST treat the delegator as the requester and MUST NOT ask the human user for plan approval. If blocked by repo rules, escalate to the delegator (not the human).
- Delegating prompts MUST explicitly state delegated mode and whether plan approval is already granted; include AC and verification requirements.
- Agents spawned in a repository read that repository's AGENTS.md and follow all rules automatically. Do not duplicate rule content in delegation prompts; focus prompts on the task description, context, and acceptance criteria.

## Read-only / no-write claims

- If a delegated agent reports read-only/no-write constraints, it MUST attempt a minimal, reversible temp-directory probe (create/write/read/delete under the OS temp directory) and report the exact failure/rejection message verbatim.

## Restricted operations

The following operations require explicit delegation from the delegating agent or user. Do not perform them based on self-judgment alone:

- Modifying rules, rulesets.
- Merging or closing pull requests.
- Creating or deleting repositories.
- Releasing or deploying.
- Force-pushing or rewriting published git history.

## Rule improvement observations

- Delegated agents must not modify rules directly.
- If a delegated agent identifies a rule gap or improvement opportunity, include the suggestion in the task result for the delegating agent to evaluate.
- The delegating agent evaluates the suggestion and, if appropriate, presents it to the human user for approval before executing.

## Authority and scope

- Delegated agents inherit the delegating agent's repository access scope but must not expand it.
- Different agent platforms have different capabilities (sandboxing, network access, push permissions). Fail explicitly when a required capability is unavailable in the current environment rather than attempting workarounds.

## Cost optimization (model selection)

- Always explicitly specify `model` and `effort` when spawning agents; never rely on defaults.
- Minimize total cost (model pricing + reasoning tokens + context + retries).
- Detailed cost optimization guidance is in the `manager` skill.

## Parallel execution safety

- Do not run multiple agents that modify the same files or repository concurrently.
- Independent tasks across different repositories may run in parallel.
- If two tasks target the same repository, assess conflict risk: non-overlapping files may run in parallel; overlapping files must run sequentially.
- When in doubt, run sequentially to avoid merge conflicts and inconsistent state.

Source: github:metyatech/agent-rules@HEAD/rules/global/planning-and-approval-gate.md

# Planning and approval gate

## Approval waiver (trivial tasks)

- In direct mode, you MAY proceed without asking for explicit approval when the user request is a trivial operational check and the action is low-risk and reversible.
- Allowed under this waiver:
  - Read-only inspection and verification (including running linters/tests/builds) that does not modify repo files.
  - Spawning a sub-agent for a read-only smoke check (no repo writes; temp-only and cleaned up).
  - Creating temporary files only under the OS temp directory (and deleting them during the task).
- Not allowed under this waiver (approval is still required):
  - Any manual edit of repository files, configuration files, or rule files.
  - Installing/uninstalling dependencies or changing tool versions.
  - Git operations beyond status/diff/log (commit/push/merge/release).
  - Any external side effects (deployments, publishing, API writes, account/permission changes).
- If there is any meaningful uncertainty about impact, request approval as usual.

- Default to a two-phase workflow: clarify goal + plan first, execute after explicit requester approval.
- In delegated mode (see Multi-agent delegation), the delegation itself constitutes plan approval. Do not re-request approval from the human user. If scope expansion is needed, fail back to the delegating agent.
- If a request may require any state-changing work, you MUST first dialogue with the requester to clarify details and make the goal explicit. Do not proceed while the goal is ambiguous.
- Allowed before approval: read-only inspection, dependency install, formatters/linters/typecheck/tests/builds (including auto-fix), and deterministic code generation/build steps.
- Before any other state-changing execution: restate as AC, produce a plan, confirm with the requester, and wait for explicit "yes" before executing. Once approved, proceed without re-requesting; re-request only when changing or expanding the plan.
- Do not treat the original task request as plan approval.
- If state-changing execution starts without the required post-plan "yes", stop immediately, report the gate miss, add/update a prevention rule, regenerate AGENTS.md, and then restart from the approval gate.
- No other exceptions: even if the user requests immediate execution (e.g., "skip planning", "just do it"), treat that as a request to move quickly through this gate, not to bypass it.

## Scope-based blanket approval

- When the user gives a broad directive that clearly encompasses multiple steps (e.g., "fix everything", "do all of these"), treat it as approval for all work within that scope; do not re-request approval for individual sub-steps, batches, or obviously implied follow-up actions.
- Obviously implied follow-up includes: rebuild linked packages, restart local services, update global installs, and other post-change deployment steps covered by existing rules.
- Re-request approval only when expanding beyond the original scope or when an action carries risk not covered by the original directive.

## Reviewer proxy approval

- When the autonomous-orchestrator skill is active, the skill invocation itself constitutes blanket approval for all operations within user-owned repositories. The orchestrator MUST approve plans via reviewer proxy without asking the human user.
- The reviewer proxy evaluates plans against all rules, known error patterns, and quality standards before approving.
- If the reviewer proxy approves (all checklist items pass), proceed without human approval.
- If the reviewer proxy flags concerns, escalate to the human user.
- The human user may override or interrupt at any time; user messages always take priority.
- Reviewer proxy does NOT apply to restricted operations (creating/deleting repositories, force-pushing, rewriting published git history) — these always require human approval per Multi-agent delegation rules.
- During autonomous operation, the orchestrator applies rule modifications directly when the reviewer proxy confirms they are safe and consistent with existing policies. Escalate to the human user only if the change conflicts with existing rules or carries ambiguous risk.

Source: github:metyatech/agent-rules@HEAD/rules/global/post-change-deployment.md

# Post-change deployment

After modifying code in a repository, check whether the changes require
deployment steps beyond commit/push before concluding.

## Globally linked packages

- If the repository is globally installed via `npm link` (identifiable by
  `npm ls -g --depth=0` showing `->` pointing to a local path), run the
  repo's build command after code changes so the global binary reflects
  the update.
- Verify the rebuilt output is functional (e.g., run the CLI's `--version`
  or a smoke command).

## Locally running services and scheduled tasks

- If the repository powers a locally running service, daemon, or scheduled
  task, rebuild and restart the affected component after code changes.
- Verify the restart with deterministic evidence (new PID, port check,
  service status query, or log entry showing updated behavior).
- Do not claim completion until the running instance reflects the changes.

Source: github:metyatech/agent-rules@HEAD/rules/global/quality-testing-and-errors.md

# Quality, testing, and error handling

For AC definition, verification evidence, regression tests, and final reporting requirements, see Delivery hard gates.

## Quality priority

- Quality (correctness, safety, robustness, verifiability) takes priority over speed or convenience.

## Verification

- If you are unsure what constitutes the full suite, run the repo's default verify/CI commands rather than guessing.
- Enforce via CI: run the full suite on pull requests and on pushes to the default branch, and make it a required status check for merges; if no CI harness exists, add one using repo-standard commands.
- Configure required status checks on the default branch when you have permission; otherwise report the limitation.
- Do not rely on smoke-only gating or scheduled-only full runs for correctness; merges must require the full suite.
- Ensure commit-time automation (pre-commit or repo-native) runs the full suite and blocks commits. This is a hard prerequisite: before making the first commit in a repository during a session, verify that pre-commit hooks are installed and functional; if not, install them before any other commits.
- If pre-commit hooks cannot be installed (environment restriction, no supported tool), manually run the repo's full verify command before every commit and confirm it passes; do not proceed to `git commit` until verify succeeds.
- Never disable checks, weaken assertions, loosen types, or add retries solely to make checks pass.
- If the execution environment restricts test execution (no network, no database, sandboxed), run the available subset, document what was skipped, and ensure CI covers the remainder.
- When delivering a user-facing tool or GUI, perform end-to-end manual verification (start the service, exercise each feature, confirm correct behavior) in addition to automated tests. Do not rely solely on unit tests for user-facing deliverables.
- When manual testing reveals issues or unexpected behavior, convert those findings into automated tests before fixing; the test must fail before the fix and pass after.
- The verify script must include a lock-file integrity check that catches manifest/lock-file drift locally, before CI. Verify that every dependency declared in the manifest has a corresponding entry in the lock file.

## Tests

- Follow test-first: add/update tests, observe failure, implement the fix, then observe pass.
- Keep tests deterministic; minimize time/random/external I/O; inject when needed.
- If a heuristic wait is unavoidable, it MUST be condition-based with a hard deadline and diagnostics, and requires explicit requester approval.

## Exceptions

- If required tests are impractical, document the coverage gap, provide a manual verification plan, and get explicit user approval before skipping.

## Error handling and validation

- Never swallow errors; fail fast or return early with explicit errors.
- Error messages must reflect actual state and include relevant input context.
- Validate config and external inputs at boundaries; fail with actionable guidance.
- Log minimally but with diagnostic context; never log secrets or personal data.
- Remove temporary debugging/instrumentation before the final patch.

Source: github:metyatech/agent-rules@HEAD/rules/global/release-and-publication.md

# Release and publication

- Include LICENSE in published artifacts (copyright holder: metyatech).
- Do not ship build/test artifacts or local configs; ensure a clean environment can use the product via README steps.
- Define a SemVer policy and document what counts as a breaking change.
- Keep package version and Git tag consistent.
- Run dependency security checks before release.
- Verify published packages resolve and run correctly before reporting done.

## Public repository metadata

- For public repos, set GitHub Description, Topics, and Homepage.
- Assign Topics from the standard set below. Every repo must have at least one standard topic when applicable; repos that do not match any standard topic use descriptive topics relevant to their domain.
  - `agent-skill`: repo contains a SKILL.md (an installable agent skill).
  - `agent-tool`: CLI tool or MCP server used by agents (e.g., task-tracker, agents-mcp, compose-agentsmd).
  - `agent-rule`: rule source or ruleset repository (e.g., agent-rules).
  - `unreal-engine`: Unreal Engine plugin or sample project.
  - `qti`: QTI assessment ecosystem tool or library.
  - `education`: course content, teaching materials, or student-facing platform.
  - `docusaurus`: Docusaurus plugin or extension.
- Additional descriptive topics (language, framework, domain keywords) may be added freely alongside standard topics.
- Review and update the standard topic set when the repository landscape changes materially (new domain clusters emerge or existing ones become obsolete).
- Verify topics are set as part of the new-repository compliance gate.

## Delivery chain gate

- Before reporting a code change as complete in a publishable package, verify the full delivery chain (commit → push → version bump → release → publish → install verify). Do not stop mid-chain.
- Detailed delivery chain procedures are in the `release-publish` skill.

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
- The task-tracker state file (`.tasks.jsonl`) must be committed to version control; do not add it to `.gitignore`.

Source: github:metyatech/agent-rules@HEAD/rules/global/thread-inbox.md

# Thread inbox

- `thread-inbox` is the persistent cross-session conversation context tracker. Use it to preserve discussion topics, decisions, and context that span sessions.
- If `thread-inbox` is not installed, install it via `npm install -g @metyatech/thread-inbox` before proceeding.
- Store `.threads.jsonl` in the workspace root directory (use `--dir <workspace-root>`). Do not commit it to version control.
- At session start, run `thread-inbox inbox` and `thread-inbox list --status waiting` to find threads needing attention; report findings before starting new work.
- Do not create threads for tasks already tracked by `task-tracker`; threads are for context and decisions, not work items.
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
- Always report whether you committed and whether you pushed; include repo(s), branch(es), and commit hash(es) when applicable.
- After completing a response, emit the Windows SystemSounds.Asterisk sound via PowerShell only when operating in direct mode (top-level agent).
- If operating in delegated mode (spawned by another agent / sub-agent), do not emit notification sounds.
- If operating as a manager/orchestrator, do not ask delegated sub-agents to emit sounds; emit at most once when the overall task is complete (direct mode only).

- When delivering a new tool, feature, or artifact to the user, explain what it is, how to use it (with example commands), and what its key capabilities are. Do not report only completion status; always include a usage guide in the same response.

## Developer-facing writing

- Write developer documentation, code comments, and commit messages in English.
- Rule modules are written in English.

## README and docs

- Every repository must include README.md covering overview/purpose, supported environments/compatibility, install/setup, usage examples, dev commands (build/test/lint/format), required env/config, release/deploy steps if applicable, and links to SECURITY.md / CONTRIBUTING.md / LICENSE / CHANGELOG.md when they exist.
- For any change, assess documentation impact and update all affected docs in the same change set so docs match behavior (README, docs/, examples, comments, templates, ADRs/specs, diagrams).
- If no documentation updates are needed, explain why in the final response.
- For CLIs, document every parameter (required and optional) with a description and at least one example; also include at least one end-to-end example command.
- Do not include user-specific local paths, fixed workspace directories, drive letters, or personal data in doc examples. Prefer repo-relative paths and placeholders so instructions work in arbitrary environments.

## Markdown linking

- When a Markdown document links to a local file, use a path relative to the Markdown file.
