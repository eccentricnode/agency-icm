# Cases — Index

> Generated index of all active and recently-closed cases. The orchestrator reads this file (not every individual case file) to discover existing `case_id`s on a new inbound. Once the team has more than ~20 cases, scanning every file becomes prompt-context bloat; this index keeps the scan O(1) cheap.

## Format

One row per case. Most recent first. Columns:

| case_id | client | property | status | agent_on_deal | last_updated |
|---|---|---|---|---|---|
| CASE-2026-0143 | Sara M. | 1845 Westwood Dr 78704 | closed | Diana | 2026-06-25 |

## How This File Is Maintained

The transaction coordinator (`04_transaction_coordinator/`) updates this index when it produces any envelope. The orchestrator (`00_orchestrator/`) updates it when it opens a new case. Specifically:

- **New case opened** — orchestrator prepends a row when it creates `cases/<case_id>.md`.
- **State change** — coordinator updates the `status` column (`active` / `under_contract` / `option_period` / `pending_close` / `closed` / `terminated`) and the `last_updated` column.
- **No human edits.** The index is regenerated; never hand-maintained.

## Cold-Start State

Until at least one real case has been opened, this file shows only this paragraph and the empty header row above. Diana on day one will see this and `EXAMPLE_CASE-2026-0143.md` (the convention template).

## Example Index After 90 Days of Use

```
| case_id          | client       | property                   | status         | agent_on_deal | last_updated |
|------------------|--------------|----------------------------|----------------|---------------|--------------|
| CASE-2026-0167   | Janet K.     | 4011 Balcones Dr 78731     | active         | Diana         | 2026-08-12   |
| CASE-2026-0166   | Tom R.       | 2611 W 10th 78703          | active         | Maria         | 2026-08-11   |
| CASE-2026-0145   | Carlos M.    | 2304 E 6th St 78702        | under_contract | Diana         | 2026-07-30   |
| CASE-2026-0144   | Mike (past)  | 78745                       | active         | Diana         | 2026-07-15   |
| CASE-2026-0143   | Sara M.      | 1845 Westwood Dr 78704     | closed         | Diana         | 2026-06-25   |
```
