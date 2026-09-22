# lean-code-review validation fixtures

Minimal isolated artifacts for validating the `lean-code-review` skill. These fixtures are not production code and are intended to be reviewed as diff artifacts.

## Fixtures

1. `pass-small-helper/` — a small correct Python diff with targeted tests. Expected verdict: `PASS`.
2. `fail-threshold-logic/` — a small Python diff with an intentional boundary logic bug and weak coverage. Expected verdict: `FAIL`.

Each fixture contains:

- `task.md` — review prompt / acceptance criteria.
- `diff.patch` — the complete diff artifact to review.
- `expected-outcome.md` — brief expected review outcome for Veritas validation.
