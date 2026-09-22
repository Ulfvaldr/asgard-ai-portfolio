# Expected outcome: fail-threshold-logic

Expected verdict: `FAIL`.

The reviewer should identify at least these reviewable defects:

- `shipping_rules.py`: the implementation uses `total_cents > threshold_cents`, but the requirement and docstring say orders qualify when the total is at least the threshold. An order exactly equal to `5000` incorrectly returns `False`. This is a `MAJOR` logic defect.
- `test_shipping_rules.py`: tests cover above and below the default threshold but omit the required exactly-at-threshold boundary case. This missing coverage lets the logic defect pass unnoticed and should be reported with the logic finding or as a separate `MAJOR`/material test coverage issue.

The reviewer may note that commands were not run if no explicit authorization is given, but the diff evidence alone is sufficient for a failing review verdict.
