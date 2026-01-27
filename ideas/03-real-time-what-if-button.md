# 03) Real-Time “What If?” Button: Predictive Traffic Decisions

## Catchy Explanation
It’s a “try it in the simulator first” button for traffic operators: test a timing change in a live digital twin, see the likely outcome in the next few minutes, then decide whether to apply it.

## What it is (precise)
A **real-time what-if button** is an operator workflow backed by a calibrated digital twin that can initialize from the current field state (phases, detector counts/occupancy, queues, probe travel times), run a small set of *bounded* candidate actions (e.g., split ±5s, offset ±3s, switch to a pre-timed plan), and return predicted KPI deltas (delay, p95 queues, spillback probability, pedestrian delay) within a strict latency budget (often **<10–30 seconds**). The system enforces guardrails (hard safety constraints, rollback, staleness checks) and logs the evaluated options for auditability.

## Benefits
- **Safer decisions**: reduces risky “guess-and-deploy” changes during incidents.
- **Faster response**: provides near-term forecasts to pick the least-bad action.
- **Audit trail**: records what was considered and why a change was chosen.
- **Operator confidence**: supports human-in-the-loop control rather than opaque automation.

## Challenges
- **State initialization**: accurate queue estimation from sparse detectors is hard.
- **Latency vs fidelity**: micro-sim can be too slow without acceleration/approximation.
- **Calibration drift**: model becomes stale as demand patterns change.
- **Automation bias**: operators may over-trust predictions unless uncertainty is shown.

## Implementation Strategies

### Infrastructure Needs
- **Live state feeds**: controller phase states, detector volumes/occupancy, pedestrian calls, (optionally) probe travel times.
- **Fast simulation stack**: micro/meso twin capable of faster-than-real-time rollouts.
- **Action generator**: templates for safe candidate actions; constraints-aware (no illegal phasing).
- **Guardrails + rollback**: bounded actions, automatic revert if KPIs degrade.
- **Observability**: ATSPM-style metrics to validate predictions and measure impacts.

### Detailed Implementation Plan
#### Phase 1: Define Operator Use Cases and Performance SLOs (Weeks 1–8)
The agency should start by selecting a small number of operator scenarios where a what-if tool would plausibly change decisions, such as incident response, construction detours, or peak-period queue management. The team should define a strict service-level objective for response time (for example, p95 < 30 seconds) and should define how fresh the input data must be before the system refuses to simulate. The team should then specify a standard KPI bundle that operators will see for every what-if run, including delay, p95 queue, spillback probability, arrivals-on-green, and pedestrian delay, because consistency is essential for decision-making under stress. Finally, the team should define a bounded list of allowable actions (plan switch, small split tweaks, small offset tweaks) and should explicitly exclude unsafe or legally constrained actions so the system cannot recommend something that cannot be implemented.

- **Roles**: operations lead (workflow owner), traffic engineer (KPI definitions and allowable actions), product/UX (operator experience), safety/accessibility reviewer (pedestrian implications), IT/security (access and audit requirements).
- **Deliverables**: requirements specification, latency and freshness SLOs, KPI definitions, allowed-action catalog, and UI mockups.
- **Risks**: too many scenarios create scope creep; unrealistic latency targets lead to unusable results.
- **Acceptance checks**: the spec is signed off, SLOs are measurable, and the allowed-action set is implementable on the target controller platforms.

#### Phase 2: Data Ingestion and State Estimation (Weeks 9–18)
The engineering team should build a reliable ingestion path for controller events (phase and interval state, detector actuations, pedestrian calls) and should optionally integrate probe travel times if a vendor feed exists. The team should implement data quality checks that reflect real signal operations, such as detector stuck-on/off checks, communications heartbeat monitoring, and time-synchronization validation, because stale or incorrect state is the most common root cause of bad predictions. The team should then implement queue and state estimation using conservative filters that respect lane storage limits and that emit a confidence score, because operators should be told when the system is guessing. The team should log the estimated state and confidence for every request so later analysis can correlate “bad recommendations” with low-confidence inputs.

- **Roles**: data engineer (pipelines), modeling engineer (state estimation), signal technician (detector validation), operations (feedback on when state is “clearly wrong”).
- **Deliverables**: live state API, state-estimation module, confidence scoring, and QA dashboards.
- **Risks**: sparse detection produces unstable queue estimates; probe coverage gaps bias results.
- **Acceptance checks**: state estimates are stable on normal days, confidence scores correlate with error, and staleness rules correctly block simulations when inputs are too old.

#### Phase 3: Twin Build and Runtime Optimization (Weeks 19–30)
The modeling team should select a simulation approach that matches the runtime SLO, because a micro-simulation that cannot respond in time is worse than a faster queue-based model that is “good enough” for short horizons. The team should build and calibrate the corridor model to match not only average travel times but also queue peaks and arrivals-on-green, and it should validate performance for the specific horizons the tool will use (typically 1–5 minutes). The infrastructure team should optimize runtime by warming the twin from the estimated current state, parallelizing candidate evaluations, and caching stable components such as route sets, because these tactics are what allow real-time decision support. Finally, the team should establish a repeatable benchmark suite that measures p50/p95 simulation time and prediction error so performance does not silently regress.

