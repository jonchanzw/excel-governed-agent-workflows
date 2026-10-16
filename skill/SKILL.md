---
name: excel-single-agent-context
description: Operational patterns for running long agentic workflows in Excel with a single agent (Microsoft 365 Copilot agent mode or similar) without hitting context-window limits. Use when designing, building, debugging, or scaling Excel-based agentic models with playbooks, workflow trackers, batch runs, staged prompting, handoffs, or human-in-the-loop escalation. Covers state management, batch sizing, resume safety, error handling, and data ingestion via Power Query/Power Automate.
---

# Excel Single-Agent Context Management

Operational guide for agentic workflows in Excel driven by ONE agent, where the context window is the binding constraint.

**Core paradigm: state-as-memory, conversation-as-disposable-execution-slot.** The workbook holds all state; each agent session holds only the current batch. The sections below are written to map cleanly onto separate Excel sheets — split them into tabs when consuming this in Excel.

## Baseline architecture (assumed)

Playbook (step-by-step procedures) + workflow tracker sheet (state) + batched prompts that auto-roll into the next prompt until human intervention. The patterns below extend this baseline.

## Workbook structure (sheet map)

| Sheet | Role | Agent access |
|---|---|---|
| `Playbook` | Numbered, self-contained procedure sections | Read one section at a time, by ID — via ContextPack, not the sheet directly |
| `Tracker` | Typed workflow state (schema below) | Read staged slices; write structured results |
| `ContextPack` | Materialised per-batch input (formulas/Power Query) | Read fully |
| `Output` | Agent writes batch results here | Write only |
| `Digest` | Rolling summary between batches | Read latest; append new |
| `Handoff` | Phase-end structured handoff note | Read latest; append new |
| `Cold` | Completed items, archived | No access |
| `Staging` | External data imports | Read only |

## Tracker schema (typed columns only — no prose)

`ItemID | Status | BatchNo | Phase | KeyFields | NextAction | ErrorCode | Attempts`

- One row per work item. All writes in structured form; narrative history is forbidden in tracker cells.
- `Attempts` counts retries; drives the retry budget (see Error taxonomy).

## Status state machine (fixed vocabulary — never invent new values)

```
Ready → In Progress → Validated → Done → Cold
              ↘ Failed → In Progress        (agent retry, Attempts + 1)
                     ↘ Escalated            (human only)
Escalated ⇄ human → In Progress | Done      (human resolves, then re-enters flow)
Failed → Done                               (human accepts failure)
```

Rules:
- Transitions only between the states above. If the workflow seems to need another state, it is a `KeyFields`/`ErrorCode` distinction, not a new status.
- Only the agent moves `Ready → In Progress → Validated`. Exits from `Failed`: the agent retries (back to `In Progress`), or the human escalates / accepts the failure as `Done`. Only validation (mechanical or human) moves `Validated → Done`. Only the human moves anything to or from `Escalated`.

## Core patterns

### P1. Context pack per batch
- `ContextPack` sheet materialises ONLY: the one relevant Playbook section (quoted verbatim), tracker rows where `Status = Ready` for this batch, and the output schema for this step.
- The agent reads `ContextPack` and `Digest` — never the full `Tracker`, `Playbook` sheet, or workbook. All playbook content reaches the agent via the context pack.

### P2. Bounded rolling digest
- After each batch, compress results into a digest entry of the fixed format below. Appended to `Digest`; the next batch reads only the latest entry.
- Bounded size is mandatory: summaries that grow with events recreate the overflow problem. Field-level counts (e.g., how many exceptions to list) are set by the adaptive sizing rule — not hard-coded — but the format and one-entry-per-batch structure never change.

### P3. Fresh session per phase
- Each playbook phase ends with a structured `Handoff` note (template below). The next phase starts as a NEW session whose only context is the latest `Handoff` + fresh `ContextPack`.
- Restarting from state beats hoping a long conversation remembers.

### P4. Mechanical validation gates
- Every agent output schema has strict columns, allowed values, required fields — enforced by formulas/data validation on `Output` BEFORE the agent reviews results.
- Failures return to the agent as a mini-batch: failed rows + error codes only.

### P5. Hot vs. cold
- Current batch and open decisions live on hot sheets; completed items move to `Cold`. Restart the session with the `Handoff` note if conversation length grows.

