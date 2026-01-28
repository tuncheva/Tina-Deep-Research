# 10) Stop-Line Intent Coordination: Precise Green Allocation

## Catchy Explanation
If the signal knew which movements are *actually* waiting at the stop line (left vs through vs right, plus bikes and pedestrians), it could stop wasting green on empty lanes and serve the real queue—without breaking safety rules or corridor progression.

## What it is (precise)
**Stop-line intent coordination** uses approach- and lane-level “intent” (which movements are queued and how much) to allocate green time more precisely.

Intent can come from:
- **Lane-level detection** (loops, radar, video) near the stop line and/or advance locations.
- **Connected-vehicle (CV) / V2X** signals and map alignment (SPaT/MAP) where available.

The system maps this “intent” to **signal phases** via a maintained **lane-to-phase (movement) configuration**. The controller then reallocates **splits** (and sometimes phase order within allowed logic) to reduce **wasted green**, while respecting:
- pedestrian minimums and clearance timing,
- minimum service guarantees (no starvation),
- safety/driver-expectancy constraints,
- corridor coordination boundaries.

A digital twin (or fast microsimulation/mesosimulation harness) is used to test reallocation policies and spillback impacts before field enablement.

## Benefits
- **Less wasted green**: green time goes to movements that can use it.
- **Better surge handling**: turn-heavy or event-driven patterns are served faster.
- **Improved queue stability**: prevents one movement from starving others.
- **Higher effective capacity**: better use of existing infrastructure.
- **More explainable operations**: decisions are grounded in observable “who is waiting” signals.

## Challenges
- **Detection quality**: misclassification yields bad reallocations.
- **Mapping correctness**: lane-to-phase configuration must be accurate and maintained.
- **Coordination risk**: myopic changes can break progression.
- **Safety constraints**: cannot trade away pedestrian service or clearance behavior.
- **Equity risk**: if intent signals are biased (coverage, occlusion, CV penetration), allocation can be unfair.

---

## The six required additions (implementation-ready)

### 1) Service guarantees / anti-starvation (including pedestrians)
Intent-based split reallocation must behave like a **bounded allocator with explicit guarantees**, not like an unconstrained optimizer.

**Define minimum guarantees per user class and per movement**:
- **Pedestrian service guarantee (hard)**: when pedestrian signal heads are used, WALK and the pedestrian change interval (flashing UPRAISED HAND) must be followed by a buffer interval; the sum of change interval + buffer must meet the calculated pedestrian clearance time. This is a constraint, not a tradeable objective. Source: MUTCD 2009 Part 4, Chapter 4E, [Section 4E.06](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm).
- **Pedestrian timing baseline**: MUTCD guidance discusses using 3.5 ft/s for clearance sufficiency, with provisions for slower pedestrians and extended pushbutton press; passive detection may adjust clearance based on the pedestrian’s actual walking speed. Source: MUTCD 2009 Part 4, Chapter 4E, [Section 4E.06](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm).
- **Vehicle movement minimum service (soft/hard depending on jurisdiction)**: each movement receives a minimum service frequency and max-wait cap unless preempted.

**Recommended MVP defaults (engineer-tunable)**
- **Max-wait cap** (anti-starvation): set a per-movement **maximum time-since-service** threshold; when exceeded, the movement is forced into the next feasible service opportunity (subject to ped timing and ring/barrier constraints).
- **Minimum service frequency**: each non-coordinated movement must be served at least once every *N* cycles under non-oversaturated conditions; if oversaturated, the system escalates to “equitable fixed-split mode” rather than continuing to chase noisy intent.
- **Periodic recalls as a safety valve**: if intent health is low or detector behavior is anomalous, switch selected phases to a recall strategy (e.g., minimum recall / pedestrian recall where policy requires). Recalls (minimum, maximum, pedestrian, soft) are standard controller concepts. Source: FHWA *Traffic Signal Timing Manual*, Chapter 5 ([`chapter5`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm)).

