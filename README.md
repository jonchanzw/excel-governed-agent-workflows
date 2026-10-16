# Governed Long-Running Workflows in Excel Agent Mode

A framework for running long, multi-phase agentic workflows in Excel with a **single agent** (Microsoft 365 Copilot agent mode / "Edit with Copilot", or similar) without hitting context-window limits — with explicit governance: autonomy profiles, escalation brakes, resume safety, and machine-readable checkpoints.

## Core paradigm

**State-as-memory, conversation-as-disposable-execution-slot.** The workbook holds all state; each agent session holds only the current batch. Restarting from workbook state always beats hoping a long conversation remembers.

## The three layers

| Layer | Lives in | Contains |
|---|---|---|
| **Protocol** (`PROTO-n`) | `Protocol` tab | Objective, autonomy profile, batch loop, brakes, approval gates, stop conditions |
| **Runtime** (`RUN-n`) | `Runtime` tab | Workbook state, context packs, status state machine, validation, retries, resume safety |
| **Playbook** (`PLAY-x`) | `Playbook` tab | Domain calculations, classifications, business rules — added per model |

Rule IDs are stable: playbooks and prompts reference rules by ID, never by paraphrase. The three-layer separation, machine-readable checkpoint formats, and stable IDs are load-bearing seams — treat changes to them as structural decisions.

## Quick start

1. Read [`docs/governed-workflows-guide.md`](docs/governed-workflows-guide.md) — the merged protocol + runtime + playbook guide (single-file edition).
2. Create a workbook on OneDrive/SharePoint with the tab map from **RUN-1** (`Protocol`, `Runtime`, `Playbook`, `Tracker`, `ContextPack`, `Output`, `Digest`, `Handoff`, `Cold`, `Staging`, `RunControl`).
3. Paste each layer into its tab (one tab per layer, not per section).
4. Author your task-specific playbook sections per the rules in **Layer 3** of the guide, using the [`templates/`](templates/) for Digest entries, Handoff records, and Context Packs.
5. Start a run: the agent asks you to pick an autonomy profile (**A – Guided**, **B – Rolling**, **C – Hands-off**), then turns the batch loop itself. Escalation brakes fire identically under all profiles.

## Repository contents

```
docs/governed-workflows-guide.md   Canonical merged guide (PROTO / RUN / PLAY, version merged-1.2)
skill/SKILL.md                     Base skill: excel-single-agent-context (operational patterns)
templates/                         Machine-readable templates (digest, handoff, context pack, tracker schema)
.github/                           Issue and pull request templates
```

## Key guarantees

- **Resume safety** — outputs are written before statuses flip; any interruption is a checkpoint, not a failure (`RUN-6`).
- **Bounded context** — per-batch context packs and fixed-format digests; batch sizes are computed from live headroom, never hard-coded (`RUN-5 P8`).
- **Human control** — fixed status state machine, retry budgets, and mandatory escalation triggers that no autonomy profile can loosen (`PROTO-3`, `RUN-7`).
- **No phantom data** — expired attachments and unreachable sources escalate (`FILE_EXPIRED` / `SOURCE_UNREACHABLE`); the agent never reconstructs file contents from memory (`RUN-5 P9/P10`).

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md). In short: playbooks and prompt patterns are welcome additions; changes to rule IDs, the state machine, or checkpoint formats are structural and need an issue first.

## License

[MIT](LICENSE)
