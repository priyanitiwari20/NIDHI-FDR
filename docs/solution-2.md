# NIDHI-FDR
### National Integrated Deposit Holdings Infrastructure for Security Deposit / FDR Management
**Smart India Hackathon 2026 · Problem Statement SIH26265 · AICTE, Ministry of Education**

> *"Nidhi" (निधि) — fund, treasury, safeguarded resource. One system of record for every deposit AICTE holds in trust.*

---

## 0. Why This Document Exists

This is a revised proposal (v2) for our team's second Problem Statement. It was built after reviewing the full text of SIH26265 **and** the published approaches of the five other teams working the same problem. Section 1 summarizes what each of them does well and where each is exposed. Section 2 onward is the resulting solution — designed specifically to be **stronger on feasibility, speed of delivery, security depth, and credibility with judges** than any single one of them.

---

## 1. Competitive Landscape — What the Other Five Teams Are Building

| Team | Core Strength | Where It's Exposed |
|---|---|---|
| **FDR SmartTrack** | Real OCR + regex field extraction, Isolation Forest anomaly scoring, Argon2id + TOTP 2FA, clean dedicated Postgres schema | Only 4 roles and no dual-source (institution vs. bank) display; no renewal *lineage* (old→new instrument chain) — renewal is a status change, not a tracked history; no explicit discrepancy severity ladder |
| **Team 2 (AICTE SD/FDR Portal)** | Best discrepancy engine of the five (12 explicit rules, R01–R12), real renewal lineage (v1→v2→v3 with "Superseded" badges), hash-chain audit log with an integrity-check button, demo-clock for judge testing | SQLite, no encryption-at-rest mentioned, no OCR/extraction (listed as future work only), no MFA, 3 roles only |
| **Team 3 (FDR-Sentinel)** | Best **honesty discipline** — an explicit "Real vs. Simulated" table; real AES-256-GCM encryption, real Ed25519-signed QR verification, ELA tamper-heatmap, a trained ML risk model with saved metrics | Stack is heavy for a hackathon prototype (Celery, Redis, MinIO, Docker Compose with 7 services) — real delivery/judging risk; implies PDF "digital signatures" from banks, which Indian banks do not currently issue on FDR receipts, so this claim is fragile under judge questioning |
| **Team 4 (AICTE FDR Portal)** | Most enterprise-grade: 13 roles with ABAC + Postgres Row-Level Security, Separation-of-Duties on release approval, SSE realtime inbox, full design system | Over-engineered for the available build window (Keycloak IAM, ClamAV, 632 tests, mobile companion app) — high risk of an unfinished or unstable demo; no OCR/AI verification layer at all |
| **Team 5a (FinMatrix)** | Strong UX polish, dual-source badges (Submitted / Bank-confirmed / Mismatch), renewal chaining, a clean 4-stage release stepper, 6 statutory reports | Integrity layer is HMAC-SHA256 only (symmetric — anyone with the secret can forge the chain); SQLite; no MFA; claims like "Turnaround reduced from 7 days to <48 hours" are asserted, not demonstrated |
| **Team 5b (SURAKSHA-FDR)** | Most ambitious vision: multi-lien/double-pledge radar against NeSL/CERSAI, NBA/NAAC accreditation cross-check, GPS-geofenced physical inspection | **Largest credibility risk of all six teams.** It presents live RBI SFMS messaging, live NeSL/CERSAI queries, and live e-NBA/NAAC API links as working features. None of these external systems grant access to a student hackathon team. Fabricated-sounding impact figures (₹41.5 Cr saved/year, ₹100 Cr swept) without a stated basis will read as overclaiming to an evaluation panel and can cost credibility on the whole submission |

### What this tells us
No team combines (a) **real, verifiable cryptography**, (b) **a disciplined real-vs-simulated boundary stated up front**, (c) **renewal lineage + dual-source trust display**, and (d) **a build scope that can actually be finished and demoed reliably** in a hackathon window. That combination is the gap NIDHI-FDR is designed to fill.

