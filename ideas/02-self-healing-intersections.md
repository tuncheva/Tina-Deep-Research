# 02) Self-Healing Intersections: Fail-Safe Traffic Reliability

## Catchy Explanation
Imagine a traffic light that can notice it’s “going blind” (a detector dies, comms drop, a camera gets glare) and calmly switches to a safe, boring backup plan instead of causing chaos.

## What it is (precise)
A **self-healing intersection** is a signal controller + monitoring layer that continuously checks sensor, controller, and communications health and automatically transitions among modes (**NORMAL → DEGRADED → FALLBACK → ISOLATED**) when inputs become unreliable. The goal is **bounded-risk operation**: maintain legal and safety invariants (pedestrian minimums, clearance intervals, max cycle rules, preemption behavior) while degrading gracefully (fixed-time or conservative actuated plans) and generating auditable logs of *what changed* and *why*. Digital twins are used to rehearse failure modes (stuck detectors, clock drift, partial comms loss), validate fallbacks, and quantify corridor-level impacts (e.g., loss of progression).

At implementation time, this becomes a **reliability layer** with three core behaviors:
- **Detection** of health faults across sensors, controller, comms, and time.
- **Graded degradation** through an explicit state machine (NORMAL / DEGRADED / FALLBACK / ISOLATED) with hysteresis and corridor awareness.
- **Safe recovery** with exit criteria, operator overrides/lockouts, and full observability.

---

## 1) Safety Assurance & Verification Strategy (Controller + Supervisory)

Self-healing has to be engineered like a safety feature: the *controller* must guarantee the legal/safe sequencing invariants, while the *supervisory layer* can decide **when** to switch strategies and **how** to route alerts.

### 1.1 Boundary of responsibility (what must be guaranteed where)
- **Controller firmware / cabinet (safety-critical)**
  - Must enforce **conflict-free phase sequencing** and clearance behavior through controller logic and cabinet conflict monitoring (you do not rely on a cloud service to keep opposing greens from happening).
  - Must enforce minimum greens, pedestrian WALK/FDW and clearance rules configured in the controller database.
  - Must implement preemption behavior deterministically in the event supervisory comms are lost.
  - Must *not* accept external commands that violate internal safety checks (e.g., commands that would truncate clearance or force conflicting greens).
- **Supervisory layer (safety-supporting, not safety-critical)**
  - Computes health scores, recommends/commands mode changes (where permitted), and drives escalation workflows.
  - Maintains auditable logs, fleet-wide dashboards, and correlation across intersections.
  - Applies policy: graded degradation, corridor-aware adjustments, cyber safeguards, and governance (who is notified, when, and how).

