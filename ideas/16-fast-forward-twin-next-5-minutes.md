# 16) Fast-Forward Twin (Next 5 Minutes): Short-Horizon Predictive Control

## Catchy Explanation
It’s like fast-forwarding the city by a few minutes—many times in parallel—to see which timing choice avoids the worst queues, then applying the safest best option.

## What it is (precise)
A **fast-forward twin** is a short-horizon model-predictive control (MPC) loop using a digital twin that repeatedly:
1) initializes from the current field state,
2) simulates multiple candidate actions over a 1–5 minute horizon,
3) scores them on a KPI bundle (delay, stops, p95 queues, spillback risk, ped delay, safety proxies),
4) selects an action within hard constraints,
5) applies it with rollback protections.

It differs from a one-off what-if tool by being a **continuous control loop** with runtime SLAs, strict action bounds, and explicit safety envelopes.

## Benefits
- **High responsiveness**: adapts to near-term demand shifts.
- **Constraint-aware**: selects actions that respect safety and policy invariants.
- **Stability control**: can reduce spillback cascades with early interventions.
- **Auditability**: logs evaluated rollouts and selected action.

## Challenges
- **Compute + latency**: must run fast enough every cycle/window.
- **State estimation**: twin initialization errors compound quickly.
- **Overfitting to the horizon**: myopic optimization can harm longer-term flow.
- **Operational risk**: requires strong rollback and supervision.

---

## Operating model (control loop + layers)

### Control layers
- **Controller layer (safety execution):** runs the actual signal safety-critical logic (conflicts, min greens, clearance, pedestrian timing). The fast-forward twin must not bypass this layer.
- **Supervisory layer (this idea):** chooses among *bounded* actions (e.g., plan selection, split/offset nudges) and validates feasibility before sending changes.
- **Observation layer (ATSPM + probes):** continuously measures outcomes and drift from predictions using high-resolution controller logs and probe data.

