# lean-code-review Skill Specification

## Purpose

Design a custom Hermes skill named `lean-code-review` for Veritas. The skill makes Veritas a diff-first reviewer that inspects the complete in-scope Git change with minimal necessary context and returns `PASS`, `PASS WITH ISSUES`, or `FAIL`.

This is a specification only. Do not implement the skill from this document unless a later task explicitly authorizes implementation.

## Core Principles

1. Diff first: start from Git status and diffs, not the whole repository.
2. Non-modifying: never edit files, apply patches, format, stage, commit, push, rebase, merge, update generated files, or auto-fix while using this review skill.
3. Tests are execution: test, build, lint, and typecheck commands may change local or external state and require explicit authorization before running.
4. Minimal context: inspect changed hunks and only the smallest surrounding definitions, call sites, tests, or configuration needed to validate the review.
5. Evidence over opinion: every issue must cite file and line, hunk, or symbol evidence from the diff or targeted context.
6. Complete coverage: cover every in-scope changed file and every required risk category before giving a verdict.
7. Low token use: summarize relevant changes, avoid full-file reads unless hunk context is insufficient, and avoid repository-wide scans unless a concrete risk requires them.
8. Requirements-aware: compare the diff against the user task, parent handoff, comments, and stated acceptance criteria.
9. Separation of review and implementation: Veritas reports issues and suggested fixes; corrective implementation must be a separate task or rework cycle.

## Intended Skill Location

If implemented later, the skill should be a user/project skill named `lean-code-review`. The exact installation path is intentionally out of scope for this spec; implementation should follow the current Hermes skill-authoring conventions at that time.

## Activation Trigger

Use when Veritas is asked to review code changes, validate a Kanban implementation task, review a pull request locally, or decide whether a diff is safe to pass.

Do not use for:

- Writing or changing code.
- Applying review feedback.
- Broad architecture review with no diff.
- Full repository audits.
- Dependency upgrade impact analysis beyond changed lock/config files unless risk evidence requires it.
- Security audits unrelated to the current diff.

## Inputs

The skill should operate from these inputs, in order of authority:

1. User or Kanban task instructions, acceptance criteria, and explicit constraints.
2. Parent task handoffs and relevant board comments.
3. Explicit review target, explicit review range, target commit, or comparison/integration base when provided.
4. Git status, branch, upstream, merge-base, changed-file lists, and diffs.
5. Minimal surrounding code context needed to understand changed hunks.
6. Existing tests directly related to changed behavior.
7. Existing build/test output supplied by the task or previous worker.
8. New build/test output only when explicit authorization to run that command was given.

## Non-Modifying Boundary

Default allowed actions:

- Read task context.
- Read Git status, branch, upstream, merge-base, changed-file lists, and diffs.
- Read changed files and minimal context around changed hunks.
- Read directly related tests, configs, schemas, migrations, and docs.
- Run read-only inspection commands such as `git status`, `git diff`, `git branch`, `git merge-base`, and `git log`.

Default forbidden actions:

- Editing files or applying patches.
- Formatting, lint-fixing, snapshot updates, generated-doc updates, fixture updates, lockfile updates, or package artifact updates.
- Staging, committing, pushing, rebasing, merging, or changing branches.
- Running tests, builds, linters, typecheckers, package installs, generators, migrations, servers, or scripts without explicit authorization for the exact command or clearly bounded command class.
- Broad scans that load the entire repository without a concrete review question.

There is no edit exception inside this skill. If a review finds required changes, report them and use the correct Kanban action; do not implement them in the review run.

## Review Procedure

### 1. Establish Review Target, Range, and Base

Collect the task goal, acceptance criteria, and explicit review target. Keep these concepts separate:

- Review target: the branch, PR, commit, working tree, or artifact being reviewed.
- Review range: the exact committed diff expression to inspect, such as `A..B` or `A...B`.
- Comparison/integration base: a trusted base branch or commit representing the line the target integrates into, such as an explicit PR base, `main`, `master`, or `develop`.

Determine committed-change scope with this precedence:

1. If the task provides an explicit review range such as `A..B` or `A...B`, review exactly that committed range. Do not reinterpret the left side as an integration base for a different target.
2. Else if the task provides a target commit, review that commit against the parent, base, or range specified by the task. If the task does not specify how to compare the commit and the appropriate parent/base is not otherwise unambiguous, treat committed scope as unresolved.
3. Else if reviewing a feature branch or PR and an explicit integration base is provided by task context, PR metadata, or an unambiguous repository default/integration-branch signal, compute `merge-base HEAD <integration-base>` and review `merge-base...HEAD`.
4. Else do not guess a committed range. Review only working-tree changes and report committed scope as unresolved, requiring clarification before any PASS that depends on committed changes.