This split is consistent with how ATSPM is positioned: ATSPM “does not solve problems by itself; technicians and engineers must interpret ATSPM reports and take action,” which is exactly the supervisory-layer role ([FHWA ATSPM Use Cases (Ch.4)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

### 1.2 Verification and testing approach

**Configuration validation (static checks before field deploy)**
- Verify phase compatibility/movement mapping is correct.
- Verify pedestrian minimums/clearance intervals are present and unchanged across NORMAL and fallback plans.
- Verify preemption inputs and priority logic are consistent across templates.
- Verify controller limits on minimum greens, red clearance, and conflict monitoring are enabled and unchanged by supervisory configuration.

**Transition safety checks (runtime rules the supervisory layer enforces)**
- Never command a transition that would truncate clearance or pedestrian service in-progress.
- Only allow plan/mode switches at *safe boundaries* (end of cycle, barrier, or defined transition points), depending on controller capability.
- Enforce **“safe denial”**: if health evidence is insufficient or contradictory, switch to the most conservative allowed mode and alert ops.
- Enforce rate limiting and dwell times for mode transitions.

**Regression testing strategy**
- Treat fallback templates as versioned artifacts; every change triggers automated replay of scenario tests and a sign-off workflow.
- Maintain a catalog of failure scenarios and assert that expected mode transitions, guardrails, and operator messages still hold for new software or plan versions.

### 1.3 Practical testing artifacts
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

## 2) Threat Model & Cyber-Resilient Self-Healing

A self-healing system can be attacked by manipulating the same signals it uses to protect itself. The implementation must explicitly consider malicious triggering and spoofing.

### 2.1 Lightweight threat model (what to assume can go wrong)
- **Spoofed detection** (false calls / false occupancy) to force split failures or create artificial congestion.
- **Replayed or forged telemetry** to hide faults or trigger fallbacks.
- **Comms MITM / credential compromise** on management links.
- **Time spoofing or time drift** to break correlation of events and create false alarms.

### 2.2 Defensive patterns (vendor-neutral)
- **Integrity and authentication**: require authenticated access for supervisory control and protect credentials; prefer network segmentation and controlled access paths in line with general hardening guidance ([CISA Cybersecurity Best Practices](https://www.cisa.gov/topics/cybersecurity-best-practices)).
- **Signed / validated telemetry** (where supported) and strict device identity mapping so one cabinet cannot impersonate another.
- **Rate limits + anomaly scoring**: throttle mode transition commands and require multiple independent health indicators before escalation.
- **Quorum checks / corroboration**: do not trust any single sensor; corroborate detector anomalies with phase termination patterns, probe travel times, and comms status.
- **Safe denial behaviors**: if you cannot trust inputs, prefer conservative operation (fixed-time or local isolated) and notify ops.

### 2.3 Preventing “fallback thrash attacks” (anti-thrash policies)
- **Hysteresis and minimum dwell**: once a mode changes, enforce a minimum time or minimum cycles before allowing another change.
- **Escalation ladder**: repeated oscillations automatically escalate to human review.
- **Manual lockout**: after N oscillations in a window, block auto-return and require acknowledgement.
- **Per-asset rate limits**: cap the number of automatic transitions per intersection per hour.

For cyber incidents specifically, align response workflows with transportation-focused cyber incident response and management frameworks ([Transportation Cybersecurity Incident Response & Management Framework (PDF)](https://rosap.ntl.bts.gov/view/dot/57007/dot_57007_DS1.pdf), [FHWA/USDOT Transportation Cybersecurity Resources (PDF)](https://ops.fhwa.dot.gov/publications/fhwahop24101/fhwahop24101.pdf)).

---

## 3) Network / Corridor-Aware Degradation Patterns

A local fallback that is safe at one intersection can still be *operationally unsafe* at the corridor level if it causes spillback cascades.

### 3.1 Coordination-safe fallback selection
- Prefer fallbacks that keep coordination structure when possible (e.g., fixed-time plan with compatible cycle length and offsets) to preserve corridor progression.
- If coordination cannot be maintained safely, degrade to local actuated but add strict queue/spillback guards.
- Mark intersections that are **bottlenecks** or **critical to progression**; treat their fallbacks as higher risk and escalate to corridor-level review when they degrade.

### 3.2 Anti-cascade patterns
- **Upstream metering**: temporarily reduce upstream release rates when downstream storage is at risk.
- **Max-green guards**: cap greens to prevent one movement from starving cross traffic during faults.
- **Queue threshold triggers**: if downstream occupancy indicates spillback, switch to spillback-protection behavior.
- **Corridor health aggregation**: if one node drops to FALLBACK, neighbors should adjust splits/offset assumptions and operators should see a corridor-level alert.

### 3.3 What the twin should simulate
- Queue spillback propagation, blocking of upstream intersections, and recovery time.
- Diversion effects if major-route progression breaks.
- Lockout scenarios where repeated oscillation at one node changes upstream/downstream mode patterns.

---

## 4) Observability Requirements (Telemetry, Time, Events, Data Quality)

Self-healing is only as good as its observability. ATSPM provides practical examples of how high-resolution logs can reveal detection failures and comms issues, and it defines specific “watchdog” conditions agencies can operationalize ([FHWA ATSPM Use Cases (Ch.4)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

### 4.1 Minimum telemetry set (baseline)
- **Signal state**: phase/interval state changes, phase termination reason (gap-out/max-out/force-off), coordination plan ID.
- **Detection**: raw detector actuations (or aggregated counts/occupancy), plus detector configuration metadata.
- **Pedestrian**: ped calls/actuations, WALK start, ped service (for delay computations).
- **Priority/preemption**: TSP and preemption events and status.
- **Comms**: heartbeat/poll success, latency, missed poll counts.
- **Cabinet/power** (if available): power status, UPS/battery status, cabinet door alarms.

### 4.2 Event schema recommendations
Create a structured event record:
- `event_id`, `correlation_id`, `device_id`, `intersection_id`
- `timestamp_utc`, `controller_local_time`, `time_quality`
- `severity` (INFO/WARN/CRITICAL)
- `reason_code` + `evidence` (metrics snapshot)
- `previous_mode` → `new_mode`
- `recommended_action` (ticket template)

### 4.3 Time sync guidance (practical)
- Use a consistent UTC timeline for all logs.
- Detect and alarm on time discontinuities (“time jumped backwards/forwards”) and treat “unknown time quality” as a health degradation.
- If time is untrusted, suppress fine-grained comparisons and fall back to conservative rules.

### 4.4 Data quality checks (reduce false alarms)
Operationalize checks similar to ATSPM’s watchdog patterns:
- “No data” / communications failure detection.
- High max-outs/force-offs in low-demand windows as detector fault indicators.
- Excessive pedestrian actuations in overnight windows as “stuck ped button” indicator.

ATSPM describes example watchdog thresholds (e.g., “no data” via record counts, max-outs/force-offs ratios, and stuck ped actuations during overnight hours) that agencies use as starting values ([FHWA ATSPM Use Cases (Ch.4)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

---

## 5) Governance, Ticketing, and Accountability (Beyond “Log Everything”)

Health events must turn into action. The default workflow should be: **event → alert → ticket → dispatch → resolution → postmortem**.

### 5.1 Recommended workflow
1. **Detect**: system generates an event with reason code and evidence.
2. **Notify**: route by severity to ops and/or maintenance.
3. **Ticket**: automatically open a maintenance ticket with device/likely root cause.
4. **Dispatch**: assign to field tech or IT/security depending on category.
5. **Resolve**: close ticket with fix applied and verification steps.
6. **Review**: weekly triage of top recurring faults; monthly reliability review.

### 5.2 Ownership model (example)
- **Signals ops/engineering**: mode policies, fallback plan library, operational acceptance.
- **Signal maintenance**: detectors, cabinet hardware, field repairs.
- **IT/network**: comms, time sync, access control.
- **Security**: incident response, suspected compromise handling.
- **Vendor**: controller firmware and integration constraints (within contract).

### 5.3 Retention and review cadence
- Retain health events long enough to support trend and root cause analysis, and align retention with cyber incident response needs ([Transportation Cybersecurity Incident Response & Management Framework (PDF)](https://rosap.ntl.bts.gov/view/dot/57007/dot_57007_DS1.pdf)).
- Weekly operations triage; monthly reliability and cyber-related incident report; annual audit bundle.

### 5.4 KPIs
- **MTTD** (mean time to detect).
- **MTTR** (mean time to repair/recover).
- **Time in DEGRADED/FALLBACK/ISOLATED** (minutes/day and % of operating time).
- **False positive rate** and **mode flap rate**.
- **Recurrence rate** of top fault categories.
- **Ticket closure time** and **reopen rate** (quality of fixes).

---

## 6) Exit Criteria, Operator Overrides, and Safety Lockouts

Automatic return-to-normal is often where risk concentrates. Define explicit exit criteria and when humans must confirm.

### 6.1 Auto-return criteria (examples)
- Comms stable (no missed polls) for X minutes.
- Detector health stable for Y cycles with no contradictions.
- No repeated oscillations in the last Z minutes.
- Time quality restored and stable.

### 6.2 Human confirmation triggers
- Repeated mode transitions (thrash risk).
- Any preemption failure or suspicious preemption behavior.
- Ped service alarms or suspected conflicting indications.
- Suspected time spoofing or cyber compromise.

### 6.3 Lockout / isolation triggers
- Suspected unsafe operation.
- Untrusted time with conflicting telemetry.
- Suspected compromise of comms or credentials.
- Documented recurring oscillations exceeding policy limits.

### 6.4 Operator UX requirements
- One-click **isolate** and one-click **revert**, each requiring a reason code.
- Timeline view of transitions and evidence snapshots.
- Suggested actions (repair detector, check comms, dispatch tech, contact security).
- Clear indication of whether auto-return is currently **enabled**, **disabled**, or **locked out** by policy.

---

## 7) State Machine for Graded Degradation (NORMAL / DEGRADED / FALLBACK / ISOLATED)

This section provides the **required visual** state machine for the reliability layer.

### 7.1 Conceptual state machine diagram

```text
                  +-----------------------------+
                  |                             |
                  v                             |
     +--------+        minor fault cleared      |
     | NORMAL | <-------------------------------+
     +--------+                                  \
        |  ^                                      \
  health |  | exit criteria met                    \
 degrade |  | (stability, no thrash)               \
        v  |                                        \
   +-----------+     worsening fault                 \
   | DEGRADED  | ----------------------+             \
   +-----------+                      |              \
        |  ^                          |               \
 severe |  | partial recovery         |                \
 fault  |  |                          |                 \
        v  |                          v                  \
   +-----------+   comms loss /       +-----------+       \
   | FALLBACK  |-- controller local ->| ISOLATED  |--------
   +-----------+   only operation     +-----------+
        ^   |                             |   ^
        |   | operator isolate            |   |
        |   +-----------------------------+   |
        |       operator revert /            |
        +----------- controlled return ------+
```

- **NORMAL**: fully healthy detection, comms, and time; uses full actuated/coordination logic.
- **DEGRADED**: some detectors or comms degraded; logic reduces reliance on bad data but keeps most capabilities.
- **FALLBACK**: reliability of detection or comms is too low; use conservative fixed-time or simple actuated patterns.
- **ISOLATED**: comms or trust boundary broken; controller runs a safe local plan with minimal external influence.

Transitions occur only at **safe boundaries** and are rate-limited with hysteresis and lockouts.

---

## 8) Fault Table: Fault → Signals → Detection Tests → Action → Operator Message

This table is the second required visual, translating the self-healing concept into concrete behavior for common failures.

| Fault type                    | Signals / symptoms                                           | Detection tests (examples)                                                                 | Automated action (mode / plan)                                      | Operator message / ticket template                                     |
|------------------------------|--------------------------------------------------------------|--------------------------------------------------------------------------------------------|------------------------------------------------------------------------|---------------------------------------------------------------------------|
| Detector stuck-on            | Continuous call, frequent max-outs in low demand            | Overnight actuations, max-out ratio vs expected; flatline high occupancy                  | Move to **DEGRADED** (reduced reliance) or **FALLBACK** plan; flag detector | "Detector likely stuck-on at approach X; switched to [mode]; inspect loop/video." |
| Detector stuck-off           | No calls, phases gapping early, low volumes vs historical   | Missing actuations vs historical; near-zero occupancy with known demand                   | Add recall / min-green guard; potentially shift to **FALLBACK**       | "Detector likely stuck-off at approach X; recall enabled; schedule field check."   |
| Stuck ped button             | Continuous ped calls, frequent overnight ped service        | Ped calls in low/zero pedestrian windows; ATSPM stuck-ped watchdog                        | Treat as auto-recall; adjust splits if needed; stay NORMAL/DEGRADED   | "Possible stuck ped button at crossing Y; operating as recall; verify on site."     |
| Comms loss                   | Missed polls, no recent telemetry                           | Heartbeat timeout; no data in ATSPM window                                                | Move to **ISOLATED** or local plan; stop remote commands              | "Comms lost to intersection Z; running local isolated plan; IT to investigate."     |
| High comms latency / flapping| Intermittent polls, out-of-order events                     | Variable latency; out-of-order timestamps; partial gaps                                   | Move to **DEGRADED**; rate-limit commands; suppress non-critical changes | "Unstable comms to intersection Z; degraded mode; check network path."             |
| Time drift / time jump       | Timestamps jump backwards/forwards, inconsistent ordering   | Compare controller time vs trusted NTP; detect discontinuities                            | Move to **DEGRADED** or **FALLBACK**; treat analytics as low-trust    | "Time sync issue at intersection Z; conservative timing enabled; verify NTP/source." |
| Cabinet power events         | UPS on battery, restarts, conflict monitor trips            | Power/UPS telemetry; controller restart logs; CM events                                   | Move to **FALLBACK** or **ISOLATED** depending on policy              | "Power/CM event at cabinet Z; now on fallback/local; inspect cabinet and CM logs."  |
| Suspicious detector pattern  | Unusual spikes in acts, inconsistent with neighbors/probes  | Cross-check vs neighboring detectors and probe speeds; anomaly score                      | Move to **DEGRADED**; consider isolation if correlated with cyber signs| "Anomalous detector behavior at Z; degraded mode; investigate sensor and security."  |
| Preemption input failure     | EVP requests not appearing or stuck active                  | Compare CAD events vs preemption log; detect stuck preempt input                          | Lock to **FALLBACK** with safe preemption disable or restrictive rule | "EVP anomaly at intersection Z; fallback preemption policy active; coordinate with EMS." |
| Repeated mode oscillation    | Frequent back-and-forth between modes                       | Count transitions over time window                                                        | Enter **lockout**; hold safe mode until human review                  | "Repeated mode oscillations at Z; auto-return locked; review thresholds and field." |

Agencies should extend this table with local failure types and specific reason codes matching their ticketing system.

---

## 9) Detailed Implementation Plan

### Phase 1: Requirements, Safety Invariants, and Failure Catalog (Weeks 1–6)
The agency should begin by cataloging the actual field assets at the pilot sites, because controller capabilities, detector types, and cabinet wiring determine what can be monitored and what can be changed automatically. The traffic engineering lead should then define non-negotiable safety and legal invariants, including pedestrian walk and clearance minimums, conflict monitor expectations, preemption and emergency vehicle priority behavior, and any jurisdictional “flash” policies. The team should build a failure-mode catalog that is grounded in real maintenance history (for example, stuck detectors, intermittent comms, time-sync drift, video occlusion, and cabinet power issues) and should explicitly label which failures are automatically detectable and which require human confirmation. Finally, the team should select a small set of pilot intersections that represent common patterns (simple 4-leg) and one or two difficult sites (heavy pedestrian demand, transit interaction, or complex geometry) so the solution does not only work in “easy mode.”

- **Roles**: traffic engineering lead (invariants and mode policy), signal maintenance supervisor (field failure history), IT/network lead (comms/power), safety/accessibility reviewer (ped constraints), data engineer (telemetry requirements).
- **Deliverables**: failure-mode catalog with indicators, mode transition policy draft, acceptance test plan, and baseline performance snapshot.
- **Risks**: the team may over-scope the failure catalog; flash and fallback policies may be politically or legally sensitive.
- **Acceptance checks**: the invariants are signed off, pilot sites are selected, and every failure mode has an agreed detection approach (automatic, manual, or “not supported”).

### Phase 2: Telemetry, Health Metrics, and Data Quality Gates (Weeks 7–16)
The engineering team should implement reliable telemetry collection from controllers and detectors so the system can distinguish “traffic changed” from “the sensor broke.” The team should ingest phase and interval state, detector actuations, pedestrian calls, conflict monitor alarms, and any available cabinet power/UPS signals, and it should also compute communications health (heartbeat latency and missed polls). The data quality layer should produce explicit health metrics such as stuck-on/off detectors, flatline variance, chattering detection, occupancy plausibility bounds, and cross-sensor consistency checks, because these are the real-world failure patterns that create bad timing decisions. The operations team should receive dashboards and alerting that is severity-based and tuned to avoid alert fatigue, and the maintenance team should receive actionable tickets that point to specific detectors or comms links.

- **Roles**: data engineer (pipeline and storage), IT/network engineer (secure comms), signal technician (field QA), operations lead (alert routing), analyst (threshold tuning).
- **Deliverables**: telemetry pipeline, health-metric service, dashboards/alerts, and a detector QA report with prioritized fixes.
- **Risks**: thresholds may be mis-tuned, causing either alert fatigue or missed failures; controller object availability may vary by vendor.
- **Acceptance checks**: key telemetry is available with high completeness during peaks (for example, >95%), time sync is stable, and health metrics behave predictably on known-good detectors.

### Phase 3: Mode Logic (Finite-State Machine) and Hysteresis (Weeks 17–24)
The software team should implement a finite-state machine that converts health signals into operational modes, because “self-healing” in practice is a controlled escalation process rather than a single switch. The state machine should use dwell times and hysteresis windows so the system does not bounce between NORMAL and FALLBACK during intermittent faults, and it should include an oscillation guard that enforces cooldown after repeated switches. The traffic engineer should define a capability matrix for each mode (for example, DEGRADED may allow actuated operation with reduced reliance on questionable detectors, while FALLBACK may use fixed-time plans), and the operations team should retain the ability to force a mode temporarily with a reason code. Every transition should create an audit event that records what changed, why it changed, and what evidence (health metrics) supported the decision.

- **Roles**: software engineer (FSM implementation), traffic engineer (policy and thresholds), QA/test engineer (scenario tests), operations lead (override workflow).
- **Deliverables**: FSM service, transition audit schema, operator override workflow, and scenario tests for common failures.
- **Risks**: mode policy may be too aggressive (unnecessary fallbacks) or too lax (unsafe reliance on broken detection).
- **Acceptance checks**: shadow testing shows low mode-flapping rate, manual override works and expires, and all transitions include reason codes and evidence.

### Phase 4: Fallback Plan Library and Twin Validation (Weeks 25–34)
The traffic engineering team should create a small, conservative fallback plan set for each pilot site because “degraded” operation needs an approved landing zone that maintenance and operations trust. The team should design at least one conservative actuated plan (for partial detection), one fixed-time plan (for unreliable detection), and an isolated local plan (for comms loss), and it should ensure that each plan preserves pedestrian minimum service and clearance behavior. The modeling team should then validate these fallback plans in a digital twin by injecting realistic failures (detector stuck, comms loss, heavy pedestrian demand) and by measuring how queues shift to adjacent intersections, because corridor coupling is where naive fallback strategies often fail. Finally, the team should document when to escalate from DEGRADED to FALLBACK or ISOLATED, and it should define a safe recovery process back to NORMAL that requires sustained health.

- **Roles**: traffic engineer (plan design), modeler (twin scenarios), safety/accessibility reviewer (ped checks), operations lead (approval).
- **Deliverables**: versioned fallback plan library with activation criteria, twin validation report, and corridor impact analysis.
- **Risks**: fallback plans may preserve safety but create unacceptable congestion; fixed-time plans may not serve pedestrians well if volumes shift.
- **Acceptance checks**: simulations show invariants always satisfied, corridor impacts are understood, and each fallback plan has a clear activation and rollback/recovery procedure.

### Phase 5: Pilot Deployment (Shadow → Assisted → Auto) (Weeks 35–46)
The agency should deploy self-healing in a phased approach because automatic switching is operationally sensitive and requires trust. In shadow mode, the system should compute health scores and recommended mode transitions without actuating the controller so the team can measure false positives and tune thresholds safely. In assisted mode, operators should confirm transitions so the team can validate that recommendations align with field reality and maintenance observations. After thresholds and procedures stabilize, the system can enable auto-switching with strict rate limits and an upper bound on time spent in fallback before escalation or human review. The team should also run planned “game day” drills (for example, temporarily disconnecting a detector) to validate that playbooks, alerts, and recovery procedures work under controlled conditions.

- **Roles**: operations (day-to-day monitoring), field technicians (drills and fixes), engineer on-call (tuning), analyst (evaluation).
- **Deliverables**: pilot evaluation report (MTTD/MTTR, false positive rate, flap rate), updated thresholds, operator and maintenance runbook.
- **Risks**: drills may disrupt traffic if poorly timed; operators may distrust the system if alerts are noisy.
- **Acceptance checks**: detection-to-recovery time improves versus baseline, false positives are within target, operators can revert safely, and drill outcomes are documented and reviewed.

### Phase 6: Scaling and Continuous Improvement (Ongoing)
The program should scale intersection-by-intersection and then corridor-by-corridor, because each controller platform and detection configuration has unique failure signatures. The engineering team should continuously incorporate new failure modes observed in production and should refresh fallback plans as geometry and demand change due to construction and development. The agency should schedule quarterly threshold reviews, periodic detector QA audits, and annual recertification of safety invariants and flash policies so the system remains defensible. Over time, the team should measure program-level outcomes such as reduced downtime, faster maintenance response, and fewer operational surprises.

- **Roles**: program owner (governance), operations (monitoring), engineering (improvements), maintenance (field fixes).
- **Deliverables**: network rollout roadmap, quarterly health report, updated plan library versions, and annual compliance/audit package.
- **Risks**: equipment aging and firmware changes may break assumptions; scaling may increase alert load without process improvements.
- **Acceptance checks**: audit logs are complete, coverage expands without increased incident rate, and MTTR improvements remain sustained.

---

## 10) Technical Mechanics

### 10.1 Key Parameters
- Health score thresholds (NORMAL/DEGRADED/FALLBACK).
- Hysteresis windows and minimum dwell times.
- Recovery criteria.
- Safety invariants and violation detectors.

### 10.2 Guardrails
- Never violate pedestrian minimum service/clearance logic.
- Rate-limit mode changes.
- Always provide manual override + safe rollback.
- Emit reason codes + evidence per transition.

---

## 11) MVP Deployment

- 2–4 intersections.
- 3 health checks (heartbeat, stuck detector, plausibility).
- 1 fallback plan per site.
- Shadow → assisted → auto progression.

---

## 12) Evaluation

- Mean time to detect and recover from faults.
- False-positive rate and mode-flap rate.
- ATSPM impacts: split failures, arrivals-on-green changes.
- Safety compliance: ped service, clearance integrity.
- Corridor-level spillback and progression impacts during degraded periods.

---

## 13) Implementation Checklist (Design & Deployment)

Use this as a build-and-launch checklist for a self-healing layer.

- [ ] Define controller vs supervisory responsibilities and document safety invariants that must never be violated.
- [ ] Build a failure-mode catalog tied to real maintenance history and explicit detection signals.
- [ ] Define NORMAL/DEGRADED/FALLBACK/ISOLATED semantics and permitted capabilities per mode.
- [ ] Implement telemetry ingestion for phase states, detection, ped, preemption/TSP, comms heartbeat, and (where available) cabinet/power.
- [ ] Implement data-quality gates and watchdog-style checks to detect comms loss and detector anomalies ([FHWA ATSPM Use Cases (Ch.4)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).
- [ ] Implement FSM mode logic with hysteresis, dwell times, oscillation guard, and rate limiting.
- [ ] Create versioned fallback templates and validate them in a twin with injected failure drills.
- [ ] Define corridor-aware anti-cascade rules (spillback guards, upstream metering patterns) and simulate them.
- [ ] Define cyber-resilient posture (access controls, identity mapping, anomaly corroboration) and integrate with incident response ([Transportation Cybersecurity Incident Response & Management Framework (PDF)](https://rosap.ntl.bts.gov/view/dot/57007/dot_57007_DS1.pdf)).
- [ ] Define explicit exit criteria, auto-return rules, operator overrides, and lockout triggers.
- [ ] Configure logging, retention, KPIs, and ticketing integration for health events.
- [ ] Deploy in shadow → assisted → auto phases; run controlled “game day” drills and capture after-action reviews.

---

## 14) Operations Runbook (SOP)

### 14.1 Severity definitions
- **CRITICAL**: suspected unsafe operation, conflicting indication suspicion, preemption malfunction, untrusted time with conflicting telemetry, suspected compromise.
- **HIGH**: comms loss > threshold, widespread detector faults, repeated mode oscillations.
- **MEDIUM**: single detector fault affecting actuation quality, intermittent comms errors.
- **LOW**: minor telemetry missingness, non-impacting sensor anomalies.

### 14.2 Triage workflow (always)
1. Confirm current mode (NORMAL/DEGRADED/FALLBACK/ISOLATED) and recent transitions in the timeline.
2. Check comms heartbeat and last-good telemetry timestamp.
3. Check detector health summary (stuck-on/off, abnormal max-outs/force-offs, missing data).
4. Check ped service status and any stuck ped indicators.
5. Check preemption/TSP event logs.
6. Decide action: **revert**, **hold in fallback**, **isolate**, or **dispatch**.
7. Open/attach ticket with evidence snapshot and correlation ID.

### 14.3 Playbooks by failure category

#### A) Communications loss
- Move to ISOLATED/local plan if comms are lost and the controller supports safe local operation.
- Notify IT/network, open ticket with last-known heartbeat time.
- Only auto-return after comms stable for the configured window and no oscillation history.

#### B) Detector failure suspected
- Confirm watchdog signals (overnight max-outs/force-offs, missing counts) and compare to field expectations ([FHWA ATSPM Use Cases (Ch.4)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).
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

## 15) Completion Checklist (Program Readiness)

Use this to decide if the **self-healing reliability layer is production-ready** for a corridor.

- [ ] All safety invariants (pedestrian, clearance, conflict monitoring, preemption) are documented and verified against controller configuration.
- [ ] NORMAL/DEGRADED/FALLBACK/ISOLATED modes are implemented with clear capability matrices and tested transitions.
- [ ] The state machine shows stable behavior in shadow mode with acceptable false-positive and flap rates.
- [ ] Fault table has been localized to agency failure modes and wired into alert templates.
- [ ] Telemetry, schemas, time sync, and data-quality checks are operational and monitored.
- [ ] Governance workflows (event → alert → ticket → dispatch → resolution → postmortem) are tested with real tickets.
- [ ] KPIs (MTTD, MTTR, time-in-mode, false positives, recurrence) are reported at least monthly.
- [ ] Cyber and anti-thrash policies (rate limits, hysteresis, lockouts) are configured and validated via drills.
- [ ] Operator overrides and lockouts work as designed, with clear UI and audit logs.
- [ ] At least one corridor-level failure drill has been run in coordination with operations, maintenance, and security.

---

## 16) Reference Links

### Standards / interoperability
- [NTCIP 1211 (Signal Control and Prioritization)](https://www.ntcip.org/file/2018/11/NTCIP1211-v0224j.pdf)

### Operations / ATSPM guidance
- [FHWA ATSPM (landing)](https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm)
- [FHWA ATSPM Use Cases (Ch.4)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)

### Cybersecurity / incident response
- [CISA Cybersecurity Best Practices](https://www.cisa.gov/topics/cybersecurity-best-practices)
- [Transportation Cybersecurity Incident Response & Management Framework (PDF)](https://rosap.ntl.bts.gov/view/dot/57007/dot_57007_DS1.pdf)
- [FHWA/USDOT Transportation Cybersecurity Resources (PDF)](https://ops.fhwa.dot.gov/publications/fhwahop24101/fhwahop24101.pdf)

---

Cross-links: Related ideas include what-if button, delay-tolerant signals, and explainable signals.