**Fairness framing (person-delay)**
When instrumentation supports it, define “harm” using **marginal person-delay** to non-served movements and pedestrians; otherwise start with seconds-based proxies and graduate. (This matches the staged approach used in Theme 09.)

**Operator-visible fairness dashboard (minimum viable)**
- Per movement: time-since-service, split-failure %, max-out/gap-out rates.
- Pedestrian: average wait, missed calls, clearance extensions invoked.

ATSPM exists specifically to provide high-resolution signal data for active performance management and includes use cases for timing, safety, and pedestrian delay reporting. Source: FHWA, *Automated Traffic Signal Performance Measures* (FHWA-HOP-20-002) [index](https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm).

### 2) Pedestrian + bicycle intent as “first-class”
The intent model must be **multi-modal** (vehicles, pedestrians, bicycles) rather than vehicles-only.

**Pedestrian intent**:
- **Call presence**: pushbutton actuation or passive detection devices; passive detection can register presence without requiring a pushbutton and can track crossing progress for adjusting timing. Source: MUTCD 2009 Part 4, Chapter 4E, [Section 4E.08](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm).
- **Timing constraints**: WALK + pedestrian clearance + buffer must meet clearance time. Source: MUTCD 2009 Part 4, Chapter 4E, [Section 4E.06](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm).
- **LPI interactions**: LPI may be used to reduce turning conflicts at high ped + turning volumes, and guidance includes minimum duration and accessibility considerations. Source: MUTCD 2009 Part 4, Chapter 4E, [Section 4E.06](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm).

**Bicycle intent**:
- Treat bicycles as a separate queue class where detection exists (bike loops, video/radar classification, or approach detection).
- If bikes share lanes, represent bicycle presence as a “slow discharge modifier” rather than a separate queue if classification is unreliable.

**Failure modes and constraints (avoid vehicle-centric allocation)**
- If ped/bike intent is missing or uncertain, the allocator must **not** infer “no demand.” It must default to policy floors (e.g., ped recall where required, minimum service frequency).
- If an LPI is enabled, the allocator must treat it as part of the phase’s non-negotiable service envelope.

### 3) Architecture choice: edge vs central vs hybrid (with latency budget + comms-loss behavior)
Make architecture explicit because it affects latency, survivability, and coordination.

**Recommended default: hybrid**
- **Edge (intersection controller / cabinet compute)** runs the real-time bounded allocator each cycle (or sub-cycle) because it needs low latency and must fail safe.
- **Central (TMS/ATS/Cloud)** hosts model training, policy authoring, corridor-level constraints, configuration management, and monitoring.

**Latency budget (practical)**
- “Real-time” decisions should be made within the current cycle (or within a sub-cycle window if the platform supports it).
- Central coordination updates (new bounds, new policy versions) can be slower (minutes) but must be safely rolled out.

**Comms-loss behavior (safe fallback)**
- Loss of comms to central: continue to operate using the last known-good policy and mapping version until TTL expires.
- If policy TTL expires or mapping mismatch is detected: disable intent reallocation and revert to baseline controller operation (actuated and/or coordination plan).

### 4) Uncertainty-aware allocation + quality budget table (sensor→failure→mitigation→disable rules)
Intent is noisy; allocation must be **uncertainty-aware**.

**Principles**:
- Use intent only when the system is confident the signal is real.
- When confidence drops, shrink the “authority” of the allocator (smaller split shifts) and/or fall back to baseline actuated/coordination behavior.

**Controller-parameter grounding** (used as guardrails)
Actuated control parameters such as minimum green, maximum green, passage time, recalls, and detection configuration are defined and discussed in FHWA’s Signal Timing Manual. Source: FHWA *Traffic Signal Timing Manual*, Chapter 5 ([`chapter5`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm)).

**Quality budget (template)**

