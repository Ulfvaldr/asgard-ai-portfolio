# Hermes Lean Agent Team — Change Log

## 2026-09-12 — Initial Lean Architecture

### Decision

Adopt a four-agent Hermes team:

- Odin — Chief Agent / Orchestrator
- Brokkr — Developer Agent
- Veritas — Quality Assurance Agent
- Bao — Research Agent

### Design Principles

- No sub-managers initially.
- Prefer direct routing: Odin → specialist.
- Use the smallest team necessary for each task.
- Use one primary model per agent by default.
- Use Mixture of Agents only when additional cost is justified.
- Require human approval for destructive, irreversible, sensitive, or high-impact actions.
- VS Code and Git are the source of truth for reusable configuration.
- Keep reports concise unless additional detail is required.

### Reason for Redesign

The first multi-agent experiment used approximately seven agents plus coordination layers.

Observed problems included:

- increased latency,
- increased model cost,
- unnecessary orchestration complexity,
- parent agents not always resuming automatically after child completion,
- unexpected MoA expense,
- backend slot and profile-loading limitations,
- excessive manual bot-by-bot configuration.

### Lessons Learned

- More agents do not automatically produce better results.
- Delegation layers should exist only when they provide measurable value.
- Independent QA is useful, but QA should remain separate from implementation.
- Repository-based configuration improves reproducibility.
- Cloning/templates should reduce manual Hermes setup.
- Failures should be preserved as engineering evidence.

### Files Created

- README.md
- team-config.md
- odin-soul.md
- brokkr-soul.md
- veritas-soul.md
- bao-soul.md
- change-log.md

### Next Step

Define the initial repository README and document the architecture before creating Hermes profiles.

## 2026-09-13 — Named Profile Routing Validation

### Finding

Hermes `delegate_task` does not automatically launch the persistent Brokkr, Veritas, or Bao profiles.

A diagnostic delegation from Odin showed that the child agent inherited:

- Odin's active profile
- Odin's model
- Odin's provider

The child was only instructed to act as Bao and did not load Bao's persistent SOUL/profile configuration.

### Architecture Correction

Use Kanban for real specialist routing.

Use `delegate_task` only for temporary child agents where specialist-specific model, provider, SOUL, memory, or profile configuration is not required.

### Kanban Validation

Kanban successfully discovered the persistent profiles:

- odin
- brokkr
- veritas
- bao
- default

The Hermes gateway is required for Kanban dispatch.

### Specialist Runtime Verification

Bao Kanban task verified:

- Profile: bao
- Model: deepseek/deepseek-v4-pro
- Provider: nous
- Persistent Bao SOUL/profile loaded

Brokkr Kanban task verified:

- Profile: brokkr
- Model: gpt-5.5
- Provider: openai-codex
- Persistent Brokkr profile configuration loaded

Veritas Kanban task verified:

- Profile: veritas
- Model: gpt-5.6-sol
- Provider: openai-codex
- Persistent Veritas SOUL/profile configuration loaded

### Lesson Learned

Prompting a temporary child agent to "act as" a specialist is not equivalent to routing work to the persistent specialist profile.

Profile-specific model selection, provider selection, SOUL configuration, and environment must be verified at runtime rather than assumed.

Kanban is the validated routing mechanism for the Hermes Lean Agent Team.

## 2026-09-16 — Market Research Demo Validation Complete

### Goal
Validate the lean Hermes team using persistent named profiles, Kanban routing, independent QA, correction loops, and portfolio artifact generation.

### Workflow

1. Bao research
   - Task: `t_8dca91a5`
   - Output: `research/2026-smb-manufacturing-ai-adoption.md`
   - Result: completed
   - Lesson: initial research was useful but contained citation-to-claim and wording issues.

2. Veritas initial QA
   - Task: `t_52a5c228`
   - Result: `PASS WITH ISSUES`
   - Findings included unsupported or overstated claims, citation mismatches, and an inaccurate self-reported duration.