Never treat a branch's same-name tracking upstream, such as `origin/<feature>`, as the integration comparison base merely because it is the upstream. On a pushed feature branch it may equal `HEAD`, causing `merge-base...HEAD` to be empty and erasing committed feature changes from review. Similarly, do not infer `origin/main`, `origin/master`, `main`, or `master` as the base merely because the ref exists; use it only when task, PR, or repository-default evidence identifies it as the integration base.

If committed scope is unresolved, do not PASS when committed changes could be material to the requested review; return that review scope is unresolved and requires clarification.

Completion criterion: the report states the target, committed range, and comparison/integration base used, or states that committed scope was unresolved and only working-tree changes were reviewable.

### 2. Establish Complete Review Scope

Inspect Git status and changed-file lists before opening files. Scope includes all of the following unless the task explicitly narrows them:

- Committed branch/PR changes: the resolved explicit review range, or `merge-base...HEAD` when a safe integration base exists.
- Staged changes: `git diff --cached`.
- Unstaged tracked changes: `git diff`.
- Untracked, non-ignored files shown by Git status.
- Deleted and renamed files.

For untracked files, there is no Git hunk against HEAD; read the file only as needed to understand its purpose and risk. Treat unexplained untracked files as in scope until explicitly excluded by task context.

Completion criterion: every in-scope changed file is listed in a scope table with status, source (`committed`, `staged`, `unstaged`, `untracked`), and a short purpose/risk hypothesis or `unexplained`.

### 3. Read the Diff First

Inspect diffs before opening full files. For each changed file, capture:

- Hunk locations.
- Added, removed, and modified behavior.
- New or removed public interfaces.
- Test changes.
- Config, dependency, migration, or generated-file changes.
- Untracked file purpose and notable public behavior, if applicable.

Completion criterion: the reviewer can explain what each hunk or untracked file is intended to do without relying on a full repository scan.

### 4. Pull Minimal Context Only When Needed

Read extra context only for a specific question raised by the diff. Examples:

- Enclosing function/class/module for a changed hunk.
- A directly called helper whose contract is unclear.
- A directly related test fixture.
- A schema or config entry referenced by the change.
- A neighboring implementation pattern in the same file.

Avoid:

- Reading unrelated files for general familiarity.
- Searching for every occurrence of a symbol unless compatibility or call-site impact is a concrete risk.
- Reading full files when the changed hunk and enclosing definition are enough.

Completion criterion: every extra file read is justified by a concrete review question.

### 5. Check Required Risk Categories

Review every in-scope changed file against every category:

#### Logic defects

Look for incorrect conditionals, off-by-one errors, inverted flags, wrong defaults, async/control-flow mistakes, state mutation bugs, incomplete error handling, and mismatch between name and behavior.

#### Regression risk

Look for changed public contracts, changed defaults, removed behavior, silent behavior changes, compatibility breaks, and paths that existing callers may rely on.

#### Missing or weak tests

Check whether new behavior, bug fixes, edge cases, failure modes, and regression risks are covered by targeted tests. Flag tests that only assert implementation details, do not fail before the change, or miss the changed branch.

#### Security-sensitive changes

Inspect authentication, authorization, token/secret handling, file paths, subprocesses, deserialization, SQL/query construction, network calls, CORS, logging, dependency changes, and privilege boundaries. Flag any new exposure or missing validation.

#### Config drift

Check environment variables, defaults, feature flags, config schemas, docs, generated examples, CI config, deployment manifests, and lockfiles for consistency with the code change.

#### Unsupported assumptions

Flag assumptions not proven by code, tests, docs, task context, or existing conventions. Examples: assuming one OS, one provider, one path layout, one user role, one data shape, or one timing/order guarantee.

#### Requirement mismatch

Compare the diff to the requested task. Flag missing acceptance criteria, extra behavior, overbroad implementation, or solving a different problem than requested.

#### Accidental unrelated changes

Flag formatting churn, unrelated refactors, drive-by renames, generated drift, lockfile noise, and changed files whose purpose is not explained by the task.

Completion criterion: for each file, each category is either cleared by targeted evidence, marked not applicable with a reason, or has an evidence-backed finding.

### 6. Decide Whether More Context Is Justified

