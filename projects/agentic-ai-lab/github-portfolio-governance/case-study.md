# GitHub Portfolio Governance Case Study

## Initial State

The GitHub account contained a mix of:

- active portfolio projects
- private development repositories
- old IBM/Coursera course material
- unfinished experiments
- duplicate projects
- empty placeholder repositories

The goal was to reduce clutter without deleting useful work by mistake.

## Workflow

### 1. Bao — Repository Evaluation

Bao reviewed repositories read-only and classified them as:

- KEEP
- KEEP BUT IMPROVE
- ARCHIVE CANDIDATE
- DELETE CANDIDATE
- MANUAL REVIEW

Bao evaluated repository purpose, originality, completeness, documentation, technical depth, redundancy, relevance, and portfolio value.

### 2. Veritas — Independent Verification

Veritas independently reviewed proposed deletion candidates before any destructive action.

That review caught an important difference in a private application repository that initially looked duplicated by a related private repository.

Bao initially classified the source repository as a DELETE CANDIDATE because its purpose appeared redundant.

Veritas found that:

- the source repository contained unique newer implementation work
- its useful repository history was not preserved in the related repository
- some duplicated generated or bulky assets were not the deletion blocker

Result:

The source repository was changed from DELETE CANDIDATE to KEEP.

That is why independent verification was required.

### 3. Human Approval Gate

No destructive action was performed automatically.

Repositories were deleted only after:

1. Bao evaluation
2. Veritas verification
3. explicit human approval

### 4. Controlled Cleanup

Repositories removed during the cleanup included low-value course artifacts, abandoned stubs, and empty placeholder repositories.

Empty private placeholder repositories were independently confirmed to contain no commits, files, releases, artifacts, issues, or useful history before deletion.

## Orchestration Failure / Recovery

During Bao's private-repository evaluation, the worker hit a provider rate limit.

Hermes reported:

`rate-limited (quota wall) — requeued without counting a failure`

The task later resumed automatically and completed successfully.

That showed the orchestration layer could recover from a temporary provider-capacity limit without incorrectly recording the task as failed.

## Final Repository Strategy

### KEEP

- `asgard-ai-portfolio`
- `asgard-ai-foundry`
- anonymized private application repository preserved after independent verification

### KEEP BUT IMPROVE

- `cybersecurity-notebooks`
- `Applied_Data_Science_Capstone_SpaceX_IBM`

### MANUAL REVIEW

- anonymized private utility/archive repository
- anonymized related private application repository
- anonymized private visualization repository

## Lessons Learned

- Research-agent recommendations should not directly trigger destructive actions.
- Independent verification can catch incorrect assumptions about duplication.
- Human approval is essential for irreversible repository deletion.
- Private repositories require authenticated inspection; public-only inventory can miss important context.
- Provider rate limits should be treated as orchestration events, not task failures.
- The same specialist agents become more useful across projects when their skills, workflows, and validation patterns are reusable.
