# PLAY-0 — Run-start checklist (worked example)

*This is a filled-in example of the recommended `PLAY-0` section — the run-start checklist every playbook should open with. Copy it into your workbook's `Playbook` tab and adapt the bracketed parts to your model.*

---

## PLAY-0. Run-start checklist — three-way reconciliation

**Purpose.** Establish run scope, authorised sources, and supervision level before any batch executes. No tracker rows move until every item below is answered.

**Objective (stated by user at run start).**
Reconcile the Q3 intercompany balances between the GL extract, the subledger detail, and the counterparty confirmation schedule; flag breaks above materiality for review.

**Scope.**
- Entities: [AU10, SG20] only
- Period: [Q3 FY26]
- Accounts: intercompany receivable/payable pairs [1410/2410 series]

**Authorised sources (per RUN-5 P10 — no others may be used).**

| Source | Path | Lands in |
|---|---|---|
| GL extract | `OneDrive/Finance/Shared/Q3/GL_extract_Q3.csv` via Power Query | `Staging` cols A–F |
| Subledger detail | `OneDrive/Finance/Shared/Q3/Subledger_Q3.csv` via Power Query | `Staging` cols H–M |
| Counterparty confirmations | chat attachment, staged on first access | `Staging` cols O–R |

**Output location.** `Output` tab, schema defined in PLAY-2.

**Materiality profile.** Breaks ≥ [50,000] per account pair → `KeyFields` flag `BREAK`; below → `OK`.

**Autonomy profile (PROTO-3 — user picks at run start).**
- [ ] A — Guided
- [x] B — Rolling *(default for this playbook; reconciliation batches are proven)*
- [ ] C — Hands-off

**Hands off to.** PLAY-1 (staging verification) once this checklist is complete.

**Failure handling.** Any unanswered item → stop, present the open items, wait for the user. The agent must not infer missing answers.

---

### Why PLAY-0 exists

Every later section assumes objective, scope, sources, and autonomy profile are settled. Settling them in one explicit checklist — recorded before batch 1 — is what makes the rest of the run safe to automate. Skipping PLAY-0 is the single most common cause of mid-run escalations.
