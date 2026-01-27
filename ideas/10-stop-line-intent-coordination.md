# 10) Stop-Line Intent Coordination: Precise Green Allocation

## Catchy Explanation
If the signal knew which movements are *actually* waiting at the stop line (left vs through vs right), it could stop wasting green on empty lanes and serve the real queue—without breaking safety rules.

## What it is (precise)
**Stop-line intent coordination** uses approach and lane-level “intent” (which movements are queued and how much) to allocate green time more precisely. Intent can come from lane-level detection (loops, radar, video) and/or connected-vehicle data, then mapped to signal phases via a maintained lane-to-phase configuration. The controller reallocates splits (and sometimes phase order within allowed logic) to reduce **wasted green** while respecting pedestrian minimums/clearances and limiting corridor disruption. A digital twin is used to test reallocation policies and spillback impacts.

## Benefits
- **Less wasted green**: green time goes to movements that can use it.
- **Better surge handling**: turns and special patterns are served faster.
- **Improved queue stability**: prevents one movement from starving others.
- **Higher effective capacity**: better use of existing infrastructure.

## Challenges
- **Detection quality**: misclassification leads to bad reallocations.
- **Mapping correctness**: lane-to-phase configuration must be accurate and maintained.
- **Coordination risk**: myopic changes can break progression.
- **Safety constraints**: cannot trade away pedestrian service or clearance behavior.

## Implementation Strategies

### Infrastructure Needs
- **Lane-level detection** at/near stop line.
- **Lane-to-phase mapping** data model (versioned).
- **(Optional) SPaT/MAP interface** for richer connected-vehicle integration.
- **Twin evaluation** for spillback and network effects.
- **ATSPM telemetry** to quantify wasted green and split failures.

### Detailed Implementation Plan
#### Phase 1: Lane-to-Phase Mapping and Baseline Measurement (Weeks 1–4)
The agency should begin by verifying lane geometry, signage, and striping at the pilot intersection(s), because the system can only allocate green correctly if it knows which lanes feed which movements. The traffic engineer should build a versioned lane-to-phase mapping registry that includes lane groups, permitted movements, detector channels, and associated controller phases, and the maintenance team should confirm this mapping matches cabinet wiring and detector configuration. The analytics team should compute a baseline for wasted green, split failures, queue lengths, and arrivals-on-green so the project can prove that “intent” improves operations rather than just adding complexity.

- **Roles**: traffic engineer (lane-phase definitions), signal technician (wiring/detector confirmation), data analyst (baseline KPIs), operations (field observations).
- **Deliverables**: mapping registry (versioned), baseline KPI report, and a detector/channel documentation pack.
- **Risks**: lane-use changes (construction, restriping) can invalidate mappings; baseline may be biased by unusual weeks.
- **Acceptance checks**: lane-to-phase mapping is signed off, and baseline KPIs are reproducible.

#### Phase 2: Intent Estimation from Detection (Weeks 5–12)
The engineering team should implement intent estimation that computes, for each movement, an estimate of queued vehicles and demand intensity based on stop-line presence, advance detection, and/or video/radar classification. The team should implement plausibility checks (for example, maximum queue bounded by lane storage and consistency between upstream and stop-line detectors) and smoothing so brief detection noise does not trigger oscillating split allocations. The system should publish a health score for intent quality so that when detection is unreliable the controller can fall back to standard actuated or fixed-time behavior.

- **Roles**: data engineer (intent pipeline), traffic engineer (movement logic), maintenance (detector QA), QA engineer (test scenarios).
- **Deliverables**: intent feed/API with health checks, detector QA report, and validation plots comparing estimated queues to spot field observations.
- **Risks**: misclassification of lanes (left vs through) can starve movements; detection outages can produce unstable estimates.
- **Acceptance checks**: intent estimates behave plausibly under normal traffic, and the system correctly flags low-health periods.

