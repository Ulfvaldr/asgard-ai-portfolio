# GitHub Portfolio Governance

A lean, documentation-first governance pattern for cleaning up a GitHub portfolio without letting agents perform destructive repository actions on their own.

## Problem
A GitHub account can accumulate outdated, incomplete, course-based, redundant, or low-value repositories over time. Cleaning them up manually risks deleting useful work or making inconsistent decisions.

## Goal
Use the existing Hermes Lean Agent Team to evaluate repositories, independently verify destructive-action and visibility-changing candidates, and require explicit human approval before any deletion, archival, rename, transfer, or visibility-change action.

## Non-Negotiable Safety Rule
No repository may be deleted, archived, renamed, transferred, or have visibility changed without independent verification and explicit human approval.

Independent verification is required before any archive, delete, rename, transfer, or visibility change. Human approval alone is not sufficient until the independent verification record is complete.

## Agent Workflow
- Bao — repository inventory and evidence-based evaluation
- Veritas — independent verification of archive, delete, rename, transfer, and visibility-change candidates
- Brokkr — reusable governance tooling, templates, and documentation
- Odin — orchestration and decision routing
- Human — approval gate for destructive or visibility-changing actions

## Governance Flow
1. Inventory repositories in read-only mode.
2. Evaluate each repository against consistent evidence fields.
3. Classify each repository as KEEP, KEEP BUT IMPROVE, ARCHIVE CANDIDATE, DELETE CANDIDATE, or MANUAL REVIEW.
4. Send every archive, delete, rename, transfer, or visibility-change candidate to independent verification.
5. Require explicit human approval before any destructive or visibility-changing action.
6. Record the approved action, final state, and lessons learned in a run log.

## Project Structure
- `README.md` — operating model, safety rule, workflow, and success criteria
- `case-study.md` — documented outcome and lessons from the portfolio cleanup
- `templates/repository-evaluation.md` — first-pass repository review template
- `templates/independent-verification.md` — second-pass verification template for destructive-action and visibility-changing candidates
- `templates/human-approval.md` — explicit human approval record template
- `templates/run-log.md` — run-level audit log template

## Reusable Templates
Use the templates in order:

1. `templates/repository-evaluation.md`
2. `templates/independent-verification.md` for any ARCHIVE CANDIDATE, DELETE CANDIDATE, rename, transfer, or visibility-change candidate
3. `templates/human-approval.md` before any destructive or visibility-changing action
4. `templates/run-log.md` to capture final decisions, before/after state, failures, and lessons

## Success Criteria
- Repository inventory completed
- Clear KEEP / KEEP BUT IMPROVE / ARCHIVE CANDIDATE / DELETE CANDIDATE / MANUAL REVIEW classifications
- Independent verification before archive, delete, rename, transfer, or visibility-change actions
- Human approval recorded
- Before/after repository state documented
- Failures and orchestration issues captured as lessons learned
