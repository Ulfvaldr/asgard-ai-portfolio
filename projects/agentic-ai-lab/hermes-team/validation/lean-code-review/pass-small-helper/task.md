# Fixture: pass-small-helper

Review the supplied `diff.patch` as an isolated artifact. The change should add a deterministic helper that normalizes display labels by trimming surrounding whitespace, collapsing internal whitespace runs to single spaces, and lowercasing the result.

Acceptance criteria:

- `normalize_label(value: str) -> str` returns a lowercase normalized label.
- Leading/trailing whitespace is removed.
- Internal whitespace runs are collapsed to one space.
- Empty or all-whitespace strings return an empty string.
- Tests cover the behavior above.
