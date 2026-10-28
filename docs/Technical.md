# Technical Approach & Architecture — NIDHI-FDR
### National Integrated Deposit Holdings Infrastructure for Security Deposit / FDR Management
**SIH26265 · AICTE, Ministry of Education**

---

## 1. System Architecture

```
                         ┌──────────────────────────────────┐
                         │        Presentation Layer          │
                         │  React + TypeScript + Tailwind     │
                         │  Role-aware SPA: Nodal Officer,     │
                         │  Scrutiny Official, Competent       │
                         │  Authority dashboards               │
                         └───────────────┬────────────────────┘
                                         │ HTTPS / REST (OpenAPI)
                         ┌───────────────▼────────────────────┐
                         │         Application Layer           │
                         │  FastAPI (Python, async)            │
                         │  ├─ Auth & RBAC Service              │
                         │  ├─ Instrument Lifecycle Engine      │
                         │  ├─ Discrepancy Rule Engine (12 rules)│
                         │  ├─ OCR / Field-Extraction Service    │
                         │  ├─ Trust & Lineage Engine            │
                         │  ├─ Alert & Escalation Scheduler      │
                         │  └─ Signed Audit Ledger Service       │
                         └───────┬───────────────┬──────────────┘
                                 │               │
                 ┌───────────────▼───┐   ┌───────▼────────────┐
                 │   PostgreSQL        │   │  Encrypted Document  │
                 │   (system of record,│   │  Store (AES-256-GCM, │
                 │   lineage graph,    │   │  SHA-256 fingerprint)│
                 │   audit ledger)     │   └───────────────────┘
                 └─────────────────────┘
                                 │
                         ┌───────▼────────────┐
                         │  Mock Bank Connector │  ← clearly badged,
                         │  (simulated CBS      │    swappable for a
                         │  confirmation layer) │    real bank API later
                         └───────────────────────┘
```

**Design intent:** a single authoritative backend, a single relational store for both transactional data and the cryptographic audit graph, and one clearly-isolated connector boundary for anything that depends on external authorization. This keeps the system auditable end-to-end and removable-part-by-part — nothing is a black box.

---

## 2. Technology Stack & Rationale

| Layer | Technology | Why this, specifically |
|---|---|---|
| Frontend | React 18 + TypeScript + Vite + Tailwind CSS | Type safety for financial-field handling; fast iteration; componentized role-based views |
| Backend | FastAPI (Python 3.11+) | Async I/O for concurrent document processing; auto-generated OpenAPI spec doubles as a live technical artifact for evaluation |
| Database | PostgreSQL 15+ | ACID guarantees for financial records; native support for recursive queries (needed for the lineage graph); row-level isolation for multi-tenant institution data |
| ORM / Migrations | SQLAlchemy 2.x + Alembic | Versioned schema evolution; explicit, reviewable migrations |
| Document storage | Local encrypted volume, S3-compatible interface | AES-256-GCM encryption at rest; swappable to managed object storage without an application rewrite |
| OCR / Extraction | Open-source OCR pipeline + rule-calibrated regex extraction | Deterministic, explainable field extraction with confidence scoring — avoids opaque "black box" AI claims |
| Anomaly scoring | Gradient-boosted / isolation-forest style classifier on engineered features | Flags statistically unusual submissions for human review; always a *signal*, never an automatic rejection |
| Scheduler | APScheduler (in-process) | Maturity-alert scanning (90/60/30/15/7-day ladder) without an external queue/broker dependency |
| Auth | Argon2id password hashing, JWT access + rotating refresh tokens, mandatory TOTP 2FA for Regulation Bureau and Competent Authority roles | Industry-standard credential hygiene with minimal operational surface |
| Audit integrity | Hash-chained ledger entries, each additionally signed with Ed25519 by the acting role's key | Asymmetric signing means tampering requires forging a private key — stronger than hash-only or symmetric-HMAC chains |

**Why this stack over a larger "enterprise" stack (Keycloak, Redis, Celery, MinIO, multi-service orchestration):** every additional moving part is a point of demo failure and a line nobody on the team can defend under questioning. This stack is intentionally the smallest set of components that still delivers every required capability — defensible, explainable, and fully working, not partially working across more services.

---

## 3. Data Model (Core Entities)