### MPC framing (why it works)
MPC repeatedly solves a rolling-horizon optimization: predict system evolution, optimize an objective, apply only the first action, then re-estimate and repeat. This “rolling horizon” structure and the ability to change predictive models and objectives are central MPC properties, including in traffic-signal applications. [A Survey of MPC Methods for Traffic Signal Control](https://www.ieee-jas.net/article/doi/10.1109/JAS.2019.1911471)

---

## 1) State estimation (inputs → estimates → uncertainty)

### 1.1 Minimum real-time inputs (per replan cycle)
**Required (minimum viable):**
- **Controller state + events:** current plan ID, ring/phase state, phase start/end times, termination type (gap-out/force-off/max-out), pedestrian calls, coordination status.
- **Detection aggregates/events:** approach/movement counts, occupancy and/or actuations; detector health signals.
- **Probe travel times/speeds:** segment-level speed/travel time (where available) to validate model and detect drift.
- **Static mapping:** geometry, lane groups, movement mapping, detector-to-movement mapping, phase-movement mapping.

**Optional (high value if available):**
- **Video/CV (counts + queues):** more direct queue proxies, but subject to occlusion and weather.
- **V2X / connected vehicle messages:** approach trajectories/queues (coverage-dependent).

### 1.2 What you can estimate well in 1–5 minutes (and what you can’t)
You can usually estimate **short-horizon demand state** (who is arriving where) better than you can estimate **true physical queue length**.

- **Volume/counts** tell you “how many passed the detector.” Under oversaturation, served volume can remain flat even while queues grow.
- **Occupancy** on advance detectors inside a queueing region can detect onset/growth of queuing even when served volumes don’t change, which is why traffic-responsive selection often uses occupancy (or a combined value like V+kO) and places detectors away from the stop line. [FHWA Traffic Signal Timing Manual, Ch. 9](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm)

Practical implication:
- Use **multi-signal state inference**: counts (flow), occupancy (queue presence), and controller termination types/max-outs (capacity stress).

### 1.3 Estimation methods (implementation-ready)
Pick the simplest estimator that you can validate and keep healthy.

#### A) Movement arrival rate estimation (λ)
- **From detectors:** estimate arrivals per movement from counts / short aggregation windows (e.g., 10–30s).
- **From probes:** infer approach demand shifts via travel time increases + upstream speed drops (useful for boundary links).

Handle missing detection:
- If a detector is unhealthy, fall back to:
  - correlated detectors on the same approach,
  - historical profile by TOD,
  - probe-based proxy,
  - or “unknown” with widened uncertainty.

#### B) Turning rate estimation (β)
- Prefer direct measurement (separate lane group detectors). If not available:
  - infer from downstream approach flows (conservation-of-flow),
  - or use a calibrated turning matrix that updates slowly (hours/days), not aggressively (minutes).

#### C) Queue / oversaturation inference (q)
Use a **queue state proxy** rather than claiming exact queues unless you have validated queue sensors.

Recommended multi-signal approach:
- **Split failure likelihood via high GOR + high ROR5** (both typically ≥ 80%) as an oversaturation/cycle-failure signal in ATSPM practice. [FHWA ATSPM Use Cases (Split Failure)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)
- **Max-outs / force-offs** frequency as another stress indicator and as a detector health clue (e.g., high max-outs overnight can indicate continuous calls). [FHWA ATSPM Use Cases (Watchdog + Termination)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)
- **Occupancy patterns**: sustained high occupancy on advance detectors within a queueing region can indicate growing queues even if served volumes do not. [FHWA Traffic Signal Timing Manual, Ch. 9](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm)

If you have only stop-bar detection:
- treat “queue length” as **unobservable**; estimate **degree-of-saturation/stress** and limit actions to conservative templates.

### 1.4 Uncertainty propagation (confidence → robust scoring)
Your twin should carry **confidence intervals** for the state it uses to score actions:
- queue proxy confidence (low/med/high)
- turning rate confidence (CI or quantiles)
- detector health confidence
- probe coverage confidence

How uncertainty affects action scoring:
- **Risk penalty:** penalize options that rely on uncertain state or that have large downside under plausible errors.
- **Worst-case ranking:** score candidate action under multiple plausible states (e.g., demand high/nominal/low, turning ±Δ) and choose the action with best *bounded downside*.
- **Conservatism knob:** when uncertainty is high, shrink allowed action magnitudes and/or choose template-only actions.

MPC literature commonly frames this as dealing with uncertain dynamics/disturbances and uses robust/stochastic variants; a practical takeaway is to favor architectures (hierarchical/distributed) and model choices that stay computationally feasible while coping with uncertainty. [A Survey of MPC Methods for Traffic Signal Control](https://www.ieee-jas.net/article/doi/10.1109/JAS.2019.1911471)

### 1.5 Staleness rules (data age gates)
Define hard gates: **if inputs are stale, do not optimize**.

Suggested starting thresholds (tune per comms reality):
- Controller events/state: **> 2× replan interval** → stale → freeze to baseline.
- Detector aggregates: **> 2× aggregation window** → stale → widen uncertainty; if sustained → freeze.
- Probe feed: **> 5 minutes** → stale → ignore (do not block control if other feeds are healthy).

If any required feed is stale:
- enter **SAFE_HOLD** mode: stop sending new actions; maintain last known good plan (or revert to base plan) until health recovers.

---

## 2) Scope and network coupling (corridor vs subnetwork)

### 2.1 Choosing control scope
Choose the smallest scope that captures the coupling you must respect.

- **Single intersection:** OK if queues do not spill across boundaries and coordination is not critical.
- **Corridor (2–10 signals):** needed when offsets/progression matter or when upstream decisions can protect downstream bottlenecks.
- **Subnetwork (corridor + key side streets):** required when spillback blocks adjacent intersections or diversion affects neighborhood streets.

MPC traffic-control surveys emphasize that centralized MPC captures interactions best but can be computationally heavy; hierarchical/distributed approaches often trade off optimality for feasibility and resilience. [A Survey of MPC Methods for Traffic Signal Control](https://www.ieee-jas.net/article/doi/10.1109/JAS.2019.1911471)

### 2.2 Coupling that must be modeled
At minimum, model these couplings if you operate in coordination:
- **Offsets/progression:** if an action changes splits/offsets at one node, it affects arrivals downstream.
- **Spillback blocking:** downstream link storage limits cap upstream discharge.
- **Queue storage constraints:** do not release more than downstream can store.

### 2.3 Control region vs observation region
Use the **observation region** to prevent boundary artifacts:
- Control region: intersections you can actuate.
- Observation region: *one intersection upstream and downstream* (and critical side street links) you observe and model as boundary flows.

Why: demand and constraints arrive from outside the controlled set; you need boundary inflows/outflows to avoid “optimizing into a wall.”

### 2.4 Runtime resizing rules (dynamic scope)
Expand scope when:
- downstream spillback risk rises (queue/storage constraints threatened)
- split failures cluster across adjacent intersections
- incident diversion is detected (probe TT spikes + volume/occupancy shifts)

Shrink scope when:
- compute SLA is threatened
- comms coverage is partial (partitioning)
- uncertainty is high outside a core set

Implementation pattern:
- keep a **pre-approved scope ladder** (Single → 3-node corridor → full corridor → corridor+side streets) and only move one step per cooldown window.

---

## 3) Stability / anti-oscillation controls (make it operational)

Traffic-responsive and adaptive selection commonly uses **hysteresis** to prevent oscillating across thresholds, and still may suffer from plans changing too frequently if not tuned. [FHWA Traffic Signal Timing Manual, Ch. 9](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm)

### 3.1 Stability guardrails (minimum set)
- **Minimum dwell time:** once you change a plan/offset group, hold it for *at least* `DWELL_MIN` (e.g., 5–15 minutes for plan selection; 2–5 minutes for bounded split nudges).
- **Hysteresis on triggers:** require the trigger condition to exceed a threshold for `PERSIST_N` consecutive windows before acting.
- **Cooldown:** after an action, disallow further changes for `COOLDOWN` unless safety/performance watchdog fires.
- **Regularization penalties:** add penalties for magnitude of changes and frequency of changes (e.g., quadratic penalty on Δsplit, Δoffset, plan switches).
- **Do-nothing dominance:** only change if predicted improvement exceeds `IMPROVE_MIN` and is robust under uncertainty.
- **Flip-flop detection:** if the selected action toggles back/forth within a time window, escalate to conservative mode.

### 3.2 “Do-nothing baseline dominance” (how to encode)
Always evaluate **BASELINE** (no change) as a candidate. Choose a non-baseline action only if:
- it passes deployability gate (constraints), and
- it improves the objective by > `IMPROVE_MIN`, and
- it does not violate stability budgets (change limits), and
- its worst-case score is better than baseline.

### 3.3 Pseudocode state machine (MPC loop)

```text
States:
  INIT -> SHADOW -> ASSISTED -> AUTO
  Any state -> SAFE_HOLD

Per replan tick:
  1) Collect inputs; compute freshness + detector health (ATSPM watchdog style checks)
  2) If required feeds stale OR detector health critical:
       enter SAFE_HOLD; issue (optional) revert-to-base; log
       continue
  3) Estimate state (λ, β, queue proxy q) + uncertainty
  4) Build candidate set A:
       - include BASELINE
       - include bounded actions allowed by current mode (SHADOW/ASSISTED/AUTO)
       - apply action-space budgets (max changes per window)
  5) For each a in A:
       - run fast rollout (surrogate/micro/meso) over horizon 1–5 min
       - compute KPIs and risk penalties
       - run constraint checks:
           controller safety minimums
           policy constraints
           transition feasibility
           coordination constraints
       - if any hard constraint fails: mark infeasible
  6) Rank feasible candidates using robust score:
       score(a) = E[KPI(a)] + risk_penalty(a) + change_penalty(a)
       prefer baseline unless improvement > IMPROVE_MIN
  7) Stability guards:
       - if within DWELL_MIN / COOLDOWN: override selection to BASELINE
       - if flip-flop detected: switch to SAFE_HOLD or TEMPLATE_ONLY
  8) Actuation:
       - SHADOW: log recommendation only
       - ASSISTED: present option card to operator; require approve/deny
       - AUTO: push action; require ack; verify post-state
  9) Post-check:
       - compare predicted vs observed short-term residuals
       - if residual too high: flag drift; widen uncertainty or SAFE_HOLD
  10) Log everything (see governance section)
```

Where the “watchdog style checks” can leverage ATSPM-like daily/periodic abnormality detection rules (no data, high max-outs/force-offs, etc.). [FHWA ATSPM Use Cases (Watchdog)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)

---

## 4) Safety + policy constraint formalization (constraints as code/config)

Adaptive control guidance emphasizes that the ability to constrain *minimum/maximum phase lengths, allowed sequences, and how rapidly parameters change* is critical to safe deployment. [FHWA Traffic Signal Timing Manual, Ch. 9](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm)

### 4.1 Constraint taxonomy
Separate constraints into three buckets:

1) **Controller safety minimums (hard, always enforced):**
   - conflict matrix / phase compatibility
   - minimum green, yellow, all-red
   - pedestrian walk + clearance
   - preemption/priority transition rules (as applicable)

2) **Operational/policy rules (hard or procedural):**
   - max pedestrian wait or service frequency targets
   - school zone timing windows
   - transit commitments (TSP boundaries, max disruption)
   - emergency routes / evacuation plans
   - “do not block the box” style spillback avoidance rules (implemented via queue/storage constraints)

3) **Soft objectives (optimize, don’t guarantee):**
   - delay, stops, travel time reliability
   - arrivals on green
   - max queue / spillback risk penalties

