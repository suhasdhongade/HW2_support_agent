# Before vs After Comparison Tracker

Use this file to record your baseline and then update it after each improvement.

## Baseline from current report.json

| Metric | Before | After | Delta |
|---|---:|---:|---:|
| system_score | 70.31 | 72.4 | +2.0900 |
| route | 0.875 | 0.9167 | +0.0417 |
| actions | 0.6042 | 0.6042 | +0.0000 |
| facts | 0.6667 | 0.7083 | +0.0416 |
| citations | 0.7083 | 0.7083 | +0.0000 |

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