### P6. Deterministic steps live outside the prompt
- "If column = X, set status = Y" belongs in formulas, Office Scripts, or Power Automate — never in agent prompts. Every step moved out shrinks the playbook the agent must hold.

### P7. Independent verification pass (judgement-heavy steps only)
- Format validation cannot catch wrong-but-well-formed output. For steps involving interpretation, add a cheap second pass: a separate prompt reviews N outputs against source and flags only discrepancies. Do not combine generation and verification in one prompt — self-review is biased and doubles context.

### P8. Attached files expire — stage everything
- Files attached directly to a prompt (uploads in the chat) are accessible for only a limited number of prompts/runs. Any workflow longer than a handful of exchanges WILL lose access mid-flight — treat attachment access as volatile, not storage.
- Rule: on FIRST access to any attached file, extract everything the workflow needs from it into a workbook sheet (`Staging` or a dedicated source sheet) immediately. After that first read, the agent works only from the sheet; the attachment is never needed again.
- If extraction didn't happen (or was partial) and access has expired, the agent must NOT guess at file contents or proceed on memory. It flags the affected tracker rows with `ErrorCode = FILE_EXPIRED`, records the file name and the fields it still needs in `NextAction`, and escalates.
- Re-provision protocol (human side): re-attach the file, or better, drop a copy into the workbook's OneDrive/SharePoint folder so Power Query can refresh from a stable path. The playbook should state, per source file: preferred stable location, which sheet it lands in, and which playbook sections consume it.
- Preferred default for anything reused across phases: reference files by stable OneDrive/SharePoint path from the start (Power Query / Office Scripts can reach these repeatedly); reserve chat attachments for true one-off inputs.

## Adaptive batch sizing (no fixed numbers)

Batch size is decided at run time against the actual context limits, not set as a fixed constant:

1. Estimate `tokens_per_row ≈ (chars in a typical row incl. KeyFields) ÷ 4`.
2. Estimate the fixed overhead for the batch: playbook section + output schema + latest digest + verification-pass reserve + output space.
3. `max_batch_rows ≈ (usable_context × target_utilisation − overhead) ÷ tokens_per_row`. Keep `target_utilisation` in the 20–30% band by default; raise it only for cheap mechanical steps, lower it when outputs are long or verification is in play.
4. The agent run decides the row count from this estimate and the observed behaviour (truncation, degraded output quality, or nearing limits → reduce). Where the platform exposes context usage, use it; otherwise infer from symptoms.
5. Sufficient headroom beats throughput: when the estimate is uncertain, round down. Small batches that finish are worth more than large batches that stall.
6. The same principle applies anywhere a count would otherwise be fixed (digest exception lists, handoff pending-ID lists, verification sample size N): size the count to remaining headroom, and when truncating always record what was cut (e.g., "+N more, see Tracker Status=Failed") so truncation never silently drops information.

## Resume safety (idempotency)

- Agent writes all `Output` rows for the batch FIRST, then flips tracker statuses. Never the reverse.
- Every written row carries `BatchNo`. A batch is **complete** only when every row in it has its tracker status flipped; output rows alone do not count.
- Resume protocol after any interruption: new session reads rows where `Status = In Progress`, then re-runs (a) those rows, and (b) any rows that have `Output` entries for a `BatchNo` whose tracker rows were never flipped (write finished, flip interrupted).
- A re-run must be safe to execute twice: outputs are keyed by `ItemID`, so duplicates overwrite rather than append.
- Resume never depends on chat attachments: if a re-run needs a source file whose access has expired, follow the P8 re-provision protocol before retrying those rows.

## Error taxonomy and escalation

Three failure classes — handle differently:

| Class | Signal | Handling |
|---|---|---|
| **Mechanical** | Schema/format violation caught by validation | Return as mini-batch to the agent; self-correct; counts as one `Attempt` |
| **Semantic** | Format OK but content wrong | Verification pass flags it; agent re-does flagged rows only |
| **Systemic** | Same failure across many rows in one batch | STOP. Do not retry the batch. Escalate — the prompt, source data, or schema is broken |
| **Source access** | Attached-file access expired / source unreachable (`ErrorCode = FILE_EXPIRED` or `SOURCE_UNREACHABLE`) | Agent stops work on affected rows only, records what it needs in `NextAction`, escalates for re-provisioning; unaffected rows continue |

