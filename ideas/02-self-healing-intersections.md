# 02) Self-Healing Intersections: Fail-Safe Traffic Reliability

## Catchy Explanation
Imagine a traffic light that can notice it’s “going blind” (a detector dies, comms drop, a camera gets glare) and calmly switches to a safe, boring backup plan instead of causing chaos.

## What it is (precise)
A **self-healing intersection** is a signal controller + monitoring layer that continuously checks sensor, controller, and communications health and automatically transitions among modes (**NORMAL → DEGRADED → FALLBACK → ISOLATED**) when inputs become unreliable. The goal is **bounded-risk operation**: maintain legal and safety invariants (pedestrian minimums, clearance intervals, max cycle rules, preemption behavior) while degrading gracefully (fixed-time or conservative actuated plans) and generating auditable logs of *what changed* and *why*. Digital twins are used to rehearse failure modes (stuck detectors, clock drift, partial comms loss), validate fallbacks, and quantify corridor-level impacts (e.g., loss of progression).

## Benefits
- **Safer failure behavior**: fewer confusing or dangerous signal states during detector/comms faults.
- **Higher uptime**: reduces “dark signal” time and speeds recovery.
- **Faster maintenance**: health telemetry pinpoints likely fault locations.
- **Operator trust**: predictable, explainable fallbacks reduce panic overrides.

## Challenges
- **False positives/negatives**: overly sensitive checks create unnecessary fallbacks; weak checks miss real faults.
- **Mode flapping**: oscillation between NORMAL and FALLBACK without hysteresis.
- **Corridor coupling**: a single fallback intersection can break coordination and shift queues.
- **Testing burden**: must simulate a large catalog of failure modes.

## Implementation Strategies

### Infrastructure Needs
- **Controller state access**: phase/interval states, detector inputs, logs (often via NTCIP objects).
- **Health monitoring**: per-detector stuck-on/off, variance/flatline checks, cross-sensor consistency, heartbeat checks.
- **Fallback plan library**: pre-approved fixed-time + conservative actuated plans (and “all-red”/flash policies where applicable).
- **ATSPM/telemetry**: performance measures to detect anomalies and validate recovery.
- **Audit logging**: append-only events with reason codes and evidence fields.

### Detailed Implementation Plan
#### Phase 1: Requirements, Safety Invariants, and Failure Catalog (Weeks 1–6)
The agency should begin by cataloging the actual field assets at the pilot sites, because controller capabilities, detector types, and cabinet wiring determine what can be monitored and what can be changed automatically. The traffic engineering lead should then define non-negotiable safety and legal invariants, including pedestrian walk and clearance minimums, conflict monitor expectations, preemption and emergency vehicle priority behavior, and any jurisdictional “flash” policies. The team should build a failure-mode catalog that is grounded in real maintenance history (for example, stuck detectors, intermittent comms, time-sync drift, video occlusion, and cabinet power issues) and should explicitly label which failures are automatically detectable and which require human confirmation. Finally, the team should select a small set of pilot intersections that represent common patterns (simple 4-leg) and one or two difficult sites (heavy pedestrian demand, transit interaction, or complex geometry) so the solution does not only work in “easy mode.”

- **Roles**: traffic engineering lead (invariants and mode policy), signal maintenance supervisor (field failure history), IT/network lead (comms/power), safety/accessibility reviewer (ped constraints), data engineer (telemetry requirements).
- **Deliverables**: failure-mode catalog with indicators, mode transition policy draft, acceptance test plan, and baseline performance snapshot.
- **Risks**: the team may over-scope the failure catalog; flash and fallback policies may be politically or legally sensitive.
- **Acceptance checks**: the invariants are signed off, pilot sites are selected, and every failure mode has an agreed detection approach (automatic, manual, or “not supported”).

#### Phase 2: Telemetry, Health Metrics, and Data Quality Gates (Weeks 7–16)
The engineering team should implement reliable telemetry collection from controllers and detectors so the system can distinguish “traffic changed” from “the sensor broke.” The team should ingest phase and interval state, detector actuations, pedestrian calls, conflict monitor alarms, and any available cabinet power/UPS signals, and it should also compute communications health (heartbeat latency and missed polls). The data quality layer should produce explicit health metrics such as stuck-on/off detectors, flatline variance, chattering detection, occupancy plausibility bounds, and cross-sensor consistency checks, because these are the real-world failure patterns that create bad timing decisions. The operations team should receive dashboards and alerting that is severity-based and tuned to avoid alert fatigue, and the maintenance team should receive actionable tickets that point to specific detectors or comms links.

- **Roles**: data engineer (pipeline and storage), IT/network engineer (secure comms), signal technician (field QA), operations lead (alert routing), analyst (threshold tuning).
- **Deliverables**: telemetry pipeline, health-metric service, dashboards/alerts, and a detector QA report with prioritized fixes.
- **Risks**: thresholds may be mis-tuned, causing either alert fatigue or missed failures; controller object availability may vary by vendor.
- **Acceptance checks**: key telemetry is available with high completeness during peaks (for example, >95%), time sync is stable, and health metrics behave predictably on known-good detectors.

#### Phase 3: Mode Logic (Finite-State Machine) and Hysteresis (Weeks 17–24)
The software team should implement a finite-state machine that converts health signals into operational modes, because “self-healing” in practice is a controlled escalation process rather than a single switch. The state machine should use dwell times and hysteresis windows so the system does not bounce between NORMAL and FALLBACK during intermittent faults, and it should include an oscillation guard that enforces cooldown after repeated switches. The traffic engineer should define a capability matrix for each mode (for example, DEGRADED may allow actuated operation with reduced reliance on questionable detectors, while FALLBACK may use fixed-time plans), and the operations team should retain the ability to force a mode temporarily with a reason code. Every transition should create an audit event that records what changed, why it changed, and what evidence (health metrics) supported the decision.

