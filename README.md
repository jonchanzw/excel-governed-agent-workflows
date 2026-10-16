# Governed Long-Running Workflows in Excel Agent Mode

**Attach one file. Copilot agent mode builds the whole governed-workflow workbook for you.**

This repo turns a blank Excel workbook into a governed, resumable, long-running agentic workflow — batch loop, autonomy profiles, escalation brakes, checkpoint logs — without you assembling a single tab by hand.

## Get started in 3 minutes

1. Create a blank workbook on **OneDrive/SharePoint** and open **Copilot agent mode** ("Edit with Copilot").
2. **Attach one file:** [`governed-excel-agent-setup.md`](governed-excel-agent-setup.md) — it contains both the setup prompt and the full governance guide.
3. Send with: **"Run the setup in the attached file."**

The agent then builds all 11 tabs, stages the protocol/runtime/playbook rules, sets up the tracker schema and status validation, and seeds your run record — ready for your first governed run. (Prefer manual setup? The tab map is in **RUN-1/RUN-2/RUN-3** of the guide.)

## What you get

- **State-as-memory architecture** — the workbook holds all state; each agent session holds only the current batch. Restarting from workbook state always beats hoping a long conversation remembers.
- **Resume safety** — outputs are written before statuses flip; any interruption is a checkpoint, not a failure (`RUN-6`).
- **Bounded context** — per-batch context packs and fixed-format digests; batch sizes computed from live headroom, never hard-coded (`RUN-5 P8`).
- **Human control** — fixed status state machine, retry budgets, and mandatory escalation triggers that no autonomy profile can loosen (`PROTO-3`, `RUN-7`).
- **No phantom data** — expired attachments and unreachable sources escalate (`FILE_EXPIRED` / `SOURCE_UNREACHABLE`); the agent never reconstructs file contents from memory (`RUN-5 P9/P10`).

## The three layers

| Layer | Lives in | Contains |
|---|---|---|
| **Protocol** (`PROTO-n`) | `Protocol` tab | Objective, autonomy profile, batch loop, brakes, approval gates, stop conditions |
| **Runtime** (`RUN-n`) | `Runtime` tab | Workbook state, context packs, status state machine, validation, retries, resume safety |
| **Playbook** (`PLAY-x`) | `Playbook` tab | Domain calculations, classifications, business rules — added per model |

Rule IDs are stable: playbooks and prompts reference rules by ID, never by paraphrase. The three-layer separation, machine-readable checkpoint formats, and stable IDs are load-bearing seams — treat changes to them as structural decisions.

## After setup runs

1. Author your task's `PLAY-x` sections in the `Playbook` tab (authoring rules in Layer 3 of the guide; start with a `PLAY-0` run-start checklist).
2. Start your first run by stating your objective — the agent asks you to pick an autonomy profile per `PROTO-3`:
   - **A – Guided:** approves each batch with you first
   - **B – Rolling** (default): loops autonomously, posts digest entries as running reports
   - **C – Hands-off:** runs to phase end, reports at the end
3. Supervise from anywhere — digest entries and decision requests are short enough for a phone (`PROTO-7`).

## Repository contents

```
governed-excel-agent-setup.md      The headline: one attachment builds the whole workbook
docs/governed-workflows-guide.md   Canonical guide text (embedded in the setup file; kept here for reading/maintenance)
skill/SKILL.md                     Base skill: excel-single-agent-context (operational patterns)
templates/                         Machine-readable templates (digest, handoff, context pack, tracker schema)
.github/                           Issue and pull request templates
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). In short: playbooks and prompt patterns are welcome additions; changes to rule IDs, the state machine, or checkpoint formats are structural and need an issue first.

## License

[MIT](LICENSE)