| Entity | Key Fields | Notes |
|---|---|---|
| **Institution** | id, name, region, nodal_officer_id | Tenant boundary for row-level isolation |
| **Instrument (FDR)** | id, institution_id, bank_id, reference_no, amount, issue_date, maturity_date, status, parent_instrument_id | `parent_instrument_id` is what makes lineage a graph, not a flag |
| **Document** | id, instrument_id, sha256_hash, storage_ref, mime_type, extracted_fields (JSON) | Hash enables cross-record duplicate/reuse detection (Rule D12) |
| **DeclaredField / ConfirmedField** | field_name, declared_value, confirmed_value, source (Institution / OCR / MockBankConnector), confidence | The dual-source model — every financial fact has a provenance |
| **DiscrepancyFlag** | rule_code, severity, status (Open/Resolved), resolution_note | Maps directly to the 12-rule engine |
| **ReleaseRequest** | id, instrument_id, requested_by, recommended_by, approved_by, stage | Three distinct actor fields — the schema itself enforces separation of duties |
| **AuditEntry** | id, prev_hash, entry_hash, signature, actor_id, action, timestamp | Append-only; `prev_hash` + `signature` make the chain independently verifiable |
| **Alert** | id, instrument_id, threshold_days, status, escalated_to | Deduplicated by (instrument_id, threshold_days) |

---

## 4. Core Algorithms & Engines

### 4.1 Trust Ladder State Machine
`SUBMITTED → MACHINE-VERIFIED → SOURCE-CONFIRMED → ACTIVE → {RENEWAL | RELEASE} → RENEWED / DISCHARGED`
Each transition is a guarded function: it may only fire if its precondition (e.g., "zero open High discrepancies") holds, and it always writes exactly one signed `AuditEntry`.

### 4.2 Discrepancy Rule Engine
A deterministic rule set (12 rules, D01–D12 — see `sol.md` for the full table) runs on every submission and every document re-upload. Each rule is a pure function of `(declared_fields, confirmed_fields, existing_records) → (triggered: bool, severity: High|Medium)`. Determinism is a deliberate choice: every flag is explainable and reproducible, which matters for a regulatory system far more than a marginal accuracy gain from a less interpretable model.

### 4.3 Lineage Graph
Renewal/replacement does not overwrite a record — it inserts a new `Instrument` row with `parent_instrument_id` pointing to the one it supersedes, and marks the parent `SUPERSEDED`. The full chain is reconstructed with a single recursive SQL query, giving a 1→N-generation audit view for any instrument in constant developer effort.

### 4.4 Signed Audit Ledger
```
entry_hash_i = SHA256(prev_hash_(i-1) || canonical_json(event_i))
signature_i  = Ed25519_Sign(private_key_role, entry_hash_i)
```
Verification recomputes the chain from genesis and checks every signature against the signer's public key. A single-byte change to any historical row breaks both the hash chain *and* the signature, and the verifier reports the exact offending entry.

---

## 5. Constraints & Rules

- **Separation of Duties (hard rule):** `requested_by`, `recommended_by`, and `approved_by` on a `ReleaseRequest` must be three distinct user IDs — enforced at the database/service layer, not only the UI, so it cannot be bypassed via direct API calls.
- **No release past an open High discrepancy** — enforced the same way, system-wide.
- **Tenant isolation:** every query touching `Institution`-scoped data is filtered server-side by the authenticated user's institution_id; cross-tenant access attempts are logged as audit events, not silently denied.
- **Immutability:** `AuditEntry` rows are insert-only; no UPDATE/DELETE grants exist on that table at the database role level.
- **Explicit provenance labeling:** any UI element showing a "confirmed" value must render its `source` field — the interface cannot present a simulated confirmation as indistinguishable from a real one.

---

## 6. Non-Functional Requirements

| Requirement | Target | Approach |
|---|---|---|
| Availability (pilot scale) | Standard single-region deployment | Stateless API layer behind a reverse proxy; DB backups on a defined schedule |
| Data integrity | Zero undetected tampering | Signed hash-chained ledger, verified on demand |
| Confidentiality | Encrypted at rest and in transit | AES-256-GCM for documents, TLS for transport, hashed/salted credentials |
| Auditability | Every state change traceable to an actor and timestamp | Mandatory audit-entry write on every lifecycle transition |
| Explainability | Every automated flag must have a human-readable reason | Deterministic rule engine with rule codes, not opaque ML-only decisions |

---

## 7. Assumptions & Dependencies

- Institutions and AICTE officials access the portal over the public internet with modern browsers; no native mobile client is assumed necessary for the prototype.
- Bank confirmation is **not** assumed to be live — the Mock Bank Connector is an explicit, swappable interface, not a placeholder pretending to be real.
- No assumption is made of pre-existing API access to AICTE's internal systems, RBI's SFMS messaging layer, or NeSL/CERSAI; these are named as future integration points requiring formal authorization (see `Feasible.md`), not built against.

---

*This document defines the technical approach only. For how this architecture solves the stated problem and what sets it apart, see `sol.md`. For feasibility, risk, and impact, see `Feasible.md`.*