| Sensor / signal | Failure mode | Detection / health checks | Mitigation | Disable / fallback rule |
|---|---|---|---|---|
| Stop-line presence (per lane) | stuck-on, occlusion | watchdog, stuck-call duration, variance | ignore lane, use phase-level demand | if stuck-on > threshold for M cycles → disable lane intent |
| Lane classification (L/T/R) | mapping drift, mislabel | mapping version, field audit diffs | freeze learning, require re-survey | if mapping version invalid → disable allocator |
| Advance detection | missed arrivals, noisy pulses | consistency with stop-line presence | smoothing, plausibility bounds | if inconsistency persists → reduce authority |
| Ped call | missed calls, passive sensor noise | compare actuation vs ped served | treat as recall during fault | if missed-call rate spikes → enforce ped recall |
| CV-derived intent | low penetration, sample bias | coverage per approach/time-of-day | CV additive only | if coverage < min → zero weight CV |
| Clock/cycle alignment | drift breaks logs | time sync checks | local-only logging | if drift > threshold → keep control local, degrade analytics |

Where possible, publish this table as part of the site’s “go-live” packet and re-evaluate after any detection change.

### 5) Corridor/network impacts: caps, recovery-to-coordination rules, and monitoring
Intent allocation that ignores coordination will harm corridor travel time and progression.

**Coordination constraint envelope**
- Bound split changes so they don’t violate:
  - time-of-day coordination plan assumptions,
  - force-off / yield points,
  - minimum pedestrian timing.

**Recovery-to-coordination rules**
- **Rate-limit** split changes cycle-to-cycle.
- **Snap-back** to baseline splits gradually (e.g., exponential decay over K cycles) to avoid oscillations.
- **Coordination-first mode** during peak progression windows.

**Monitoring**
Use ATSPM-style measures and diagrams to detect coordination harm:
- split failures,
- phase termination patterns,
- progression quality proxies (AOG-like),
- spillback indicators from occupancy/blocked-box proxies.

ATSPM documentation includes timing and corridor-related use cases (e.g., coordination and split failure diagrams). Source: FHWA, *Automated Traffic Signal Performance Measures* (FHWA-HOP-20-002) [index](https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm).

### 6) Deployable artifacts (copy/paste)

#### A) Lane-to-phase mapping QA checklist
- [ ] Field verify lane markings/signage match the map.
- [ ] For each lane group, confirm allowed movements (L/T/R, protected/permitted).
- [ ] Confirm detector channel ↔ lane association in cabinet wiring docs.
- [ ] Confirm controller phase diagram (rings/barriers) and overlaps.
- [ ] Confirm pedestrian phases and which vehicle phases conflict.
- [ ] Confirm bike detection (if any) and whether it maps to a vehicle phase or separate call.
- [ ] Version the registry and require sign-off (engineer + technician).

#### B) Intent logic state machine (pseudocode)
```text
Inputs each cycle:
  intent[movement] = {queue_estimate, arrival_rate, confidence}
  ped[movement]    = {call_present, clearance_required, confidence}
  coord            = {cycle, offset, split_bounds, force_off_schedule}

Hard constraints:
  enforce MUTCD ped timing rules for any called crosswalk
  enforce min_green / max_green bounds

Compute movement priority:
  base_score = wq * queue_estimate + wa * arrival_rate
  fairness_penalty = f(time_since_service)
  uncertainty_penalty = g(1 - confidence)
  score = base_score - fairness_penalty - uncertainty_penalty

Allocator authority:
  authority = clamp(confidence_global, 0..1)
  max_split_shift = authority * SHIFT_MAX_SECONDS

Decision:
  propose split deltas within [-max_split_shift, +max_split_shift]
  project against coordination envelope and min service guarantees
  if any hard constraint violated -> reject and revert to baseline

Fallback:
  if confidence_global < MIN_CONF or mapping_version_invalid:
     operate baseline actuated/coordination plan
     log degraded-mode reason
```

#### C) Bounded controller-parameter envelope table (starter)
Use this as an engineer-facing “allowed range” document; values should be chosen to match the site plan and controller capabilities.

