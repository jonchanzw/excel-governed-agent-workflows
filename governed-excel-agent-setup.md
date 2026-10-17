# Governed Excel Agent — One-Step Setup

> **This is the file for Excel Copilot agent mode users** — the only attachment you need.

**This one file is the entire setup.** Attach it to a single prompt in Excel Copilot agent mode and the agent builds the complete governed-workflow workbook for you.

## How to use (30 seconds)

1. Create a **blank workbook** on OneDrive/SharePoint and open **Copilot agent mode** ("Edit with Copilot").
2. **Attach this file** (`governed-excel-agent-setup.md`) to your prompt.
3. Send with a short message: **"Run the setup in the attached file."**

The agent will create all 11 tabs, stage the protocol/runtime/playbook rules, set up the tracker schema and status validation, and seed your run record. When it reports back, setup is done — author your `PLAY-x` sections (see Layer 3 below) and start your first run.

> Fallback: if the agent treats this file as reference material instead of instructions, copy the prompt block below and **paste it as your message text**, with this file still attached.

> ⚠️ Attachments expire after a limited number of prompts (see RUN-5 P9). This works in one pass because the guide is consumed once, up front, and immediately staged into workbook tabs — after that, the workbook is self-sufficient.

---

## THE SETUP PROMPT

```
You are setting up a governed agentic-workflow workbook. The attached file has
two parts: this prompt (top) and the GOVERNANCE GUIDE (below the divider marked
"GUIDE — SOURCE TEXT"). Execute these steps in order. Do not improvise beyond
them; if anything is ambiguous, stop and ask me.

1. Create these tabs, in this order, exactly named:
   Protocol, Runtime, Playbook, Tracker, ContextPack, Output, Digest,
   Handoff, Cold, Staging, RunControl

2. Populate the rule tabs from the GUIDE section of the attached file
   (do NOT copy this prompt into any tab):
   - Protocol tab  <- the full text of LAYER 1 (PROTO-1 through PROTO-7),
     one section per block, keeping rule IDs verbatim.
   - Runtime tab   <- the full text of LAYER 2 (RUN-1 through RUN-12),
     same treatment.
   - Playbook tab  <- the full text of LAYER 3, plus a placeholder line:
     "PLAY-0 to be authored for this model - see guide Layer 3 authoring rules."

3. Create the working tabs with these exact header rows (row 1):
   - Tracker:     ItemID | Status | BatchNo | Phase | KeyFields | NextAction | ErrorCode | Attempts
   - RunControl:  RunID | PlaybookVersion | WorkbookVersion | Objective | ApprovalProfile |
                  StartedAt | LastCheckpointAt | RunStatus | CurrentPhase | CurrentBatch |
                  TriggeredBy | OutputLocation
   - Digest:      (one header cell) "Digest entries - append one row per batch, format per RUN-8"
   - Handoff:     (one header cell) "Handoff records - append one row per phase end, format per RUN-8"
   - Output:      leave empty except a note cell: "Output schema is defined per playbook step"
   - ContextPack: leave empty except a note cell: "Materialised per batch - see RUN-8 assembly spec"
   - Cold:        same header row as Tracker
   - Staging:     leave empty except a note cell: "External data imports land here - read only"

4. Apply data validation on Tracker!Status allowing only:
   Ready, In Progress, Validated, Done, Cold, Failed, Escalated

5. Fill RunControl row 2 with: a generated RunID in format RUN-YYYYMMDD-01,
   WorkbookVersion = 1.0, StartedAt = now, RunStatus = Setup,
   ApprovalProfile = (leave blank - ask me at run start per PROTO-3).

6. Report back: tab list created, any steps you could not complete, and the
   RunID you generated. Then stop - do not begin any workflow work.
```

### Troubleshooting

| Symptom | Fix |
|---|---|
| Agent treats the file as reference, not instructions | Paste the prompt block as your message text, keep the file attached |
| Tabs created but rule text missing | Ask: "Re-read the attached file and populate the Protocol tab only" - one tab per prompt |
| Validation list rejected | Some tenants limit validation lists; write the allowed values into a note cell instead and rely on RUN-4 discipline |

---

# GUIDE — SOURCE TEXT
*(Everything below this divider is copied into the workbook tabs by the setup prompt. It is also the canonical reference for the framework.)*

# Governed Long-Running Workflows in Excel Agent Mode
### Combined protocol + runtime + playbook guide (single-file edition)

**Generated from:** a generic governed-workflow protocol + the excel-single-agent-context skill — merged for direct consumption by Microsoft 365 Copilot agent mode in Excel ("Edit with Copilot").