---

## 2. Design Principles

1. **Feasibility over spectacle.** Every claimed capability is either real in the prototype or explicitly labeled simulated, with the exact reason it can't be real yet (matching Team 3's best instinct, applied more consistently and without heavy infra).
2. **Dual-source truth, always.** Every financial field shows what the institution *declared* separately from what was *independently confirmed*, so a reviewer never has to trust a single source blindly.
3. **Lineage, not just status.** A renewed or replaced FDR keeps its full ancestry (old instrument → new instrument), never silently overwritten.
4. **Separation of duties.** The person who submits a release request can never be the person who approves it — enforced in the data layer, not just the UI.
5. **Lean, defensible stack.** One backend, one primary database, no dependency the team cannot explain line-by-line to a judge.
6. **Terminology matches the PS exactly** — Security Deposit, FDR, Regulation Bureau, Competent Authority, CBS, lien, discharge/release, renewal, maturity, nodal officer — so scoring against the official rubric is unambiguous.

---

## 3. Terminology (used consistently throughout)

| Term | Meaning in this system |
|---|---|
| **Security Deposit / FDR** | The Fixed Deposit Receipt (or equivalent instrument) an institution places as a statutory financial guarantee as part of AICTE's approval process |
| **Custodian / Issuing Bank** | The scheduled bank that issued the FDR |
| **Nodal Officer** | The institution-side user authorized to submit and manage that institution's deposits |
| **Regulation Bureau Official** | AICTE-side user who scrutinizes submissions |
| **Competent Authority** | AICTE-side role with final authority to approve release/discharge — never the same person as the submitting nodal officer or the recommending official |
| **CBS (Core Banking System)** | The bank's internal system; in this prototype, CBS confirmation is produced by a clearly labeled **Mock Bank Connector**, not a live bank API |
| **Discharge / Release** | The authorized return or release of a matured/no-longer-required deposit to the institution |
| **Lineage** | The traceable chain linking an original FDR to any instrument that later renewed or replaced it |

---

## 4. System Overview

NIDHI-FDR is a centralized portal with three user-facing surfaces (Institution, Scrutiny/Reviewing Official, Competent Authority) sitting on one authoritative register of deposits. Its differentiating layer is the **Trust & Lineage Engine**, which does three things none of the other five teams do together:

- Keeps a **dual-source record** per field (declared vs. independently confirmed)
- Maintains a **versioned lineage graph** across renewals/replacements
- Produces a **cryptographically verifiable audit trail** using asymmetric signatures (not just a hash chain), so the chain cannot be regenerated by anyone holding only database access

### 4.1 Recommended Stack (deliberately lean)

| Layer | Choice | Why |
|---|---|---|
| Frontend | React + TypeScript + Vite + Tailwind | Fast to build, easy to demo responsively |
| Backend | FastAPI (Python) | Async, auto-generated OpenAPI docs for judges to inspect live |
| Database | PostgreSQL | Relational integrity for lineage graphs + row-level tenant isolation |
| File storage | Local encrypted volume (AES-256-GCM), S3-compatible interface for a production path | No MinIO/Redis/Celery dependency required to demo |
| Background jobs | APScheduler in-process | One fewer moving part than Celery/Redis for a prototype |
| Auth | Argon2id password hashing + TOTP 2FA for Regulation Bureau and Competent Authority roles | Matches best-in-class (Team 1) without Keycloak's operational overhead |
| Audit integrity | **Hash-chained ledger, each entry additionally Ed25519-signed by the acting role's key** | Stronger than a plain HMAC/SHA-256 chain (Teams 2, 5a) because signatures are independently verifiable with a public key — tampering requires forging a private key, not just recomputing a hash |

A production note is included for every "simulated" component explaining exactly what authorized integration (AICTE SSO, bank CBS API, NeSL) would replace it with — so the roadmap is credible rather than hand-waved.

---

## 5. Trust Ladder (Instrument Lifecycle State Machine)

