# 13) Red-Team the Signals: Adversarial Testing for Resilience

## Catchy explanation

Before real attackers find the weak spots, you hire your own “attackers” and break the system in a simulator and test lab—so the real intersection stays safe.

## What it is (precise)

**Red-teaming the signals** is a continuous adversarial testing program for connected traffic signal ecosystems (controllers, detectors, comms, central software, and supporting OT/IT). It blends:

- **Threat modeling + scoped authorization** (what is allowed, where, and when)
- **Repeatable fault/attack injection** (sensor streams, network conditions, time sync, configs)
- **Twin + lab validation** (safety-preserving realism)
- **Operationally meaningful scoring** (queues, split failures, pedestrian service, preemption behavior)
- **Mitigation validation** (detection, response modes, recovery, change control)

The outputs are production-ready artifacts: a versioned scenario library, tuning targets and stop conditions, detection rules and dashboards, playbooks for operations and incident response, and procurement/acceptance requirements.

---

## 1) Formal Scope, Threat Model, and Worst-Outcomes Taxonomy

### 1.1 Scope (systems in and out)

**In scope (typical):**
- Field controllers (NEMA/ATC/UTCS etc.) and local timing plans
- Detection: loops, radar, video analytics, ped buttons, probe/AVL feeds
- Field cabinets and communications (fiber, copper, wireless, VPNs)
- Central ATMS/TMC software, data brokers, and APIs
- Performance analytics systems (ATSPM, dashboards)
- Digital twins and simulation environments used for testing
- Vendor-hosted cloud services that influence signal operations

**Out of scope (unless explicitly approved):**
- Production police/fire CAD systems (beyond simulated interfaces)
- City-wide IT infrastructure outside the signal network
- Non-transport safety-critical systems not integrated with signals

Red-team exercises against systems “out of scope” require separate governance and approvals.

### 1.2 Threat model (attacker types and capabilities)

**Adversary classes:**
- **External attacker**: internet-based or on-street adversary probing exposed services or wireless links.
- **Insider (malicious or careless)**: staff/contractor with legitimate access misusing credentials or exporting data.
- **Compromised vendor/subprocessor**: vendor-side breach giving access to management portals or data stores.
- **Opportunistic attacker**: e.g., exploiting exposed/default devices on field cabinets.

**Capabilities considered:**
- Network scanning, MITM, replay, and injection on field or backhaul networks
- Configuration tampering (timing plans, preemption rules)
- Data tampering (sensor spoofing, bogus priority requests, time spoofing)
- Denial of service (DoS) against central/edge components

### 1.3 Worst-outcomes taxonomy (what we explicitly want to prevent)

Categorize worst outcomes to prioritize testing and mitigations:

- **Category A — Safety-critical traffic outcomes**
  - Conflicting greens / loss of safe phasing
  - Systematic failure of pedestrian protections (no WALK, no clearance)
  - Incorrect or disabled emergency vehicle preemption
  - Persistent queue spillback blocking crosswalks or intersections

- **Category B — Operational disruption**
  - Large-scale corridor or network freeze, flash, or miscoordination
  - Inability to return from degraded/fallback modes
  - Loss of visibility (operators blind to field states)

- **Category C — Data and privacy compromise**
  - Unauthorized access to sensitive telemetry or logs
  - Exfiltration of more-than-aggregate data from signal analytics

- **Category D — Integrity of governance and trust**
  - Undetected tampering with logs, audit trails, or alarms
  - Misleading or suppressed alerts during incidents

Red-team scenarios should always map to one or more categories and define **success criteria** in terms of: prevented Category A, reduced probability/impact of B–D, improved detection and response metrics.

---

## 2) Validation Ladder: From Tabletop to Field (Required Visual: Ladder Table)

Red-teaming must progress in **stages**, with strict guardrails. Treat this as a ladder; you don’t jump straight to live field experiments.

### 2.1 Attack/failure injection validation ladder (tabletop → sim → HIL → field)

| Stage | Environment | Examples | Allowed attack/failure types | Exit criteria |
|---|---|---|---|---|
| **Tabletop** | Conference room / whiteboard / documents | Walk through "what if" scenarios; incident reviews | Hypothetical attacks, process failures, social/operational gaps | Scenario card defined; mitigations and detection hypotheses captured; no live systems touched |
| **Simulation (Twin-only)** | Digital twin; no real hardware | Simulate detector spoofing, comms delays, corrupted messages | Synthetic data/attack patterns into twin; no real controllers | Safety invariants always maintained in sim; Category A outcomes ruled out under modeled defenses; metrics baselined |
| **HIL (Hardware-in-the-loop)** | Lab controllers + twin / test cabinet | Real controller configs; network impairments; mis-timed plans | Inject faults/attacks on lab networks; config tampering on non-production hardware | Controllers recover to safe modes; logs/audit entries complete; detection thresholds tuned; no impact on field devices |
| **Controlled Field** | Limited real intersections; off-peak; with approvals | Simulated comms degradations; benign fault injections with strict bounds | Very constrained exercises (e.g., disabling central influence; increasing latency), under SOP + stop conditions | No safety regressions; Category A outcomes remain impossible; on-street KPIs within pre-approved bounds; debrief + sign-off |

