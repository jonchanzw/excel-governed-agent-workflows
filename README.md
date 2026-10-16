# Governed Long-Running Workflows in Excel Agent Mode

[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](LICENSE)
[![Version](https://img.shields.io/badge/version-merged--1.2-blue.svg)](CHANGELOG.md)

**Attach one file. Copilot agent mode builds the whole governed-workflow workbook for you.**

This repo turns a blank Excel workbook into a governed, resumable, long-running agentic workflow — batch loop, autonomy profiles, escalation brakes, checkpoint logs — without you assembling a single tab by hand.

## Before you start (prerequisites)

- A **Microsoft 365 Copilot licence** with agent mode ("Edit with Copilot") enabled
- The workbook stored on **OneDrive or SharePoint** (agent mode doesn't work on local files)
- Your tenant admin **hasn't disabled** agent mode (if "Edit with Copilot" isn't visible, check with IT first)

**Privacy:** everything runs inside your own Microsoft 365 tenant — no data leaves your environment beyond normal Copilot processing.

## Get set up in one prompt

1. Create a blank workbook on **OneDrive/SharePoint** and open **Copilot agent mode** ("Edit with Copilot").
2. **Attach one file:** [`governed-excel-agent-setup.md`](governed-excel-agent-setup.md) — it contains both the setup prompt and the full governance guide.
3. Send with: **"Run the setup in the attached file."**

The agent then builds all 11 tabs, stages the protocol/runtime/playbook rules, sets up the tracker schema and status validation, and seeds your run record. (Prefer manual setup? The tab map is in **RUN-1/RUN-2/RUN-3** of the guide.)

<!-- Screenshots: add docs/screenshots/after-setup.png (the built 11-tab workbook) and docs/screenshots/mid-run-digest.png (a Digest entry mid-run) after your first real run, then uncomment:
![The built workbook](docs/screenshots/after-setup.png)
![A batch digest mid-run](docs/screenshots/mid-run-digest.png)
-->

## See a real run

New here? Start with the worked example — a complete three-way reconciliation run, from PLAY-0 checklist to Digest entries, a mid-run systemic failure, and the phase-end Handoff:

- [`examples/play-0-run-start.md`](examples/play-0-run-start.md) — a filled-in run-start checklist
- [`examples/worked-example-reconciliation.md`](examples/worked-example-reconciliation.md) — the full run, end to end

## What you get

- **State-as-memory architecture** — the workbook holds all state; each agent session holds only the current batch. Restarting from workbook state always beats hoping a long conversation remembers.
- **Resume safety** — outputs are written before statuses flip; any interruption is a checkpoint, not a failure (`RUN-6`).
- **Bounded context** — per-batch context packs and fixed-format digests; batch sizes computed from live headroom, never hard-coded (`RUN-5 P8`).
- **Human control** — fixed status state machine, retry budgets, and mandatory escalation triggers that no autonomy profile can loosen (`PROTO-3`, `RUN-7`).
- **No phantom data** — expired attachments and unreachable sources escalate (`FILE_EXPIRED` / `SOURCE_UNREACHABLE`); the agent never reconstructs file contents from memory (`RUN-5 P9/P10`).

## What this won't do (limitations)

- Agent runs have **action/time caps** — long batches checkpoint mid-way and resume; no batch is guaranteed to complete in one run.
- **Licence, metering, and context limits vary by tenant** and change over time — re-verify before sizing batches (`RUN-9`).
- **Judgement-heavy steps still need verification** — format checks can't catch wrong-but-well-formed output; use the independent verification pass (`RUN-5 P7`).
- The agent **cannot fetch external data itself** — sources must arrive via Power Query, a scheduled flow, or a staged extract (`RUN-5 P10`).

## The three layers

| Layer | Lives in | Contains |
|---|---|---|
| **Protocol** (`PROTO-n`) | `Protocol` tab | Objective, autonomy profile, batch loop, brakes, approval gates, stop conditions |
| **Runtime** (`RUN-n`) | `Runtime` tab | Workbook state, context packs, status state machine, validation, retries, resume safety |
| **Playbook** (`PLAY-x`) | `Playbook` tab | Domain calculations, classifications, business rules — added per model |

Rule IDs are stable: playbooks and prompts reference rules by ID, never by paraphrase. The three-layer separation, machine-readable checkpoint formats, and stable IDs are load-bearing seams — treat changes to them as structural decisions.

## After setup runs

1. Author your task's `PLAY-x` sections in the `Playbook` tab (authoring rules in Layer 3 of the guide; start with a `PLAY-0` run-start checklist — see [`examples/play-0-run-start.md`](examples/play-0-run-start.md)).
2. Start your first run by stating your objective — the agent asks you to pick an autonomy profile per `PROTO-3`:
   - **A – Guided:** approves each batch with you first
   - **B – Rolling** (default): loops autonomously, posts digest entries as running reports
   - **C – Hands-off:** runs to phase end, reports at the end
3. Supervise from anywhere — digest entries and decision requests are short enough for a phone (`PROTO-7`).

## Common issues

| Symptom | Fix |
|---|---|
| Agent treats the setup file as reference, not instructions | Paste the prompt block as your message text, file still attached |
| Tabs created but rule text missing | Ask for one tab at a time: "populate the Protocol tab only" |
| "Edit with Copilot" not visible | Check prerequisites above — licence, OneDrive/SharePoint storage, tenant setting |

Full troubleshooting table: [`governed-excel-agent-setup.md`](governed-excel-agent-setup.md).

## Repository contents

```
governed-excel-agent-setup.md      The headline: one attachment builds the whole workbook
docs/governed-workflows-guide.md   Canonical guide text (embedded in the setup file; kept here for reading/maintenance)
examples/                          Filled-in playbook + a complete worked run
skill/SKILL.md                     Base skill: excel-single-agent-context (operational patterns)
templates/                         Machine-readable templates (digest, handoff, context pack, tracker schema)
.github/                           Issue and pull request templates
```

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). In short: playbooks and prompt patterns are welcome additions; changes to rule IDs, the state machine, or checkpoint formats are structural and need an issue first.

## License

[MIT](LICENSE)
