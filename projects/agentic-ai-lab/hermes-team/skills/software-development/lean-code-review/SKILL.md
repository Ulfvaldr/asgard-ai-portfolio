---
name: lean-code-review
description: Review diffs without modifying code.
version: 0.1.0
author: Asgard AI Portfolio
platforms: [linux, macos, windows]
metadata:
  hermes:
    tags: [code-review, qa, git, kanban, veritas]
    related_skills: []
---

# Lean Code Review Skill

Diff-first review discipline for Veritas. Use the smallest evidence needed to review the complete in-scope Git change, then return `PASS`, `PASS WITH ISSUES`, or `FAIL`. This skill is read-only: report findings and suggested fixes, but never modify files or Git state during review.

## When to Use

- Veritas is asked to review code changes, a local diff, a pull request, or a Kanban implementation handoff.
- A task asks whether a change is safe to pass, merge, or return for rework.
- A reviewer must validate implementation against task requirements with low token use.

Do not use for:

- Writing or changing code.
- Applying review feedback.
- Broad architecture review with no diff.
- Full repository audits.
- Dependency impact analysis beyond changed lock/config files unless the diff shows concrete risk.
- Security audits unrelated to the current diff.

## Prerequisites

- A Git repository or explicit artifact to review.
- Task requirements, acceptance criteria, parent handoff, comments, or user instructions when available.
- Explicit authorization before running any test, build, lint, typecheck, install, generator, migration, server, or package script command.

## Procedure

1. Establish target, committed range, and integration base.
   - Keep separate: review target, committed review range, and comparison/integration base.
   - If an explicit range such as `A..B` or `A...B` is provided, review exactly that range.
   - If a target commit is provided, compare it only against the parent/base/range specified by the task or otherwise unambiguous evidence.
   - For a feature branch or PR, use `merge-base HEAD <integration-base>` only when task, PR metadata, or repository-default evidence identifies the integration base.
   - Do not treat a same-name tracking upstream such as `origin/<feature>` as the integration base just because it exists.
   - Do not infer `origin/main`, `origin/master`, `main`, or `master` merely because the ref exists.
   - Completion: the report states target, committed range, and compared-against base, or states committed scope is unresolved and only working-tree changes were reviewable.

2. Establish complete review scope before opening files.
   - Inspect Git status, changed-file lists, and diff stats first with read-only Git commands through `terminal`.
   - Include committed branch/PR changes from the resolved range, staged changes, unstaged tracked changes, untracked non-ignored files, deleted files, and renamed files unless task context explicitly excludes them.
   - Treat unexplained untracked files as in scope.
   - Completion: every in-scope file appears in the final scope table with status, source, and a purpose/risk hypothesis or `unexplained`.

3. Read diffs before file bodies.
   - Inspect hunks before reading full files.
   - Capture hunk locations, behavior changes, public interface changes, test changes, config/dependency/migration/generated-file changes, and untracked file purpose.
   - Completion: every hunk or untracked file can be summarized without relying on a broad repository scan.

4. Pull minimal context only for concrete review questions.
   - Use `read_file` for enclosing functions/classes/modules, directly called helpers, directly related tests/fixtures, schema/config entries, or a neighboring pattern in the same file.
   - Use `search_files` only for bounded symbol/config/test lookup when a concrete risk requires call-site or parallel-definition evidence.
   - Avoid unrelated files, whole-file reads when a hunk plus enclosing definition is enough, and repository-wide scans for general familiarity.
   - Completion: each extra file or search has a stated review question.

5. Cover every required risk category for every in-scope file.
   - Logic defects: wrong conditionals, off-by-one errors, inverted flags, wrong defaults, async/control-flow mistakes, mutation bugs, incomplete error handling, name/behavior mismatch.
   - Regression risk: changed public contracts, changed defaults, removed behavior, compatibility breaks, silent behavior changes.
   - Missing or weak tests: risky behavior, edge cases, failure modes, and regression paths lacking targeted tests.
   - Security-sensitive changes: auth, authorization, tokens/secrets, file paths, subprocesses, deserialization, queries, network calls, CORS, logging, dependencies, privilege boundaries.
   - Config drift: env vars, defaults, flags, config schemas, docs, generated examples, CI, deployments, lockfiles.
   - Unsupported assumptions: OS, provider, path layout, user role, data shape, timing, ordering, or caller guarantees not proven by evidence.
   - Requirement mismatch: missing acceptance criteria, extra behavior, overbroad implementation, or solving the wrong task.
   - Accidental unrelated changes: formatting churn, unrelated refactors, drive-by renames, generated drift, lockfile noise, unexplained files.
   - Completion: each file/category pair is cleared by targeted evidence, marked not applicable with reason, or represented by an evidence-backed finding.

6. Treat tests and commands as authorization-only execution.
   - Prefer reviewing test diffs and supplied output.
   - Run tests, builds, linters, typecheckers, package scripts, generators, migrations, or servers only when the task explicitly authorizes the exact command or a clearly bounded command class.
   - If not authorized, report `not run: no explicit authorization`.
   - Never update snapshots, fixtures, generated files, lockfiles, or package artifacts during review.
   - If an authorized command runs, record the command, result, and any observed working-tree state change afterward.
   - Completion: final report states reviewed tests/output, commands run, or why commands were not run.