**How to load this document:**
1. Paste each layer into its own workbook tab: `Protocol`, `Runtime`, `Playbook`. (Section headers double as tab names; one tab per layer is enough — do not create a tab per section.)
2. Alternatively, attach this file to the first prompt of a run. NOTE: chat attachments expire after a limited number of prompts — always stage a copy in the workbook tabs as the durable source.
3. Batches never read this document wholesale. The `ContextPack` tab quotes exactly the sections a batch needs (see RUN-1).

**ID scheme:** `PROTO-n` = orchestration rules · `RUN-n` = workbook runtime rules · `PLAY-x` = task-specific playbook sections (added per model). Rule IDs are stable — playbooks and prompts reference rules by ID, never by paraphrase.

**Flagged architectural consideration (not a plan):** the three-layer separation, machine-readable checkpoint formats, and stable rule IDs are load-bearing beyond their current purpose — if this framework is ever re-platformed (e.g. rules moved out of workbook tabs into code), these are the seams a future build would cut along. Do not erode them when editing; treat any change to them as a structural decision, not a tidy-up.

---

# LAYER 1 — PROTOCOL (`Protocol` tab)

## PROTO-1. Core paradigm

**State-as-memory, conversation-as-disposable-execution-slot.** The workbook holds all state; each agent session holds only the current batch. Restarting from workbook state always beats hoping a long conversation remembers.

## PROTO-2. The batch loop

The agent turns the crank itself — no external scheduler required:

```
LOOP until phase complete or stopped:
  1. READ    ContextPack (bounded slice for this batch) + latest Digest entry
  2. EXECUTE the batch per the quoted PLAY section
  3. WRITE   all Output rows FIRST, then flip Tracker statuses
  4. CHECKPOINT: append a Digest entry
  5. DECIDE  per PROTO-4: continue / stop at gate / escalate
```

Interruption at any point is a checkpoint, not a failure — the resume protocol (RUN-6) recovers from workbook state alone.

## PROTO-3. Autonomy profiles (user's choice at run start)

At the start of every governed run, ask the user to pick one:

| Profile | Behaviour | Best for |
|---|---|---|
| **A — Guided** | Propose each batch and wait for approval before executing | First runs of a new playbook; high-stakes data; calibrating trust |
| **B — Rolling** (default) | Loop batches autonomously; post each Digest entry as a running report; stop at brakes | Proven playbooks with active supervision available |
| **C — Hands-off** | Run to phase end without per-batch chatter; present Handoff note + exception summary | Well-validated workflows, long runs |

Rules:
- **The profile modulates reporting and prompting — never the brakes.** Escalation triggers (RUN-7) fire identically under all profiles. Choosing hands-off means less chatter, not less protection.
- The chosen profile is recorded in the run record (`ApprovalProfile` field, RUN-3) and can be changed at any checkpoint ("switch to Guided for the next phase").
- If a run escalates under Profile C, it resumes under Profile B until the user re-authorises C.

## PROTO-4. Continue / stop / escalate decision (after every batch)

```
Digest.decisions_needed = none  AND  no escalation triggers  → continue to next batch
Digest.decisions_needed ≠ none  OR  any RUN-7 trigger fires  → STOP; present the decision/exception to the user
```

The agent must never answer its own escalation questions, and never guess at an ambiguous playbook branch — that is a stop condition, not a judgement call.

## PROTO-5. Precedence order (when instructions conflict)

```
safety & approval rules
  > task-specific playbook (PLAY-x)
  > this protocol (PROTO-n)
  > workbook runtime profile (RUN-n)
  > operational defaults
```

Record any unresolved conflict as an exception and stop at the affected decision point.

## PROTO-6. Layer responsibilities

- **Protocol layer (this tab):** objective, scope, autonomy profile, loop, brakes, approval gates, reporting cadence, stop conditions, final report.
- **Runtime layer (`Runtime` tab):** workbook state, context-pack construction, batch execution mechanics, status transitions, validation recording, retry handling, restart safety.
- **Playbook layer (`Playbook` tab):** domain calculations, classifications, source hierarchy, business decisions. Domain profiles (e.g. validation or materiality rules for a specific reporting regime) live here, referenced by ID — never inflate the runtime layer with domain content.

## PROTO-7. Operating context: chat, projects, and mobile control plane

The same state-as-memory paradigm applies to how the human supervises runs — sessions are disposable, files are durable:

- **Chat orchestration:** when supervision happens through an AI chat tool (e.g. a Kimi Project), one chat cannot manage other chats. Use the project as the shared workspace instead: canonical files live as project files, conventions live in project instructions, and a short `STATUS` file acts as the cross-chat digest — every chat reads it first and updates it last. Start a separate chat per distinct output (a run, a playbook draft, a review) so no single conversation bloats.
- **Phone as control plane:** the supervisor's interactions are short — approve a batch, answer a decision request, read a Digest entry — so they work from a mobile chat client. Execution stays in Excel on the desktop; the phone is for state visibility and gate decisions only. Keep the `STATUS` file short, append-at-bottom, so it reads well on a small screen.
- **The human is the crank across tools:** within Excel, the PROTO-2 loop turns itself per the autonomy profile; across chat sessions, the human routes. This is Profile A/B supervision by design, not a limitation.

---

# LAYER 2 — RUNTIME (`Runtime` tab)

## RUN-1. Workbook structure (tab map)

| Tab | Role | Agent access |
|---|---|---|
| `Protocol` / `Runtime` / `Playbook` | This document, staged | Read one section at a time, by ID — via ContextPack, not the tab directly |
| `Tracker` | Typed workflow state (schema in RUN-2) | Read staged slices; write structured results |
| `ContextPack` | Materialised per-batch input (formula-driven) | Read fully |
| `Output` | Agent writes batch results here | Write only |
| `Digest` | Machine-readable checkpoint log | Read latest; append new |
| `Handoff` | Phase-end structured handoff record | Read latest; append new |
| `Cold` | Completed items, archived | No access |
| `Staging` | External data imports + staged attachment extracts | Read only |
| `RunControl` | One-row run record (RUN-3) | Read; update designated fields |

The agent reads `ContextPack` and the latest `Digest` entry — never the full `Tracker`, rule tabs, or workbook.

## RUN-2. Tracker schema (typed columns only — no prose)

`ItemID | Status | BatchNo | Phase | KeyFields | NextAction | ErrorCode | Attempts`

- One row per work item. All writes in structured form; narrative history is forbidden in tracker cells.
- `Attempts` counts retries; drives the retry budget (RUN-7).

## RUN-3. Run record (`RunControl` tab, one row per run)

`RunID | PlaybookVersion | WorkbookVersion | Objective | ApprovalProfile | StartedAt | LastCheckpointAt | RunStatus | CurrentPhase | CurrentBatch | TriggeredBy | OutputLocation`

- `ApprovalProfile` holds the PROTO-3 choice (A/B/C). `LastCheckpointAt` updates with every Digest entry — a stale value signals an interrupted run.

## RUN-4. Status state machine (fixed vocabulary — never invent new values)

```
Ready → In Progress → Validated → Done → Cold
              ↘ Failed → In Progress        (agent retry, Attempts + 1)
                     ↘ Escalated            (human only)
Escalated ⇄ human → In Progress | Done      (human resolves, then re-enters flow)
Failed → Done                               (human accepts failure)
```

Rules:
- Transitions only between the states above. If the workflow seems to need another state, it is a `KeyFields`/`ErrorCode` distinction, not a new status. If a specific build genuinely needs more (e.g. `Superseded`), that is a documented extension in that build's playbook — the base vocabulary does not change.
- Only the agent moves `Ready → In Progress → Validated`. Exits from `Failed`: agent retry, human escalate, or human accept-as-Done. Only validation (mechanical or human) moves `Validated → Done`. Only the human moves anything to or from `Escalated`.

## RUN-5. Core execution patterns

**P1. Context pack per batch.** `ContextPack` materialises ONLY: the one relevant PLAY/RUN section (quoted verbatim), tracker rows where `Status = Ready` for this batch, the output schema for this step, and the latest Digest entry. All rule content reaches the agent via the context pack — never read from the rule tabs directly.

**P2. Bounded rolling digest.** After each batch, append one machine-readable Digest entry (template in RUN-8). Bounded size is mandatory: summaries that grow with events recreate the overflow problem. The format never changes; field-level list lengths follow the adaptive sizing rule (RUN-5 P8).

**P3. Fresh session per phase.** Each phase ends with a Handoff record. The next phase starts as a NEW session whose only context is the latest Handoff + fresh ContextPack.

**P4. Mechanical validation gates.** Every output schema has strict columns, allowed values, required fields — enforced by formulas/data validation on `Output` BEFORE the agent reviews results. Failures return as a mini-batch: failed rows + error codes only. The playbook defines any additional validation layers per output schema (referential, calculation, reconciliation, business-rule, reasonableness, presentation — as the domain requires); the runtime enforces that they run.