### 4.2 Encode constraints as config
Use a versioned config structure (YAML/JSON) so it is reviewable and testable.

Example (sketch):

```yaml
constraints:
  controller:
    min_green_s: {"P2": 7, "P4": 7}
    yellow_s: {"P2": 3.5, "P4": 3.5}
    all_red_s: {"P2": 1.0, "P4": 1.0}
    ped:
      min_walk_s: {"X1": 7}
      clearance_s: {"X1": 15}
  policy:
    max_ped_wait_s: {"X1": 90}
    plan_change:
      dwell_min_s: 600
      max_changes_per_hr: 4
  network:
    storage_veh:
      "link_A_to_B": 18
    spillback_guard:
      max_occupancy: 0.85
```

### 4.3 Deployability gate (feasibility checks)
Every candidate action must pass:
- **Action feasibility:** is the target controller/corridor capable of applying it (supported API/feature) and within bounds?
- **Transition feasibility:** is the change safe to transition into given the current ring/phase state?
- **Coordination feasibility:** if in coordination, does the action respect group constraints and do offsets remain consistent?

If a candidate fails any hard constraint → it is infeasible and cannot be selected.

### 4.4 Constraint enforcement table

| Constraint | Data needed | Enforcement layer | Violation response |
|---|---|---|---|
| Conflict matrix / phase compatibility | phase definitions, conflict table | Controller | block action; log as infeasible |
| Min green / yellow / all-red | controller settings | Controller | block action; log |
| Ped walk + clearance | ped timing settings; ped calls | Controller | block action; log |
| Max ped wait (policy) | ped actuation timestamp; walk start timestamp | Supervisory + monitoring (ATSPM) | if predicted/past wait too high: penalize or forbid actions; if observed violations: SAFE_HOLD + alert |
| Split failure / oversaturation guard | GOR/ROR5, max-outs, occupancy | Supervisory scoring + monitoring | raise spillback risk; limit action magnitude; possibly trigger metering templates |
| Detector health minimum | watchdog rules (no data, max-outs/force-offs anomalies) | Supervisory | widen uncertainty or SAFE_HOLD | 
| Plan change dwell time | action history | Supervisory | override to BASELINE |