#### Phase 3: Split Reallocation Policy, Guardrails, and Twin Validation (Weeks 13–20)
The traffic engineer should implement a bounded reallocation policy that shifts split time by small amounts within the cycle (for example, ±3–8 seconds) and that preserves minimum service for all movements and pedestrian phases. The policy should include coordination caps so it does not break corridor progression and should include rate limits so the plan does not oscillate across cycles. The team should validate the reallocation policy in a twin across representative demand patterns and should explicitly test surge and turn-heavy conditions, because those are the cases where intent allocation should provide value. Finally, the team should define revert logic that returns to the baseline plan when intent health drops or when performance degrades.

- **Roles**: traffic engineer (policy design), modeler (twin scenarios), software engineer (implementation), safety/accessibility reviewer (ped constraints).
- **Deliverables**: validated reallocation policy, guardrail specification, and simulation report.
- **Risks**: overreacting to short-term noise; coordination degradation across a corridor.
- **Acceptance checks**: pedestrian minimums and clearance are always preserved, and simulation shows reduced wasted green without unacceptable side-street delay.

#### Phase 4: Connected-Vehicle Enrichment (Optional) (Weeks 21–32)
If the city has connected-vehicle infrastructure, the team can enrich intent estimation using SPaT/MAP alignment and (where available) CV-derived turning intent and approach trajectories, because CV data can improve lane-level understanding when detectors are limited. The team should treat CV data as additive and should maintain robust fallback to detection-only operation, because CV penetration may be low or uneven. The team should also validate that CV-derived intent does not introduce privacy or bias concerns and should ensure that any identifiers are not persisted.

- **Roles**: ITS engineer (CV integration), data engineer (fusion), security/privacy reviewer (data governance), traffic engineer (policy impact).
- **Deliverables**: hybrid intent estimator, coverage report, and updated validation results.
- **Risks**: low penetration yields limited value; integration complexity increases maintenance burden.
- **Acceptance checks**: hybrid approach improves intent quality when coverage exists and falls back cleanly when it does not.

#### Phase 5: Expansion and Operationalization (Ongoing)
The agency should expand intent-based allocation intersection-by-intersection, prioritizing sites with high wasted-green rates and reliable lane-level detection, because those locations offer the best return. The team should implement a maintenance process that updates lane-to-phase mapping after restriping or construction, and it should monitor intent health continuously using ATSPM-style performance measures. Over time, the agency can add incident-specific policies (for example, turn surge mode) but should keep the bounded reallocation principle so operations remain stable.

- **Roles**: program owner (prioritization), traffic engineering (policy tuning), operations (monitoring), maintenance (mapping updates).
- **Deliverables**: rollout plan, mapping update procedure, and monthly performance reports.
- **Risks**: mapping drift over time; inconsistent detection quality across the network.
- **Acceptance checks**: performance improvements persist and the mapping registry remains accurate and versioned.

### Choices
- **Detection-only**: simplest, widely deployable.
- **CV-enhanced**: richer intent where coverage exists.
- **Hybrid**: best practical approach.

## Technical Mechanics

### Key Parameters
- Wasted green %
- Queue length and discharge estimates per movement
- Split shift bounds (seconds)
- Minimum service constraints

### Guardrails
- Maintain pedestrian minimums and clearance behavior.
- Rate-limit split shifts.
- Cap deviation from corridor coordination.

## MVP Deployment
- One intersection.
- 1–2 bounded split tweaks.
- Shadow mode first with KPI reporting.

## Evaluation
- Wasted green reduction.
- Split failure rate change.
- Queue balance across movements.
- Corridor travel time and progression impacts.

## References / Standards / Useful Sources
- NOCoE SPaT Challenge Implementation Guide: https://www.transportationops.org/spatchallenge/resources/Implementation-Guide
- Guidance Document for MAP Message Preparation (PDF): https://engineering.virginia.edu/sites/default/files/Connected-Vehicle-PFS/Resources/MAP%20Guidance%20Document%20-%20Revision%202_06232023.pdf

---

Cross-links: Related ideas include green waves, what-if button, and fair priority credits.
