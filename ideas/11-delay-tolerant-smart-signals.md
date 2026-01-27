# 11) Delay-Tolerant Smart Signals: Resilient Control Under Uncertainty

## Catchy Explanation
When sensors or comms get flaky, a good signal shouldn’t “panic optimize.” It should fall back gracefully and stay safe and predictable until the data is trustworthy again.

## What it is (precise)
**Delay-tolerant smart signals** are control systems designed to operate safely when inputs are **late, missing, or erroneous**. They continuously assess data health (latency, loss, plausibility), run a graded degradation policy (NORMAL → DEGRADED → FALLBACK/ISOLATED), and rely on pre-vetted timing plans that preserve invariants (pedestrian minimums/clearances, max cycles, preemption behavior). Digital twins are used to rehearse comms outages and sensor degradation, test transition logic, and quantify performance bounds during degraded operation.

## Benefits
- **Resilience**: fewer disruptions from data pipeline failures.
- **Safety assurance**: graceful degradation preserves constraints.
- **Operational clarity**: explicit modes + logs make troubleshooting easier.
- **Bounded performance**: predictable behavior under stress.

## Challenges
- **Over-conservatism**: safer modes can increase delay.
- **Tuning**: thresholds must avoid false triggers.
- **Mode oscillation**: need dwell times/hysteresis.
- **Maintenance**: fallback plans must stay current.

## Implementation Strategies

### Infrastructure Needs
- **Health monitoring**: latency/heartbeat checks, missing-data rate, stuck detector detection.
- **Plausibility checks**: cross-sensor consistency, bounds on volumes/occupancy.
- **Fallback plan library**: conservative actuated + fixed-time plans.
- **High-resolution logs / ATSPM**: measure failures and impacts.
- **Twin drills**: repeatable failure injection and transition testing.

### Detailed Implementation Plan
#### Phase 1: Failure Catalog and Safety/Service Invariants (Weeks 1–3)
The agency should begin by identifying the data dependencies of its signal operations, including controller polling, detector streams, probe travel time feeds, and any central optimization service, because “delay tolerance” is mostly about knowing what can go wrong. The team should document failure modes such as comms outages, delayed data, partial detector loss, time drift, and implausible values, and it should define unacceptable outcomes, including pedestrian service violations, loss of preemption behavior, and sustained queue spillback. The team should then define invariants that must hold in all modes, such as pedestrian minimums/clearance, maximum cycle constraints where applicable, and a clear safe local plan that can run autonomously.

- **Roles**: traffic engineering (invariants), operations (failure history), IT/network (data paths), maintenance (controller constraints), safety/accessibility reviewer (ped service).
- **Deliverables**: failure catalog, invariants list, and an initial mode policy draft.
- **Risks**: missing a key dependency leads to unhandled failures; invariants may conflict with desired performance.
- **Acceptance checks**: all critical data feeds are inventoried and each has a documented failure response.

#### Phase 2: Health Checks, Scoring, and Alerting (Weeks 4–10)
The engineering team should implement health checks for each input stream, including comms heartbeat latency, missing-data rate, detector stuck/chatter checks, and plausibility bounds on occupancy and flow, because delayed or corrupt data must be detected before it drives control decisions. The team should compute a unified health score per intersection and per corridor so that mode selection can be driven by consistent logic rather than ad-hoc alarms. The operations team should receive dashboards and alerting that indicate which component is unhealthy and what the recommended mode change is, because troubleshooting is a core goal of delay-tolerant design.

- **Roles**: data engineer (health metrics), IT/network (heartbeat monitoring), maintenance (detector QA), operations (alert routing), QA (threshold testing).
- **Deliverables**: health scoring service, dashboards, alert rules, and a detector/comms QA report.
- **Risks**: false alarms create unnecessary fallback; weak thresholds miss real failures.
- **Acceptance checks**: health scoring is stable on normal days and detects known failure cases in controlled tests.