Escalate from diff-only review to targeted repository search only when a specific risk cannot be resolved otherwise. Acceptable triggers:

- A changed public function/class/type may have external call sites.
- A renamed or removed symbol needs call-site validation.
- A config key, env var, route, permission, or feature flag may have parallel definitions.
- A security boundary depends on caller guarantees.
- Tests reference fixtures or helpers whose behavior controls the assertion.

When searching, use narrow terms from the diff and stop only when the review question is answered or further search would be disproportionate.

Completion criterion: all in-scope files and required risk categories are covered; no broad repository scan occurs without a stated review question and bounded target.

### 7. Test and Command Review

Prefer reviewing test diffs and known output. Running commands is optional and never assumed safe.

Rules:

- Treat tests, builds, linters, typecheckers, package scripts, generators, migrations, and servers as state-changing execution.
- Run a command only when the user/task explicitly authorizes that exact command or a clearly bounded class such as `run targeted pytest tests for changed Python files`.
- If authorization is absent, say `not run: no explicit authorization`.
- If a command would be expensive, destructive, require missing services, or affect external state, do not run it unless that risk was explicitly accepted.
- Never update snapshots, fixtures, generated files, or lockfiles during review.
- If any authorized command is run, capture the command, result, and any relevant working-tree state change observed afterward.

Completion criterion: the final report states which tests were reviewed, which commands were run, or why commands were not run.

## Verdict Rules

### PASS

Use only when:

- All in-scope changed files were reviewed.
- Every required risk category was cleared or is not applicable with reason.
- The diff satisfies the stated requirements.
- No `BLOCKER`, `MAJOR`, or `MINOR` findings remain.
- Tests are adequate for the risk level, or the lack of tests is justified by the nature of the change.
- No unexplained unrelated changes remain.
- Positive evidence is reported for the main acceptance criteria.

### PASS WITH ISSUES

Use only when:

- All in-scope changed files were reviewed.
- Every required risk category was cleared or has findings.
- The change is probably acceptable.
- Findings are only `MINOR` and do not appear to break requested behavior, safety, or compatibility.
- Any `NOTE` items are limitations or context, not unresolved material risks.

Examples: minor test weakness, small documentation/config follow-up, low-risk maintainability concern, or a limitation that should be noted but does not block acceptance.

### FAIL

Use when any `BLOCKER` or `MAJOR` finding is present, including:

- Likely logic defect.
- Requirement not met.
- High or material regression risk.
- Security issue or secret exposure.
- Missing tests for a risky behavior change.
- Config/deployment drift likely to break runtime behavior.
- Unexplained unrelated changes that materially expand review scope.
- Reviewer cannot validate the diff because necessary context, scope, or base is missing and the risk is material.

## Severity Guidance

- `BLOCKER`: must be fixed before acceptance; always yields `FAIL`.
- `MAJOR`: likely bug, regression, missing critical test, security/config issue, or requirement mismatch; yields `FAIL`.
- `MINOR`: useful correction that does not block acceptance; yields `PASS WITH ISSUES` unless accompanied by higher severity.
- `NOTE`: non-actionable context, limitation, or positive/neutral observation; does not affect verdict.

Do not use `low`, `medium`, or `high` severity labels in the final findings; map them to `MINOR`, `MAJOR`, or `BLOCKER` before issuing the verdict.

## Required Output Format

The skill should produce concise reports in this shape:

```text
VERDICT: PASS | PASS WITH ISSUES | FAIL

BASE:
- Target: <branch/PR/commit/working tree/artifact>
- Committed range: <explicit A..B/A...B, merge-base...HEAD, single-commit comparison, or unresolved/not applicable>
- Compared against: <comparison/integration base, parent/base for commit, or limitation>

SCOPE:
- <file>: <status/source> — <purpose/risk summary>

POSITIVE EVIDENCE:
- <acceptance criterion or risk category>: <concise evidence>

FINDINGS:
- [BLOCKER|MAJOR|MINOR] <file>:<line or hunk> — <issue>
  Evidence: <specific diff/context evidence>
  Impact: <why it matters>
  Suggested fix: <review-level suggestion, not an applied edit>

TESTS / COMMANDS:
- Reviewed: <test files/output or "none">
- Run: <commands and results, or "not run: <reason>">

NOTES:
- <limitations, assumptions, or non-blocking observations>
```

If there are no findings:

