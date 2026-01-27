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
- **Controller state access**: phase/interval states, detector inputs, logs (often via NTCIP objects) ([`NTCIP 1211 (Signal Control and Prioritization)`](https://www.ntcip.org/file/2018/11/NTCIP1211-v0224j.pdf)).
- **Health monitoring**: per-detector stuck-on/off, variance/flatline checks, cross-sensor consistency, heartbeat checks.
- **Fallback plan library**: pre-approved fixed-time + conservative actuated plans (and “all-red”/flash policies where applicable).
- **ATSPM/telemetry**: performance measures to detect anomalies and validate recovery ([`FHWA ATSPM (landing)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm)).
- **Audit logging**: append-only events with reason codes and evidence fields.

---

### 1) Safety Assurance & Verification Strategy (Controller + Supervisory)
Self-healing has to be engineered like a safety feature: the *controller* must guarantee the legal/safe sequencing invariants, while the *supervisory layer* can decide **when** to switch strategies and **how** to route alerts.

#### Boundary of responsibility (what must be guaranteed where)
- **Controller firmware / cabinet (safety-critical)**
  - Must enforce **conflict-free phase sequencing** and clearance behavior through controller logic and cabinet conflict monitoring (you do not rely on a cloud service to keep opposing greens from happening).
  - Must enforce minimum greens, pedestrian WALK/FDW and clearance rules configured in the controller database.
  - Must implement preemption behavior deterministically in the event supervisory comms are lost.
- **Supervisory layer (safety-supporting, not safety-critical)**
  - Computes health scores, recommends/commands mode changes (where permitted), and drives escalation workflows.
  - Maintains auditable logs, fleet-wide dashboards, and correlation across intersections.

This split is consistent with how ATSPM is positioned: ATSPM “does not solve problems by itself; technicians and engineers must interpret ATSPM reports and take action,” which is exactly the supervisory-layer role ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

#### Verification approaches
- **Configuration validation (static checks before field deploy)**
  - Verify phase compatibility/movement mapping is correct.
  - Verify pedestrian minimums/clearance intervals are present and unchanged across NORMAL and fallback plans.
  - Verify preemption inputs and priority logic are consistent across templates.
- **Transition safety checks (runtime rules the supervisory layer enforces)**
  - Never command a transition that would truncate clearance or pedestrian service in-progress.
  - Only allow plan/mode switches at *safe boundaries* (end of cycle, barrier, or defined transition points), depending on controller capability.
  - Enforce “safe denial”: if health evidence is insufficient or contradictory, switch to the most conservative allowed mode and alert ops.
- **Regression testing strategy**
  - Treat fallback templates as versioned artifacts; every change triggers automated replay of scenario tests and a sign-off workflow.

#### Practical testing artifacts
- **Failure-mode test cases** (minimum set)
  - Detector stuck-on (continuous call) → verify health detection and degradation policy.
  - Detector stuck-off (no actuation) → verify ped recall / min green guard remains safe.
  - Intermittent comms loss → verify dwell/hysteresis prevents thrash.
  - Controller clock drift / time jump → verify event ordering detection and conservative behavior.
  - Power sag / cabinet restart → verify mode selection and safe return.
- **Twin “failure drills”**
  - Inject failures while the corridor is oversaturated and confirm spillback does not cascade.
- **Field acceptance test (FAT) steps**
  - Run shadow mode first and confirm that (1) health scoring flags known injected faults and (2) no unsafe transitions are commanded.
  - Perform controlled drills (disconnect a detector, simulate comms loss) in off-peak windows and document outcomes.

---

### 2) Threat Model & Cyber-Resilient Self-Healing
A self-healing system can be attacked by manipulating the same signals it uses to protect itself. The implementation must explicitly consider malicious triggering and spoofing.

#### Lightweight threat model (what to assume can go wrong)
- **Spoofed detection** (false calls / false occupancy) to force split failures or create artificial congestion.
- **Replayed or forged telemetry** to hide faults or trigger fallbacks.
- **Comms MITM / credential compromise** on management links.
- **Time spoofing or time drift** to break correlation of events and create false alarms.

#### Defensive patterns (vendor-neutral)
- **Integrity and authentication**: require authenticated access for supervisory control and protect credentials; prefer network segmentation and controlled access paths in line with general hardening guidance ([`CISA Cybersecurity Best Practices`](https://www.cisa.gov/topics/cybersecurity-best-practices)).
- **Signed / validated telemetry** (where supported) and strict device identity mapping so one cabinet cannot impersonate another.
- **Rate limits + anomaly scoring**: throttle mode transition commands and require multiple independent health indicators before escalation.
- **Quorum checks / corroboration**: do not trust any single sensor; corroborate detector anomalies with phase termination patterns, probe travel times, and comms status.
- **Safe denial behaviors**: if you cannot trust inputs, prefer conservative operation (fixed-time or local isolated) and notify ops.

#### Preventing “fallback thrash attacks”
- **Hysteresis and minimum dwell**: once a mode changes, enforce a minimum time or minimum cycles before allowing another change.
- **Escalation ladder**: repeated oscillations automatically escalate to human review.
- **Manual lockout**: after N oscillations in a window, block auto-return and require acknowledgement.

For cyber incidents specifically, align response workflows with transportation-focused cyber incident response and management frameworks ([`Transportation Cybersecurity Incident Response & Management Framework (PDF)`](https://rosap.ntl.bts.gov/view/dot/57007/dot_57007_DS1.pdf), [`FHWA/USDOT Transportation Cybersecurity Resources (PDF)`](https://ops.fhwa.dot.gov/publications/fhwahop24101/fhwahop24101.pdf)).

---

### 3) Network / Corridor-Aware Degradation Patterns
A local fallback that is safe at one intersection can still be *operationally unsafe* at the corridor level if it causes spillback cascades.

#### Coordination-safe fallback selection
- Prefer fallbacks that keep coordination structure when possible (e.g., fixed-time plan with compatible cycle length and offsets) to preserve corridor progression.
- If coordination cannot be maintained safely, degrade to local actuated but add strict queue/spillback guards.

#### Anti-cascade patterns
- **Upstream metering**: temporarily reduce upstream release rates when downstream storage is at risk.
- **Max-green guards**: cap greens to prevent one movement from starving cross traffic during faults.
- **Queue threshold triggers**: if downstream occupancy indicates spillback, switch to spillback-protection behavior.
- **Corridor health aggregation**: if one node drops to FALLBACK, neighbors should adjust splits/offset assumptions and operators should see a corridor-level alert.

#### What the twin should simulate
- Queue spillback propagation, blocking of upstream intersections, and recovery time.
- Diversion effects if major-route progression breaks.

---

### 4) Observability Requirements (Telemetry, Time, Events, Data Quality)
Self-healing is only as good as its observability. ATSPM provides practical examples of how high-resolution logs can reveal detection failures and comms issues, and it defines specific “watchdog” conditions agencies can operationalize ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

#### Minimum telemetry set (baseline)
- **Signal state**: phase/interval state changes, phase termination reason (gap-out/max-out/force-off), coordination plan ID.
- **Detection**: raw detector actuations (or aggregated counts/occupancy), plus detector configuration metadata.
- **Pedestrian**: ped calls/actuations, WALK start, ped service (for delay computations).
- **Priority/preemption**: TSP and preemption events and status.
- **Comms**: heartbeat/poll success, latency, missed poll counts.
- **Cabinet/power** (if available): power status, UPS/battery status, cabinet door alarms.

#### Event schema recommendations
Create a structured event record:
- `event_id`, `correlation_id`, `device_id`, `intersection_id`
- `timestamp_utc`, `controller_local_time`, `time_quality`
- `severity` (INFO/WARN/CRITICAL)
- `reason_code` + `evidence` (metrics snapshot)
- `previous_mode` → `new_mode`
- `recommended_action` (ticket template)

#### Time sync guidance (practical)
- Use a consistent UTC timeline for all logs.
- Detect and alarm on time discontinuities (“time jumped backwards/forwards”) and treat “unknown time quality” as a health degradation.
- If time is untrusted, suppress fine-grained comparisons and fall back to conservative rules.

#### Data quality checks (reduce false alarms)
Operationalize checks similar to ATSPM’s watchdog patterns:
- “No data” / communications failure detection.
- High max-outs/force-offs in low-demand windows as detector fault indicators.
- Excessive pedestrian actuations in overnight windows as “stuck ped button” indicator.

ATSPM describes example watchdog thresholds (e.g., “no data” via record counts, max-outs/force-offs ratios, and stuck ped actuations during overnight hours) that agencies use as starting values ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

---

### 5) Governance, Ticketing, and Accountability (Beyond “Log Everything”)
Health events must turn into action. The default workflow should be: **event → alert → ticket → dispatch → resolution → postmortem**.

#### Recommended workflow
1. **Detect**: system generates an event with reason code and evidence.
2. **Notify**: route by severity to ops and/or maintenance.
3. **Ticket**: automatically open a maintenance ticket with device/likely root cause.
4. **Dispatch**: assign to field tech or IT/security depending on category.
5. **Resolve**: close ticket with fix applied and verification steps.
6. **Review**: weekly triage of top recurring faults; monthly reliability review.

#### Ownership model (example)
- **Signals ops/engineering**: mode policies, fallback plan library, operational acceptance.
- **Signal maintenance**: detectors, cabinet hardware, field repairs.
- **IT/network**: comms, time sync, access control.
- **Security**: incident response, suspected compromise handling.
- **Vendor**: controller firmware and integration constraints (within contract).

#### Retention and review cadence
- Retain health events long enough to support trend and root cause analysis, and align retention with cyber incident response needs ([`Transportation Cybersecurity Incident Response & Management Framework (PDF)`](https://rosap.ntl.bts.gov/view/dot/57007/dot_57007_DS1.pdf)).
- Weekly operations triage; monthly reliability report.

#### KPIs
- **MTTD** (mean time to detect)
- **MTTR** (mean time to repair/recover)
- **Time in FALLBACK/ISOLATED** (minutes/day)
- **False positive rate** and **mode flap rate**
- **Recurrence rate** of top fault categories

---

### 6) Exit Criteria, Operator Overrides, and Safety Lockouts
Automatic return-to-normal is often where risk concentrates. Define explicit exit criteria and when humans must confirm.

#### Auto-return criteria (examples)
- Comms stable (no missed polls) for X minutes.
- Detector health stable for Y cycles with no contradictions.
- No repeated oscillations in the last Z minutes.

#### Human confirmation triggers
- Repeated mode transitions (thrash risk).
- Any preemption failure or suspicious preemption behavior.
- Ped service alarms or suspected conflicting indications.

#### Lockout / isolation triggers
- Suspected unsafe operation.
- Untrusted time with conflicting telemetry.
- Suspected compromise of comms or credentials.

#### Operator UX requirements
- One-click **isolate** and one-click **revert**, each requiring a reason code.
- Timeline view of transitions and evidence snapshots.
- Suggested actions (repair detector, check comms, dispatch tech, contact security).

---

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

---

## Implementation Checklist
- Define controller vs supervisory responsibilities and document safety invariants that must never be violated.
- Build a failure-mode catalog tied to real maintenance history and explicit detection signals.
- Implement telemetry ingestion for phase states, detection, ped, preemption/TSP, comms heartbeat, and (where available) cabinet/power.
- Implement data-quality gates and watchdog-style checks to detect comms loss and detector anomalies ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).
- Implement FSM mode logic with hysteresis, dwell times, oscillation guard, and rate limiting.
- Create versioned fallback templates and validate them in a twin with injected failure drills.
- Define corridor-aware anti-cascade rules (spillback guards, upstream metering patterns).
- Define cyber-resilient posture (access controls, identity mapping, anomaly corroboration) and integrate with incident response ([`Transportation Cybersecurity Incident Response & Management Framework (PDF)`](https://rosap.ntl.bts.gov/view/dot/57007/dot_57007_DS1.pdf)).
- Define explicit exit criteria, auto-return rules, and lockout triggers.
- Deploy in shadow → assisted → auto phases; run controlled “game day” drills and capture after-action reviews.

## Operations Runbook (SOP)
### Severity definitions
- **CRITICAL**: suspected unsafe operation, conflicting indication suspicion, preemption malfunction, untrusted time with conflicting telemetry, suspected compromise.
- **HIGH**: comms loss > threshold, widespread detector faults, repeated mode oscillations.
- **MEDIUM**: single detector fault affecting actuation quality, intermittent comms errors.
- **LOW**: minor telemetry missingness, non-impacting sensor anomalies.

### Triage workflow (always)
1. Confirm current mode (NORMAL/DEGRADED/FALLBACK/ISOLATED) and recent transitions in the timeline.
2. Check comms heartbeat and last-good telemetry timestamp.
3. Check detector health summary (stuck-on/off, abnormal max-outs/force-offs, missing data).
4. Check ped service status and any stuck ped indicators.
5. Check preemption/TSP event logs.
6. Decide action: **revert**, **hold in fallback**, **isolate**, or **dispatch**.
7. Open/attach ticket with evidence snapshot and correlation ID.

### Playbooks by failure category
#### A) Communications loss
- Move to ISOLATED/local plan if comms are lost and the controller supports safe local operation.
- Notify IT/network, open ticket with last-known heartbeat time.
- Only auto-return after comms stable for the configured window and no oscillation history.

#### B) Detector failure suspected
- Confirm watchdog signals (overnight max-outs/force-offs, missing counts) and compare to field expectations ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).
- Switch to DEGRADED (reduced reliance) or FALLBACK (fixed-time) depending on severity.
- Dispatch maintenance for physical inspection and repair.

#### C) Time quality / clock drift
- If time is untrusted, suppress fine-grained analytics and move to conservative mode.
- Notify IT; verify time sync source.
- Require human confirmation before returning to NORMAL.

#### D) Suspected compromise / malicious manipulation
- Isolate the intersection (or segment) according to policy.
- Escalate to security incident response and preserve logs.
- Do not auto-return; require human sign-off after investigation.

---

## Reference Links
### Standards / interoperability
- [`NTCIP 1211 (Signal Control and Prioritization)`](https://www.ntcip.org/file/2018/11/NTCIP1211-v0224j.pdf)

### Operations / ATSPM guidance
- [`FHWA ATSPM (landing)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm)
- [`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)

### Cybersecurity / incident response
- [`CISA Cybersecurity Best Practices`](https://www.cisa.gov/topics/cybersecurity-best-practices)
- [`Transportation Cybersecurity Incident Response & Management Framework (PDF)`](https://rosap.ntl.bts.gov/view/dot/57007/dot_57007_DS1.pdf)
- [`FHWA/USDOT Transportation Cybersecurity Resources (PDF)`](https://ops.fhwa.dot.gov/publications/fhwahop24101/fhwahop24101.pdf)

---

Cross-links: Related ideas include what-if button, delay-tolerant signals, and explainable signals.