| Parameter | What it does | Typical guidance / notes | Source |
|---|---|---|---|
| Minimum green | shortest green duration | set to satisfy driver expectancy; typical ranges vary by facility type | FHWA STM Ch. 5 Table 5-3 [`chapter5`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm) |
| Maximum green | longest green w/ conflicting demand | example ranges vary by facility type; must prevent starvation and guard detector failures | FHWA STM Ch. 5 Table 5-5 [`chapter5`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm) |
| Passage time / gap | max allowable time between calls before gap-out | typical efficient range depends on detection; discussion includes MAH guidance | FHWA STM Ch. 5 Section 5.4.2 [`chapter5`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm) |
| Recall modes | ensure service even under low demand / failures | minimum/maximum/ped/soft recall definitions and uses | FHWA STM Ch. 5 Section 5.4.1 [`chapter5`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm) |
| Ped intervals (WALK/FDW/buffer) | ensures safe ped service | buffer interval at least 3 s prior to releasing conflicting vehicles; clearance calculations | MUTCD Ch. 4E Section 4E.06 [`part4e`](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm) |

#### D) Shadow-mode acceptance criteria + rollback triggers
**Shadow-mode acceptance criteria** (before enabling live control):
- ≥X% of cycles: proposed deltas stay within the coordination envelope.
- 0 pedestrian timing violations (WALK/FDW/buffer/clearance).
- Proposed decisions reduce wasted green without increasing split failures beyond threshold.

**Rollback triggers (production)**
- ped missed-call spike,
- split failures increase above threshold,
- confidence falls below minimum,
- mapping version mismatch.

#### E) Test plan (field + simulation)
- **Unit tests**: mapping parsing, constraint projection, fallback triggers.
- **Replay tests**: run allocator against recorded high-resolution events; confirm no ped violations.
- **Twin scenarios**:
  - turn-surges (event release),
  - spillback / blocked intersection,
  - detector stuck-on and stuck-off,
  - CV penetration dropouts.
- **Field shadow mode**: compute proposed split deltas but do not apply; compare KPIs.

---

## Implementation Strategies

### Infrastructure Needs
- **Lane-level detection** at/near stop line.
- **Lane-to-phase mapping** data model (versioned).
- **(Optional) SPaT/MAP interface** for richer connected-vehicle integration.
- **Twin evaluation** for spillback and network effects.
- **ATSPM telemetry** to quantify wasted green and split failures.

### Detailed Implementation Plan

#### Phase 1: Lane-to-Phase Mapping and Baseline Measurement (Weeks 1–4)
The agency should begin by verifying lane geometry, signage, and striping at the pilot intersection(s), because the system can only allocate green correctly if it knows which lanes feed which movements.

Build a versioned lane-to-phase mapping registry that includes lane groups, permitted movements, detector channels, and associated controller phases, and confirm this mapping matches cabinet wiring and detector configuration.

Compute a baseline for wasted green, split failures, queue lengths, arrivals-on-green, and pedestrian delay so the project can prove that “intent” improves operations rather than just adding complexity. ATSPM is explicitly intended to provide high-resolution data for actively managing traffic signal performance. Source: FHWA ATSPM [`index`](https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm).

- **Roles**: traffic engineer (lane-phase definitions), signal technician (wiring/detector confirmation), data analyst (baseline KPIs), operations (field observations).
- **Deliverables**: mapping registry (versioned), baseline KPI report, detector/channel documentation pack.
- **Risks**: lane-use changes (construction, restriping) can invalidate mappings; baseline may be biased by unusual weeks.
- **Acceptance checks**: mapping signed off; baseline KPIs reproducible.

#### Phase 2: Intent Estimation from Detection (Weeks 5–12)
Implement intent estimation that computes, for each movement, an estimate of queued users and demand intensity based on stop-line presence, advance detection, and/or video/radar classification.