ATSPM definitions used here include:
- split failures via high GOR and high ROR5 (typically ≥ 80%) [FHWA ATSPM Use Cases](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)
- pedestrian delay definition (pushbutton to walk) [FHWA ATSPM Use Cases](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)

---

## 5) Failure modes and safe degradation

### 5.1 Compute / latency failure
Define SLAs and thresholds:
- If `runtime_p95 > REPLAN_INTERVAL * 0.8` → reduce candidate count or switch to cheaper surrogate.
- If `runtime > HARD_DEADLINE` on a cycle → drop actuation for that tick (BASELINE) and log.

Fallback ladder:
1) Full candidate set + robust scoring
2) Smaller candidate set (template-only)
3) Plan selection only (no parameter tweaks)
4) SAFE_HOLD (revert to base plan)

### 5.2 Comms disruption and partitioning
Partition rule:
- If central cannot confirm controller state/ack within `ACK_TIMEOUT`, treat that controller as **autonomous local**.

Behavior:
- Central supervisory stops sending commands.
- Local controller continues running last known safe plan/coordination.
- When comms return: require re-sync + re-entry through ASSISTED mode for N minutes.

### 5.3 Silent wrongness (model divergence)
Detect drift by comparing predicted vs observed outcomes:
- residuals for travel time / arrivals on green / occupancy
- rising split failures despite predicted improvements

ATSPM provides the kind of objective, repeated measurements (split failures, arrivals on green, termination types) that make drift detection practical. [FHWA ATSPM Use Cases](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)

Policy:
- If residual exceeds `DRIFT_THRESHOLD` for `K` consecutive windows:
  - widen uncertainty
  - reduce action magnitudes
  - if still high → SAFE_HOLD and alert engineer

