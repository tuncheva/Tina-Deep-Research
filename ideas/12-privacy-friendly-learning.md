# 12) Privacy-Friendly Learning: Smart Signals Without Tracking

## Catchy Explanation
Signals can learn from traffic patterns the way you learn from a crowd: you don’t need to track individuals—aggregates are often enough.

## What it is (precise)
**Privacy-friendly learning** improves signal timing using **data minimization** and privacy-preserving methods (aggregates, on-device/edge processing, strict retention, access controls) so optimization does not rely on storing individual trajectories.

Typical inputs include counts, occupancy, travel time summaries, phase states, and derived KPIs (e.g., split failures). A digital twin can quantify the privacy–performance tradeoff by comparing outcomes when using aggregate-only vs richer data, and can help design safe aggregation windows.

## Benefits
- **Trust and legitimacy**: reduces surveillance concerns and public pushback.
- **Regulatory alignment**: easier compliance with privacy frameworks (e.g., GDPR principles).
- **Lower breach impact**: less sensitive raw data stored.
- **Operational viability**: aggregates can still support meaningful optimization.

## Challenges
- **Accuracy tradeoffs**: aggregates may hide important variation.
- **Re-identification risk**: small cells or rare patterns can still leak identity.
- **Engineering overhead**: secure pipelines, edge compute, audits.
- **Explainability burden**: must show what is collected and why.

---

## Core Principle: Separate “Operations” From “Publication”

### Two boundary types
1. **Operational boundary (internal, real-time):**
   - Goal: safe and reliable signal operations.
   - Data: aggregated telemetry (detector counts/occupancy, controller event logs, binned travel-time summaries).
   - Privacy posture: strict minimization + access control + retention.
   - Typically **not** differential privacy (DP), because DP is designed for *intentional releases* of statistics and carries utility loss.

