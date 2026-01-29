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

### 1) Safety validation (shadow→prod gates + measurable proxies)

Safety evaluation is a core requirement because calibration can inadvertently “ratchet” the system toward outcomes that look efficient but increase risk.

#### 1.1 Safety outcomes/proxies to monitor when parameters change
Monitor both **direct outcomes** (if available) and **proxied risk**:

- **Speed distributions / speeding proxies**
  - probe speeds (segment/approach), radar spot speeds where available.
  - metrics: mean, 85th percentile, % above posted or target speed.
- **Harsh braking / hard acceleration proxies** (if probe/CV data supports)
  - metrics: events per 1,000 vehicles, by approach and time-of-day.
  - limitation: biased toward equipped vehicles; validate against spot observations.
- **Red-light running (RLR) proxies**
  - from detection/video: red entry events / red occupancy when available.
  - limitation: detection coverage and legal definitions vary.
- **Conflict proxies (if video analytics used)**
  - metrics: PET/TTC distributions.
  - limitation: surrogates; do not equate to crashes.

#### 1.2 “Non-negotiables” expressed as measurable gates
Calibration must not justify relaxing minimum safety constraints.

- **Start-up lost time is not a performance target**
  - If observed start-up lost time decreases, treat it as an *observation* not an objective.
  - Gate: never modify timing in a way that requires unsafe acceleration to “hit” an assumed saturation headway.

- **Clearance integrity**
  - Gate: any proposal that reduces yellow/all-red below configured minimums → **reject**.

- **Pedestrian minimums**
  - Gate: any predicted/measured pedestrian timing reduction below policy/standard → **reject**.

- **No illegal phase sequences**
  - Gate: any plan that produces conflicting greens → **reject**.

#### 1.3 Shadow → limited production → broad production protocol
Use staged deployment so updates cannot silently degrade safety.

- **Stage A — Shadow calibration (no field changes)**
  - Run new parameters in the twin only.
  - Validate predicted discharge/queues against observed data.

- **Stage B — Limited production (pilot windows)**
  - Apply small, reversible timing changes at limited sites.
  - Enforce cooldown/freeze windows after changes.

- **Stage C — Broad production**
  - Expand to corridors once safety + equity gates pass.

**Rollback triggers**
- Any gate violation above.
- Increase in RLR/conflict proxies beyond tolerance.
- Speed distribution worsening beyond policy target.
- Data quality failure (freeze).

---

### 2) Representativeness + equity (bias risks, guardrails, sign-off)

Calibration is not neutral: where you calibrate, when you calibrate, and what you weight changes who benefits.

#### 2.1 How calibration can bias outcomes
Common bias channels:
- **Time-of-day bias**: peak-only data sets lock in peak behavior for off-peak conditions.
- **Instrumentation bias**: better detection coverage in certain neighborhoods yields better-tuned performance there.
- **User-mix bias**: freight-heavy corridors dominate parameter estimation if not stratified.
- **Outcome bias**: optimizing for vehicle delay alone pushes delay onto pedestrians/buses/side streets.

#### 2.2 Equity guardrails (implementation-ready)
- **Stratify by place and policy**
  - Keep separate parameter sets for school zones / high-VRU places.
- **Do not degrade vulnerable-user service**
  - Track pedestrian wait distributions and apply stricter caps in equity-focus areas.
- **Publish person-delay and distributional impacts**
  - Report person-delay deltas by movement/time-of-day; don’t report only aggregate intersection delay.

#### 2.3 Equity reporting template (copy/paste)
```markdown
# Calibration Equity Review — Parameter Set vX.Y.Z

## A) Where the data came from
- Sites included/excluded:
- Time windows:
- Data quality summary:
- Stratification scheme:

## B) Who might be impacted
- Equity focus areas included:
- School zones / high-VRU places:
- Transit routes / detours:

## C) Before/after (or baseline/new-parameter) indicators
- Person-delay change by movement (median/p95):
- Pedestrian wait change (median/p95/max):
- Bus travel-time reliability (if applicable):
- Safety proxies (speeds/RLR/conflicts):

## D) Decision
- Approve / Approve with conditions / Reject
- Conditions and required monitoring:
- Rollback triggers:

## E) Sign-off
- Signals engineer:
- Operations supervisor:
- Safety reviewer:
- Equity reviewer:
```

#### 2.4 Explicit sign-off roles
- Signals engineer (bounds, movement definitions)
- Operations supervisor (field feasibility, monitoring)
- Safety reviewer (gates)
- Equity reviewer (distributional checks)