**Rule:** no scenario moves to a higher rung without documented success and safety review on the previous rung.

---

## 3) Red-Team Lifecycle Diagram (Required Visual: discover → simulate → mitigate → retest → deploy)

### 3.1 Lifecycle overview

```text
[Discover]
  - Threat modeling
  - Vulnerability reviews
  - Incident postmortems
        │
        ▼
[Design Scenario]
  - Define scope and objectives
  - Map to worst-outcome categories
  - Choose ladder stage (tabletop/sim/HIL/field)
  - Define safety boundaries + stop conditions
        │
        ▼
[Simulate / Inject]
  - Run scenario at chosen ladder stage
  - Collect logs, KPIs, operator responses
        │
        ▼
[Analyze & Score]
  - Classify findings (vuln, gap, false alarm, etc.)
  - Quantify impact on safety & operations
        │
        ▼
[Mitigate]
  - Design and implement fixes:
    * configs, code, procedures, training, vendor changes
        │
        ▼
[Retest]
  - Re-run scenario at same or higher ladder rung
  - Confirm mitigation effectiveness
        │
        ▼
[Institutionalize & Deploy]
  - Update SOPs, detection rules, dashboards
  - Update procurement/acceptance criteria
  - Feed lessons into new scenario design
        └───────────────────────────────▶ (back to [Discover])
```

Use this lifecycle as the **governance backbone**: every scenario must be somewhere on this loop, and no mitigation is “done” until re-tested.

---

## 4) Scenario Card Template + Sample Scenarios (Required Visual)

### 4.1 Scenario card template (what every red-team scenario must specify)

Each scenario is a structured card; this becomes a versioned library.

```markdown
# Scenario Card — RT-XXX

## A. Metadata
- Scenario ID:
- Title:
- Category (A/B/C/D from worst-outcomes taxonomy):
- Owner:
- Status: [Proposed / In design / Active / Retired]

## B. Scope
- Systems in scope:
- Environments allowed: [Tabletop / Sim / HIL / Field]
- Preconditions (plans active, time-of-day, events):

## C. Attack / Failure Description
- Type (e.g., detector spoofing, comms delay, config tampering):
- Steps (high level):
- Tooling required:

## D. Safety Boundaries and Stop Conditions
- Prohibited actions:
- Hard stop conditions (what immediately halts test):
- Required monitoring (KPIs, alarms):

## E. Success Criteria
- Detection metrics:
- Response metrics:
- Residual risk notes:

## F. Mitigation & Closure
- Mitigation actions:
- Owners and target dates:
- Retest plan and date:
- Closure evidence (link to logs/reports):
```

### 4.2 Scenario card table with sample scenarios

| Scenario ID | Type | Ladder stage(s) | Brief description | Worst-outcome category | Stop conditions (summary) | Primary owner |
|---|---|---|---|---|---|---|
| RT-COMMS-01 | Comms delay / loss | Sim → HIL → limited field | Inject increasing latency and drop rates between central and field controllers to test delay-tolerant modes | B (operational disruption), A (safety if mis-handled) | Any unexpected flash; ped min violations; preemption misbehavior; unplanned loss of visibility | Signals engineering + IT/network |
| RT-DETECT-01 | Detector spoofing / stuck | Sim → HIL | Feed unrealistic occupancy/volume to mimic stuck-on or spoofed detector and see if plausibility checks and delay-tolerant modes trigger | A (safety via RLR/spillback), B | Any unbounded queue spillback; crosswalk blockages; uncontrolled phase max-outs | Signals engineering + maintenance |
| RT-TSP-ABUSE-01 | Priority abuse (transit/fleet) | Sim | Send high-frequency TSP/priority requests that meet nominal rules but create unfair harm | B, D (trust) | Cross-street delay caps exceeded; repeated starvation of protected movements | Signals + transit ops |
| RT-CONFIG-01 | Config tampering | Tabletop → HIL | Attempt unauthorized config change in lab (timing plans, preemption order) to test change-control and detection | C, D | Any unlogged config change; bypass of approval workflow | Security + signals engineering |
| RT-VENDOR-01 | Vendor backdoor / over-privileged access | Tabletop → controlled vendor test | Simulate vendor support account misuse or credential leak to test access controls and audit logs | C, D | Vendor access without ticket; unlogged exports; failure to notify | Security + vendor management |

New scenarios should follow this pattern and be added only after scope, boundaries, and owners are defined.