**P5. Hot vs. cold.** Current batch and open decisions live on hot tabs; completed items move to `Cold`.

**P6. Deterministic steps live outside the prompt.** "If column = X, set status = Y" belongs in formulas or Office Scripts — never in agent prompts.

**P7. Independent verification pass (judgement-heavy steps only).** For steps involving interpretation, a separate prompt reviews N outputs against source and flags only discrepancies. Never combine generation and verification in one prompt — self-review is biased and doubles context.

**P8. Adaptive batch sizing (no fixed numbers).** Batch size is decided at run time against actual limits, which are driven by more than text: workbook actions, write volume, formula/recalc load, response length, and run-time caps all bind.
1. Estimate `tokens_per_row ≈ (chars in a typical row incl. KeyFields) ÷ 4`, plus fixed overhead (rule section + schema + digest + verification reserve + output space).
2. `max_batch_rows ≈ (usable_capacity × target_utilisation − overhead) ÷ tokens_per_row`, target utilisation 20–30% by default — lower when outputs are long or verification is in play.
3. The run adjusts from observed behaviour: truncation, degraded output, slowing recalc, or nearing action/time caps → reduce.
4. The Digest entry records `ObservedDuration`, `ObservedFailureMode`, and `RecommendedNextBatchSize` so sizing calibrates from real runs instead of estimates alone.
5. When uncertain, round down. Small batches that finish beat large batches that stall.
6. The same principle governs any list length (digest exceptions, handoff pending IDs, verification sample N): size to headroom, and when truncating always record what was cut ("+N more, see Tracker Status=Failed") — truncation must never silently drop information.

**P9. Attached files expire — stage everything.** Files attached to a prompt are accessible for only a limited number of prompts. On FIRST access to any attached file, extract everything the workflow needs into `Staging`; afterwards the agent works only from the tab. If access expired before extraction, the agent flags affected rows `ErrorCode = FILE_EXPIRED`, records what it needs in `NextAction`, and escalates — it must never reconstruct file contents from memory. Preferred default for anything reused across phases: a stable OneDrive/SharePoint path that Power Query can refresh; chat attachments are for one-off inputs only.

**P10. External data access.** The agent must not assume direct external-source access. External data must arrive through an explicitly authorised and tested path — Power Query, a scheduled flow, a database connector, or a staged extract. If no such path exists, treat the source as unreachable (`ErrorCode = SOURCE_UNREACHABLE`) and escalate. Keep `Staging` separate from `Tracker` so batching treats import as an upstream source.

**P11. Workbook protection.** Sheet-level access rules (RUN-1) are not sufficient — within allowed tabs the agent must also respect: allowed write ranges only; protected formula columns and data-validation cells are never overwritten; no insert/delete row operations or table resizing unless the playbook explicitly permits; no editing queries, connections, or reporting layouts; pre-write and post-write row counts are recorded per batch; structural changes (any of the above exceptions) require a workbook version checkpoint first.

## RUN-6. Resume safety (idempotency)

- Write all `Output` rows for the batch FIRST, then flip tracker statuses. Never the reverse.
- Every written row carries `BatchNo`. A batch is **complete** only when every row has its tracker status flipped; output rows alone do not count.
- Resume after any interruption: read rows where `Status = In Progress`, then re-run (a) those rows, and (b) any rows with `Output` entries for a `BatchNo` whose tracker rows were never flipped.
- A re-run must be safe to execute twice: outputs keyed by `ItemID` — duplicates overwrite, never append.
- Resume never depends on chat attachments: expired sources follow the P9 re-provision protocol before those rows retry.
- Only one session operates the workbook at a time; a second session finding an active run (fresh `LastCheckpointAt`) must not process rows.

## RUN-7. Error taxonomy and escalation

| Class | Signal | Handling |
|---|---|---|
| **Mechanical** | Schema/format violation caught by validation | Return as mini-batch; self-correct; counts as one `Attempt` |
| **Semantic** | Format OK but content wrong | Verification pass flags it; agent re-does flagged rows only |
| **Systemic** | Same failure across many rows in one batch | STOP. Do not retry. Escalate — prompt, source data, or schema is broken |
| **Source access** | Attached-file access expired / source unreachable (`FILE_EXPIRED` / `SOURCE_UNREACHABLE`) | Stop affected rows only; record need in `NextAction`; escalate; unaffected rows continue |