**Human intervention is REQUIRED when:**
- `Attempts ≥ 2` for any item (retry budget exhausted)
- A systemic failure is detected
- `ErrorCode` is one the playbook doesn't classify
- An external file or source is no longer accessible (`FILE_EXPIRED` / `SOURCE_UNREACHABLE`) — re-attach or restage it, then let the flagged rows resume
- The playbook itself has an ambiguous branch (agent must not guess)

Anything else rolls forward automatically.

## Playbook authoring rules

- Playbook = numbered, self-contained sections (P1, P2, …). Each section contains: purpose, inputs (which tracker fields), procedure, output schema reference, failure handling, and the section ID(s) it hands off to.
- Sections must make sense read in isolation — no "as mentioned above". Context packs quote exactly one section.
- If a section needs another section's content, that content gets restated or referenced by ID, not assumed.

## Templates

**Digest entry (fixed format):**
```
BATCH {BatchNo} | PHASE {Phase} | rows: {n} ok / {m} failed
exceptions: {ItemID list in "ID:ErrorCode" format; list as many as headroom allows;
             if truncated, append "+N more, see Tracker Status=Failed"}
decisions_needed: {none | list}
next: {next playbook section ID and batch number}
```

**Handoff note (fixed format):**
```
HANDOFF | FROM phase {X} TO phase {Y} | session ended {timestamp optional}
done: {n} items, batches {a–b}
pending: {count + ItemIDs as headroom allows; if truncated, append
          "+N more, see Tracker Status=Ready, BatchNo>{b}"}
anomalies: {anything the next phase must know; if none, write "none"}
decisions_needed: {none | list with required input}
```

**Context pack assembly (per batch):**
```
1. Playbook section {ID} text, verbatim
2. Tracker rows: Status=Ready AND BatchNo={N} (formula-driven)
3. Output schema for this step (column names + allowed values)
4. Latest Digest entry
```

## Data ingestion (agent cannot fetch)

Copilot/agent cannot download from the web. Use the M365 automation stack as the ingestion layer; the agent consumes staged results on `Staging`:

1. **Power Query → Get Data from Web**: public APIs, CSV/JSON endpoints, HTML tables; refresh on open. Copilot can help author the M steps.
2. **Power Automate scheduled flow**: HTTP action (handles auth) → Parse JSON → "Add a row into a table" into `Staging`. The workhorse for authenticated APIs/databases.
3. **Direct DB connectors**: Power Query / Power Automate native SQL connectors (data gateway if on-prem).
4. **Middleman API**: if the source needs complex auth/pagination, normalise behind a clean endpoint first.
5. Keep `Staging` separate from `Tracker` so batching logic treats import as an upstream source.

## Copilot environment notes (verify in your tenant — limits vary by licence)

- Workbook must be stored on OneDrive or SharePoint for agent mode.
- Agent runs have action/time limits; long batches should checkpoint mid-way (write partial outputs + update Attempts) rather than assume one run completes a batch.
- Exact context window size, action caps, and supported actions differ across licences/tenants. Confirm before sizing batches, and re-run the sizing estimate whenever observed behaviour changes.

## Skill persistence

This skill file may live on an ephemeral filesystem and be missing in a new session. Keep a canonical backup copy alongside the workbooks/models that use it; if the file is missing at session start, restore from that backup before advising on workflow design.

## Anti-patterns (review checklist)

- Narrative history in tracker cells
- Status values outside the fixed state machine
- Growing summaries instead of bounded digests
- Hard-coded batch sizes or list caps instead of headroom-based sizing
- Truncating a list without recording how many items were cut
- Agent reading full workbook / full tracker
- Flipping status before outputs are written (breaks resume)
- Retrying a batch with a systemic failure
- Agent guessing at an ambiguous playbook branch instead of escalating
- Deterministic state-machine logic inside agent prompts
- Relying on a chat-attached file staying accessible across a long workflow
- Agent reconstructing an expired file's contents from memory instead of escalating
- Generation and verification in the same prompt
- Expecting the agent itself to fetch URLs or query databases

## When to escalate to multi-agent

Only when a single-agent version demonstrably fails: context overflow despite the patterns above, a genuine specialised-skill gap, or one unreliable phase poisoning the rest. Multi-agent buys context isolation and separation of concerns at the price of coordination complexity — pay it only when the single-agent version is proven inadequate.
