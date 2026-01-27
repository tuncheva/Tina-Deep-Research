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
- **Observability**: ATSPM-style metrics to validate predictions and measure impacts ([`FHWA ATSPM (landing)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm)).

---

## 1) State initialization & uncertainty handling
A what-if tool only works if it can **initialize a realistic state** quickly and admit when it cannot. ATSPM guidance makes clear that the tooling supports engineers/technicians by exposing operational issues and requires interpretation and action; a what-if tool should follow the same philosophy by surfacing evidence and confidence rather than pretending it “knows” ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

### Minimum required inputs (baseline)
- **Controller state**: current phase/interval, plan ID, coordination state (if applicable), phase termination types (gap/max/force-off).
- **Detection summary**: stop-bar calls/occupancy (where present), approach volumes/counts, detector health flags.
- **Pedestrian state**: active calls, recall status, pedestrian service timing (WALK/FDW).
- **Geometry registry**: lane groups, storage lengths, turn bay lengths, permitted/protected movements.
- **Optional enrichment**: probe speeds/travel times for corridor state, incident feeds/work zones.

### Sparse/partial detection + contradictory sensors
Use a **layered state estimator**:
1. **Hard constraints**: lane storage limits, minimum discharge/saturation constraints, and “cannot exceed storage” queue caps.
2. **Primary sensors**: stop-bar detection and phase state (if good health).
3. **Secondary corroboration**: advance detectors, probe travel times, and “symptom” signals like max-outs/force-offs patterns.

ATSPM shows how high-resolution event patterns can be used to identify detection and comms problems (e.g., watchdog conditions and phase termination diagrams) and therefore can be used as “sensor health” inputs to a what-if state estimator ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

### Queue estimation methods for short horizons (1–5 min)
Use methods that can run fast and degrade gracefully:
- **Simple lane-group queue balance**: update queue by arrivals minus departures per cycle/phase.
- **Detector-driven queue proxies**: infer queue growth when occupancy remains high during green and red.
- **Spillback/blocking detection**:
  - look for sustained high downstream occupancy (or near-continuous presence) coupled with discharge suppression (few departures even when green)
  - incorporate “red occupancy” patterns and split failures as warning signals.

ATSPM defines split failures and describes occupancy-ratio logic (GOR/ROR5) used to detect split failures when demand exceeds service, which can be used as an operational proxy for “queues not clearing” ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm), [`FHWA ATSPM (PDF)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/fhwahop20002.pdf)).

### Representing and propagating uncertainty
Represent uncertainty explicitly and carry it into scenario ranking:
- Attach a **confidence score** to each critical state element (queue length, spillback probability, arrival rate).
- Run scenarios as **ensembles**: evaluate each action across plausible states (“low/med/high queue” or a sampled distribution).
- Rank actions by a **conservative criterion** (e.g., worst-case score or percentile score), not only the mean.

### “Staleness” rules (when to refuse to simulate)
Define clear data freshness checks:
- If controller state is older than the freshness SLO, the UI should disable “Apply” and label results as unavailable.
- If detection health checks indicate “no data” or suspicious patterns, the system should either (a) reduce action vocabulary to “plan switch only” or (b) refuse recommendations.

ATSPM’s watchdog examples explicitly include “no data” conditions to identify comms failure, illustrating why staleness needs formal handling ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

### UI behavior for uncertainty (operator-facing)
Operators should see:
- a top-line **State Confidence** label (HIGH/MED/LOW)
- uncertainty bands / “worst case” outcomes, not only a single predicted delta
- explicit “data missing → result unknown” behavior

---

## 2) Controller/ATMS feasibility constraints + action vocabulary
The what-if tool must recommend **only actions that can be deployed safely** on the target controllers and within agency policy. Signal timing guidance describes plan development and coordinated operation, and priority/preemption considerations that bound what is operationally feasible ([`FHWA Traffic Signal Timing Manual (PDF)`](https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf), [`FHWA Timing Manual — Chapter 7 (Plan development)`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter7.htm), [`FHWA Timing Manual — Chapter 9 (Priority/Preemption)`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm)).

### Action Vocabulary (vendor-neutral)
Feasible, common action types:
- **Plan selection**: switch to a pre-approved plan ID from a plan library (time-of-day / traffic-responsive plan selection).
- **Bounded split tweaks**: adjust green splits within configured min/max and without violating pedestrian timings.
- **Bounded offset shift**: adjust offsets within coordination limits (where supported) to improve progression.
- **Green extension (bounded)**: extend a coordinated phase by a small amount within max green constraints.
- **Hold/force-off (policy-limited)**: only if agency policy allows and only via controller-supported transitions; these are higher risk.
- **No phasing topology changes**: permissive/protected changes or ring/barrier changes should only be done via **pre-approved configurations**, not ad-hoc.

