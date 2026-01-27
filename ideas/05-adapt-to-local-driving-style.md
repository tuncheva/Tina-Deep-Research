# 05) Adapt to Local Driving Style: Calibrated Traffic Behavior

## Catchy Explanation
Traffic signals shouldn’t assume everyone drives like the textbook. If a city’s drivers start slower, leave bigger gaps, or have more trucks, the signal plan should account for that—without rewarding unsafe behavior.

## What it is (precise)
**Adapt to local driving style** means estimating local operational parameters (not personal identities) such as **start-up lost time**, **saturation headway / saturation flow**, reaction patterns, heavy-vehicle effects, and turn friction from signal controller logs and detection. These parameters feed into timing design (splits, clearance assumptions) and digital-twin calibration so predictions and optimizations match reality. The key is **guardrailed calibration**: allow small parameter adjustments where supported by stable evidence, but cap changes so the system doesn’t “learn” unsafe norms or overfit noise.

## Benefits
- **Better throughput forecasts**: calibrated models predict queues and discharge more accurately.
- **More realistic timing**: less wasted green where discharge is slower/faster than assumed.
- **Operational diagnostics**: highlights unusual intersections (bad markings, poor visibility, aggressive blocking).
- **Improved simulation validity**: twin outputs are closer to field reality.

## Challenges
- **Noisy measurements**: detector errors and missing data can corrupt estimates.
- **Feedback loops**: timing changes alter behavior; you must avoid chasing your own tail.
- **Equity/safety risk**: “adapting” must not privilege aggressive driving.
- **Data governance**: even aggregate behavior analysis must be privacy-safe.

## Implementation Strategies

### Infrastructure Needs
- **High-resolution detection/logs**: phase state changes, detector actuations, (ideally) per-vehicle headways.
- **Data quality gates**: stuck detector detection, time sync validation.
- **Parameter estimation pipeline**: robust statistics by movement and time-of-day.
- **Twin calibration loop**: sensitivity analysis and validation against field KPIs.
- **Guardrails**: caps on parameter drift, freeze windows after deployments.

### Detailed Implementation Plan
#### Phase 1: Baseline Audit and Data Readiness (Weeks 1–4)
The agency should begin by selecting a small set of intersections (for example, 3–5) that already have reliable detection and high-resolution controller logging, because parameter estimation is only as good as the event data. The traffic engineer and signal maintenance supervisor should verify that the phase-to-movement mapping is correct, that detector channels are documented, and that time synchronization is stable across cabinets, because clock drift can corrupt headway calculations. The data team should then compute baseline estimates of start-up lost time and saturation headway for each movement using robust statistics, and it should produce a data quality report that flags detectors with stuck, chattering, or missing data behavior.

- **Roles**: traffic engineer (movement definitions and bounds), maintenance/signal technician (field validation), data analyst (baseline estimation), IT/network (time sync and access).
- **Deliverables**: baseline parameter map by movement and time-of-day, detector QA report, and a documented parameter dictionary.
- **Risks**: detectors may be mis-mapped; insufficient samples for some movements; time sync drift creates artificial headway variation.
- **Acceptance checks**: baseline estimates are reproducible, detector QA passes minimum thresholds, and movement mappings are signed off.

#### Phase 2: Estimation, Segmentation, and Confidence Scoring (Weeks 5–12)
The data team should build a parameter estimation pipeline that computes local parameters by movement and by operating context, such as time-of-day plans, day-of-week, weather conditions (if available), and heavy-vehicle prevalence on freight routes. The pipeline should use robust estimators (medians, trimmed means, outlier rejection) and should publish confidence scores based on sample size, stability over time, and detector health, because operators need to know when an estimate is reliable enough to influence timing decisions. The team should also implement drift detection so sudden changes in estimated saturation flow trigger investigation rather than automatic retiming, because sudden jumps often indicate detection faults or construction impacts.

- **Roles**: data engineer (pipeline), analyst/statistician (estimators and confidence), traffic engineer (segmentation rules), maintenance (investigation of drift alarms).
- **Deliverables**: parameter service/API, confidence and drift metrics, and weekly parameter reports for pilot sites.
- **Risks**: over-segmentation reduces sample sizes; detector errors produce false drift alarms.
- **Acceptance checks**: confidence scores behave sensibly (low samples → low confidence), drift alarms align with known field changes, and parameter outputs are stable across weeks on normal days.