- **Roles**: software engineer (FSM implementation), traffic engineer (policy and thresholds), QA/test engineer (scenario tests), operations lead (override workflow).
- **Deliverables**: FSM service, transition audit schema, operator override workflow, and scenario tests for common failures.
- **Risks**: mode policy may be too aggressive (unnecessary fallbacks) or too lax (unsafe reliance on broken detection).
- **Acceptance checks**: shadow testing shows low mode-flapping rate, manual override works and expires, and all transitions include reason codes and evidence.

#### Phase 4: Fallback Plan Library and Twin Validation (Weeks 25–34)
The traffic engineering team should create a small, conservative fallback plan set for each pilot site because “degraded” operation needs an approved landing zone that maintenance and operations trust. The team should design at least one conservative actuated plan (for partial detection), one fixed-time plan (for unreliable detection), and an isolated local plan (for comms loss), and it should ensure that each plan preserves pedestrian minimum service and clearance behavior. The modeling team should then validate these fallback plans in a digital twin by injecting realistic failures (detector stuck, comms loss, heavy pedestrian demand) and by measuring how queues shift to adjacent intersections, because corridor coupling is where naive fallback strategies often fail. Finally, the team should document when to escalate from DEGRADED to FALLBACK or ISOLATED, and it should define a safe recovery process back to NORMAL that requires sustained health.

- **Roles**: traffic engineer (plan design), modeler (twin scenarios), safety/accessibility reviewer (ped checks), operations lead (approval).
- **Deliverables**: versioned fallback plan library with activation criteria, twin validation report, and corridor impact analysis.
- **Risks**: fallback plans may preserve safety but create unacceptable congestion; fixed-time plans may not serve pedestrians well if volumes shift.
- **Acceptance checks**: simulations show invariants always satisfied, corridor impacts are understood, and each fallback plan has a clear activation and rollback/recovery procedure.

#### Phase 5: Pilot Deployment (Shadow → Assisted → Auto) (Weeks 35–46)
The agency should deploy self-healing in a phased approach because automatic switching is operationally sensitive and requires trust. In shadow mode, the system should compute health scores and recommended mode transitions without actuating the controller so the team can measure false positives and tune thresholds safely. In assisted mode, operators should confirm transitions so the team can validate that recommendations align with field reality and maintenance observations. After thresholds and procedures stabilize, the system can enable auto-switching with strict rate limits and an upper bound on time spent in fallback before escalation or human review. The team should also run planned “game day” drills (for example, temporarily disconnecting a detector) to validate that playbooks, alerts, and recovery procedures work under controlled conditions.

- **Roles**: operations (day-to-day monitoring), field technicians (drills and fixes), engineer on-call (tuning), analyst (evaluation).
- **Deliverables**: pilot evaluation report (MTTD/MTTR, false positive rate, flap rate), updated thresholds, operator and maintenance runbook.
- **Risks**: drills may disrupt traffic if poorly timed; operators may distrust the system if alerts are noisy.
- **Acceptance checks**: detection-to-recovery time improves versus baseline, false positives are within target, operators can revert safely, and drill outcomes are documented and reviewed.

#### Phase 6: Scaling and Continuous Improvement (Ongoing)
The program should scale intersection-by-intersection and then corridor-by-corridor, because each controller platform and detection configuration has unique failure signatures. The engineering team should continuously incorporate new failure modes observed in production and should refresh fallback plans as geometry and demand change due to construction and development. The agency should schedule quarterly threshold reviews, periodic detector QA audits, and annual recertification of safety invariants and flash policies so the system remains defensible. Over time, the team should measure program-level outcomes such as reduced downtime, faster maintenance response, and fewer operational surprises.

- **Roles**: program owner (governance), operations (monitoring), engineering (improvements), maintenance (field fixes).
- **Deliverables**: network rollout roadmap, quarterly health report, updated plan library versions, and annual compliance/audit package.
- **Risks**: equipment aging and firmware changes may break assumptions; scaling may increase alert load without process improvements.
- **Acceptance checks**: audit logs are complete, coverage expands without increased incident rate, and MTTR improvements remain sustained.

### Choices
- **Simple**: comms heartbeat + fixed-time fallback.
- **Moderate**: detector plausibility + conservative actuated fallback.
- **Advanced**: multi-sensor fusion + graded degradation + corridor-aware coordination.

## Technical Mechanics

### Key Parameters
- Health score thresholds (NORMAL/DEGRADED/FALLBACK)
- Hysteresis windows and minimum dwell times
- Recovery criteria
- Safety invariants and violation detectors

### Guardrails
- Never violate pedestrian minimum service/clearance logic.
- Rate-limit mode changes.
- Always provide manual override + safe rollback.
- Emit reason codes + evidence per transition.

## MVP Deployment
- 2–4 intersections.
- 3 health checks (heartbeat, stuck detector, plausibility).
- 1 fallback plan per site.
- Shadow → assisted → auto progression.

## Evaluation
- Mean time to detect and recover from faults.
- False-positive rate and mode-flap rate.
- ATSPM impacts: split failures, arrivals-on-green changes.
- Safety compliance: ped service, clearance integrity.

## References / Standards / Useful Sources
- FHWA Automated Traffic Signal Performance Measures (ATSPM): https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm
- NTCIP 1211 (Signal Control and Prioritization): https://www.ntcip.org/file/2018/11/NTCIP1211-v0224j.pdf
- FHWA Traffic Signal Timing Manual (chapters on timing fundamentals and operational constraints): https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf

---

Cross-links: Related ideas include what-if button, delay-tolerant signals, and explainable signals.