Add plausibility checks (e.g., maximum queue bounded by lane storage, consistency between upstream and stop-line detectors) and smoothing so brief detection noise does not trigger oscillating allocations.

Publish a health score for intent quality so that when detection is unreliable the controller can fall back to baseline behavior.

- **Roles**: data engineer (intent pipeline), traffic engineer (movement logic), maintenance (detector QA), QA engineer (test scenarios).
- **Deliverables**: intent feed/API with health checks, detector QA report, validation plots vs spot field observations.
- **Risks**: lane misclassification can starve movements; detection outages produce unstable estimates.
- **Acceptance checks**: intent behaves plausibly; low-health periods correctly flagged.

#### Phase 3: Split Reallocation Policy, Guardrails, and Twin Validation (Weeks 13–20)
Implement a **bounded reallocation policy** that shifts split time by small amounts and preserves minimum service for all movements and all pedestrian timing constraints.

Use actuated-controller concepts (minimum green, maximum green, passage time, recalls) as the guardrail envelope for what the allocator is allowed to change. Source: FHWA Signal Timing Manual Chapter 5 ([`chapter5`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm)).

Validate the policy in a twin across representative demand patterns and explicitly test surge and turn-heavy conditions.

Define revert logic that returns to baseline when intent health drops or performance degrades.

- **Roles**: traffic engineer (policy), modeler (twin scenarios), software engineer (implementation), safety/accessibility reviewer (ped constraints).
- **Deliverables**: validated policy, guardrail spec, simulation report.
- **Risks**: overreacting to noise; coordination degradation.
- **Acceptance checks**: ped constraints preserved; simulation shows reduced wasted green without unacceptable side-street delay.

#### Phase 4: Connected-Vehicle Enrichment (Optional) (Weeks 21–32)
If the city has V2X infrastructure, enrich intent estimation using SPaT/MAP alignment and (where available) CV-derived approach trajectories.

Treat CV data as additive and maintain robust fallback to detection-only operation.

SPaT deployments require producing SPaT outputs and pairing them with MAP geometry so vehicles can interpret which approach/phase applies; the NOCoE SPaT Challenge Implementation Guide summarizes SPaT and MAP roles and a typical SPaT deployment architecture. Source: NOCoE SPaT Challenge Implementation Guide [`Implementation-Guide`](https://www.transportationops.org/spatchallenge/resources/Implementation-Guide).

- **Roles**: ITS engineer (CV integration), data engineer (fusion), security/privacy reviewer (governance), traffic engineer (policy impact).
- **Deliverables**: hybrid intent estimator, coverage report, updated validation results.
- **Risks**: low penetration yields limited value; integration complexity increases maintenance burden.
- **Acceptance checks**: hybrid improves intent quality when coverage exists and falls back cleanly.

#### Phase 5: Expansion and Operationalization (Ongoing)
Expand intersection-by-intersection, prioritizing sites with high wasted-green rates and reliable lane-level detection.

Implement a maintenance process to update lane-to-phase mapping after restriping or construction, and monitor intent health continuously with ATSPM-style measures.

- **Roles**: program owner (prioritization), engineering (policy tuning), operations (monitoring), maintenance (mapping updates).
- **Deliverables**: rollout plan, mapping update procedure, monthly performance reports.
- **Risks**: mapping drift; inconsistent detection quality.
- **Acceptance checks**: improvements persist; mapping registry remains accurate and versioned.

---

## Choices
- **Detection-only**: simplest, widely deployable.
- **CV-enhanced**: richer intent where coverage exists.
- **Hybrid**: best practical approach.

## Technical Mechanics

### Key Parameters
- Wasted green %
- Queue length and discharge estimates per movement
- Split shift bounds (seconds)
- Minimum service constraints
- Pedestrian call and clearance obligations

### Guardrails
- Maintain pedestrian minimums and clearance behavior (MUTCD 4E). Source: [`part4e`](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm).
- Rate-limit split shifts.
- Cap deviation from corridor coordination envelope.
- Freeze on mapping drift or low intent health.

## MVP Deployment
- One intersection.
- 1–2 bounded split tweaks.
- Shadow mode first with KPI reporting.
- Explicit ped/bike fairness metrics published.

## Evaluation
- Wasted green reduction.
- Split failure rate change.
- Queue balance across movements.
- Corridor travel time and progression impacts.
- Pedestrian delay and missed-call rate.

---

## Implementation Checklist
- [ ] Confirm pedestrian timing constraints and accessibility requirements are satisfied (WALK/FDW/buffer/clearance) per MUTCD 4E [`part4e`](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm).
- [ ] Build and version lane-to-phase mapping registry; complete QA checklist; obtain sign-offs.
- [ ] Baseline KPIs using ATSPM measures (wasted green, split failures, ped delay) [`index`](https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm).
- [ ] Implement intent estimation + confidence scoring + fault detection.
- [ ] Define bounded reallocation envelope using controller parameters (min/max green, passage time, recalls) [`chapter5`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm).
- [ ] Twin validation across surge, spillback, and fault scenarios.
- [ ] Shadow deployment with acceptance criteria + rollback triggers.
- [ ] Limited production enablement with monitoring alarms.
- [ ] Document maintenance process for mapping drift and detector health.

## Operations Runbook (SOP)

### Daily / weekly
- Review ATSPM dashboards:
  - split failures and termination patterns,
  - ped delay and missed calls,
  - abnormal max-out frequency on specific phases.
- Review intent health:
  - stuck-on detectors,
  - classification confidence drops,
  - mapping version mismatches.

### Incident response
- If detector faults or mapping drift detected:
  1) switch to degraded mode (baseline actuated/coordination plan),
  2) open maintenance ticket,
  3) record time window for KPI analysis.