### “Can apply now?” gating rules
For each candidate action, compute an explicit feasibility verdict:
- **Controller capability check**: action supported by the controller configuration and communications objects (often via NTCIP management/control objects) ([`NTCIP 1211 (Signal Control and Prioritization)`](https://www.ntcip.org/file/2018/11/NTCIP1211-v0224j.pdf)).
- **Safety/legal constraints**: do not violate minimum greens, clearance intervals, or pedestrian service requirements.
- **Coordination transition constraints**: if coordinated, only allow actions at safe boundaries and with documented transition logic.
- **Health/staleness constraints**: if state confidence is low, restrict the action set.
- **Rate limiting**: prevent rapid oscillation between actions.

### Action table (what operators actually need)
| Action | Preconditions | Risks | Rollback method | Typical use cases |
|---|---|---|---|---|
| Switch to plan ID | plan exists + approved; safe transition point; comms healthy | wrong plan for conditions; coordination disruption | revert to previous plan ID; timed auto-revert | incident detour; peak shift; planned event |
| Split tweak (± seconds) | min/max split margins; ped constraints satisfied | side-street starvation; queue migration | revert splits to baseline; revert plan | relieve short queue; protect storage |
| Offset shift (bounded) | coordinated; offset changes supported; corridor agreement | breaks progression elsewhere | revert offset; revert plan | restore progression after disruption |
| Green extension (bounded) | max green margin; conflicting demand acceptable | increases red delay for others | stop extension; revert plan | clear spillback risk; late bus recovery |
| Hold/force-off (policy-limited) | explicitly allowed; operator trained; safety boundary | hazardous if misused; confusion | immediate revert to baseline plan | unusual incidents; emergency management |

---

## 3) Objective design, tradeoffs, and anti-gaming guardrails
The what-if button needs a consistent scoring rubric that separates **hard constraints** from **soft objectives**.

### Recommended KPI set (short horizon)
Use a compact but multi-modal KPI bundle:
- **Queue / spillback risk**: max queue and probability of spillback at critical links.
- **Delay**: person-delay (preferred) and vehicle-delay.
- **Throughput / discharge**: vehicles served per cycle or per minute.
- **Progression**: arrivals on green / progression ratio (where applicable).
- **Pedestrian delay**: especially max wait at priority crossings.
- **Transit delay**: bus delay / reliability if TSP corridors.
- **Safety proxies** (if available): red-light running risk proxies, near-miss/conflict proxies.

ATSPM provides definitions for progression evaluation and describes arrivals-on-green in the Purdue Coordination Diagram context, and it defines split failures and the GOR/ROR5 logic used to detect them ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

### Hard constraints vs soft objectives
- **Hard constraints** (must not violate): pedestrian minimums/clearance, maximum queue/storage constraints at critical approaches, “do-not-block” boundary constraints.
- **Soft objectives** (optimize within constraints): minimize person-delay, improve reliability, protect transit, improve progression.

### Anti-gaming patterns
- **Caps and monotonic constraints**: cap how much any approach delay can increase even if corridor delay improves.
- **Penalty terms**: penalize actions that increase spillback probability or reduce pedestrian service.
- **Robust scoring**: rank by worst-case (or p90) performance across uncertainty ensembles.

### Sample scoring rubric template (adjust per corridor)
- Hard constraints:
  - spillback probability at protected links must stay < threshold
  - pedestrian max wait at priority crossings must stay < threshold
  - no increase in critical side-street p95 queue beyond cap
- Soft objective weights (example):
  - 0.35 person-delay (corridor)
  - 0.25 spillback risk (penalty)
  - 0.15 transit delay (if applicable)
  - 0.15 arrivals-on-green / progression
  - 0.10 pedestrian delay

---

## 4) Operator workflow, accountability, and governance
A what-if tool is a decision-support workflow, not a model demo.

### Roles and permissions (recommended)
- **Operator**: can run what-if; can apply only low-risk actions (plan switch, bounded split tweaks) if permitted.
- **Supervisor**: can approve higher-risk actions and extended holds; can override rate limits with explicit justification.
- **Engineer (on-call)**: can modify action bounds and plan library, but changes must go through change control.

### Escalation paths (incident-time)
- Define when operators must escalate (e.g., repeated spillback, suspected bad detection, frequent rollback triggers).
- Link the workflow to the incident log so decisions can be reviewed.

### Required audit log fields
Every run should record:
- who initiated it, intersection/corridor, timestamp, incident context
- input data freshness and health summary
- scenario set version and action library version
- predicted deltas with confidence bands
- selected action, applied by whom, rollback plan, and actual outcome metrics

### Post-event review cadence
- **Weekly**: review outliers (large errors, rollbacks, overrides).
- **Monthly**: recalibrate, review equity impacts (which movements consistently lose), and update training.

### Training and human factors (anti-automation-bias)
- Require a “10-second card” view: state confidence, top 3 options, key risks/constraints, rollback path.
- Train with drills that include wrong predictions and low-confidence states.

---

## 5) Compute/infra reliability + safe degradation behavior
If compute and data pipelines are unreliable, the feature will be ignored in real incidents.

### Latency budgets and SLOs
Define SLOs per stage and enforce them:
- ingest + state estimation
- simulation/rollout
- scoring + UI render

### Degradation behaviors
- **Sim engine down**: disable recommendations; show “unavailable” state and link to manual playbooks.
- **Missing feeds**: reduce action vocabulary; show confidence drop and warnings.
- **High latency**: refuse “apply now” and show stale indicator.

### Monitoring/observability
Track and alert on:
- simulation runtime p50/p95
- dropped/late feed counts
- queue estimation confidence distribution
- rollback frequency and reason codes

---

## 6) Network effects and interaction boundaries
Local actions can harm adjacent areas.

### When local actions cause harm elsewhere
- Queue migration to upstream/downstream intersections.
- Diversion shifts due to changed progression.
- Transit priority interactions on adjacent corridors.

### Boundary setting
- Define a **control region** (where the tool can recommend actions) and an **observation region** (adjacent links/intersections that must be simulated/checked).
- Include boundary links in the simulation to capture spillback risk, even if the tool cannot control those nodes.

### “Do no harm” guardrails
- Require boundary spillback constraints to remain below caps.
- Reject actions that increase protected approach queue beyond storage.

---

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

---

## Implementation Checklist
- Define SLOs: end-to-end latency, data freshness, and safe-degradation behaviors.
- Implement live state ingestion (controller + detectors + ped + optional probe) and health/staleness checks; reuse watchdog patterns where possible ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).
- Build state estimator with explicit confidence scoring and spillback detection.
- Implement a bounded action vocabulary tied to controller capabilities and agency policy; include a per-action feasibility gate ([`FHWA Timing Manual — Chapter 7`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter7.htm)).
- Implement multi-objective scoring with hard constraints and anti-gaming (caps, penalties, robust scoring).
- Add audit logging for every run (inputs, versions, confidence, chosen action, rollback) and link to incident logs.
- Build UI that shows confidence/uncertainty and requires explicit confirmation.
- Deploy shadow-first; validate prediction error vs realized ATSPM/probe metrics.
- Run drills and train operators to manage automation bias.

## Operations Runbook (SOP)
### Normal use
1. Confirm data freshness and state confidence; if LOW, treat results as advisory only.
2. Select scenario set (incident type / objective template).
3. Run what-if; review top 3 options and their risks/constraints.
4. If applying, confirm feasibility gates (safe boundary, controller health, coordination constraints).
5. Apply action with rollback enabled; monitor KPIs for the rollback window.

### When to refuse to apply
- Controller state or detection feeds are stale or “no data” is detected ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).
- Estimated state confidence is low *and* the action is high risk (hold/force-off).
- Protected boundary link spillback risk increases.