```text
VERDICT: PASS

BASE:
- Target: <branch/PR/commit/working tree/artifact>
- Committed range: <explicit A..B/A...B, merge-base...HEAD, single-commit comparison, or unresolved/not applicable>
- Compared against: <comparison/integration base, parent/base for commit, or limitation>

SCOPE:
- <file>: <status/source> — <purpose/risk summary>

POSITIVE EVIDENCE:
- Requirements: <why the diff satisfies the task>
- Risk coverage: <how required categories were cleared>
- Tests: <why tests/output are adequate or not needed>

FINDINGS:
- None.

TESTS / COMMANDS:
- Reviewed: <test files/output or "none">
- Run: <commands and results, or "not run: <reason>">

NOTES:
- <optional limitation or "None.">
```

`POSITIVE EVIDENCE` is required for `PASS`, optional but encouraged for other verdicts, and must stay concise.

## Evidence Requirements

Each finding must include:

- File path.
- Line number, hunk location, or nearest stable symbol if line numbers are unavailable.
- Direct evidence from the diff or targeted context.
- Why the issue matters for correctness, security, regressions, or requirements.
- A suggested fix direction.

Do not report speculative issues without labeling the unsupported assumption and explaining what evidence is missing.

## Token Budget Strategy

The implemented skill should keep review context small by default:

1. Use Git status, changed-file lists, and `git diff --stat` for orientation.
2. Resolve target, committed range, and comparison/integration base before judging.
3. Read patch hunks before full files.
4. Read only enclosing definitions or nearby tests when needed.
5. Prefer targeted symbol searches over broad repository scans.
6. Summarize scope once; do not paste large diff blocks into the final report.
7. Stop gathering context only after every in-scope file and required risk category has been covered.

## Tooling Expectations

The skill should prefer Hermes read tools and non-mutating inspection commands:

- `kanban_show` when operating as a Kanban reviewer.
- `terminal` for Git inspection commands and explicitly authorized test/build commands.
- `read_file` for minimal file/context reads.
- `search_files` for bounded symbol/config/test lookup.

The skill must not call write-capable tools such as `write_file` or `patch`, and must not run formatters or fixers during review.

## Kanban Review Behavior

When used by Veritas on a Kanban review task:

1. Load the task context with `kanban_show`.
2. Inspect parent handoff metadata and artifacts before reviewing the diff.
3. Review only the implementation under review.
4. Determine the lane before selecting the terminal action:
   - Same-card review lane: the implementation card itself is in review and assigned to Veritas. Use `kanban_complete` for PASS/PASS WITH ISSUES, `kanban_request_changes` for actionable implementation rework, and `kanban_block` only for genuine external blockers.
   - Separate downstream QA/review card: the current card is a child or standalone QA task reviewing a parent artifact. Do not call `kanban_request_changes` on the QA card. Use `kanban_complete` with verdict metadata and clear findings; the parent/orchestrator should create or requeue corrective implementation work.
5. If pre-created downstream review/QA children depend on the implementation card, completing the implementation card releases them; do not also request same-card review.

Completion criterion: the final Kanban action targets the current card according to its lane and cannot accidentally requeue the QA task instead of the implementation task.

## Non-Goals

The skill must not become:

- A full static analyzer.
- A repository indexing workflow.
- A style or formatting reviewer unless style changes create functional risk.
- An auto-fixer.
- A replacement for CI.
- A broad security audit independent of the current diff.

## Acceptance Criteria for Future Implementation

A future implementation of `lean-code-review` is complete when:

- It loads with the trigger for code review and local diff review tasks.
- It instructs Veritas to resolve review target, committed range, and comparison/integration base before reviewing conclusions.
- It covers committed branch/PR changes, staged changes, unstaged changes, untracked files, deleted files, and renamed files unless explicitly excluded.
- It instructs Veritas to inspect diffs before reading full files.
- It strictly forbids modifications during review.
- It treats tests/builds/linters as explicit-authorization-only execution.
- It requires checks for logic defects, regression risk, tests, security-sensitive changes, config drift, unsupported assumptions, requirement mismatch, and unrelated changes.
- It maps `BLOCKER` and `MAJOR` to `FAIL`, `MINOR` to `PASS WITH ISSUES`, and `NOTE` to no verdict impact.
- It requires complete file and risk-category coverage before stopping context gathering.
- It requires file/line evidence for all findings and concise positive evidence for `PASS`.
- It includes low-token context discipline and bounded escalation rules.
- It documents same-card versus downstream Kanban review behavior.
- It is validated against at least one clean diff and one intentionally flawed diff before use.
