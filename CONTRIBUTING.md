# Contributing

Thanks for helping improve this framework. A few ground rules before you start.

## What belongs where

- **Protocol (`PROTO-n`)** — orchestration rules: loop, autonomy profiles, brakes, precedence, gates.
- **Runtime (`RUN-n`)** — workbook mechanics: state, context packs, validation, retries, resume.
- **Playbook (`PLAY-x`)** — domain content: calculations, classifications, business rules.

Domain content must never inflate the runtime layer, and runtime mechanics must never leak into the protocol layer. If your contribution doesn't fit cleanly, open an issue before writing it.

## Structural vs. non-structural changes

The following are **load-bearing seams** and any change to them is structural:

- The three-layer separation
- Stable rule IDs (`PROTO-n` / `RUN-n` / `PLAY-x`)
- Machine-readable checkpoint formats (Digest, Handoff, ContextPack)
- The status state machine vocabulary

Structural changes require an issue first, with a rationale and a migration note. Non-structural changes (new playbook examples, clarifications, template additions, typo fixes) can go straight to a pull request.

## Rules for edits

- The setup flow is described in two places — README "Get set up" and `governed-excel-agent-setup.md` "How to use". Change both together, or make one link to the other.
- The setup file embeds the guide verbatim below its divider. When `docs/governed-workflows-guide.md` changes, regenerate the setup file (header + guide concatenated) in the same commit — they must never drift.
- Reference rules by ID, never by paraphrase.
- Keep playbook sections self-contained — no "as mentioned above".
- Keep templates machine-readable: fixed fields, one row per entry.
- Never introduce hard-coded batch sizes or list caps; sizing is headroom-based (`RUN-5 P8`).
- When truncating any list in a template or example, always record what was cut.
- Check your change against the anti-pattern checklist (`RUN-11`) before submitting.

## Pull requests

- One concern per PR.
- Describe which layer(s) you touched and whether the change is structural.
- If you add a template, include a filled-in example.