2. **Publication boundary (external sharing / open data / public dashboards):**
   - Goal: transparency and reporting.
   - Data: heavily aggregated and/or privacy protected.
   - Privacy posture: small-cell suppression/coarsening, and **DP where appropriate**.
   - Key point: DP does **not** protect raw collection/storage; it protects privacy at the *release* boundary ([`NIST.SP.800-226`](https://doi.org/10.6028/NIST.SP.800-226)).

---

## 1) Privacy Risk Assessment (PIA) — Quantified, Repeatable

### Why: treat privacy like an engineering risk
A Privacy Impact Assessment (PIA) is a process to evaluate collection of personal data in information systems and determine whether collected personal information is necessary and relevant ([`USDOT Privacy Impact Assessments`](https://www.transportation.gov/individuals/privacy/privacy-impact-assessments)).

NIST frames privacy risk assessment as a process to analyze and assess privacy risks for individuals arising from the processing of their data ([`NIST Privacy Risk Assessment`](https://www.nist.gov/itl/applied-cybersecurity/privacy-engineering/collaboration-space/privacy-risk-assessment)).

### Threat model template (use in every PIA)
Define a *system-specific* threat model. Minimum required elements:

**Adversaries**
- External attacker (data breach, scraping public releases, correlation with outside datasets)
- Insider (authorized access misused; “curious analyst”) — common in analytics programs
- Vendor/subprocessor (support access, backups, off-shore processing)
- Well-resourced third party with auxiliary data (data brokers, platform logs)

**Capabilities & attack surfaces**
- **Re-identification/linkage**: joining aggregates with auxiliary datasets
- **Membership inference**: “Was person X in this dataset?”
- **Sensitive inference**: learning attributes from group-level stats (e.g., rare movements)
- **Reconstruction**: deriving underlying records from “too many” aggregates (risk even for aggregate statistics) ([`NIST.SP.800-226`](https://doi.org/10.6028/NIST.SP.800-226))

**Data actions that create privacy risk (examples)**
- Persisting identifiers (MACs, device IDs, plate text)
- Long retention of raw video
- Exporting raw logs to ad-hoc analyst workspaces
- Public release of fine-grained time/location tables

### Risk scoring rubric (repeatable)
Use a 1–5 scale for both likelihood and impact, compute **risk = likelihood × impact (1–25)**.

**Likelihood (1–5)**
1. Highly unlikely: strong controls, no plausible auxiliary data
2. Unlikely: controls present, limited access
3. Possible: common data actions, moderate access
4. Likely: broad access and/or weak monitoring, common auxiliary data
5. Almost certain: public release or known adversary interest; “attack assumed” for open data

**Impact (1–5) – impact to individuals + operational impacts**
1. Minimal: no credible individual harm
2. Low: inconvenience, limited inference
3. Moderate: sensitive inference plausible, reputational harm
4. High: discrimination/enforcement risk, safety or personal harm plausible
5. Severe: serious harm at scale

**Go/No-go thresholds**
- **1–6 (Low):** go with standard controls
- **7–12 (Medium):** go only with mitigation plan + approval
- **13–25 (High):** no-go unless redesigned (minimize further / coarsen / DP at release) and executive privacy sign-off

### Low-volume / small-cells: “aggregate safety” rules
Aggregates can still be identifying in low-volume contexts. Aggregate data is **not automatically de-identified**; a risk assessment must still be conducted ([`IPC Ontario De-Identification Guidelines`](https://www.ipc.on.ca/en/media/5946/download)).

Practical rules for sparse contexts:
- Treat any metric that can isolate a single household, vehicle, or rare movement as **high risk**.
- Apply *suppression and combining* until groups are “large enough” to avoid uniqueness.
- For public releases, assume attack probability = 1 (public release implies adversary certainty) ([`IPC Ontario De-Identification Guidelines`](https://www.ipc.on.ca/en/media/5946/download)).

### Required artifacts (minimum deliverables)
Align to the **NIST Privacy Framework “Inventory and Mapping”** concept: inventory systems, owners/operators, data actions, purposes, data elements, and the data processing environment ([`NIST Privacy Framework Core v1.0`](https://www.nist.gov/document/nist-privacy-framework-version-1-core-pdf)).

Artifacts your program must produce and keep current:
- **Data inventory** (systems/products/services; owners/operators; data elements; purposes)
- **Data flow diagram** (edge → transport → store → analytics → reporting)
- **Retention schedule** by data class
- **Access matrix** (roles → permissions; + break-glass)
- **Export register** (who exported what, when, why, under what approval)

### Review cadence and sign-off
- **Initial PIA:** before pilot launch
- **Change-triggered re-review:** sensor type change, new vendor, new data field, new external sharing
- **Cadence:** quarterly quick review + annual full review

**Sign-off (minimum):**
- Privacy/legal
- IT security
- Signals operations lead
- Data governance owner

---

## 2) Differential Privacy (DP) — Operational Guidance (Where/When/How)

### What DP is for (and what it is not)
DP is designed to bound privacy loss to individuals when releasing statistics, typically by adding calibrated random noise to results; it is a formal framework to protect against a wide range of attacks, including linkage using auxiliary data ([`NIST.SP.800-226`](https://doi.org/10.6028/NIST.SP.800-226)).

**DP is not a substitute for security controls**: if raw data is breached, the DP guarantees for leaked records are nullified; DP does not protect data during collection and storage ([`NIST.SP.800-226`](https://doi.org/10.6028/NIST.SP.800-226)).

### Separation of use cases

| Use case | DP recommended? | Why |
|---|---:|---|
| Public open data / transparency dashboards | Yes (often) | Releases are adversary-facing; assume attacks; DP can bound incremental privacy loss ([`NIST.SP.800-226`](https://doi.org/10.6028/NIST.SP.800-226)). |
| Research sharing to external parties | Sometimes | If data leaves your control, DP can reduce linkage risk; still need contracts/controls. |
| Internal signal operations / control loops | Usually no | Noise can cause unsafe or unstable operational decisions; apply DP at publication boundary, not inside control loop (unless explicitly justified and tested). |

### Decision tree: “Should we use DP here?”

1) **Is the output leaving the organization’s control (public/open data or broad sharing)?**
- If **yes** → DP is a candidate.
- If **no** → prefer access controls + minimization + retention.

2) **Is the output safety-critical or used for real-time control?**
- If **yes** → **do not** use DP-noised data in the control loop.
- If **no** → DP can be applied at reporting boundary.

3) **Can you bound sensitivity (worst-case impact of one person’s data)?**
- If **no** → you are not ready; redesign aggregation (cap contributions, define units).
- If **yes** → proceed.

4) **Can you manage a privacy loss budget across repeated releases?**
- If **no** → redesign release cadence or reduce query surface.
- If **yes** → proceed.

### Practical parameter selection (engineering-level)
DP choices must be set based on context and constraints; ε/δ values depend on goals and constraints ([`NIST.SP.800-226`](https://doi.org/10.6028/NIST.SP.800-226)). Use this decision approach:

1. **Pick the unit of privacy**
- For signals, common units are: “one trip,” “one device-day,” or “one person-day.”
- Choose the unit you can enforce in preprocessing (cap contributions).

2. **Choose aggregation level (space/time bins)**
- Coarser bins reduce sensitivity and improve utility for a given privacy budget.
- Start with corridor-level or zone-level, with time windows ≥ 15 minutes for public releases unless volumes are high.

3. **Set sensitivity bounds**
- Counts: sensitivity often 1 if each unit contributes to at most one bin.
- If the same unit can contribute to many bins (e.g., repeated pings), cap contributions per unit/time.

4. **Choose ε and δ (policy decision + utility testing)**
- Treat ε as a **risk/utility dial** (smaller ε → stronger privacy, more noise) ([`NIST.SP.800-226`](https://doi.org/10.6028/NIST.SP.800-226)).
- Establish a governance policy: e.g., “public dashboards must fit within a quarterly privacy loss budget, with ε allocated per table and per update.”

5. **Run utility analysis before release**
- DP adds noise; evaluate confidence intervals and ensure key conclusions remain stable.

### Operational safety guardrail
- **Never** feed DP-noised aggregates into safety-critical real-time actuation (split, offset, recall decisions).
- If DP is used anywhere near operational analytics, require a safety case: simulation evidence + fallback to non-noised internal telemetry.

---

## 3) Edge Analytics Scoping — Output Minimization + Aggregation Windows

### Goal
Make “edge processing” actually privacy-preserving by constraining **outputs** and **retention**, not just where compute runs.

### Safe output patterns (default allow-list)
Edge devices may emit only:
- **Counts** (by movement/class) per fixed time bin
- **Occupancy** / detector on-time per bin
- **Signal performance aggregates**: split failures, max-out rate, arrivals-on-green (aggregated), pedestrian service metrics
- **Binned speeds** (e.g., 0–10, 10–20, …) and **binned travel times** (corridor-level) — no per-vehicle series
- **Phase/event state logs** from controllers (already operational telemetry)

### Output minimization rules (hard requirements)
- No **unique IDs** (no plate, no face embedding, no MAC, no hashed device IDs, no stable pseudonyms).
- No raw trajectories or path reconstructions.
- No raw video storage by default. If unavoidable, use tightly controlled, short retention and documented justification.

Rationale: identifiers and fine-grained traces turn “traffic analytics” into tracking.

### Aggregation windows + minimums (“k-anonymity-like” guardrails)
Aggregated data can still be identifying; you must manage small cells ([`IPC Ontario De-Identification Guidelines`](https://www.ipc.on.ca/en/media/5946/download)).

Policy:
- **Internal ops aggregates:** allow lower thresholds, but still suppress extremely small cells in analyst-facing exports.
- **External/public releases:** require minimum cell sizes and/or DP.

Practical rules:
- Define minimum count threshold **k** per published cell.
- If a bin has count < k, apply one of:
  - **Suppress** (no value)
  - **Combine** adjacent time windows
  - **Coarsen** spatial bin (merge approaches or zones)
  - **Switch to DP** release with budget accounting

The IPC guidance provides an example of a minimum cell size requirement (e.g., a “minimal cell size of 11” in an illustrative table) and emphasizes that aggregated data still requires risk assessment ([`IPC Ontario De-Identification Guidelines`](https://www.ipc.on.ca/en/media/5946/download)).

### No-go list (never collect / never persist)
- License plates (ALPR) for signal learning
- Face detection/recognition outputs
- Persistent device identifiers (MAC, advertising IDs)
- Full-resolution trajectories (per vehicle/person)
- Any dataset enabling “where did this person go” queries

### Requires exception approval (documented justification + time limit)
- Raw video clips (maintenance/incident QA only)
- High-frequency per-object tracking (temporary engineering studies)
- Pedestrian near-miss analytics using video (high privacy risk; must be separated, minimized, and governed)

---

## 4) Data Governance Lifecycle — Vendor Access, Audits, Leak Response

### Governance model (vendor-neutral)
Use the NIST Privacy Framework core as a vocabulary for governance: inventory/mapping, risk assessment, access control, audit logs, ongoing monitoring and review ([`NIST Privacy Framework Core v1.0`](https://www.nist.gov/document/nist-privacy-framework-version-1-core-pdf)).

### Vendor / contract language checklist
Include these clauses in procurement and DPAs (Data Processing Addendums):

**Data scope + permitted types**
- Explicitly list permitted fields (aggregate-only; no IDs)
- Prohibit collection/storage of identifiers unless explicitly approved

**Purpose limitation + secondary use**
- Data only for defined signal operations/twin calibration
- No secondary use (training vendor models, resale, advertising)

**Retention + deletion**
- Retention limits per data class (see retention defaults below)
- Secure deletion requirements + deletion attestations

**Subprocessors**
- Disclose subprocessors; require written approval for changes
- Flow down the same obligations

**Access + support**
- Support access only via agency-controlled channels
- MFA required; time-bound access; session recording for privileged actions

**Breach notification**
- Define notification timeline (e.g., “without undue delay,” and a hard maximum like 72 hours) aligned to your jurisdictional requirements

**Audit rights**
- Right to audit (or receive SOC2/ISO reports), plus allow targeted audits after incidents

### Access control (minimum technical + procedural)
- RBAC (role-based access control) and least privilege ([`NIST Privacy Framework Core v1.0`](https://www.nist.gov/document/nist-privacy-framework-version-1-core-pdf))
- Separate production and development environments ([`NIST Privacy Framework Core v1.0`](https://www.nist.gov/document/nist-privacy-framework-version-1-core-pdf))
- Audit logs for access and exports; review logs routinely ([`NIST Privacy Framework Core v1.0`](https://www.nist.gov/document/nist-privacy-framework-version-1-core-pdf))

### Export gates (stop “shadow datasets”)
- Require approval for any export of non-public datasets.
- Log: requester, dataset, fields, time range, purpose, destination, retention.
- Maintain an export register as a standing artifact (ties back to your PIA).

### Audit cadence
- **Quarterly:** access review (who has access; remove stale accounts)
- **Quarterly:** retention verification (purge jobs succeed)
- **Annual:** privacy review + update PIA
- **Annual:** security assessment / pen-test scope including data stores and dashboards

---

## 5) Privacy–Performance Tradeoffs — What Tasks Need What Fidelity

### Practical mapping table

| Task | Acceptable data (aggregate-only?) | Degradation risks | Mitigations |
|---|---|---|---|
| 1–5 minute queue/spillback forecasting | Yes: counts/occupancy per 10–60s + phase states | Less accurate on short queues; harder to detect lane-specific spillback | Use multiple detectors per approach; fuse with controller states; conservative thresholds |
| Arrivals-on-green (AOG) estimation | Yes: detector time stamps (binned) + phase states | Coarse bins blur platoon structure | 10–30s bins internally; validate vs short-term studies; avoid public release at high resolution |
| Multimodal compliance monitoring (ped wait caps) | Mostly yes: ped call, service time, clearance events | Cannot detect “informal crossings” without richer sensing | Use button/call logs + field audits; add privacy-preserving counts (no tracking) |
| Incident detection (blocked lane, detector failures) | Yes: occupancy anomalies, sudden count drops, max-out rate | May miss subtle incidents; false positives | Add quality gates; correlate across detectors; operator confirmation workflow |
| Safety proxies / near-miss analytics | Often no (higher risk) | Near-miss needs trajectories/video; high re-identification risk | Use short-term, exception-approved studies; strong minimization; consider synthetic/DP only for reporting outcomes |

### Explicitly not feasible (without higher-granularity data)
- Attribution to specific users/vehicles
- Origin-destination reconstruction for individuals
- Behavioral profiling (compliance per person)

Alternative approaches:
- Use aggregate surrogates (counts, occupancy, delay distributions)
- Use targeted, time-limited studies with strict governance

---

## 6) Minimum Technical Controls + Example Architectures

### Reference architecture (aggregate-first)

```text
[Sensors: loops/radar/video]            [Signal Controller]
        | (edge extraction)                    |
        v                                     v
  [EDGE AGGREGATOR] ------------------> [Controller Event Stream]
        |  counts/occ/speed bins                |
        |  NO IDs / NO raw traj                 |
        v                                       v
   (TLS/mTLS)                             (TLS/mTLS)
        \                                     /
         \                                   /
          v                                 v
        [Secure Ingest API / Broker (authN/Z, rate limits)]
                          |
                          v
                 [Aggregate Store (time-series + parquet)]
                          |
        +-----------------+-------------------+
        |                                     |
        v                                     v
[Ops Analytics / QA Dashboards]        [Digital Twin Calibration]
        |                                     |
        +-----------------+-------------------+
                          |
                          v
             [Reporting Layer (public release boundary)]
                          |
        small-cell rules + coarsening + optional DP
```

### Security basics that directly support privacy
DP and aggregation don’t remove the need for security; if raw/less-minimized data is breached, privacy impact increases and DP guarantees do not apply to leaked records ([`NIST.SP.800-226`](https://doi.org/10.6028/NIST.SP.800-226)).

Minimum controls:
- **Encryption in transit:** TLS/mTLS edge → ingest → storage
- **Encryption at rest:** for aggregate stores and backups
- **Key management:** rotate keys, restrict KMS access
- **Time sync:** NTP/PTP alignment for accurate bins (prevents over-collection “for debugging”)
- **Logging:** immutable access logs; export logs
- **Separation:** dev/test vs prod environments ([`NIST Privacy Framework Core v1.0`](https://www.nist.gov/document/nist-privacy-framework-version-1-core-pdf))

### Retention defaults by data class (starting point)
Treat retention as a policy knob tied to risk.

| Data class | Examples | Default retention | Notes |
|---|---|---:|---|
| Edge aggregates (ops) | counts/occ per 10–60s, phase states | 30–180 days | enough for QA and seasonal calibration; minimize long tails |
| Aggregated historical (planning) | hourly/daily summaries | 1–3 years | acceptable if sufficiently aggregated; monitor small cells |
| Raw sensor artifacts (exception only) | short video clips, debug logs | 24–72 hours | require ticket + justification + auto purge |
| Exports (research) | curated extracts | project-limited | must be logged in export register and deleted per plan |

---

## Implementation Strategies

### Infrastructure Needs
- **Aggregate pipelines**: counts/occupancy/travel time summaries with quality checks.
- **Edge processing**: discard raw video/plate/trajectory data at source.
- **Access controls**: RBAC, audit logs, export controls.
- **Retention controls**: short windows, automatic purge.
- **Twin evaluation**: assess how aggregation choices affect decisions.

### Detailed Implementation Plan

#### Phase 1: Privacy Requirements, Data Inventory, and “No-Go” Decisions (Weeks 1–4)
The agency should begin by documenting what data is currently collected, where it is stored, and who can access it, because privacy risk often comes from “forgotten” logs and vendor backends rather than from the optimization algorithm itself. The privacy and legal teams should define explicit “no-go” data elements (for example, license plates, persistent device IDs, or raw trajectories) and should define the permissible use cases and retention periods under the relevant regulatory context. The engineering team should define aggregation windows and minimum cell sizes for reporting so that small samples do not create re-identification risk, and it should define a clear separation between operational telemetry and research datasets. Finally, the program should create a governance model that includes role-based access control, audit logging, and a process for approving any data export.

- **Roles**: privacy/legal (policy), data governance lead (controls), IT/security (RBAC and logging), traffic engineering (operational needs), vendor manager (contract requirements).
- **Deliverables**: privacy policy and data inventory, no-go list, retention schedule, access control matrix, and an initial transparency statement.
- **Risks**: existing vendor systems may already store sensitive data; ambiguity about “operational” vs “research” use.
- **Acceptance checks**: no-go data is formally prohibited, retention is defined for all datasets, and access is limited to named roles.

#### Phase 2: Build Aggregate-Only and Edge-Minimized Data Pipeline (Weeks 5–12)
The engineering team should redesign the pipeline so that aggregation happens as close to the sensor as possible, because edge aggregation reduces the risk of storing sensitive raw data. For video analytics, this means extracting counts and classifications and discarding raw video unless a short, justified retention window is required for maintenance. For probe travel times, this means storing only corridor-level summaries and not persistent device identifiers. The team should implement data quality gates (detector stuck checks, time drift detection) so aggregate statistics are reliable, because bad aggregates can create unsafe timing decisions. The pipeline should enforce retention automatically and should log access and exports.

- **Roles**: data engineer (pipeline), IT/security (hardening and access control), signal maintenance (data quality feedback), privacy reviewer (design approval).
- **Deliverables**: validated aggregate pipeline, edge processing configuration, automated purge jobs, and audit log reports.
- **Risks**: edge compute constraints; vendor tooling may not support desired minimization; data quality issues may be harder to debug without raw data.
- **Acceptance checks**: raw data is not retained beyond approved windows, aggregates are accurate enough for operations, and access logs are complete.

#### Phase 3: Twin Calibration and Optimization Using Aggregates (Weeks 13–20)
The modeling team should calibrate the digital twin using aggregate inputs such as approach counts, occupancy, travel time summaries, and high-resolution phase state, because the goal is to prove that privacy-friendly data is sufficient for meaningful improvements. Where the agency has access to richer historical datasets (for example, short-term trajectory studies), the team should compare performance to quantify the privacy–performance tradeoff and to identify which aggregates matter most. The team should also test how aggregation window choices affect decisions (for example, 30 seconds vs 5 minutes) and should choose settings that preserve operational responsiveness without creating small-cell risk.

- **Roles**: modeler (twin calibration), analyst (tradeoff study), traffic engineer (acceptance thresholds), privacy reviewer (aggregation settings).
- **Deliverables**: privacy–performance report, recommended aggregation windows and minimum cell sizes, and a documented calibration workflow.
- **Risks**: aggregates may be too coarse for certain optimizations; comparison datasets may be unavailable.
- **Acceptance checks**: the twin can be calibrated and validated using aggregates, and chosen aggregation settings meet both privacy and operational needs.

#### Phase 4: Hardening, Audits, and Re-identification Testing (Weeks 21–32)
The agency should harden the system by implementing periodic privacy audits that verify retention policies, access controls, and export behavior, because privacy-friendly design is a continuous practice rather than a one-time decision. The team should perform re-identification risk testing on released aggregates, focusing on small cells, unusual time windows, and rare patterns, because these are the common failure modes in aggregate-only systems. If certain aggregates remain high-risk, the team can add protective measures such as suppressing small cells, coarsening time windows, or applying differential privacy noise where appropriate. The agency should also ensure that vendor contracts reflect the same retention and access requirements.

- **Roles**: privacy officer/auditor (audit), security (controls verification), data governance (export approval), engineering (mitigations), procurement/vendor manager (contract alignment).
- **Deliverables**: audited production pipeline, re-identification test report, mitigation plan, and updated vendor requirements.
- **Risks**: audits uncover legacy data stores that are hard to remediate; differential privacy can reduce operational utility if misapplied.
- **Acceptance checks**: audit findings are resolved on schedule, and high-risk aggregates are suppressed or protected.

#### Phase 5: Monitoring, Transparency, and Continuous Compliance (Ongoing)
The agency should publish a plain-language transparency report that explains what data is collected, how long it is retained, and what it is used for, because public trust is a core benefit of privacy-friendly learning. The program should run regular compliance reviews and should treat privacy incidents and near-misses as operational events with documented remediation. Over time, the team should incorporate privacy constraints into new features so the system does not drift toward more invasive data collection.

- **Roles**: program owner (governance), privacy/legal (compliance), operations (incident reporting), data governance (ongoing controls).
- **Deliverables**: periodic transparency report, quarterly compliance review, and updated privacy requirements for new features.
- **Risks**: scope creep toward more detailed data; inconsistent practices across vendors.
- **Acceptance checks**: retention and access controls remain enforced, and new deployments pass privacy review before launch.

### Choices
- **Aggregates only**: simplest and often sufficient.
- **Edge analytics**: strongest minimization (recommended).
- **Differential privacy**: added protection at publication boundary where necessary.

---

## Technical Mechanics

### Key Parameters
- Aggregation window (e.g., 10–60s internal; 15–60min for public)
- Minimum cell sizes for release (k threshold) and suppression/coarsening rules
- Retention period by data class
- Access log completeness and export logging
- DP budget (ε/δ) where used

### Guardrails
- Default to smallest useful data.
- Block exports for small cells; require export approvals.
- Enforce retention automatically.
- Separate ops telemetry from research datasets.
- Apply DP at publication boundary, not inside control loops.

---

## MVP Deployment
- One corridor.
- Counts/occupancy aggregates only.
- 30–180 day retention (ops aggregates).
- Quarterly access + retention audit.

## Evaluation
- Performance gap vs richer data (if measured).
- Privacy audit findings (small-cell and linkage risk).
- Pipeline reliability and data quality.

---

## Implementation Checklist
- [ ] Produce data inventory + data flow diagram (PIA artifacts) ([`NIST Privacy Framework Core v1.0`](https://www.nist.gov/document/nist-privacy-framework-version-1-core-pdf))
- [ ] Define no-go list + exception process
- [ ] Define retention schedule + automate purge
- [ ] Implement RBAC + MFA + logging + export register ([`NIST Privacy Framework Core v1.0`](https://www.nist.gov/document/nist-privacy-framework-version-1-core-pdf))
- [ ] Implement edge output allow-list (counts/occupancy/etc.)
- [ ] Define small-cell rules (k threshold; suppress/combine/coarsen)
- [ ] Decide DP usage for public reporting; define privacy loss budget governance ([`NIST.SP.800-226`](https://doi.org/10.6028/NIST.SP.800-226))
- [ ] Run twin calibration + utility tests across aggregation windows
- [ ] Establish quarterly access review + annual privacy review
- [ ] Publish transparency statement

---

## Data Governance & Access Runbook

### Roles
- **Data owner (Signals Ops):** approves operational use
- **Data steward (Governance):** maintains inventory/retention/access matrix
- **Privacy/legal:** approves PIA and external sharing
- **Security:** enforces IAM, logging, encryption
- **Vendors:** limited support access under contract

### Access request process
1. Requester submits ticket: purpose, datasets, time range, fields.
2. Data steward checks against no-go list, retention class, and purpose limitation.
3. Security verifies role + MFA; grants least-privilege access.
4. Access is time-bounded; reviewed quarterly.

### Export process (internal/external)
1. Export request must include: business purpose, recipients, storage location, retention/deletion plan.
2. Approval: Data owner + Data steward + Privacy (for external) + Security (for sensitive stores).
3. Log export in **export register**.
4. Verify deletion at end of retention.

### Ongoing monitoring
- Weekly: failed purge jobs, unusual access patterns
- Quarterly: access review and attestation
- Annual: PIA refresh and vendor compliance check

---

## Incident Response (Privacy) Runbook

### Triggers
- Suspected breach of any store containing more-than-aggregate data
- Evidence of unintended identifier capture (e.g., MACs, plate text)
- Public release complaint or evidence of re-identification

### First 60 minutes (containment)
1. Freeze exports; disable non-essential access.
2. Preserve logs (access logs, export logs, config history).
3. Identify affected data classes and time ranges.

### Assessment (same day)
- Determine if raw/less-minimized data was exposed. If yes, assume higher individual risk.
- If DP releases exist, note that DP does not protect leaked raw records; DP still applies to records not leaked ([`NIST.SP.800-226`](https://doi.org/10.6028/NIST.SP.800-226)).
- Determine whether external/public data needs to be pulled back or corrected.

### Notification + remediation
- Notify internal stakeholders (privacy/legal, security, signals ops) immediately.
- Notify vendors/subprocessors as required by contract.
- Follow jurisdictional breach notification requirements.

### Post-incident improvements
- Root cause analysis: which data action failed (collection, retention, access, export)
- Update no-go/exceptions, monitoring, and PIA
- Add tests: “no IDs emitted” assertions at edge ingest

---

## Reference Links
- [`NIST.SP.800-226`](https://doi.org/10.6028/NIST.SP.800-226) — Guidelines for Evaluating Differential Privacy Guarantees
- [`NIST Privacy Framework Core v1.0`](https://www.nist.gov/document/nist-privacy-framework-version-1-core-pdf)
- [`NIST Privacy Risk Assessment`](https://www.nist.gov/itl/applied-cybersecurity/privacy-engineering/collaboration-space/privacy-risk-assessment)
- [`USDOT Privacy Impact Assessments`](https://www.transportation.gov/individuals/privacy/privacy-impact-assessments)
- [`IPC Ontario De-Identification Guidelines`](https://www.ipc.on.ca/en/media/5946/download)

---

## Completion Checklist
- ✅ **(1) Privacy risk assessment method (quantified, repeatable):** see **“1) Privacy Risk Assessment (PIA)”**.
- ✅ **(2) DP operational guidance (where/when/how):** see **“2) Differential Privacy (DP) — Operational Guidance”**.
- ✅ **(3) Edge analytics scoping (output minimization + aggregation windows):** see **“3) Edge Analytics Scoping”**.
- ✅ **(4) Data governance lifecycle (vendor access, audits, leak response):** see **“4) Data Governance Lifecycle”** + runbooks.
- ✅ **(5) Privacy–performance tradeoffs mapping:** see **“5) Privacy–Performance Tradeoffs”** table.
- ✅ **(6) Minimum technical controls + example architectures:** see **“6) Minimum Technical Controls + Example Architectures”**.

---

Cross-links: Related ideas include city rules and explainable signals.