### Tuning changes
- Only change bounded envelope parameters (SHIFT_MAX_SECONDS, authority thresholds) through change control; never hot-edit in the field without a rollback plan.

## Governance / Change-Control Runbook
- **Version everything**: lane-to-phase mapping registry, policy parameters, and code.
- **Sign-offs required**: signals engineer + maintenance lead + safety/accessibility reviewer.
- **Deployment process**:
  - shadow mode → limited enablement → broader enablement.
- **Rollback triggers**:
  - ped missed-call spike,
  - split failures increase above threshold,
  - confidence falls below minimum,
  - mapping version mismatch.

## Reference Links
- MUTCD 2009 Part 4, Chapter 4E (Pedestrian Control Features): https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm
- FHWA Automated Traffic Signal Performance Measures (FHWA-HOP-20-002): https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm
- FHWA Traffic Signal Timing Manual (archived), Chapter 5: https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm
- NOCoE SPaT Challenge Implementation Guide: https://www.transportationops.org/spatchallenge/resources/Implementation-Guide
- Guidance Document for MAP Message Preparation (PDF): https://engineering.virginia.edu/sites/default/files/Connected-Vehicle-PFS/Resources/MAP%20Guidance%20Document%20-%20Revision%202_06232023.pdf

## Completion Checklist
- [ ] Service guarantees / anti-starvation are explicitly stated (max-wait caps, minimum service frequency, recalls).
- [ ] Ped + bike intent treated as first-class, with failure modes and constraints.
- [ ] Architecture choice (edge/central/hybrid) plus latency and comms-loss behavior documented.
- [ ] Uncertainty-aware allocation design + quality budget table included.
- [ ] Corridor/network impacts: caps, recovery-to-coordination rules, ATSPM monitoring included.
- [ ] Deployable artifacts included: mapping QA checklist, pseudocode/state machine, parameter envelope table, shadow-mode acceptance + rollback triggers.
- [ ] End sections appended: Implementation Checklist, Operations Runbook (SOP), Governance/Change-Control Runbook, Reference Links.

---

Cross-links: Related ideas include green waves, what-if button, and fair priority credits.