```
SUBMITTED
   │  (required fields + document present, chronological validation passed)
   ▼
MACHINE-VERIFIED
   │  (OCR-extracted fields reconciled against declared fields; 0 open High discrepancies)
   ▼
SOURCE-CONFIRMED
   │  (Mock Bank Connector returns a match — clearly badged "Simulated Confirmation")
   ▼
ACTIVE (locked, monitored for maturity)
   │
   ├──► RENEWAL REQUESTED ──► RENEWED (new FDR; old FDR marked "Superseded", lineage preserved)
   │
   └──► RELEASE REQUESTED ──► RECOMMENDED (Scrutiny Official) ──► APPROVED (Competent Authority) ──► DISCHARGED
```

Every transition is an immutable, signed audit entry. A High-severity open discrepancy blocks progression past *Machine-Verified* and blocks release, mirroring Team 2's strongest control but made mandatory system-wide rather than only on release.

---

## 6. Core Capabilities (mapped to the PS's 10 expected capabilities)

1. **Centralized Repository** — institution-wise and instrument-wise records, one register, Postgres-backed.
2. **Digital Document Management** — encrypted upload/storage (AES-256-GCM at rest), SHA-256 fingerprint on every file to detect byte-level duplication or reuse across records.
3. **Verification & Discrepancy Detection** — OCR extraction with confidence scores + a 12-rule discrepancy engine (below), clearly separating *declared* data from *extracted/confirmed* data.
4. **Automated Validity & Maturity Tracking** — 90 / 60 / 30 / 15 / 7-day alert ladder, deduplicated, with a demo time-jump control for judges (Team 2's best demo idea, retained because it genuinely solves the "how do we show a 10-year lifecycle in 5 minutes" problem).
5. **Renewal & Replacement Monitoring** — full lineage graph (parent → child instrument), not a status flag.
6. **Release & Refund Workflow** — 3-stage separation of duties: Nodal Officer requests → Scrutiny Official recommends → Competent Authority approves. Blocked while any High discrepancy is open.
7. **Automated Alerts & Escalation** — unacknowledged alerts past threshold escalate automatically to a senior role.
8. **Dashboard & Management Reports** — active / expiring / matured / renewed / released / pending views; CSV export for statutory reporting.
9. **Audit Trail & Access Control** — Ed25519-signed, hash-chained ledger; role-based access; one-click chain-integrity verification that pinpoints the exact tampered entry if verification fails.
10. **Integration & Data Security** — a documented, named integration boundary (Section 8) instead of a vague "future integration" line.

### 6.1 Discrepancy Rule Engine (12 rules — synthesizing the strongest checks across all five teams)

| Code | Severity | Trigger |
|---|---|---|
| D01 | High | No document attached |
| D02 | High | Declared vs. extracted maturity date mismatch |
| D03 | High | Declared vs. extracted amount mismatch |
| D04 | High | Declared vs. extracted FDR number mismatch |
| D05 | Medium | Bank name mismatch (normalized) |
| D06 | Medium | Issue date mismatch |
| D07 | High | Maturity date already elapsed at submission |
| D08 | High | Maturity date earlier than issue date |
| D09 | High | Duplicate FDR number across the system |
| D10 | High | Matured instrument with no renewal/release submitted past grace period |
| D11 | Medium | Replacement instrument amount lower than the one it supersedes |
| D12 | High | Document hash (SHA-256) matches another institution's document — possible reuse/fraud |

Release is blocked while any High-severity rule is open, system-wide — not only at the release step.

---

## 7. Security & Compliance

- **Authentication:** Argon2id password hashing; mandatory TOTP 2FA for Regulation Bureau and Competent Authority accounts; short-lived access tokens with rotating refresh tokens.
- **Authorization:** Role-based access control with institution-level tenant isolation enforced server-side; release approval structurally cannot be performed by the requester or the recommender (separation of duties).
- **Data at rest:** AES-256-GCM for stored documents; database-level encryption for sensitive fields.
- **Integrity:** Hash-chained, Ed25519-signed audit ledger — append-only, independently verifiable, with a judge-facing "Verify Chain" action that names the exact row if tampering is detected.
- **Uploads:** MIME whitelist, file-size caps, SHA-256 fingerprinting, storage outside the web root, served only via an authenticated streaming route.
- **Honesty boundary:** every page that shows a "bank-confirmed" or "source-verified" badge is labeled with whether that confirmation came from the Mock Bank Connector (prototype) or a live integration (future, pending formal authorization) — no capability is presented ambiguously.

---

## 8. What's Real vs. What's Simulated (stated up front, not discovered by a judge)

| Capability | Status in Prototype | Path to Real |
|---|---|---|
| Document encryption, audit signing, hashing, OCR extraction, discrepancy engine, alert scheduler | **Real** | Already production-shaped; hardening only |
| Bank confirmation of FDR details | **Simulated** via Mock Bank Connector | Requires a bilateral agreement + CBS API access from a scheduled bank — not obtainable by a student team |
| Cross-council / multi-lien checks (PCI, NMC, UGC, NeSL) | **Not attempted in this prototype** | Requires formal data-sharing agreements between regulatory bodies; flagged as a Phase 2 policy item, not claimed as built |
| AICTE SSO / existing-system integration | **Interface stubbed, not connected** | Requires AICTE IT team coordination and credentials |

Stating this plainly is itself a differentiator: it is the single biggest credibility gap in Team 5b's otherwise ambitious submission, and a partial gap in Team 4 (silent about AI/OCR) and Team 1/5a (silent about MFA/encryption gaps).

---

## 9. Judge Demonstration Script (5 minutes)

1. **Submission & verification (90s):** Nodal Officer submits an FDR with a planted amount mismatch → system auto-flags D03 (High) → release stays blocked.
2. **Scrutiny (60s):** Scrutiny Official reviews the side-by-side declared-vs-extracted panel, resolves the discrepancy with a note, triggers Mock Bank Connector confirmation (clearly badged "Simulated").
3. **Release, with separation of duties (60s):** Nodal Officer requests release → Scrutiny Official recommends → a *different* Competent Authority account approves → instrument reaches **Discharged**. Attempting to approve from the requester's own account is shown being rejected.
4. **Maturity alerts via time-jump (45s):** Jump the simulated clock forward; the 90/60/30/15/7-day alert ladder fires and an unacknowledged alert auto-escalates.
5. **Renewal lineage + audit integrity (45s):** Show an instrument's full renewal chain (v1 → v2 → v3, with superseded badges), then run "Verify Chain" — inject a tamper and show it is caught and localized instantly.

---

## 10. Measurable Outcomes We Will Actually Test (per the PS's testing expectation)

- Record accuracy: % of submissions with zero open discrepancies after first scrutiny pass, before vs. after the discrepancy engine.
- Verification turnaround: time from submission to Source-Confirmed status, measured in-system across test cases.
- Renewal monitoring: % of matured instruments with a renewal/release action initiated before the grace-period deadline, across simulated time-jumped scenarios.
- Manual effort: count of distinct manual touch-points eliminated, documented against the legacy fragmented-record process described in the PS background.

No externally-sourced rupee-savings or percentage claims are asserted without a stated method of calculation — unlike the unsupported headline figures in the FinMatrix and SURAKSHA-FDR write-ups.

---

## 11. Roadmap

**Phase 1 (hackathon prototype):** everything in Section 8's "Real" column, full lifecycle state machine, 12-rule discrepancy engine, dual-source display, lineage graph, judge demo script.

**Phase 2 (pilot-ready):** formal integration points — AICTE SSO, one pilot bank's CBS API under a signed agreement, document digital-signature support if/when banks issue them, configurable multi-region deployment.

**Phase 3 (scale):** the cross-regulatory checks (NeSL/CERSAI, inter-council) explored by Team 5b — pursued only once the relevant data-sharing agreements exist, not claimed earlier.

---

*Prepared as a revision of our original proposed solution for SIH26265, incorporating a structured review of five competing team approaches.*