### 5.4 Biased data during incidents/weather
During unusual conditions (incidents, special events, extreme weather), detection reliability and saturation flows can change; traffic-responsive/adaptive operation relies on reliable detectors and may need special plans for unusual conditions. [FHWA Traffic Signal Timing Manual, Ch. 9](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm)

Policy:
- widen uncertainty bounds
- reduce action magnitudes
- prefer pre-approved playbook plans (event plans)
- increase monitoring cadence and operator involvement

---

## 6) Governance, auditability, and operator integration

### 6.1 Required logs per decision cycle
Log every cycle/tick with enough detail to replay decisions:
- state snapshot identifier (hash)
- raw input freshness per feed
- detector health flags (watchdog conditions)
- estimated state + uncertainty (CI/quantiles)
- candidate set identifiers + versions
- constraint check results per candidate (pass/fail + reason)
- chosen action + expected KPI deltas
- runtime metrics (wall time, p95/p99 over day)
- operator override actions (approve/deny/edit) and reason codes

ATSPM practice highlights systematic use of high-resolution logs to identify abnormal activity and support troubleshooting workflows (e.g., watchdog alert categories). [FHWA ATSPM Use Cases (Watchdog)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)

### 6.2 Explainability requirements
Provide two levels:
- **10-second card:** “Why this action now” with top 3 drivers (e.g., split failures rising, occupancy high, predicted spillback risk) and key constraints respected.
- **Deep dive:** candidate comparison table and constraint audit.

### 6.3 Change control
Version and review:
- action menu definitions
- constraint configs
- scoring weights
- estimator model versions
- simulation/surrogate model versions

### 6.4 Ops KPIs (program health)
- override rate and reasons
- safety constraint violations (target: zero)
- stale-feed minutes per corridor
- runtime p95/p99
- mode distribution (SHADOW/ASSISTED/AUTO/SAFE_HOLD)
- benefit vs harm slices (e.g., travel time and reliability; ped delay)

---

## 7) Implementation artifacts (make it deployable)

### 7.1 Action menu / vocabulary (bounded)
Keep actions few, reversible, and within pre-tested bounds.

| Action | Typical magnitude bounds | Notes |
|---|---:|---|
| Plan selection (TOD/response plan) | from approved plan set | Use hysteresis + dwell (avoid frequent changes). TRPS uses thresholds/hysteresis and can still change too frequently if not tuned. [FHWA Traffic Signal Timing Manual, Ch. 9](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm) |
| Offset nudge (corridor) | ±2–5 s per step | Rate limit; coordinate group must remain coherent |
| Split nudge (phase group) | ±1–3 s per cycle or ±5% | Use penalty on Δsplit and dwell |
| “Hold/force-off” (if supported) | seconds-level bounded | Used by some adaptive systems (e.g., SCOOT/OPAC send holds/force-offs); treat as advanced mode requiring extra safety review. [FHWA Traffic Signal Timing Manual, Ch. 9](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm) |
| Meter/flush templates | pre-approved templates | Use only where spillback protection is needed; validate with ATSPM |

### 7.2 Shadow-mode acceptance criteria and rollout gates
**Shadow mode (recommend-only) acceptance:**
- runtime SLA met (p95) under expected candidate counts
- prediction errors within agreed bounds for selected KPIs
- “deployability gate” rejects infeasible actions correctly
- stability logic prevents frequent flip-flops

**Assisted actuation gate:**
- operator UX is usable (approve/deny in seconds)
- rollback works reliably
- no constraint violations during assisted period

**Auto gate:**
- sustained benefit without excessive overrides
- drift watchdog stable

### 7.3 Test plan
- **Offline simulation regression:** compare baseline vs candidate actions across representative days.
- **Replay tests:** run estimator and decision logic on recorded high-resolution event logs; validate reproducibility.
- **Field trial checklist:**
  - confirm detector health monitoring
  - confirm controller capabilities and rollback
  - confirm coordination and offset transition behavior

NCHRP work on performance-based arterial signal management emphasizes using high-resolution event data and cycle-by-cycle measures to support systematic performance measurement and operations improvement. [NCHRP 03-79A Final Report Volume 1](https://onlinepubs.trb.org/onlinepubs/nchrp/docs/NCHRP03-79A_FR-Volume1.pdf)

### 7.4 Interfaces and rollback contract
**Controller command interface should be explicit about:**
- command id + version
- expected resulting state
- acknowledgement semantics
- rollback command and timeout

Minimum contract:
1) **Push**: send action with id and target state.
2) **Ack**: controller returns accepted/rejected with reason.
3) **Verify**: supervisory reads back state after `VERIFY_DELAY`.
4) **Rollback**: if verify fails or KPIs degrade → revert to safe plan.