#### Phase 3: Graded Degradation Modes and Transition Logic (Weeks 11–18)
The software team should implement explicit modes (NORMAL, DEGRADED, FALLBACK, ISOLATED) and should define what control actions are permitted in each mode, because the safest response to missing data is often to reduce complexity rather than guess. The transition logic should include dwell times and hysteresis so the system does not oscillate during intermittent comms, and it should require multiple consecutive “good” windows before returning to NORMAL. The team should validate the transitions in a twin by injecting delayed data, dropped messages, and detector failures, and it should quantify the performance bounds of each mode so operators know what to expect.

- **Roles**: software engineer (mode controller), traffic engineer (mode behaviors and plans), modeler (failure injection tests), operations (override workflow).
- **Deliverables**: mode controller, transition policies, twin validation report, and operator-facing mode definitions.
- **Risks**: degraded modes may cause unacceptable delay; recovery may be too slow and reduce performance.
- **Acceptance checks**: transitions are explainable and rate-limited, and the system always has a safe fallback plan.

#### Phase 4: Shadow Drills, Training, and Assisted Activation (Weeks 19–26)
The agency should run delay-tolerant logic in shadow mode first, where it detects failures and recommends mode changes without actuating, because this provides real-world validation without operational risk. Operators and maintenance staff should be trained on what each mode means, how to override it, and how to troubleshoot common causes such as time sync drift or detector faults. After shadow results are stable, the agency should enable assisted activation where operators confirm mode changes, because this step builds trust and reveals operational edge cases.

- **Roles**: operations (pilot users), maintenance (troubleshooting), engineering (on-call), analyst (evaluation).
- **Deliverables**: readiness report, updated thresholds, training materials, and a runbook for common failure patterns.
- **Risks**: operators may ignore recommendations if early alerts are noisy; shadow mode may not capture stress behavior.
- **Acceptance checks**: false-trigger rate is within target, and operators can reliably interpret and act on mode recommendations.

#### Phase 5: Rollout, Audits, and Continuous Maintenance (Ongoing)
The agency should roll out delay-tolerant operation corridor-by-corridor, prioritizing areas where comms reliability is known to be challenging or where detector quality is variable. The program should conduct periodic audits of mode events, including why mode changes occurred and what performance impacts resulted, because auditing is how thresholds and plans remain defensible over time. The team should also maintain fallback plan libraries and update them after construction, timing changes, and controller upgrades so “safe mode” does not become stale.

- **Roles**: program owner (rollout planning), operations (monitoring), traffic engineering (plan maintenance), IT (network improvements), analyst (audits).
- **Deliverables**: rollout roadmap, quarterly mode-event audit reports, updated fallback plan library, and maintenance tickets.
- **Risks**: scaling increases alert volume; stale fallback plans reduce safety and performance.
- **Acceptance checks**: mode events decrease over time as infrastructure improves, and fallback plans remain current and validated.

### Choices
- **Heartbeats only**: simplest detection.
- **Plausibility + fusion**: stronger detection, more complexity.
- **Graded modes**: step-down behavior (recommended).

## Technical Mechanics

### Key Parameters
- Latency thresholds and missing data rate
- Health score boundaries
- Dwell times / hysteresis windows
- Recovery criteria

### Guardrails
- Preserve pedestrian service and clearance invariants.
- Rate-limit mode transitions.
- Require reason codes + evidence logs.

## MVP Deployment
- One corridor.
- 3 modes: NORMAL, DEGRADED, FALLBACK.
- Shadow mode first.

## Evaluation
- Time-to-detect data failures.
- False trigger rate.
- Performance bounds during degradation (delay, queues, split failures).

## References / Standards / Useful Sources
- FHWA ATSPM: https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm
- FHWA Traffic Signal Timing Manual: https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf

---

Cross-links: Related ideas include self-healing intersections, explainable signals, and safety-first signals.
