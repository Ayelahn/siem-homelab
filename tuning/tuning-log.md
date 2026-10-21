# Tuning log

Every change to a rule's logic, level, or exclusions gets a row. Before and after counts are average alerts per day over a 7 day window, pulled from the Wazuh dashboard with the same query both times. A change without an "after" number isn't finished.

## How counts are measured

- **Window:** 7 days before the change, 7 days after. Shorter windows get labeled in the Window column.
- **Query:** filter on `rule.id` in the dashboard, count per day, average.
- **Lab activity caveat:** attack runs from Kali inflate counts. Note any test runs in the Notes column so they can be excluded.

## Log

| Date | Rule ID | Detection | Change | Reason | Alerts/day before | Alerts/day after | Window | Notes |
|---|---|---|---|---|---|---|---|---|

<!--
Example row, delete once the first real one exists:
| 2027-01-12 | 100101 | 001 SSH brute force | Raised threshold from 5 to 8 failures in 60s | Fired on my own typos after a key rotation | 14.3 | 0.6 | 7d / 7d | No Kali runs in either window |
-->

## Baseline noise worth knowing

Built in Wazuh rules that are loud in this lab and were deliberately left alone or silenced, with the reason.

| Rule ID | Description | Alerts/day | Action | Reason |
|---|---|---|---|---|