---

## 5) Mitigation Ownership, Closure Workflow, and RACI

### 5.1 Mitigation and closure workflow

1. **Finding captured**
   - Scenario run produces findings (vulnerability, missing detection, unsafe behavior, process gap).
   - Classify severity and category (A/B/C/D) and environment (tabletop/sim/HIL/field).

2. **Ticket & owner assignment**
   - Open a tracked ticket (or issue) per distinct finding.
   - Assign **clear owners** (signals, IT, security, vendor) and due dates.

3. **Mitigation design & implementation**
   - Define technical, procedural, and governance changes (configs, code, SOPs, training, contract updates).

4. **Retest**
   - Re-run the scenario (or an equivalent) at the appropriate ladder rung.
   - Document that the worst-outcome category is mitigated or risk reduced and accepted.

5. **Closure**
   - Mark ticket closed only when: mitigation deployed, retest passed, documentation updated.
   - Archive evidence: logs, configs, sign-offs.

### 5.2 RACI for red-team program

| Activity | Program Owner | Signals Ops | Traffic Engineering | IT/Network | Security | Vendors | Legal/Privacy | Leadership |
|---|---|---|---|---|---|---|---|---|
| Define scope & threat model | R | C | C | C | C | C | C | A |
| Approve scenario cards | R | C | C | C | C | C | I | A |
| Run tabletop exercises | R | R | C | C | C | I | I | I |
| Run sim/HIL tests | R | C | R | C | C | C | I | I |
| Approve field tests | R | R | R | C | C | C | C | A |
| Implement mitigations | C | C | R | R | R | R | C | I |
| Maintain logs & evidence | R | C | C | C | R | C | C | I |
| Publish leadership scorecard | R | C | C | C | C | I | C | A |

R = Responsible, A = Accountable, C = Consulted, I = Informed.

---

## 6) Responsible Disclosure and Sensitivity Controls

### 6.1 Internal vs external red-teaming

- **Internal red team / blue team**: operates under agency governance, with strict logging and approvals.
- **External testers (consultants, researchers)**:
  - Must operate under a contract / engagement letter.
  - Scope, ladders, and stop conditions must be documented.

### 6.2 Responsible disclosure policy

- Provide a clear, public channel for vulnerability reporting (e.g., security email, web form).
- Commit to **safe harbor** for good-faith researchers within published rules.
- Define timelines for triage and remediation.

### 6.3 Sensitivity, classification, and sharing controls

- Scenario details that could materially aid attackers should be classified (e.g., internal-only).
- Public-facing materials may reference scenario classes and mitigations, but not step-by-step exploits.
- Logs and artifacts must be handled under your security classification policy.

### 6.4 Integration with incident response

- Red-team findings must feed into existing incident response and cybersecurity practices (e.g., aligned with CISA ICS guidance).
- Ensure playbooks cover both **simulated** and **real** incidents consistently.

---

## 7) Safety-Critical Realism Boundaries and Stop Conditions

### 7.1 Safety realism boundaries

- **Never** intentionally generate conflicting greens or known unsafe phase sequences in the field.
- **Never** disable or bypass pedestrian protections or emergency preemption in live traffic purely for testing.
- Safety-focused behaviors (e.g., delay-tolerant fallback, self-healing fail-safe modes) are **in scope** for red-teaming, but only via simulations/HIL or highly constrained field tests with robust observation.

### 7.2 Stop conditions (hard abort rules)

Define **immediate stop** conditions for any field or HIL test:

- Any indication of ped minimum WALK/clearance not being served per policy.
- Any observed or reported conflicting greens.
- Unexpected transition to flash or dark at an unapproved site.
- Queue spillback into rail crossings, critical crosswalks, or high-risk junctions.
- Unexpected loss of comms to a cluster of intersections.
- Operator or field staff reports unsafe behavior.

Stop conditions must be encoded in procedures and, where feasible, automated via watchdogs.

---

## 8) Metrics, Cadence, and Leadership Scorecard

### 8.1 Program metrics

Track metrics across **scenarios**, **systems**, and **time**:

- **Coverage metrics**
  - # of active scenarios by category (A/B/C/D).
  - % of priority systems covered by at least one scenario.

- **Detection & response metrics**
  - % of simulated attacks detected at each ladder stage.
  - Time-to-detect and time-to-mitigate (per scenario).

- **Quality of mitigations**
  - % of mitigations passing retest on first attempt.
  - # of repeated findings (regressions) per quarter.

- **Safety & stability**
  - # of field tests halted by stop conditions (and why).
  - No Category A outcome ever induced in field tests.

- **Governance health**
  - % of scenarios with current owners and status.
  - Age of open high-severity findings.

### 8.2 Cadence

- **Monthly**: red-team review
  - New scenario proposals
  - Review findings and mitigations
  - Approve next month’s test plan