3. Odin review
   - Task: `t_a87f4538`
   - Result: `REVISE`
   - Decision: return research to Bao before implementation.

4. Bao revision
   - Task: `t_5d9c5fc2`
   - Result: completed
   - Changes:
     - corrected citation-to-claim mismatches
     - added direct vendor citations
     - qualified global and segment extrapolations
     - softened unsupported wording
     - labeled opportunity conclusions as hypotheses/inference
     - corrected the original task runtime record to 170 seconds

5. Veritas re-verification
   - Task: `t_1e0e79c2`
   - Result: `PASS`
   - Decision: research approved for Brokkr.

6. Brokkr portfolio artifact build
   - Task: `t_fc8a4cfc`
   - Output: `projects/agentic-ai-lab/hermes-team/market-research-portfolio-brief.md`
   - Result: completed
   - No Git commit made.

7. Veritas final artifact QA
   - Tasks:
     - `t_f831220d`
     - `t_92ba5ef3`
   - Result: `FAIL` on both runs
   - Finding:
     - Brokkr introduced unsupported wording around "back-office automation"
     - the artifact also presented broader small-business evidence as if it were manufacturing-specific

8. Brokkr targeted correction
   - Task: `t_306ba44f`
   - Result: completed
   - Change:
     - removed unsupported "back-office automation" wording
     - separated manufacturer-supported workflow evidence from broader small-business evidence
     - no new research added
     - no commit made

9. Veritas final re-check
   - Task: `t_e278c5e0`
   - Result: `PASS`
   - Final status: portfolio artifact approved as portfolio-ready

### Key Lessons Learned

- Persistent named-profile routing through Kanban successfully exercised Bao, Brokkr, Veritas, and Odin as distinct agents.
- Independent QA adds real value even after upstream research has already passed review.
- A downstream artifact can introduce new unsupported claims even when the source research is correct.
- Verification should occur both before and after artifact generation.
- Failures should be preserved as engineering evidence rather than hidden.
- Small targeted correction tasks are more efficient than restarting the entire workflow.
- Measured runtime and task IDs provide useful traceability for AgentOps-style documentation.
- The lean 4-agent architecture is sufficient for this workflow without additional managers or unnecessary agent hops.

### Final Validation State

Research:
- validated

Portfolio brief:
- validated

Final artifact:
- `projects/agentic-ai-lab/hermes-team/market-research-portfolio-brief.md`

Final QA:
- `PASS`

Final QA task:
- `t_e278c5e0`

## 2026-09-19 — Rebuild Guide Validation Cycle

### Goal
Validate `projects/agentic-ai-lab/hermes-team/rebuild-guide.md` as a reproducible Hermes Lean Agent Team rebuild guide.

### Workflow

1. Brokkr initial guide creation
   - Task: `t_445f3af1`
   - Output: `projects/agentic-ai-lab/hermes-team/rebuild-guide.md`
   - Result: completed

2. Veritas initial QA
   - Task: `t_f9f41ecf`
   - Result: `FAIL`
   - Findings:
     - exact per-profile skill configuration was missing
     - Windows PowerShell assumptions were unclear
     - Hermes version was not bounded
     - profile-scoped verification had ambiguities

3. Brokkr revision
   - Task: `t_7d27db3b`
   - Result: completed
   - Changes:
     - documented exact per-profile skill configuration
     - clarified Windows PowerShell platform and shell assumptions
     - bounded the validated Hermes version context
     - clarified profile-scoped verification requirements

4. Veritas re-check
   - Task: `t_a40078f3`
   - Result: `PASS`
   - Final status: guide declared ready to commit

### Key Lesson Learned

Reproducibility requires documenting exact live configuration, platform and shell assumptions, version boundaries, and profile-scoped verification.

### Final Validation State

Rebuild guide:
- validated

Final QA:
- `PASS`

Final QA task:
- `t_a40078f3`

