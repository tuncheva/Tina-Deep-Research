# 17) Safety-First Signals: Optimizing for Near-Misses

## Catchy Explanation
Instead of waiting for crashes to prove a problem, safety-first signals treat “near-misses” as warnings—and optimize timing to reduce them.

## What it is (precise)
**Safety-first signal control** treats safety as a primary optimization target by using **surrogate safety measures (SSMs)** such as **TTC (time-to-collision)**, **PET (post-encroachment time)**, and conflict counts derived from trajectories (from simulation, video analytics, or connected data). Candidate timing changes are evaluated in a digital twin (or fast predictor) and selected to reduce risk subject to mobility constraints (delay, queues) and policy constraints (pedestrian service). This enables explicit, auditable safety–mobility tradeoffs.

## Benefits
- **Proactive safety**: reduces risk before crashes accumulate.
- **Measurable tradeoffs**: safety impacts can be reported like performance impacts.
- **Targeted interventions**: focuses on high-risk movements and times.
- **Better justification**: supports safety-driven retiming decisions.

## Challenges
- **Validation**: SSMs must be correlated with local crash history.
- **Data intensity**: needs trajectories or good proxies.
- **Political sensitivity**: safety improvements may increase delay.
- **Bias risk**: incomplete detection can undercount vulnerable users.

## Implementation Strategies

### Infrastructure Needs
- **Trajectory source**: micro-sim twin, video/radar tracking, or CV data.
- **SSM computation**: TTC/PET/conflict metrics.
- **Policy constraints**: required ped service, clearance behavior.
- **Twin evaluation**: test timing alternatives with safety scoring.
- **Reporting**: safety impact statements + audits.

### Detailed Implementation Plan
#### Phase 1: Select Safety Measures, Thresholds, and Decision Rules (Weeks 1–4)
The agency should begin by selecting a small set of surrogate safety measures that it can compute reliably, because a complex safety score that cannot be defended will not be used for operational decisions. The team should define TTC/PET thresholds and conflict definitions that match local context and should define how safety will be used in decisions, such as treating certain risk levels as hard constraints that prohibit a timing change or using safety as a high-weight objective that is balanced against mobility. The team should define reporting formats and acceptance criteria so that every deployment can be explained as a safety-driven choice rather than an opaque optimization.

- **Roles**: safety program lead (Vision Zero alignment), traffic engineering (timing implications), data/analytics (SSM definitions), accessibility reviewer (vulnerable user focus).
- **Deliverables**: SSM framework, threshold definitions, decision rules, and reporting template.
- **Risks**: SSMs may not correlate with local crash patterns; thresholds may be contested.
- **Acceptance checks**: chosen SSMs are computable with existing data sources and have clear definitions.

#### Phase 2: Data Pipeline, Calibration, and Validation Against Crash History (Weeks 5–12)
The agency should establish a trajectory data pipeline using either video/radar tracking, connected data, or a calibrated simulation twin, because SSMs require trajectory-like information. The team should validate that trajectory quality is sufficient across lighting and weather conditions and should measure detection bias for pedestrians and cyclists, because undercounting vulnerable users undermines safety claims. The team should compare SSM hotspots to historical crash and near-miss reports to confirm that high-risk SSM locations align with known safety problems, and it should document limitations where the data is incomplete.

- **Roles**: ITS/video analytics team (trajectory extraction), modeler (if twin-based), analyst (validation), safety lead (crash data interpretation), privacy reviewer (data governance).
- **Deliverables**: calibrated SSM pipeline, validation report, bias assessment, and site prioritization list.
- **Risks**: video analytics may degrade in bad weather; crash data may be sparse or delayed.
- **Acceptance checks**: trajectory pipeline is stable, and SSMs show plausible correlation with known risk areas.

#### Phase 3: Generate Safety-Driven Timing Alternatives and Impact Statements (Weeks 13–20)
The traffic engineering team should generate timing alternatives targeted to the movements and periods where conflicts occur, such as adding LPIs, adjusting protected/permitted left timing, modifying clearance intervals, or changing coordination to reduce speed variance. The team should evaluate candidates using the twin or a fast predictor and should produce safety impact statements that report expected changes in SSM distributions alongside mobility impacts, because safety-first control explicitly trades mobility for risk reduction when needed. The team should choose a small set of plans that meet safety targets and remain operationally feasible.

- **Roles**: traffic engineer (plan design), safety lead (review), analyst/modeler (evaluation), operations (feasibility).
- **Deliverables**: safety-first plan set, safety impact statements, and a deployment/rollback plan.
- **Risks**: safety improvements may increase delay and create political pushback; overly aggressive changes can harm transit reliability.
- **Acceptance checks**: safety targets are met in evaluation and pedestrian service constraints remain satisfied.

#### Phase 4: Pilot Deployment, Monitoring, and Adjustment (Weeks 21–32)
The agency should deploy changes in assisted mode with close monitoring, because safety-driven changes often require careful field observation and community communication. The team should monitor both SSMs and mobility KPIs, and it should treat unexpected increases in near-miss measures as a trigger for rollback or adjustment. The team should also conduct field observations where possible to validate that behavioral changes (turning yields, vehicle speeds) match the intended safety improvement.

- **Roles**: operations (monitoring), traffic engineering (on-call tuning), safety lead (field review), analyst (reporting), communications (public messaging if needed).
- **Deliverables**: pilot report, updated thresholds if needed, and documented adjustments.
- **Risks**: SSM measurements may be noisy; public may perceive added delay as unnecessary.
- **Acceptance checks**: SSM distributions improve without introducing new operational hazards, and field observations support the data.

#### Phase 5: Continuous Revalidation and Network Expansion (Ongoing)
The program should revalidate SSM-to-crash correlation periodically and should update thresholds and models as data improves, because safety analytics is an evolving practice. The agency should expand to additional sites using a prioritization process that targets high-risk locations and that accounts for equity and vulnerable road user exposure. The agency should integrate safety-first scoring into policy-driven signal rules so safety constraints remain consistently enforced.

- **Roles**: safety program owner (governance), analytics (revalidation), traffic engineering (expansion), operations (monitoring).
- **Deliverables**: periodic safety impact reports, updated site list, and expansion roadmap.
- **Risks**: correlation may drift as infrastructure changes; scaling increases data and maintenance requirements.
- **Acceptance checks**: safety improvements persist and are reflected in longer-term crash trends where measurable.

### Choices
- **Hard safety constraints**: do not allow actions above risk thresholds.
- **Weighted objectives**: trade safety vs mobility explicitly.

## Technical Mechanics

### Key Parameters
- TTC/PET thresholds
- Conflict rate targets
- Data quality thresholds

### Guardrails
- Preserve pedestrian minimums and clearance behavior.
- Require evidence for safety claims and periodic audits.

## MVP Deployment
- One intersection with known conflict history.
- SSM computed from micro-sim twin.
- Monthly safety impact report.

## Evaluation
- Changes in SSM distributions (TTC/PET) and conflict counts.
- Mobility impacts (delay, queues).
- Longer-term crash trend validation.

## References / Standards / Useful Sources
- FHWA Surrogate Safety Measures from Traffic Simulation Models (PDF): https://ntlrepository.blob.core.windows.net/lib/38000/38000/38015/FHWA-RD-03-050.pdf
- FHWA Leading Pedestrian Interval (LPI) fact sheet: https://highways.dot.gov/sites/fhwa.dot.gov/files/2022-06/04_Leading%20Pedestrian%20Interval_508.pdf

---

Cross-links: Related ideas include city rules and explainable signals.