- **Roles**: simulation/modeling engineer (twin), infrastructure engineer (runtime/performance), traffic engineer (calibration targets), QA engineer (benchmarks and regression testing).
- **Deliverables**: calibrated twin in shadow mode, runtime benchmark report, and validation report.
- **Risks**: micro-simulation remains too slow; mismatch between estimated queues and simulated queues reduces trust.
- **Acceptance checks**: the system meets the response-time SLO on realistic hardware and produces prediction error within the agreed tolerance for key KPIs.

#### Phase 4: Candidate Actions Library and Safety Envelope (Weeks 31–40)
The traffic engineer should define a library of action templates that match how controllers are actually configured in practice, such as switching between pre-approved plan IDs or applying limited split/offset adjustments that preserve minimum green and pedestrian clearance requirements. The software team should encode hard constraints that reject illegal actions before simulation and should also reject actions if the current state is stale or the self-healing health checks indicate unreliable detection. The team should implement rollback behavior that operators understand, such as a time-based revert to a known plan and a KPI-triggered revert if queues worsen beyond a pre-set threshold, because rollback is the practical safety net for decision support. Finally, the team should implement rate limits and full audit logging so plan changes do not oscillate and so post-incident reviews can determine why a change was made.

- **Roles**: traffic engineer (bounds and legality), software engineer (constraints and rollback), safety reviewer (ped rules), operations lead (workflow and training inputs).
- **Deliverables**: versioned action library, constraint/validation module, rollback controller, and audit log schema.
- **Risks**: incomplete constraints allow unsafe recommendations; rollback triggers are too sensitive and create instability.
- **Acceptance checks**: every action is validated against constraints prior to simulation, rollback works in staging and in pilot drills, and operators can revert within minutes.

#### Phase 5: Operator UI and Workflow Integration (Weeks 41–52)
The UX team should design the interface for speed and clarity, because operators make decisions under stress and cannot interpret a research dashboard. The UI should show the current state estimate and its confidence, and it should present a short list of ranked options with predicted deltas and uncertainty rather than a single “best” answer. The tool should require explicit confirmation before any actuation and should clearly label when the system is only simulating, because this reduces automation bias. The team should integrate the tool into existing TMC workflows by linking decisions to incident logs and shift notes, and it should train operators through tabletop exercises that include both successful and failed predictions.

- **Roles**: UX engineer (UI), operations lead (training), traffic engineer (interpretation), product owner (scope and rollout).
- **Deliverables**: pilot-ready UI, training materials, operator runbook, and integration to incident logging.
- **Risks**: operators may over-trust or ignore recommendations; too much information reduces usability.
- **Acceptance checks**: operators complete drills successfully, UI meets latency targets, and logs capture decisions, overrides, and outcomes.

#### Phase 6: Pilot Deployment, Evaluation, and Scaling (Weeks 53+ / Ongoing)
The agency should run the tool in shadow mode first and should compare predicted KPI deltas to realized KPIs, because this is the fastest way to measure whether the model is trustworthy without creating operational risk. The team should then move to assisted actuation where operators apply recommended actions with rollback enabled, and it should measure how often operators accept recommendations and why they override them. The modeling team should implement a calibration and drift-detection cadence (monthly, or sooner during construction) so predictions remain relevant. To scale to new corridors, the team should reuse the architecture and UI but should redo calibration, action bounds, and policy constraints for each corridor, because local phasing and pedestrian requirements differ.

- **Roles**: program owner (governance), operations (daily use), modeling team (calibration), data analyst (evaluation).
- **Deliverables**: pilot report, calibration playbook, and corridor expansion roadmap.
- **Risks**: model drift; field changes invalidate assumptions; inconsistent operator use reduces measured benefit.
- **Acceptance checks**: prediction error decreases over time, KPI improvements are achieved without policy violations, and scaling templates are documented.

### Choices
- **Recommend-only**: operators view outcomes but apply changes manually.
- **Assisted control**: operator-confirmed actuation with rollback.
- **Continuous shadow**: always-on evaluation for learning and validation.

## Technical Mechanics

### Key Parameters
- Horizon: 1–5 minutes
- Candidates per request: 3–20 (bounded by runtime)
- Runtime SLA: p95 response < 30s
- KPI bundle: delay, stops, p95 queues, spillback probability, ped delay

### Guardrails
- Reject actions that violate pedestrian minimums/clearances.
- Reject actions when state is stale or health checks fail.
- Rate-limit plan changes; enforce revert paths.

## MVP Deployment
- One corridor (5–12 signals).
- 3 candidate actions.
- Shadow mode for 2–4 weeks, then assisted activation.

## Evaluation
- Prediction error (queues/travel times).
- Decision latency distribution.
- Operator acceptance/override rate.
- KPI change vs baseline, plus rollback frequency.

## References / Standards / Useful Sources
- FHWA Traffic Signal Timing Manual: https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf
- FHWA Automated Traffic Signal Performance Measures (ATSPM): https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm

---

Cross-links: Related ideas include fast-forward twin, green waves, self-healing intersections, and city rules.
