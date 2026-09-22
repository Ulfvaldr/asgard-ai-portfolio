# Fixture: fail-threshold-logic

Review the supplied `diff.patch` as an isolated artifact. The change should add a deterministic helper that decides whether an order qualifies for free shipping.

Acceptance criteria:

- `qualifies_for_free_shipping(total_cents: int, threshold_cents: int = 5000) -> bool` returns `True` when `total_cents` is at least the threshold.
- It returns `False` below the threshold.
- The default threshold is 5000 cents.
- Tests must cover below-threshold, exactly-at-threshold, and above-threshold behavior.
