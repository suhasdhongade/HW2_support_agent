# Before vs After Comparison Tracker

Use this file to record your baseline and then update it after each improvement.

## Baseline from current report.json

| Metric | Before | After | Delta |
|---|---:|---:|---:|
| system_score | 70.31 | 95.52 | +25.2100 |
| route | 0.875 | 1.0 | +0.1250 |
| actions | 0.6042 | 0.9792 | +0.3750 |
| facts | 0.6667 | 0.9167 | +0.2500 |
| citations | 0.7083 | 0.8889 | +0.1806 |

## How to update it

1. Run the evaluation:
   ```bash
   python scripts/evaluate_dev.py --json report.json
   ```
2. Copy the new values from `report.json` into the `After` column.
3. Calculate delta as:
   ```text
   after - before
   ```
4. Keep one row per milestone or change set.

## Notes

- This is the easiest way to track whether a change actually improved the agent.
- Keep the baseline values unchanged until you intentionally start a new milestone.
- Use the same command after every major code change.
