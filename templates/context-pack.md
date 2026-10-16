# Context Pack Template

Materialised on the `ContextPack` tab per batch, formula-driven. This is the **only** thing the agent reads (plus the latest Digest entry) — never the full Tracker or rule tabs. See `RUN-8` and `RUN-5 P1`.

## Assembly (per batch)

```
1. Rule/playbook section {ID} text, verbatim
2. Tracker rows: Status=Ready AND BatchNo={N}   (formula-driven slice)
3. Output schema for this step (column names + allowed values)
4. Latest Digest entry
5. RunID + PlaybookVersion   (so a stale session's output can be recognised and rejected)
```

## Layout sketch

| Block | Source | Notes |
|---|---|---|
| Section text | `Playbook` / `Runtime` tab, by ID | Quoted verbatim — one section only |
| Batch rows | `Tracker` filter: `Status=Ready AND BatchNo=N` | Typed columns only; no prose |
| Output schema | Playbook section reference | Strict columns, allowed values, required fields |
| Latest digest | `Digest`, last row | Bounded entry only |
| Run identity | `RunControl` | `RunID`, `PlaybookVersion` |

## Rules

- All rule content reaches the agent via the ContextPack — never read from rule tabs directly.
- Batch size is computed at run time from live headroom (`RUN-5 P8`), not hard-coded here.