**Human intervention is REQUIRED when:**
- `Attempts ≥ 2` for any item (retry budget exhausted)
- A systemic failure is detected
- `ErrorCode` is one the playbook doesn't classify
- The playbook has an ambiguous branch (agent must not guess)
- An external file or source is no longer accessible — re-attach or restage, then flagged rows resume

Anything else rolls forward automatically, per the autonomy profile.

## RUN-8. Templates (machine-readable — one row per entry, fixed fields)

**Digest entry (per batch, appended to `Digest`):**
```
RunID | CheckpointID | CheckpointType=BATCH | Phase | BatchNo
CompletedCount | FailedCount | PendingCount | ExceptionCount
Exceptions: {ItemID:ErrorCode list sized to headroom; if truncated "+N more, see Tracker Status=Failed"}
DecisionRequired: {none | description} | DecisionCode
ObservedDuration | ObservedFailureMode | RecommendedNextBatchSize
NextSectionID | NextBatchNo | CreatedAt
```

**Handoff record (per phase end, appended to `Handoff`):**
```
RunID | CheckpointID | CheckpointType=HANDOFF | FromPhase | ToPhase | CreatedAt
CompletedCount | BatchRange {a–b}
PendingCount + ItemIDs sized to headroom; if truncated "+N more, see Tracker Status=Ready, BatchNo>{b}"
Anomalies: {anything the next phase must know; if none, "none"}
DecisionRequired: {none | description + required input}
```

**Context pack assembly (per batch):**
```
1. Rule/playbook section {ID} text, verbatim
2. Tracker rows: Status=Ready AND BatchNo={N} (formula-driven)
3. Output schema for this step (column names + allowed values)
4. Latest Digest entry
5. RunID + PlaybookVersion (so a stale session's output can be recognised and rejected)
```

## RUN-9. Environment notes (verify in your tenant — limits vary by licence)

- Workbook must be stored on OneDrive or SharePoint for agent mode (branded "Agent Mode" / "Edit with Copilot" depending on surface).
- Runs have action/time limits; long batches checkpoint mid-way (partial outputs + Attempts) rather than assume one run completes a batch.
- Context window size, action caps, attachment longevity, and supported actions differ across licences/tenants and change over time — confirm before sizing batches, and recalibrate when observed behaviour changes.

## RUN-10. Skill persistence

This document may live on an ephemeral filesystem or an expiring chat attachment. Keep the canonical copy in the workbook tabs themselves (they are the durable store) and a backup alongside the models that use it; restore from backup before advising on workflow design if any copy is missing.

## RUN-11. Anti-patterns (review checklist)

- Narrative history in tracker cells
- Status values outside the fixed state machine (undocumented extensions included)
- Growing summaries instead of bounded digests
- Hard-coded batch sizes or list caps instead of headroom-based sizing
- Truncating a list without recording how many items were cut
- Agent reading full workbook / full tracker / rule tabs directly
- Flipping status before outputs are written (breaks resume)
- Retrying a batch with a systemic failure
- Agent guessing at an ambiguous playbook branch instead of escalating
- Agent answering its own escalation questions
- Deterministic state-machine logic inside agent prompts
- Relying on a chat-attached file staying accessible across a long workflow
- Agent reconstructing an expired file's contents from memory instead of escalating
- Two sessions operating the same workbook concurrently
- Generation and verification in the same prompt
- Assuming direct external-source access without an authorised, tested path
- Loosening escalation brakes because the autonomy profile is hands-off

## RUN-12. When to escalate to multi-agent

Only when the single-agent version demonstrably fails: context overflow despite the patterns above, a genuine specialised-skill gap, or one unreliable phase poisoning the rest. Multi-agent buys context isolation at the price of coordination complexity — pay it only when single-agent is proven inadequate.

---

# LAYER 3 — PLAYBOOK (`Playbook` tab)

Task-specific sections, added per model. Authoring rules:

- Playbook = numbered, self-contained sections (`PLAY-1`, `PLAY-2`, …). Each contains: purpose, inputs (which tracker fields), procedure, output schema reference, validation layers required (per RUN-5 P4), failure handling, and the section ID(s) it hands off to.
- Sections must make sense read in isolation — no "as mentioned above". Context packs quote exactly one section.
- If a section needs another section's content, restate it or reference by ID — never assume.
- Domain profiles (validation rules, materiality thresholds, classification policies, authorised-source lists) are defined here and referenced by ID from the procedure — they are playbook content, not runtime content.
- `PLAY-0` (recommended): run-start checklist — objective, scope, authorised sources, output location, and the autonomy-profile question (PROTO-3).

---

*End of document. Version: merged-1.2*
