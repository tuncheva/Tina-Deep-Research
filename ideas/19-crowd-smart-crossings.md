# 19) Crowd-Smart Crossings: Dynamic Pedestrian Phasing

## Catchy Explanation
When a crowd shows up, the intersection should behave like a good event coordinator: give pedestrians a safer, more efficient crossing pattern—then return to normal when the crowd is gone.

## What it is (precise)
**Crowd-smart crossings** dynamically adjust pedestrian phasing when large groups are present by switching among strategies such as:
- **Concurrent crossing** (baseline),
- **Leading Pedestrian Interval (LPI)** (pedestrians start 3–7s before turning vehicles),
- **Exclusive pedestrian phase (“scramble”)** in high-conflict, high-ped situations.

Crowd size is estimated from sensors (video, radar, thermal) and the system chooses a strategy under safety constraints, using a digital twin to test impacts on conflicts, queues, and cycle length.

## Benefits
- **Reduced conflicts**: LPI/scramble reduce turning conflicts.
- **Better surge handling**: handles bursts without repeated calls or chaos.
- **Accessibility**: supports slower and vulnerable pedestrians with clearer right-of-way.
- **Safety justification**: easier to explain changes during events.

## Challenges
- **Detection reliability**: weather/lighting can degrade crowd estimation.
- **Vehicle delay tradeoffs**: exclusive phases can increase delay.
- **Oscillation risk**: frequent switching if thresholds are unstable.
- **Compliance**: must align with local standards and signal controller capability.

## Implementation Strategies

### Infrastructure Needs
- **Ped detection**: queue length and arrival rate estimation.
- **Controller capability**: configurable LPI and (if used) scramble phasing.
- **Twin evaluation**: test crowd scenarios and recovery.
- **Safety constraints**: ped minimums, clearance, max vehicle queue caps.

### Detailed Implementation Plan
#### Phase 1: Site Selection, Safety Context, and Baseline (Weeks 1–4)
The agency should start by selecting 1–3 intersections with known pedestrian surges, such as transit stations, stadium-adjacent crossings, or central business district locations, because crowd-responsive phasing is most valuable where bursts create real conflict and delay. The team should review existing safety context, including turning conflicts, yielding compliance, and any crash history, and it should define objectives such as reduced turning conflicts and bounded pedestrian delay during surges. The analytics team should compute baseline pedestrian delay distributions and any available conflict proxies (or field observations) so the pilot can be evaluated credibly.

- **Roles**: traffic engineering (site selection), safety/accessibility reviewer (ped needs), operations (feasibility), analyst (baseline), field staff (observations).
- **Deliverables**: site list, baseline report, and controller capability assessment (LPI/scramble feasibility).
- **Risks**: selected sites may have limited controller flexibility; baseline may miss rare surge events.
- **Acceptance checks**: pilot sites have a clear surge pattern and measurable baseline metrics.

#### Phase 2: Detection Upgrade and Crowd Metric Pipeline (Weeks 5–12)
The agency should deploy or configure pedestrian detection capable of estimating queue length and arrival rate, using video, radar, thermal sensors, or improved pushbutton analytics, because crowd-smart control requires reliable surge detection. The team should implement data quality checks that flag degraded sensor conditions (night glare, rain, occlusion) and should define a fallback strategy that preserves safe pedestrian service when detection is unreliable. The engineering team should publish a crowd metric feed with confidence scoring so downstream control logic can avoid oscillating on noisy inputs.

- **Roles**: ITS/sensor team (deployment), IT/data engineer (pipeline), maintenance (sensor upkeep), privacy reviewer (if video analytics), operations (monitoring).
- **Deliverables**: crowd metric feed/API, confidence scoring, sensor QA dashboards, and documented fallback behavior.
- **Risks**: detection quality varies by weather; privacy concerns can block deployment.
- **Acceptance checks**: crowd metrics align with field observations and the system flags low-confidence periods.

#### Phase 3: Strategy Selection Logic, Guardrails, and Twin Validation (Weeks 13–20)
The traffic engineering team should define thresholds for switching between concurrent ped phases, LPI, and scramble, because not every surge warrants a full exclusive phase. The logic should include hysteresis and minimum dwell times so the intersection does not switch strategies every cycle during borderline crowd conditions. The team should validate candidate strategies in a twin or controlled analysis to quantify impacts on conflicts, vehicle delay, and queue spillback, and it should ensure that pedestrian minimums and clearance intervals are always preserved.

- **Roles**: traffic engineer (strategy logic), safety lead (conflict focus), modeler/analyst (evaluation), operations (workflow).
- **Deliverables**: switching logic specification, validated thresholds, and an impact report.
- **Risks**: scramble phases can increase vehicle queues and create spillback; thresholds may oscillate.
- **Acceptance checks**: switching is stable, safety constraints are preserved, and expected tradeoffs are documented.

#### Phase 4: Pilot Deployment with Assisted Mode and Tuning (Weeks 21–32)
The agency should deploy the switching logic in assisted mode first, where operators can enable or disable crowd mode during known surge periods, because early deployments require trust and operational oversight. The team should monitor pedestrian delay, vehicle queues, and conflict proxies during real events and should adjust thresholds and dwell times based on observed performance. If the intersection is near a venue, the team should coordinate activation windows with event schedules to reduce reliance on noisy detection.

- **Roles**: operations (activation), traffic engineering (tuning), analyst (evaluation), field staff (spot checks), venue liaison (if applicable).
- **Deliverables**: pilot report, tuned thresholds, and updated playbooks for crowd events.
- **Risks**: operators may disable the system if early behavior is confusing; crowd patterns may differ by event.
- **Acceptance checks**: pedestrian service improves during surges and vehicle spillback remains within caps.

#### Phase 5: Integration and Expansion (Ongoing)
The program should integrate crowd-smart crossings with event-aware timing and city rules so that pedestrian protection strategies are consistent with broader policy and operational modes. The agency should expand to additional sites only after confirming sensor reliability and operational capacity to maintain detection, because poorly maintained sensors create unsafe or annoying behavior. The team should maintain periodic safety reviews and should refresh detection calibration as conditions change.

- **Roles**: program owner (governance), traffic engineering (policy alignment), operations (monitoring), ITS/maintenance (sensor upkeep).
- **Deliverables**: expansion roadmap, periodic safety reports, and updated sensor maintenance procedures.
- **Risks**: sensor maintenance burden increases; inconsistent policy across districts.
- **Acceptance checks**: expanded sites meet reliability and safety thresholds and remain auditable.

### Choices
- **LPI first**: low-cost safety improvement.
- **Scramble**: for high-volume, high-conflict sites.
- **Hybrid**: LPI normally, scramble during events.

## Technical Mechanics

### Key Parameters
- Ped queue length / arrival rate
- LPI duration (seconds)
- Dwell times for mode switching

### Guardrails
- Maintain pedestrian minimums and clearance.
- Cap vehicle queue spillback.
- Rate-limit switching.

## MVP Deployment
- One intersection.
- LPI activation based on crowd queue threshold.
- Monthly safety review.

## Evaluation
- Conflict proxies (TTC/PET if available) and turning yield behavior.
- Ped delay distribution.
- Vehicle delay impacts and queue spillback.

## References / Standards / Useful Sources
- FHWA Leading Pedestrian Interval (LPI) fact sheet: https://highways.dot.gov/sites/fhwa.dot.gov/files/2022-06/04_Leading%20Pedestrian%20Interval_508.pdf

---

Cross-links: Related ideas include safety-first signals and event-aware timing.