## Non-Modifying Boundary

Allowed by default:

- `kanban_show` for review task context.
- `terminal` for read-only Git inspection: `git status`, `git diff`, `git diff --stat`, `git diff --name-status`, `git branch`, `git merge-base`, `git log`.
- `read_file` for changed hunks and minimal surrounding context.
- `search_files` for bounded symbol/config/test lookup tied to a stated risk.

Forbidden while using this skill:

- `write_file`, `patch`, formatters, fixers, snapshot updates, generated-doc updates, fixture updates, lockfile updates, or package artifact updates.
- Staging, committing, pushing, rebasing, merging, switching branches, or changing Git state.
- Package installs, tests, builds, lint, typecheck, generators, migrations, servers, or scripts without explicit authorization.
- Broad scans that load the whole repository without a concrete review question.

There is no edit exception. If required changes are found, report them and use the proper Kanban review action; corrective implementation belongs in a separate rework cycle.

## Verdict Rules

Use `PASS` only when:

- All in-scope changed files were reviewed.
- Every required risk category is cleared or not applicable with reason.
- The diff satisfies the stated requirements.
- No `BLOCKER`, `MAJOR`, or `MINOR` findings remain.
- Tests are adequate for the risk level, or lack of tests is justified by the change type.
- No unexplained unrelated changes remain.
- Positive evidence is reported for the main acceptance criteria.

Use `PASS WITH ISSUES` only when:

- All in-scope changed files were reviewed.
- Every required risk category is cleared or has findings.
- The change is probably acceptable.
- Findings are only `MINOR` and do not appear to break requested behavior, safety, or compatibility.
- Any `NOTE` items are limitations or context, not unresolved material risks.

Use `FAIL` when any `BLOCKER` or `MAJOR` finding is present, or when material scope/base/context is unresolved and prevents validation.

Severity labels:

- `BLOCKER`: must be fixed before acceptance; verdict is `FAIL`.
- `MAJOR`: likely bug, regression, missing critical test, security/config issue, or requirement mismatch; verdict is `FAIL`.
- `MINOR`: useful correction that does not block acceptance; verdict is `PASS WITH ISSUES` unless accompanied by higher severity.
- `NOTE`: non-actionable context, limitation, or positive/neutral observation; no verdict impact.

Do not use `low`, `medium`, or `high` in final findings; map them to `MINOR`, `MAJOR`, or `BLOCKER`.

## Required Output

```text
VERDICT: PASS | PASS WITH ISSUES | FAIL

BASE:
- Target: <branch/PR/commit/working tree/artifact>
- Committed range: <explicit A..B/A...B, merge-base...HEAD, single-commit comparison, unresolved, or not applicable>
- Compared against: <integration base, commit parent/base, or limitation>

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

If there are no findings, write `FINDINGS: - None.` and include concise positive evidence for requirements, risk coverage, and tests or test justification.

## Kanban Review Behavior

1. Start with `kanban_show` on the current review task.
2. Inspect parent handoff metadata, artifacts, comments, and task acceptance criteria before reviewing the diff.
3. Review only the implementation under review.
4. Determine the lane before choosing the final Kanban action:
   - Same-card review lane: the implementation card itself is in review and assigned to Veritas. Use `kanban_complete` for `PASS` or `PASS WITH ISSUES`, `kanban_request_changes` for actionable implementation rework, and `kanban_block` only for genuine external blockers.
   - Separate downstream QA/review card: the current card is a child or standalone QA task reviewing a parent artifact. Do not call `kanban_request_changes` on the QA card. Use `kanban_complete` with verdict metadata and clear findings so the parent/orchestrator can route rework.
5. If pre-created downstream review/QA children depend on the implementation card, completion releases them; do not also request same-card review.

Completion: the final Kanban action targets the current card according to its lane and cannot accidentally requeue a QA card instead of the implementation card.

## Pitfalls

- Empty `merge-base...HEAD` can mean the wrong base was chosen, especially when same-name upstream equals `HEAD`.
- Existing refs are not evidence of an integration base by themselves.
- A review that misses untracked, staged, unstaged, deleted, or renamed files is incomplete.
- Missing command authorization means commands are not run; it is not permission to guess results.
- Findings without file/line/hunk/symbol evidence are speculation and must be labeled as unsupported or omitted.

## Verification

Before finalizing, confirm:

- Target, committed range, and compared-against base are stated or unresolved scope is called out.
- Every in-scope changed file appears in `SCOPE`.
- Every required risk category was considered for every in-scope file.
- Every finding has evidence, impact, and suggested fix direction.
- `PASS`, `PASS WITH ISSUES`, and `FAIL` rules were applied consistently.
- Tests/commands section states reviewed evidence, executed authorized commands, or `not run: no explicit authorization`.
- No files, Git state, generated artifacts, snapshots, fixtures, or lockfiles were modified during review.