#### Phase 3: Twin Validation and Sensitivity Analysis (Weeks 13–20)
The modeling team should integrate the locally estimated parameters into the corridor or network twin and should re-run calibration so the simulation reproduces observed discharge rates and queue formation. The team should run sensitivity analysis to identify which parameters materially affect key KPIs such as split failures, arrivals-on-green, and p95 queue lengths, because this avoids spending effort on parameters that do not change outcomes. The engineering team should define a validation tolerance (for example, travel time and queue errors within agreed bounds) and should document when the twin is trustworthy enough to support timing changes.

- **Roles**: traffic modeler (twin calibration), data analyst (validation datasets), traffic engineer (acceptance thresholds), operations representative (operational realism).
- **Deliverables**: validated calibration workflow, sensitivity report, and a “use conditions” document for when parameter-driven updates are permitted.
- **Risks**: the twin can be overfit to the pilot period; missing pedestrian/transit interactions reduce realism.
- **Acceptance checks**: simulation accuracy improves when using local parameters, and sensitivity results justify which parameters will be used operationally.

#### Phase 4: Guardrailed Deployment into Timing Practice (Weeks 21–32)
The agency should treat the parameter estimates as decision support for retiming rather than as an automatic optimizer at first, because timing changes affect behavior and can create feedback loops. The traffic engineer should apply small, reversible adjustments to splits, clearance assumptions, or coordination settings where the parameter evidence is strong, and the team should implement a cooldown window after major plan changes so the estimation pipeline does not “chase” the post-change transient. The operations team should monitor KPIs (split failures, arrivals on green, queue spillback, pedestrian delay) and should roll back changes that degrade safety proxies or violate policy constraints, even if throughput improves.

- **Roles**: traffic engineer (timing changes), operations (monitoring and rollback), analyst (before/after evaluation), maintenance (field checks if behavior changes).
- **Deliverables**: operational retiming procedure that references parameter reports, rollback criteria, and change logs.
- **Risks**: unintended incentives for aggressive driving if parameters are interpreted incorrectly; changes may shift delay to pedestrians or side streets.
- **Acceptance checks**: timing changes improve targeted KPIs without violating pedestrian and safety constraints, and parameter drift remains within caps.

#### Phase 5: Scaling and Recertification (Ongoing)
The agency should expand the approach intersection-by-intersection, prioritizing corridors where detection and logging quality is high, because scaling without data quality control produces unreliable results. The team should automate periodic recertification of parameter bounds and confidence scoring, and it should revalidate estimates after construction, detector upgrades, or major land-use changes. Over time, the agency can integrate this work with privacy-friendly learning and with corridor-level optimization, but it should maintain human review and auditability so the system remains defensible.

- **Roles**: program owner (governance), data/analytics (pipeline maintenance), traffic engineering (retiming policy), operations (monitoring), IT (data access/security).
- **Deliverables**: network rollout plan, periodic parameter recertification reports, and updated calibration playbooks.
- **Risks**: scaling increases maintenance burden; vendor firmware changes can alter log semantics.
- **Acceptance checks**: network coverage increases while parameter estimates remain stable and explainable, and retiming decisions remain auditable.

### Choices
- **Manual**: parameter reports inform human retiming.
- **Hybrid**: automated estimates with operator approval.
- **Automated**: continuous re-estimation with strict caps + audits.

## Technical Mechanics

### Key Parameters
- Start-up lost time
- Saturation headway / saturation flow
- Turn friction and heavy-vehicle adjustment
- Confidence scores and drift limits

### Guardrails
- Cap parameter changes per week/month.
- Do not use individually identifiable trajectories.
- Require confidence thresholds before applying updates.
- Maintain pedestrian minimums and clearance logic regardless of calibration.

---

## New: Implementation-Ready Calibration Details (with safety + equity guardrails)

### 0) Scope: what “local driving style” is (and is not)
**Local driving style (for signal operations)** is the *aggregate* way vehicles discharge and arrive at a stop line under specific operating conditions.

