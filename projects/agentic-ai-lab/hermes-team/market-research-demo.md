# Market Research Assistant — End-to-End Demo

## Goal

Validate the Hermes Lean Agent Team on a realistic multi-agent workflow with persistent specialist profiles and Kanban routing.

## Workflow

1. Odin receives the research request.
2. Odin assigns research to Bao through Kanban.
3. Bao gathers and summarizes evidence.
4. Odin reviews Bao's findings.
5. If implementation is required, Odin assigns the work to Brokkr through Kanban.
6. Brokkr creates or modifies the required artifacts.
7. Odin assigns independent verification to Veritas through Kanban.
8. Veritas returns PASS, PASS WITH ISSUES, or FAIL.
9. Odin integrates the results into a concise final report.

## Required Routing

Odin → Kanban → Bao

When implementation is needed:

Odin → Kanban → Brokkr

For final independent verification:

Odin → Kanban → Veritas

## Rules

- Use actual persistent specialist profiles.
- Do not use `delegate_task` as a substitute for named-profile routing.
- Use the smallest necessary team.
- Preserve errors and failed attempts.
- Record model/provider usage.
- Record notable latency or orchestration issues.
- Do not use MoA unless clearly justified.
- Require human approval for destructive, irreversible, sensitive, or high-impact actions.

## Success Criteria

The demo succeeds when:

- Bao runs as the actual Bao profile.
- Brokkr runs as the actual Brokkr profile when implementation is required.
- Veritas independently validates the result.
- Odin coordinates the work instead of performing every specialist step itself.
- Kanban handoffs complete successfully.
- Final results can be reproduced.
- Failures and limitations are recorded.

## Evidence to Capture

- Kanban task IDs
- Assigned profile
- Active model/provider
- Task summaries
- Files created or modified
- Test results
- QA verdict
- Errors or retries
- Approximate task duration
- Architecture lessons learned