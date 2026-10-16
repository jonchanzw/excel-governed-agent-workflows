# Digest Entry Template

Appended to the `Digest` tab after **every** batch. One row per entry, fixed fields, bounded size. See `RUN-8`.

```
RunID | CheckpointID | CheckpointType=BATCH | Phase | BatchNo
CompletedCount | FailedCount | PendingCount | ExceptionCount
Exceptions: {ItemID:ErrorCode list sized to headroom; if truncated "+N more, see Tracker Status=Failed"}
DecisionRequired: {none | description} | DecisionCode
ObservedDuration | ObservedFailureMode | RecommendedNextBatchSize
NextSectionID | NextBatchNo | CreatedAt
```

## Filled example

```
RUN-2026-0412-01 | CKPT-014 | BATCH | 2 | 7
18 | 1 | 42 | 1
Exceptions: {ITM-1187:SCHEMA_VIOLATION}
DecisionRequired: none | DEC-000
4m10s | none | 20
PLAY-4 | 8 | 2026-10-16T09:41Z
```

## Rules

- Format never changes; list lengths follow adaptive sizing (`RUN-5 P8`).
- Truncation always records what was cut — never silently drop items.
- `RecommendedNextBatchSize` calibrates future sizing from observed behaviour.