It is **not**:
- an excuse to “follow the fastest drivers”,
- a speed-limit setting method,
- a reason to shorten pedestrian protections, or
- a proxy for enforcement patterns.

Practical framing: treat calibration as **measurement for realism** (to predict queues and failures) while treating optimization as **policy-constrained control** (what the city allows the controller to do).

Authoritative context: FHWA describes ATSPMs as a way to leverage *high-resolution traffic controller data* to proactively manage signal performance ([`fhwahop20002.pdf`](https://ops.fhwa.dot.gov/publications/fhwahop20002/fhwahop20002.pdf)).

---

### 1) Parameter catalog (deployable)

> Note: “Typical ranges” vary by jurisdiction and geometry; include them only where you can cite an authoritative source. This doc therefore provides **definitions + methods + failure modes**, and uses citations for the data/log concepts.

| Parameter | Definition (operational) | Data required | Estimation method (high-level) | Update cadence | Failure modes / pitfalls |
|---|---|---|---|---|---|
| Start-up lost time | Time lost at start of green due to driver reaction/acceleration before reaching steady discharge | Stop-bar detector actuation (or video) + phase change times; queued discharge events | For each saturated green, estimate “effective green” vs actual; or measure first several headways and compare to saturation headway; robust median across days | Quarterly or after major geometry/plan changes | Detector timestamp drift; queue not fully formed; first vehicle not at stop line; permissive left gaps contaminate;
| Saturation headway / saturation flow | Steady-state headway (s/veh) / discharge rate under continuous queue | Stop-bar presence/actuation per vehicle (or video trajectories) + phase start/end | Extract headways after the queue reaches steady state (e.g., vehicles 4+), compute trimmed mean/median; convert to sat flow | Monthly/seasonal; freeze after plan deploy | Spillback interrupts discharge; mis-mapped detectors; mixed lane groups; short greens provide too few steady vehicles |
| Heavy-vehicle adjustment / PCE | Adjustment for trucks/buses on discharge/headway | Vehicle classification (video, WIM, CV class) or proxy route class; headways by class | Stratify headways by class; estimate multiplicative factor vs passenger-car-only; publish confidence bands | Semiannual; corridor freight changes | Misclassification; low truck samples; bus stop near stop line biases |
| Turn-movement discharge modifiers (esp. permissive left) | Reduced discharge for permissive movements due to gap acceptance/conflicts | Phase timing + detection for subject movement + opposing occupancy/arrival pattern; optional video conflict classification | Separate protected vs permissive intervals; for permissive, model as gap-acceptance regime: estimate service rate conditional on opposing flow | Plan-dependent; reevaluate when phasing changes | Treating permissive like protected; opposing detector faults; pedestrian recall changes the conflict state |
| Platoon dispersion / arrival-type parameter (if used) | How arrivals spread between upstream and downstream signals (coordination realism) | Probe travel times (Bluetooth/CV) or upstream detection arrivals + downstream arrivals | Fit dispersion/arrival profile that matches observed arrival-on-green or arrival distributions; validate with ATSPM measures | Seasonal; after major offset changes | Probe sample bias; incidents; offset transitions; downstream bottleneck dominates |
| Near-capacity / spillback regime discharge | Discharge/served flow when downstream blocking or oversaturation interrupts flow | Detector occupancy patterns, queue spillback indicators, downstream link status (if available), split-failure proxies | Identify regimes (normal vs blocked) using occupancy/green utilization; estimate separate discharge rates; use as twin regime model | Triggered recalibration after geometry/network change | Misclassifying blockage as “slow drivers”; missing downstream sensing; construction/special events |

ATSPM concept grounding: ATSPMs are built around a **high-resolution controller log** (events + timestamps) and can work with existing infrastructure; detection enables additional measures ([`fhwahop20002.pdf`](https://ops.fhwa.dot.gov/publications/fhwahop20002/fhwahop20002.pdf)).

---

### 2) Data Quality & Missingness Budget (vendor-neutral)

FHWA emphasizes that ATSPMs rely on controller event data with timestamps (high-resolution logs) and optional detection for richer measures ([`fhwahop20002.pdf`](https://ops.fhwa.dot.gov/publications/fhwahop20002/fhwahop20002.pdf)). That makes **data quality gates** a first-class safety feature.

#### 2.1 Common failure modes (what breaks calibration)
- **Stop-bar detector issues**: misplacement (not at stop bar), stuck-on/stuck-off, chattering, sensitivity drift.
- **Video detection issues**: occlusion (queues, trucks), nighttime glare, rain/snow, lens contamination, calibration drift.
- **Classification errors**: heavy vehicles mislabeled (especially buses vs trucks), turning movements confused, pedestrian/bike miss-detections.
- **Clock/timestamp drift**: cabinet clock drift, server ingest delays, time-zone/DST mistakes.
- **Phase/movement mapping errors**: channel reassignment after maintenance, incorrect lane group mapping in the database.

#### 2.2 Minimum viable data requirements (per estimate)
Use these as *floor* requirements; stricter is better.

| Estimate | Minimum viable inputs | Minimum sample concept | Notes |
|---|---|---|---|
| Start-up lost time | Accurate phase start time + per-vehicle departure times at stop line | Many saturated cycles across multiple days | Avoid “near-saturated” greens; require queued first vehicle |
| Saturation headway/flow | Per-vehicle headways during sustained queue discharge | Enough steady-state vehicles per movement/time bin | Require evidence of continuous queue (e.g., occupancy/queue present at green start) |
| Heavy-vehicle factor | Vehicle class + headway/time at stop line | Enough truck/bus observations per bin | Consider pooling across similar sites/corridors |
| Permissive left modifier | Protected vs permissive interval identification + opposing flow | Enough permissive service opportunities | Separate by opposing volume regimes |
| Dispersion/arrival type | Upstream release + downstream arrival distribution | Enough matched platoons | Use probes when detection lacks upstream match |

#### 2.3 Quantitative gates (missingness + samples + outliers)
Set agency-specific numeric thresholds, but enforce the *structure* below:

- **Missingness gate** (per detector stream per day):
  - compute % minutes with invalid state (missing/no data), % minutes stuck-on, % minutes stuck-off.
  - if any exceed threshold → **freeze updates** for all dependent parameters.
- **Sample size gate** (per parameter per bin):
  - require minimum number of saturated cycles **and** minimum number of steady-state headways.
  - if insufficient → increase pooling (wider time window / similar sites) or fall back to defaults.
- **Outlier gate**:
  - reject headways outside physically plausible bounds (agency-defined) and reject cycles with obvious clock jumps.
  - prefer robust statistics (median/trimmed mean) over mean.

#### 2.4 Confidence representation + fallback strategy
Every published parameter should ship with:
- `estimate`, `confidence_score` (0–1), `sample_size`, `lookback_window`, `data_health_status`, and `last_good_version`.

Fallback rules:
- If `data_health_status != OK` → **do not update**; use last-good.
- If confidence is low but stable → allow **bounded drift** only.
- If confidence is low and unstable → revert to baseline defaults and flag for field review.

---

### 3) Distinguish behavior vs geometry/control (avoid misattribution)

#### 3.1 Required site metadata schema (minimum)
To avoid “learning driving style” when the real cause is geometry/control, require a metadata record per intersection and per movement:

- Intersection ID, movement ID/lane group ID, phase(s)
- # lanes serving lane group; lane widths (if available)
- Grade (approach), curvature/sight distance proxies (if available)
- Turn bay length/storage; channelization/islands
- Curbside friction indicators: parking/loading, bus stop near stop line, driveway density
- Posted speed limit and (if policy exists) **target speed**
- School zone / high-pedestrian area flags
- Transit priority/preemption presence
- Downstream bottleneck flags and distance to downstream stop line
- Detector type + placement notes (stop-bar vs advanced) + last maintenance date

#### 3.2 Stratification patterns (recommended)
Stratify estimates so you don’t mix regimes:
- **By movement** (through vs left vs right; protected vs permissive)
- **By plan/time-of-day** (AM peak, midday, PM peak; weekend)
- **By weather/visibility** (if available)
- **By incident/event flags** (construction, special events)
- **By blockage regime** (normal discharge vs spillback)

#### 3.3 Detect “this isn’t driving style” (exclusion/regime split)
Use indicators that suggest interruption/oversaturation:
- high stop-bar occupancy *during green* without corresponding discharge,
- repeated split failures (queue not served within green),
- downstream occupancy indicates blocking,
- abrupt drop in discharge rate aligned with downstream signal cycles.

Operational rule:
- **Exclude** interrupted cycles from “normal saturation flow” estimation.
- **Model separately** a “blocked/near-capacity” regime parameter used only when spillback indicators are active.

ATSPM alignment: split failure and related measures are commonly derived from high-resolution controller + detection data as part of performance monitoring programs ([`fhwahop20002.pdf`](https://ops.fhwa.dot.gov/publications/fhwahop20002/fhwahop20002.pdf)).

---

### 4) Safety validation: prevent encoding unsafe norms

Safety context: FHWA’s Safe System speed management guidance emphasizes prioritizing safety (injury minimization) and ongoing monitoring/evaluation rather than optimizing only motor-vehicle throughput ([`Safe_System_Approach_for_Speed_Management.pdf`](https://highways.dot.gov/sites/fhwa.dot.gov/files/Safe_System_Approach_for_Speed_Management.pdf)).

#### 4.1 Safety outcomes/proxies to monitor when parameters change
Monitor both **direct outcomes** (if available) and **proxied risk**:

- **Speed distributions / speeding proxies**
  - probe speeds (segment/approach), radar spot speeds where available.
  - metrics: mean, 85th percentile, % above posted or target speed.
  - rationale: operating speed and speed distributions are central to speed management practice (definitions and use discussed by FHWA) ([`Safe_System_Approach_for_Speed_Management.pdf`](https://highways.dot.gov/sites/fhwa.dot.gov/files/Safe_System_Approach_for_Speed_Management.pdf)).
- **Harsh braking / hard acceleration proxies** (if probe/CV data supports)
  - metrics: events per 1,000 vehicles, by approach and time-of-day.
  - limitation: biased toward equipped vehicles; validate against spot observations.
- **Red-light running (RLR) proxies**
  - from detection/video: red entry events, “red-light actuation” style indicators if present in local ATSPM stack.
  - limitation: detection coverage and legal definitions vary.
- **Pedestrian/bike conflict proxies** (if video analytics used)
  - metrics: post-encroachment time (PET), time-to-collision (TTC) distributions.
  - limitation: video conflict measures are *surrogates*; do not equate to crashes.

#### 4.2 Non-negotiables expressed as measurable gates
Calibration must not change controller/timing rules that protect safety:

- **Clearance interval integrity**
  - yellow + all-red must meet agency policy/standards; do not shorten below required.
  - gate: any change proposal that reduces clearance below configured minimum → **reject**.
- **Pedestrian minimums**
  - walk + flashing don’t-walk and ADA features must remain compliant.
  - gate: predicted or measured reduction below minimum walk/clearance → **reject**.
- **No illegal phase sequences**
  - maintain valid ring-barrier structure and intergreens.
  - gate: any simulated/field plan that produces conflicting greens → **reject**.
- **Max pedestrian wait (if policy exists)**
  - gate: do not exceed policy maximum (or publish exception justification).

#### 4.3 Shadow → production go/no-go protocol
Use a staged deployment so “learning” cannot silently degrade safety.

**Stage A — Shadow calibration (no field changes)**
- Run new parameters in the twin only.
- Compare predicted discharge/queues to observed for a holdout period.

**Stage B — Limited production (pilot windows)**
- Apply small, reversible timing changes at limited sites.
- Enforce a cooldown window after deployments (already noted above).

**Stage C — Broad production**
- Expand to corridors once safety + equity gates pass.

**Pre/post windows**
- Minimum: compare matched weeks (e.g., 2–4 weeks before vs 2–4 weeks after), excluding abnormal days.

**Acceptance criteria (examples; agency must set numeric targets)**
- No statistically meaningful increase in:
  - 85th percentile approach speeds beyond policy target,
  - RLR proxy events,
  - harsh braking events,
  - pedestrian wait time in priority areas.

**Rollback triggers**
- Any gate violation above.
- Data quality failure (freeze).
- Operator reports of near-misses, unusual queues, or complaint spikes.

#### 4.4 Safety regression tests in the twin
Treat parameter updates like software releases:
- **Speed policy test**: model must not increase predicted approach speeds beyond target-speed policy (where policy exists) ([`Safe_System_Approach_for_Speed_Management.pdf`](https://highways.dot.gov/sites/fhwa.dot.gov/files/Safe_System_Approach_for_Speed_Management.pdf)).
- **RLR risk test**: red occupancy / red-entry proxy counts must not increase beyond tolerance.
- **Clearance test**: verify all clearance intervals and pedestrian timings remain unchanged (unless explicitly reviewed).
- **Blocking test**: ensure changes do not increase frequency of spillback regime activation.

---

### 5) Representativeness + equity impacts (operational)

Equity context: FHWA’s Safe System speed management guidance explicitly ties safe road users and equity considerations to speed management decisions and recognizes unequal burdens across communities ([`Safe_System_Approach_for_Speed_Management.pdf`](https://highways.dot.gov/sites/fhwa.dot.gov/files/Safe_System_Approach_for_Speed_Management.pdf)).

#### 5.1 Why “local driving style” can embed inequity
“Local driving style” is shaped by:
- street design and land use context,
- enforcement patterns and compliance culture,
- freight routing and transit operations,
- who is exposed to danger and delay (especially VRUs).

If you calibrate purely to maximize vehicle throughput, you can unintentionally:
- privilege dominant movements (often mainline vehicles),
- worsen pedestrian wait in equity focus areas,
- normalize speeding/aggressive discharge as “efficient”.

#### 5.2 Concrete equity guardrails
- **Calibrate for realism; optimize only within city-rule constraints**
  - parameters describe reality; optimization must respect policy (speed targets, ped minimums, school zone constraints).
- **Segment by mode/class/time-of-day**
  - do not let freight-heavy peaks define all-day parameters; separate bus routes, school arrival windows.
- **Do not degrade vulnerable-user service**
  - track pedestrian delay, max ped wait, and (where available) bicycle service measures.
- **Report person-delay (not just vehicle-delay)**
  - combine volumes + occupancy assumptions to publish person-delay changes by movement.
- **Equity focus corridors require higher scrutiny**
  - tighter bounds, more conservative auto-updates, explicit sign-off.

#### 5.3 Equity review template (who checks what)

**Inputs**
- Before/after (or baseline/new-parameter) report by corridor:
  - person-delay by movement and time-of-day
  - pedestrian wait distributions (median/p95/max)
  - bus performance (on-time, dwell impacts) if relevant
  - safety proxies from Section 4

**Checks**
- Any equity-focus area shows pedestrian wait increase beyond tolerance?
- Any school-zone or high-place area shows speed proxy increase beyond target?
- Any minor-street or side-street person-delay worsens materially while mainline improves?

**Sign-off roles**
- Signals engineer (technical), operations supervisor (field), safety reviewer, equity reviewer.

**Internal publication**
- Publish a short “parameter change note” + before/after dashboard snapshot to an internal log.

---

### 6) Change management + governance for continuous updates

ATSPM practice context: FHWA positions ATSPMs as enabling proactive, objectives-based signal management and continuous performance monitoring ([`fhwahop20002.pdf`](https://ops.fhwa.dot.gov/publications/fhwahop20002/fhwahop20002.pdf)). That implies calibration updates must be treated as controlled releases.

#### 6.1 Roles
- **Signals engineer (owner)**: sets allowable bounds, reviews movement definitions, approves timing use.
- **Data engineer**: maintains pipeline, QA gates, time sync checks.
- **Ops supervisor / TMC**: monitors performance dashboards, triggers rollback.
- **Safety reviewer**: enforces Section 4 gates and safe-system policy alignment.
- **Equity reviewer**: enforces Section 5 guardrails and reporting.

#### 6.2 Versioning strategy (semver-like)
- Parameter set ID: `city-signalcal/vMAJOR.MINOR.PATCH`.
  - **MAJOR**: methodology changes (new estimator, new regime model, new metadata schema).
  - **MINOR**: new corridors/intersections, new stratification dimensions.
  - **PATCH**: data refresh within same method, small bounded drift.

Store with:
- diff summary (what changed, where, why),
- data health snapshot,
- safety/equity check results,
- approval signatures.

#### 6.3 Approval workflow
- **Auto-update allowed** (PATCH only):
  - small drift within confidence and within caps, *and* safety/equity regression tests pass in twin.
- **Human review required**:
  - large parameter shifts,
  - equity-focus corridors,
  - school zones / high VRU areas,
  - new regime detection logic,
  - clearance/ped timing touching (generally disallowed here).

#### 6.4 Cadence rules
- **Routine update frequency**: monthly (parameters), seasonal re-baseline.
- **Freeze windows**: construction periods, special events, immediately after major retiming.
- **Recertification**: after detector upgrades, firmware changes, geometry changes.

#### 6.5 Operator-facing communications
- Provide “what changed / expected effect / rollback plan” to field staff.
- Keep a simple mapping from parameter change → likely observable effect (e.g., shorter queues on approach A, possible longer ped wait on cross street unless constrained).

---

## MVP Deployment
- 3–5 intersections.
- Weekly parameter estimates.
- Operator-reviewed timing adjustments.

## Evaluation
- Prediction accuracy improvement (queues/travel times).
- Change in split failures and arrivals on green.
- Stability: parameter drift and update frequency.

---

## Implementation Checklist
- [ ] Select pilot intersections with reliable detection + high-resolution logs (Phase 1).
- [ ] Build and validate phase-to-movement mapping and detector inventory.
- [ ] Implement data quality & missingness budget gates (Section 2).
- [ ] Implement metadata schema and stratification plan (Section 3).
- [ ] Stand up parameter estimation pipeline with confidence scoring (Phase 2).
- [ ] Integrate parameters into twin; run sensitivity + holdout validation (Phase 3).
- [ ] Define and implement safety gates + regression tests (Section 4).
- [ ] Define and implement equity guardrails + reporting (Section 5).
- [ ] Establish parameter versioning, approvals, and rollback workflow (Section 6).
- [ ] Run shadow → limited production go/no-go; document acceptance and rollback triggers (Section 4.3).

## Governance / Change-Control Runbook
1. **Propose change**: new parameter set draft produced by pipeline.
2. **Data QA gate**: verify detector health, timestamp integrity, sample-size gates.
3. **Twin regression**: run safety regression tests + performance validation.
4. **Equity review**: run person-delay + pedestrian wait reports; check equity-focus corridors.
5. **Approval**:
   - PATCH: auto-approve if all gates pass.
   - MINOR/MAJOR: require signals engineer + ops + safety + equity sign-off.
6. **Deploy**: publish version tag, update configs, set cooldown window.
7. **Monitor**: ATSPM dashboards + safety proxies; daily checks for first week.
8. **Rollback**: revert to last-good version if triggers fire; open incident ticket.
9. **Postmortem**: document root cause, update gates/metadata.

## Reference Links
- FHWA Traffic Signal Timing Manual (2008): [`fhwa_hop_08_024.pdf`](https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf)
- FHWA Automated Traffic Signal Performance Measures (ATSPM): [`fhwahop20002.pdf`](https://ops.fhwa.dot.gov/publications/fhwahop20002/fhwahop20002.pdf)
- FHWA Safe System Approach for Speed Management (2023): [`Safe_System_Approach_for_Speed_Management.pdf`](https://highways.dot.gov/sites/fhwa.dot.gov/files/Safe_System_Approach_for_Speed_Management.pdf)

## Completion Checklist
- ✅ Safety validation (non-negotiables, proxies, go/no-go, twin regression): see **Section 4**.
- ✅ Representativeness + equity impacts operationalized (guardrails, reporting, template review): see **Section 5**.
- ✅ Sensor/data limitations + explicit data quality budgets: see **Section 2**.
- ✅ Behavior vs geometry/control distinguished (metadata + stratification + regime detection): see **Section 3**.
- ✅ Change management + governance (roles, versioning, approvals, cadence, comms): see **Section 6** + **Runbook**.
- ⚠️ Concrete “typical ranges” for parameters: not added (requires accessible HCM/agency numeric tables); doc is structured to plug in agency-specific ranges once sourced.

---

## References / Standards / Useful Sources
- FHWA Traffic Signal Timing Manual: https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf
- JRC report on calibration of traffic simulation models: https://publications.jrc.ec.europa.eu/repository/bitstream/JRC68403/lbna25188enn.pdf

---

Cross-links: Related ideas include green waves, what-if button, and privacy-friendly learning.
