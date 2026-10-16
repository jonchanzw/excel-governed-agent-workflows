# Tracker & RunControl Schemas

Typed columns only — no prose in tracker cells (`RUN-2`, `RUN-3`).

## `Tracker`

```
ItemID | Status | BatchNo | Phase | KeyFields | NextAction | ErrorCode | Attempts
```

- One row per work item.
- `Status` uses only the fixed state-machine vocabulary: `Ready`, `In Progress`, `Validated`, `Done`, `Cold`, `Failed`, `Escalated` (`RUN-4`).
- `Attempts` counts retries and drives the retry budget (`RUN-7`): `Attempts ≥ 2` → mandatory human escalation.

## `RunControl` (one row per run)

```
RunID | PlaybookVersion | WorkbookVersion | Objective | ApprovalProfile | StartedAt | LastCheckpointAt | RunStatus | CurrentPhase | CurrentBatch | TriggeredBy | OutputLocation
```

- `ApprovalProfile` ∈ {`A`, `B`, `C`} per `PROTO-3`; changeable at any checkpoint.
- A stale `LastCheckpointAt` signals an interrupted run (`RUN-3`, `RUN-6`).

## Error codes (seed set — playbooks may extend by documented ID)

| Code | Meaning | Handling |
|---|---|---|
| `SCHEMA_VIOLATION` | Mechanical validation failure | Mini-batch retry; counts as one Attempt |
| `FILE_EXPIRED` | Chat attachment access lost | Escalate for re-provisioning (`RUN-5 P9`) |
| `SOURCE_UNREACHABLE` | No authorised data path | Escalate (`RUN-5 P10`) |

Unclassified codes always escalate — the agent never invents handling for them.