---

### 3) Sensor limits + data-quality budgets (freeze-update rules)

FHWA’s signal timing guidance explicitly notes that it takes a few seconds for the first driver/vehicles to start moving on green, and that this **start-up lost time is commonly assumed to be ~2 seconds** ([`chapter3.htm`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter3.htm)). That’s a reminder that estimates must be robust to noise and measurement limits.

#### 3.1 Data quality budgets (recommended structure)
Define gates for each parameter family:

- **Timestamp integrity**
  - Gate: cabinet/server time offset beyond threshold → freeze.

- **Detector health**
  - Gate: stuck-on/stuck-off/chattering beyond threshold → freeze.

- **Missingness**
  - Gate: missing data minutes beyond threshold → freeze.

- **Sample size**
  - Gate: not enough saturated cycles or steady-state headways per bin → do not update; pool or fall back.

- **Stability**
  - Gate: sudden parameter jump → investigate (construction, detection faults) rather than update.

#### 3.2 Confidence scoring (must ship with every estimate)
Each published value should include:
- `estimate`, `confidence_score (0–1)`, `sample_size`, `lookback_window`, `data_health_status`, `last_good_version`.

#### 3.3 Freeze windows
Freeze updates:
- immediately after any major retiming/plan deployment,
- during major construction and planned special events affecting the corridor,
- during detection maintenance or firmware upgrades.

---

### 4) Disentangling behavior vs geometry/control (metadata + spillback regime separation)

FHWA’s timing guidance notes that saturation flow varies with geometry and constraints (lanes, grades, conflicts, parking, bus movements, etc.), and commonly ranges **~1,500–2,000 pcphpl** ([`chapter3.htm`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter3.htm)). Treat geometry/control as first-class features so you don’t mislabel design problems as “driving style.”

#### 4.1 Required site metadata schema (minimum)
Per intersection and per lane group:
- intersection ID, movement ID/lane group ID, served phase(s)
- lane count and lane use (exclusive/shared)
- grade, curb radii / turn bay length (where available)
- transit stops near stop line, parking/loading presence
- posted speed + any target-speed policy
- downstream bottleneck distance + spillback risk notes
- detector type + placement + maintenance date

#### 4.2 Regime detection: normal vs blocked (spillback/downstream blocking)
Explicitly classify cycles into regimes:
- **Normal saturated discharge** (queue clears, steady flow reached)
- **Blocked/oversaturated** (green wasted due to downstream blockage)

Operational rule:
- Do **not** use blocked cycles to estimate saturation headway.
- Publish a separate “blocked regime” model parameter used only when spillback indicators are active.

---

### 5) Change management (versioning, approvals, audit/rollback, cadence)

Treat calibration updates like controlled software releases.

#### 5.1 Versioning scheme
- Parameter set ID: `city-signalcal/vMAJOR.MINOR.PATCH`.
  - MAJOR: estimator/method changes, new metadata schema
  - MINOR: new sites/strata
  - PATCH: data refresh within bounds

#### 5.2 Approval workflow
- PATCH auto-merge allowed only if:
  - data-quality gates pass,
  - confidence is high,
  - safety regression checks pass,
  - equity review passes.
- Anything else requires human review + signatures.

#### 5.3 Update cadence + freeze windows
- Routine updates: monthly/seasonal.
- Freeze windows: after retiming, construction, special events.

#### 5.4 Operator communications
- Publish a “what changed / expected impacts / rollback triggers” note.

---

### 6) Deployable parameter table (with cited typical ranges where available)

The table below is “deployable” because it is explicit about inputs, methods, cadence, and failure modes; it includes **typical ranges only where a public FHWA source provides them**.

| Parameter | Definition | Data | Method | Cadence | Typical ranges (cited) | Failure modes |
|---|---|---|---|---|---|---|
| Start-up lost time | Additional time used by the first few queued vehicles on green before steady discharge | Phase change timestamps + stop-line discharge events | Measure the early headways vs steady headway; robust aggregation across days | Quarterly / after major change | Commonly assumed ~2 s ([`chapter3.htm`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter3.htm)) | Clock drift; no true queue; detector not at stop bar |
| Saturation headway / flow | Steady headway (s/veh) and resulting flow under continuous queue discharge | Stop-line per-vehicle departure times | Use vehicles after startup (e.g., 4th onward); trimmed mean/median | Monthly/seasonal | sat flow commonly ~1,500–2,000 pcphpl ([`chapter3.htm`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter3.htm)) | Spillback interrupts; mixed lane groups; short greens |
| Blocked-regime discharge modifier | Discharge under downstream blocking/oversaturation | Occupancy/queue indicators + downstream status | Classify blocked cycles; estimate separate regime | Event-triggered | — | Misclassification; missing downstream sensing |
| Heavy-vehicle factor | Effect of heavy vehicles on discharge/headway | Classification + headways | Stratify by class; publish confidence intervals | Semiannual | — | Misclassification; low samples |
| Permissive-turn service rate | Service rate under permissive conflicts | Protected vs permissive intervals + opposing flow | Regime model by opposing demand | Plan-dependent | — | Treating permissive as protected; opposing detection faults |

