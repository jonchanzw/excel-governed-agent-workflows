# Worked example — a reconciliation run end to end

*What a real run looks like once the workbook is set up. Playbook: three-way intercompany reconciliation (PLAY-0 → PLAY-3). Autonomy profile: B — Rolling. Numbers are illustrative.*

---

## 1. Run start — PLAY-0 completed

User attaches the counterparty confirmation file (staged immediately per RUN-5 P9), confirms Power Query sources refreshed, picks **Profile B**. RunControl row 2:

```
RUN-20261016-01 | PB-1.0 | 1.3 | Q3 intercompany rec | all 214 pairs reconciled, breaks ≥ 50k routed, zero unclassified errors | B | 2026-10-16T09:02Z | 2026-10-16T09:02Z | Active | 2 | 1 | User | Output!A:H
```

## 2. Tracker before batch 1

214 account pairs loaded, split into batches sized per RUN-5 P8 (25 rows/batch — tokens_per_row ≈ 90, overhead ≈ 3,200 tokens, 25% utilisation target):

```
ItemID | Status | BatchNo | Phase | KeyFields | NextAction | ErrorCode | Attempts
ICP-0001 | Ready | 1 | 2 | AU10:1410/SG20:2410 amt=182,400 | — | — | 0
ICP-0002 | Ready | 1 | 2 | AU10:1410/SG20:2410 amt=64,150 | — | — | 0
...    | ...   | ...    | ...  | ... | — | — | 0
ICP-0214 | Ready | 9 | 2 | SG20:2410/AU10:1410 amt=12,980 | — | — | 0
```

## 3. Batch 1 executes (PLAY-2: match and classify)

ContextPack materialises: PLAY-2 text verbatim, the 25 Ready rows, the output schema, the latest Digest entry. The agent matches GL ↔ subledger ↔ confirmation amounts and writes Output rows **first**, then flips tracker statuses (RUN-6). Two rows fail mechanical validation (missing currency code) and return as a mini-batch; both self-correct on one attempt.

## 4. Digest entry after batch 1

```
STATUS: Batch 1 of 9 | 8% complete (17 of 214) | 2 breaks flagged | decisions: none
RUN-20261016-01 | CKPT-003 | BATCH | 2 | 1
25 | 0 | 189 | 2
Exceptions: {ICP-0011:BREAK, ICP-0019:BREAK}
DecisionRequired: none | DEC-000
3m40s | none | 25
PLAY-2 | 2 | 2026-10-16T09:11Z
```

Per Profile B, this posts as a running report; the run rolls on without waiting.

## 5. Mid-run: a systemic failure (batch 4)

Every row in batch 4 comes back with `ErrorCode = SOURCE_UNREACHABLE` — the subledger Power Query stopped refreshing (source file moved). Per RUN-7 this is systemic: the agent **stops**, does not retry, escalates:

```
DecisionRequired: Subledger source unreachable since batch 4 — file moved from
/Shared/Q3/. Re-point Power Query or restore the file. Batches 4–9 paused;
batches 1–3 (75 rows) unaffected. | DEC-104
```

The user fixes the path from their phone, replies "fixed", and the run resumes at batch 4. Escalation brakes fired identically to how they would under Profile C — the profile only changed the chatter, not the brakes (PROTO-3).

## 6. Phase end — Handoff record

All 214 pairs reconciled; phase 2 closes:

```
RUN-20261016-01 | CKPT-021 | HANDOFF | 2 | 3 | 2026-10-16T11:47Z
214 | 1–9
0
Anomalies: 11 BREAKs ≥ 50,000 routed to review queue; subledger source path changed mid-run (re-pointed, verified)
DecisionRequired: none — proceed to PLAY-3 (disclosure checklist) in a new session
```

## 7. Next phase starts fresh

A **new chat session** opens with only this Handoff record plus a fresh ContextPack (RUN-5 P3). It carries no memory of batches 1–9 — and needs none, because the workbook holds the state.

---

### What this example demonstrates

| Framework rule | Where it showed up |
|---|---|
| RUN-5 P8 adaptive sizing | 25-row batches computed from token estimates |
| RUN-6 resume safety | Outputs written before status flips; mid-run source failure recovered cleanly |
| RUN-7 systemic escalation | Batch 4 stopped, not retried; user re-provisioned the source |
| PROTO-3 profiles | Rolling reporting; brakes unchanged |
| RUN-5 P3 fresh session | Phase 3 starts from the Handoff alone |