## 2026-09-20 — Lean Code Review Skill Behavioral Validation

### Goal
Validate the repository-managed `lean-code-review` skill as a non-modifying, low-context code review workflow.

### Workflow

1. Brokkr skill implementation
   - Task: `t_48cf8953`
   - Output: `projects/agentic-ai-lab/hermes-team/skills/software-development/lean-code-review/SKILL.md`
   - Result: completed
   - Scope: implemented the repository-managed Hermes skill for lean, diff-first code review.

2. Veritas implementation QA
   - Task: `t_4e218033`
   - Result: `FAIL`
   - Finding: the skill content was not rejected for static structure alone; the task failed because the approved acceptance criteria required behavioral validation against both a clean diff and an intentionally flawed diff.

3. Brokkr validation fixture creation
   - Task: `t_62369114`
   - Output: `projects/agentic-ai-lab/hermes-team/validation/lean-code-review/`
   - Result: completed
   - Fixtures:
     - `pass-small-helper` — clean helper/test diff expected to pass
     - `fail-threshold-logic` — intentionally flawed threshold logic diff expected to fail

4. Veritas behavioral validation
   - Task: `t_88ee0c20`
   - Result: `PASS`
   - Validation: Veritas ran the installed `lean-code-review` skill against both isolated fixtures.
   - Outcomes:
     - `pass-small-helper` returned `PASS`
     - `fail-threshold-logic` returned `FAIL`
   - The failing fixture was correctly identified as a strict `>` versus `>=` logic defect with missing exactly-at-threshold coverage.

### Review Constraints Verified

- Review remained non-modifying.
- Review stayed low-context and diff-focused.
- No unauthorized test, build, lint, or typecheck execution occurred.

### Key Lesson Learned

Static skill validation is not sufficient when behavioral acceptance criteria require scenario testing. Skills that define review behavior need fixture-based validation against representative passing and failing diffs.

### Final Validation State

Lean code review skill:
- behaviorally validated

Final QA:
- `PASS`

Final QA task:
- `t_88ee0c20`

## 2026-09-20 — Final Runtime Migration and Documentation Pass

### Goal
Capture the final validated Hermes Lean Agent Team architecture and the operational lessons needed to rebuild it without relying on remembered profile state.

### Final Validated Runtime

- Odin: GPT-5.6 Sol through `openai-codex` OAuth/subscription.
- Brokkr: GPT-5.5 through `openai-codex`.
- Veritas: GPT-5.6 Sol through `openai-codex`.
- Bao: `deepseek/deepseek-v4-pro` through Nous.

Nous remains available, but the validated final team uses it primarily for Bao research to reduce cost. `openai-codex` is profile-scoped OAuth/subscription usage, not raw API-key usage.

### Migration Lessons

- Hermes v0.21.1 profile-scoped auth mattered: changing Odin's model/provider config alone did not make Odin runnable because Odin lacked profile-scoped `openai-codex` auth. Running `hermes -p odin setup model` created the correct profile auth and completed the migration.
- After a Hermes update, a running gateway can continue serving pre-update modules until it is restarted.
- `gateway.multiplex_profiles` was set to `true` so named profiles could be dispatched through the gateway.
- Native Windows rebuild warnings are orchestration/platform lessons to document and re-validate, not hidden failures.

### Validation Lessons Preserved

- The `lean-code-review` skill is repository-managed, read-only, diff-first, and behaviorally validated.
- Behavioral validation used one clean PASS fixture and one intentionally flawed FAIL fixture.
- Final Veritas validation task `t_88ee0c20` passed.
- The portfolio case study remains structured as: Problem -> First Design -> Observed Failures -> Redesign -> Validation -> Rebuild.

### Final Documentation Scope

Updated only final documentation where needed: `team-config.md`, `change-log.md`, `rebuild-guide.md`, and `README.md`. No agent behavior, model runtime configuration, skills, validation fixtures, production code, or unrelated experiment files were changed.

