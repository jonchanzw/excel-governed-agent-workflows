# Handoff Record Template

Appended to the `Handoff` tab at **phase end**. The next phase starts as a new session whose only context is the latest Handoff + a fresh ContextPack. See `RUN-8` and `RUN-5 P3`.

```
RunID | CheckpointID | CheckpointType=HANDOFF | FromPhase | ToPhase | CreatedAt
CompletedCount | BatchRange {a–b}
PendingCount + ItemIDs sized to headroom; if truncated "+N more, see Tracker Status=Ready, BatchNo>{b}"
Anomalies: {anything the next phase must know; if none, "none"}
DecisionRequired: {none | description + required input}
```

## Filled example

```
RUN-2026-0412-01 | CKPT-022 | HANDOFF | 2 | 3 | 2026-10-16T11:05Z
96 | 1–7
14 + ITM-1201, ITM-1202, ITM-1205 (+11 more, see Tracker Status=Ready, BatchNo>7)
Anomalies: source file rates_q3.csv staged partially — 3 rows missing region code; carried as-is
DecisionRequired: confirm materiality threshold for phase 3 (needs user input: 5% or 10%)
```

## Rules

- A stale `RunControl.LastCheckpointAt` with no Handoff signals an interrupted run, not a completed phase.
- `Anomalies` is never left blank — write `none` explicitly.
