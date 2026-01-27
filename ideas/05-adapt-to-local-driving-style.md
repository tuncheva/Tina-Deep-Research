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

## MVP Deployment
- 3–5 intersections.
- Weekly parameter estimates.
- Operator-reviewed timing adjustments.

## Evaluation
- Prediction accuracy improvement (queues/travel times).
- Change in split failures and arrivals on green.
- Stability: parameter drift and update frequency.

## References / Standards / Useful Sources
- FHWA Traffic Signal Timing Manual: https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf
- JRC report on calibration of traffic simulation models: https://publications.jrc.ec.europa.eu/repository/bitstream/JRC68403/lbna25188enn.pdf

---

Cross-links: Related ideas include green waves, what-if button, and privacy-friendly learning.