### Rollback steps
1. Revert to last known-good plan ID (or baseline plan).
2. Log rollback reason code and attach observed KPIs.
3. Escalate to supervisor/engineer if repeated rollbacks occur.

### Incident-time escalation
- If spillback persists after two actions or if queues threaten critical facilities (hospitals/fire routes), escalate to supervisor and consider corridor-level response.

---

## Reference Links
- [`FHWA Traffic Signal Timing Manual (PDF)`](https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf)
- [`FHWA Timing Manual — Chapter 7 (Develop timing plans)`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter7.htm)
- [`FHWA Timing Manual — Chapter 9 (Priority/Preemption)`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm)
- [`FHWA ATSPM (landing)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm)
- [`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)
- [`FHWA ATSPM (PDF)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/fhwahop20002.pdf)
- [`NTCIP 1211 (Signal Control and Prioritization)`](https://www.ntcip.org/file/2018/11/NTCIP1211-v0224j.pdf)
- [`NCHRP Report 812 — Signal Timing Manual, Second Edition (PDF mirror)`](https://transops.s3.amazonaws.com/uploaded_files/Signal%20Timing%20Manual%20812.pdf)

---

## Completion Checklist
- ✅ State initialization + uncertainty — see `1) State initialization & uncertainty handling`
- ✅ Feasibility + action vocabulary — see `2) Controller/ATMS feasibility constraints + action vocabulary`
- ✅ Objectives + tradeoffs + anti-gaming — see `3) Objective design, tradeoffs, and anti-gaming guardrails`
- ✅ Operator workflow + accountability — see `4) Operator workflow, accountability, and governance`
- ✅ Compute/infra reliability — see `5) Compute/infra reliability + safe degradation behavior`
- ✅ Network effects — see `6) Network effects and interaction boundaries`
- ✅ End sections — see `Implementation Checklist`, `Operations Runbook (SOP)`, `Reference Links`

---

Cross-links: Related ideas include fast-forward twin, green waves, self-healing intersections, and city rules.
