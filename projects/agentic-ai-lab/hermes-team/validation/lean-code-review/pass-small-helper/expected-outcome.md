# Expected outcome: pass-small-helper

Expected verdict: `PASS`.

The reviewer should find that the complete diff satisfies the task requirements:

- `normalize_label` trims leading/trailing whitespace.
- It collapses internal whitespace runs with `re.sub(r"\s+", " ", ...)`.
- It lowercases the normalized value.
- Blank input becomes an empty string after stripping/collapsing.
- Tests directly cover mixed whitespace/case, a simple single-word value, and blank input.

No findings are expected. If commands are not explicitly authorized during review, the reviewer should state that tests were reviewed but not run.