---

## Implementation Checklist
- [ ] Define replan interval, horizon, runtime SLA, and fallback ladder.
- [ ] Inventory available inputs (controller events, detectors, probe feeds) and define staleness gates.
- [ ] Build mapping (geometry → movements → phases → detectors).
- [ ] Define action menu (small, bounded, reversible) and version it.
- [ ] Implement constraint config (controller safety + policy) and the deployability gate.
- [ ] Implement estimator with uncertainty outputs and health-aware fallbacks.
- [ ] Implement stability guards (dwell, hysteresis, cooldown, baseline dominance).
- [ ] Implement drift detection (predicted vs observed residual checks).
- [ ] Build operator UX (10-second card + approve/deny + reason codes).
- [ ] Shadow mode for 2–4 weeks; produce validation report.
- [ ] Assisted pilot; verify rollback and monitoring.
- [ ] Auto only after meeting acceptance gates.

---

## Operations Runbook (SOP)

### Daily
- Review detector health and comms uptime (watchdog-style alerts).
- Review runtime p95/p99 and SAFE_HOLD minutes.
- Review top corridors by split failures / arrivals-on-green degradation.

### Per-incident / special event
- Switch to event playbook plans if needed.
- Increase operator involvement (ASSISTED) and tighten action bounds.
- Monitor drift residuals; if high → SAFE_HOLD.

### When alarms trigger
- **Stale feed:** stop actuation; revert to safe plan; open ticket for comms.
- **Detector health critical:** widen uncertainty; template-only; if persists → SAFE_HOLD.
- **Oscillation detected:** increase dwell/cooldown; reduce candidate set; consider SAFE_HOLD.
- **Model drift:** freeze optimization; revert; schedule calibration.

---

## Governance / Audit Runbook
- Maintain a change log for:
  - constraints config
  - action menu
  - estimator model
  - scoring weights
- Require review/approval for changes (engineering + ops + accessibility).
- Keep replay artifacts for every deployed version (inputs + decision logs).
- Audit quarterly:
  - constraint violations (should be zero)
  - overrides and reasons
  - equity/accessibility proxies (e.g., ped delay distribution)
  - safety proxies (yellow/red actuations trends)

---

## Reference Links
- FHWA Traffic Signal Timing Manual (archived), Chapter 9 (advanced concepts incl. traffic responsive plan selection, adaptive control, hysteresis, detection dependency): https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm
- FHWA Automated Traffic Signal Performance Measures (ATSPM), report index: https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm
- FHWA ATSPM Sample Use Cases (watchdog, termination diagrams, split failures, pedestrian delay): https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm
- NCHRP 03-79A Final Report Volume 1 (performance-based arterial signal management; high-resolution event data): https://onlinepubs.trb.org/onlinepubs/nchrp/docs/NCHRP03-79A_FR-Volume1.pdf
- Ye et al., “A Survey of Model Predictive Control Methods for Traffic Signal Control” (MPC frameworks, models, architectures, uncertainty considerations): https://www.ieee-jas.net/article/doi/10.1109/JAS.2019.1911471

---

## Completion Checklist
- ✅ (1) State estimation methods + observability limits + uncertainty propagation + staleness rules: see **“1) State estimation”**
- ✅ (2) Scope/network coupling + control vs observation region + resizing rules: see **“2) Scope and network coupling”**
- ✅ (3) Stability/anti-oscillation controls + pseudocode/state machine: see **“3) Stability”**
- ✅ (4) Safety + policy constraints as code/config + deployability gate + constraint table: see **“4) Safety + policy constraint formalization”**
- ✅ (5) Failure modes/safe degradation (compute, latency, comms partitioning, silent wrongness, biased data): see **“5) Failure modes and safe degradation”**
- ✅ (6) Governance/auditability/operator integration (required logs, explainability card, change control, ops KPIs): see **“6) Governance…”**
- ✅ (7) Implementation artifacts (action menu table, rollout gates, test plan, interfaces/ack/rollback): see **“7) Implementation artifacts”**
- ✅ Appended sections: Implementation Checklist, Operations Runbook (SOP), Governance / Audit Runbook, Reference Links, Completion Checklist

---

Cross-links: Related ideas include what-if button, anti-jam nudges, and green waves.