---

### 7) Calibration lifecycle diagram (instrument → estimate → shadow → rollout → monitor)

This is the **end-to-end calibration lifecycle chart** for adapting to local driving style, showing how measurement, estimation, validation, rollout, and monitoring connect.

```text
[1] Instrument & Data Quality Gates
      - Detector inventory + placement
      - Time sync checks
      - Data-quality budgets (missingness, health, stability)
      ↓
[2] Estimate Parameters (per movement / context)
      - Start-up lost time, saturation flow, heavy-vehicle factors
      - Regime separation (normal vs blocked)
      - Confidence scores + freeze rules
      ↓
[3] Shadow Validation in Twin
      - Inject parameters into digital twin
      - Reproduce queues, split failures, arrivals-on-green
      - Safety & equity checks (Section 1 & 2)
      ↓
[4] Governed Rollout (Pilot → Corridor)
      - Limited pilot deployment with cooldown windows
      - Versioned parameter sets (vMAJOR.MINOR.PATCH)
      - Approval workflow + equity review
      ↓
[5] Monitor, Drift Detection & Recertification
      - ATSPM-style KPIs + safety proxies
      - Drift + data-quality gates; rollback triggers
      - Periodic recertification & scope expansion
      ↺  Feeds back into [1] Instrument & Gates
```

This diagram is the **calibration chart** for Idea 05 and should be treated as the reference lifecycle for any implementation.

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
- [ ] Select pilot intersections with reliable detection + high-resolution logs.
- [ ] Build and validate phase-to-movement mapping and detector inventory.
- [ ] Define and implement data-quality budgets + freeze rules (Section 3).
- [ ] Define metadata schema and regime classification (Section 4).
- [ ] Stand up parameter estimation pipeline with confidence scoring.
- [ ] Integrate parameters into twin; run holdout validation.
- [ ] Define safety proxies, go/no-go gates, and rollback triggers (Section 1).
- [ ] Define equity guardrails, reporting template, and sign-off roles (Section 2).
- [ ] Establish versioning, approvals, audit logging, and update cadence (Section 5).

## Governance / Change-Control Runbook
1. **Propose**: pipeline produces a parameter set candidate (versioned).
2. **QA gates**: detector health, missingness, timestamp integrity, sample-size gates.
3. **Shadow validation**: twin prediction checks + safety regression checks.
4. **Equity review**: run the equity template report; review equity-focus places.
5. **Approve**:
   - PATCH: auto-approve only if all gates pass.
   - MINOR/MAJOR: require signals engineer + ops + safety + equity sign-off.
6. **Deploy**: publish version tag, update configs, set cooldown.
7. **Monitor**: dashboards + safety proxies; daily checks for 1 week.
8. **Rollback**: revert to last-good version if triggers fire; open an incident ticket.
9. **Postmortem**: document root cause; update gates/metadata.

## Reference Links
- FHWA Traffic Signal Timing Manual — Chapter 3 (lost time, saturation flow ranges): [`chapter3.htm`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter3.htm)
- FHWA Automated Traffic Signal Performance Measures (ATSPM): [`fhwahop20002.pdf`](https://ops.fhwa.dot.gov/publications/fhwahop20002/fhwahop20002.pdf)
- FHWA Safe System Approach for Speed Management (2023): [`Safe_System_Approach_for_Speed_Management.pdf`](https://highways.dot.gov/sites/fhwa.dot.gov/files/Safe_System_Approach_for_Speed_Management.pdf)

---

## References / Standards / Useful Sources
- FHWA Traffic Signal Timing Manual: https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf
- JRC report on calibration of traffic simulation models: https://publications.jrc.ec.europa.eu/repository/bitstream/JRC68403/lbna25188enn.pdf

---

Cross-links: Related ideas include green waves, what-if button, and privacy-friendly learning.