- **Quarterly**: leadership briefing
  - Scorecard of metrics above
  - Trends in resilience and safety
  - Budget/roadmap alignment

- **Annually**: program audit
  - External/internal review of scope, controls, evidence, and outcomes
  - Update threat model and ladder policies

### 8.3 Leadership scorecard (example)

A high-level scorecard for leadership might include:

- % of critical corridors covered by at least 1 active scenario
- % of simulated critical-category attacks detected and mitigated within target times
- # open high-severity findings older than N days
- # field tests halted by safety stop conditions (and corrective actions completed)
- Trend of ATSPM/anomaly indicators associated with issues found during red-teaming

This turns red-teaming into a **managed, reportable risk-reduction program** rather than an ad-hoc test.

---

## MVP Deployment

- 3–5 initial scenarios (e.g., comms delay, detector spoofing, priority abuse).
- Simulation and HIL only (no field tests in MVP).
- One corridor or control cluster as primary focus.

### Evaluation

- # of significant findings per quarter.
- Time from finding to mitigated+retested.
- Improvement in detection/response metrics between cycles.
- No safety incidents induced by tests.

---

## Implementation Checklist

- [ ] Define scope (systems, environments) and threat model.
- [ ] Define worst-outcomes taxonomy (A/B/C/D) and map to KPIs.
- [ ] Establish validation ladder (tabletop → sim → HIL → field) with guardrails.
- [ ] Design scenario card template and seed scenario library with owners.
- [ ] Define mitigation and closure workflow; integrate with ticketing.
- [ ] Define RACI for program governance.
- [ ] Establish responsible disclosure policy and sensitivity controls.
- [ ] Define safety realism boundaries and hard stop conditions.
- [ ] Define metrics, cadence, and leadership scorecard.
- [ ] Run initial tabletop + sim/HIL scenarios; produce first report.

---

## Operations Runbook (SOP)

### SOP 0 — Running a scenario (any stage)

1. Confirm scenario card is approved and in “Active” status.
2. Verify scope, stage (tabletop/sim/HIL/field), and safety boundaries.
3. Ensure monitoring/telemetry and logging are configured.
4. Execute scenario steps as specified.
5. Monitor for stop conditions; abort if any are met.
6. Collect logs and KPIs.
7. Debrief with participants.
8. File findings, tickets, and mitigation actions.

### SOP 1 — Approving a new scenario

1. Review scope and threat model alignment.
2. Confirm mapping to worst-outcomes categories.
3. Validate safety boundaries and stop conditions.
4. Assign owners (scenario owner + mitigation owners).
5. Approve ladder stage(s) and schedule.

### SOP 2 — Escalation

- If a test uncovers a safety-critical behavior (even in sim/HIL), escalate to safety and engineering leadership.
- Suspend related scenarios until mitigations are defined.

---

## Governance & Change-Control Runbook

### Ownership

- **Program owner** (e.g., Signals Resilience Lead): accountable for red-team scope and outcomes.
- **Scenario owners**: accountable for specific scenario design and maintenance.
- **Mitigation owners**: accountable for fixes and retests.

### Change control

- Version scenario cards, ladders, and policies (SemVer-style or equivalent).
- Any change to scopes, safety boundaries, or allowed field tests must be approved by program owner + safety lead.
- Keep an audit trail of scenarios run, findings, and mitigations.

### Evidence management

- Store logs, reports, and approvals in a structured repository.
- Ensure retention and access are consistent with security and privacy policies.

---

## Reference Links

- CISA ICS Cybersecurity best practices (for OT/ICS hardening and incident response)
- FHWA ATSPM resources for anomaly detection and watchdog design
- NIST Cybersecurity Framework and NIST Privacy Framework (governance vocabulary)

(These can be expanded with concrete URLs in [`ideas/sources.md`](ideas/sources.md).)

---

## Completion Checklist

- ✅ Formal scope, threat model, and worst-outcomes taxonomy: see **Section 1**.
- ✅ Validation ladder (tabletop → sim → HIL → field) with table: see **Section 2**.
- ✅ Red-team lifecycle diagram (discover → simulate → mitigate → retest → deploy): see **Section 3**.
- ✅ Scenario card template + sample scenarios table: see **Section 4**.
- ✅ Mitigation ownership, closure workflow, and RACI: see **Section 5**.
- ✅ Responsible disclosure and sensitivity controls: see **Section 6**.
- ✅ Safety-critical realism boundaries and stop conditions: see **Section 7**.
- ✅ Metrics, cadence, and leadership scorecard: see **Section 8**.
- ✅ End sections provided: **MVP Deployment**, **Evaluation**, **Implementation Checklist**, **Operations Runbook (SOP)**, **Governance & Change-Control Runbook**, **Reference Links**, **Completion Checklist**.
