# Feasibility, Viability & Impact — NIDHI-FDR
### Security Deposit / FDR Management Portal for AICTE
**SIH26265 · AICTE, Ministry of Education**

---

## 1. Feasibility Assessment

### 1.1 Technical Feasibility
NIDHI-FDR is built entirely from mature, well-documented, open technology — React, FastAPI, PostgreSQL, Argon2id, Ed25519, AES-256-GCM. Nothing in the architecture depends on an unreleased library, an unobtainable license, or an external authorization that doesn't already exist. Every "real" capability claimed (encryption, signed audit ledger, discrepancy engine, OCR extraction, lifecycle state machine) is implementable and demonstrable within the hackathon build window, because the stack was deliberately chosen to be **the smallest set of components that satisfies every PS requirement** — not the largest set that sounds impressive.

### 1.2 Operational Feasibility
The workflow mirrors how AICTE's Regulation Bureau already operates — submission, scrutiny, approval — so adoption requires **process digitization, not process redesign**. The three-actor separation of duties (requester / recommender / approver) maps directly onto an existing regulatory-approval pattern rather than introducing a new one. Training burden is low: each role sees only the screens relevant to its function.

### 1.3 Economic Feasibility
The stack uses open-source components throughout — no licensing cost for the core system. Hosting a prototype or a pilot-scale deployment is achievable on standard commodity cloud infrastructure. The largest future cost driver is not the software itself but **formal integration work** with external systems (bank APIs, AICTE's existing IT estate) — which is correctly scoped as a negotiated, authorized Phase 2 activity rather than hidden inside the prototype's cost.

### 1.4 Legal & Regulatory Feasibility
The system is designed to operate **within** existing custody arrangements rather than requiring new statutory authority: it digitizes and strengthens accountability for a process AICTE already performs. Where a capability would require authorization beyond what a prototype can obtain (live bank API access, inter-regulator data sharing), the system names that requirement explicitly instead of assuming it away — so the path to legal/regulatory sign-off is clear, not glossed over.

---

## 2. Viability — Why It Holds Up Under Scrutiny

| Dimension | Why NIDHI-FDR is viable, not just plausible |
|---|---|
| **Correctness** | Deterministic, rule-coded discrepancy detection — every flag is explainable, reproducible, and auditable, which is what a regulatory process actually needs |
| **Trustworthiness** | A cryptographically signed, hash-chained audit ledger means the system's own claims about "what happened and when" are independently verifiable, not just logged |
| **Honesty under questioning** | Every simulated component (e.g., bank confirmation) is explicitly labeled as such in the UI itself — the system cannot be caught overstating what it does |
| **Low operational risk** | A lean, explainable stack with no unnecessary service dependencies means fewer failure points in a live demo or a pilot deployment |
| **Alignment with existing process** | The lifecycle state machine and release workflow mirror AICTE's actual scrutiny → approval pattern, so adoption doesn't require reinventing institutional behavior |

---

## 3. Risks & Mitigation

| Risk | Mitigation |
|---|---|
| OCR misreads a document field | Every extracted field carries a confidence score and is shown side-by-side with the declared value — a human always makes the final call, the system never silently overwrites |
| A single compromised account attempts to discharge a deposit alone | Structurally prevented — requester, recommender, and approver must be three distinct accounts, enforced at the data layer |
| Institutions resist a new mandatory digital process | Submission mirrors the existing document-based process; the system reduces institution effort (status visibility, proactive reminders) rather than adding new burden |
| Future bank API integration is delayed or denied | The system already functions fully on declared + OCR-confirmed data; a live bank connector is an enhancement to confidence, not a dependency for operation |
| Audit ledger storage grows large over a 10-year deposit lifecycle | Append-only ledger is lightweight per entry (hash + signature + metadata); standard archival/partitioning strategies apply without touching the verification logic |

---

## 4. Impact & Benefits

### 4.1 For AICTE's Regulation Bureau
- A single, always-current view of every security deposit/FDR across all institutions, replacing fragmented manual tracking.
- Discrepancies surfaced automatically and explained, instead of discovered late through manual cross-checking.
- A verifiable, non-repudiable audit trail for every decision — strengthening accountability in exactly the area (release/discharge of financial instruments) where it matters most.
- Proactive visibility into upcoming maturities, reducing the risk of deposits lapsing unnoticed.

### 4.2 For Technical Institutions
- One digital submission instead of manual correspondence.
- Clear, real-time visibility into what has been confirmed versus what is still pending — no ambiguity about status.
- Advance reminders before renewal deadlines, reducing the risk of inadvertent non-compliance.

### 4.3 For the Regulatory Process as a Whole
- Faster verification turnaround through automated discrepancy pre-screening.
- A defensible, tamper-evident record suitable for internal review or external audit.
- A foundation that can extend — once formally authorized — to live bank confirmation and inter-system integration, without re-architecting anything already built.

### 4.4 Measurable Indicators (testable directly in the prototype)
- Reduction in records with unresolved discrepancies after first scrutiny pass.
- Time from submission to Source-Confirmed status.
- Proportion of maturing instruments with a renewal/release action initiated before the deadline, under simulated time-jump testing.
- Reduction in distinct manual touch-points compared to the fragmented-record process described in the problem statement.

No outcome is asserted without a stated method of measurement — every number above is something the prototype can actually be tested against, not a projected or assumed figure.

---

## 5. Scalability Outlook

The relational data model (institution → instrument → lineage graph) scales horizontally by institution/region without schema changes. The audit ledger and discrepancy engine are stateless per-transaction, so additional load is handled by scaling the application layer, not by redesigning the core logic. The system is architected so that moving from prototype to a multi-region AICTE-wide deployment is an infrastructure change, not a rebuild.

---

*This document should be read alongside `sol.md` (what the solution does and why it's novel) and `Technical.md` (how it is built).*
