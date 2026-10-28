# Proposed Solution — NIDHI-FDR
### Detailed Explanation, Problem Fit, Innovation & USP
**SIH26265 · AICTE, Ministry of Education**

---

## 1. The Problem, Precisely

AICTE's Regulation Bureau holds thousands of **Security Deposits / Fixed Deposit Receipts (FDRs)** in trust on behalf of technical institutions as part of the approval-related regulatory process. Today this is managed through **fragmented records, manual correspondence, and multiple disconnected tracking mechanisms**, which creates four compounding failures:

1. **No single source of truth** — institution-wise and instrument-wise status lives in scattered files, not one register.
2. **No independent verification** — what an institution *declares* about a deposit is taken largely at face value; there is no systematic cross-check against the document or the issuing bank.
3. **No proactive timeline management** — maturity, renewal, and replacement deadlines are tracked manually, so time-sensitive action is often reactive rather than scheduled.
4. **No tamper-evident accountability** — changes to deposit records, once made, are not provably traceable to who made them or whether they were altered afterward.

## 2. The Solution, In One Sentence

**NIDHI-FDR is a centralized, tamper-evident portal that gives AICTE one verified, continuously monitored, cryptographically accountable register for every security deposit and FDR it holds — from submission to discharge.**

---

## 3. How It Addresses Every Element of the Problem Statement

| PS Requirement | NIDHI-FDR Mechanism |
|---|---|
| Institution-wise & instrument-wise records | Centralized Postgres register with tenant-isolated institution views |
| Digital submission & storage of FDR documents | Encrypted Document Enclave (AES-256-GCM), SHA-256 fingerprinted |
| Review, verification, flagging of discrepancies | **Discrepancy Rule Engine** — 12 deterministic rules (D01–D12), severity-graded |
| Distinguishing institution-submitted vs. independently-confirmed data | **Dual-Source Field Model** — every financial fact carries a provenance tag |
| Automated monitoring of maturity/validity/renewal timelines | **Alert Ladder** — 90/60/30/15/7-day thresholds with auto-escalation |
| Recording and tracking release/discharge/refund requests | **Three-Actor Release Workflow** — request → recommend → approve, structurally separated |
| Centralized dashboard | Role-aware dashboards: active / expiring / matured / renewed / released / pending |
| Search, document history, scrutiny observations | Full-text search + per-instrument audit timeline |
| Role-based access, encryption, audit trails | RBAC + TOTP 2FA + signed hash-chained ledger |
| Integration-ready for future bank/AICTE systems | Explicit, swappable connector boundary (Mock Bank Connector today, real CBS API later) |

Every one of the ten "Expected Solution" capabilities listed in the official PS text is addressed directly and by name — nothing is reinterpreted or substituted.

---

## 4. What Makes It Innovative — The USPs

### USP 1 — The Trust Ladder™ (staged, provable confidence, not a binary "verified" flag)
Most tracking tools treat a deposit as either "pending" or "verified." NIDHI-FDR advances every instrument through **four distinct, auditable trust stages** — *Submitted → Machine-Verified → Source-Confirmed → Active (Locked)* — so a reviewer always knows *how* a record earned its current status, not just *that* it has one.

### USP 2 — Dual-Source Verification by Design
Every financial field (amount, dates, bank name, reference number) is stored **twice**: once as *declared* by the institution and once as *independently confirmed* (via OCR extraction or the bank connector). The UI never blends these into a single number — the system shows its work, which is what regulatory audit actually requires.

### USP 3 — Lineage Graph, Not a Status Flag
When a deposit is renewed or replaced, most systems simply update a date field, destroying history. NIDHI-FDR treats every renewal as a **new linked instrument**, preserving a full, queryable ancestry chain (original → renewal 1 → renewal 2 …) — so a 10-year regulatory relationship with an institution remains fully reconstructable at any point.

### USP 4 — Structural Separation of Duties
Release of a deposit requires **three distinct human actors** — requester, recommender, approver — and this is enforced in the data layer itself, not just a UI convention. No single compromised or colluding account can discharge a deposit alone.

### USP 5 — Cryptographically Signed Audit Mesh
Every state change produces an append-only ledger entry that is both **hash-chained** (tamper-evident) and **Ed25519-signed** (tamper-*proof*, since forging it requires a private key, not just recomputing a hash). A single click reconstructs and verifies the entire chain and pinpoints the exact entry if anything was altered.

### USP 6 — Deterministic, Explainable Discrepancy Detection
Twelve explicit rules (not an opaque model) catch missing documents, date/amount/number mismatches, premature maturity, duplicate references, under-valued replacements, and document reuse across institutions. Every flag comes with a rule code and a plain-language reason — defensible in a regulatory context where "the model said so" is not an acceptable answer.

### USP 7 — Honest Capability Boundary
NIDHI-FDR explicitly labels what is live versus simulated at every point a bank confirmation appears, instead of presenting a mocked integration as if it were real. This is a credibility feature, not a limitation — it means every claim in the system can be defended under scrutiny.

---

## 5. Why This Is Different From "Just Digitizing a Spreadsheet"

A digitized spreadsheet still has one source of truth per field, no cryptographic accountability, and no concept of lineage. NIDHI-FDR is built around the idea that a **regulatory financial register's job is not just to store data — it is to make every fact provable, every change traceable, and every action attributable.** That shift — from *storage* to *provable custody* — is the core innovation.

---

## 6. Target Users & Their Experience

| User | What changes for them |
|---|---|
| **Nodal Officer** (institution) | Submits once, digitally; sees exactly which fields are confirmed vs. still pending; gets proactive renewal reminders instead of discovering a lapse after the fact |
| **Scrutiny Official** (AICTE) | Reviews a pre-flagged queue instead of manually cross-checking every document; sees discrepancies with severity and explanation already attached |
| **Competent Authority** (AICTE) | Approves releases with full confidence the request has already been independently recommended and has no open High-severity issue |

---

*For the underlying architecture and engineering detail behind these claims, see `Technical.md`. For feasibility, risk mitigation, and expected impact, see `Feasible.md`.*
