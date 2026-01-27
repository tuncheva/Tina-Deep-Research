# 14) Weather + Traffic + Power Together: Integrated Constraints Mode

## Catchy Explanation
When a storm hits and the grid is unstable, “optimal timing” is not the priority. The signal should switch into a clear, safer mode that respects weather risk and power/comms constraints.

## What it is (precise)
**Weather + traffic + power together** is a mode-selection framework that fuses three constraint families:
1) **Traffic state** (demand surges, queues, spillback risk),
2) **Weather risk** (visibility, friction, flooding risk, snow/ice),
3) **Power/comms status** (UPS on battery, comms degraded, controller isolation).

The system chooses conservative, pre-vetted timing plans (storm-safe, outage/local mode, evacuation bias) that favor predictability and safety over efficiency. Digital twins are used to test triggers, plan transitions, and recovery steps.

## Benefits
- **Safety in adverse conditions**: timing can reduce risky conflicts when visibility/friction are poor.
- **Operational continuity**: graceful behavior during power and comms degradation.
- **Predictability**: clear modes reduce confusion for operators and road users.
- **Coordinated response**: enables regional playbooks across corridors.

## Challenges
- **Data fusion**: disparate sources with different latency/quality.
- **Trigger tuning**: avoid false activations and late activations.
- **Mode transitions**: switching plans can create temporary instability.
- **Maintenance**: scenario plans must be refreshed with infrastructure changes.

## Implementation Strategies

### Infrastructure Needs
- **Weather feeds**: road weather stations, forecast APIs, precipitation/visibility.
- **Traffic monitoring**: detectors, probe travel times, CCTV analytics.
- **Power status**: UPS alarms, cabinet telemetry, comms heartbeat.
- **Plan library**: storm-safe plans, outage/local plans, evacuation plans.
- **Operator workflow**: activate, monitor, and recover with logging.

### Detailed Implementation Plan
#### Phase 1: Define Modes, Authorities, and Trigger Bundles (Weeks 1–4)
The agency should define a small number of operational modes that reflect real operational needs, such as NORMAL, STORM-SAFE, OUTAGE/LOCAL, EVACUATION BIAS, and RECOVERY, because an overly complex mode library will not be used. The program should assign authority and ownership for mode activation (for example, when emergency management can request EVACUATION mode, or when operations can initiate STORM-SAFE) and should define what happens if comms are lost and the controller must decide locally. The engineering team should define trigger bundles that combine weather indicators (visibility, precipitation intensity, surface friction risk), traffic indicators (occupancy, travel time spikes, spillback risk), and power/comms indicators (UPS on battery, heartbeat loss), and it should include dwell times so the system does not oscillate during fluctuating conditions.

- **Roles**: operations (mode workflow), traffic engineering (plan feasibility), IT/network (comms and telemetry), emergency management liaison (evacuation triggers), maintenance (power/UPS telemetry).
- **Deliverables**: mode definitions, trigger specification, activation authority matrix, and a draft operator runbook.
- **Risks**: triggers may be ambiguous; interagency coordination may be unclear.
- **Acceptance checks**: each mode has a clear purpose, an owner, and measurable trigger conditions.

#### Phase 2: Build Playbooks and Plan Sets per Mode (Weeks 5–14)
The traffic engineering team should create plan sets for each mode that match real controller capabilities, such as conservative splits and longer clearance assumptions during STORM-SAFE, and stable fixed-time or local actuated behavior during OUTAGE/LOCAL mode. For evacuation bias, the team should coordinate with emergency management to define which routes receive priority and for how long, and it should design recovery plans that restore normal coordination gradually rather than abruptly. The team should document each playbook with activation steps, expected impacts, and rollback criteria, because operators need clear guidance during stressful conditions.

- **Roles**: traffic engineer (plan design), operations (playbook usability), safety/accessibility reviewer (ped timing), emergency management (route priorities), communications lead (public messaging inputs).
- **Deliverables**: plan library (versioned) by mode and corridor, playbooks with checklists, recovery rules, and rollback criteria.
- **Risks**: plans may push queues to parallel routes; evacuation bias may conflict with local access needs.
- **Acceptance checks**: each mode plan preserves pedestrian minimums/clearance and includes a tested recovery path.

#### Phase 3: Twin-Based Validation and Transition Testing (Weeks 15–22)
The modeling team should validate each mode plan in a twin using representative storms, outages, and demand surges, because switching plans at the wrong time can create instability. The team should simulate transitions between modes and should measure temporary impacts on queues and spillback, because transitions are where many field failures occur. The analytics team should produce expected-outcome summaries for operators, including what KPIs will likely worsen (for example, delay) and what safety benefits are expected.

- **Roles**: modeler (simulation), analyst (KPI scoring), traffic engineering (interpretation), operations (review).
- **Deliverables**: simulation validation report, transition impact report, and operator-facing outcome summaries.
- **Risks**: the twin may not capture all weather-driven behavior; limited data for calibrating storm conditions.
- **Acceptance checks**: simulated transitions meet stability criteria and do not violate safety invariants.

#### Phase 4: Integrate Monitoring and Switching Logic (Weeks 23–34)
The engineering team should integrate weather, traffic, and power/comms telemetry into a fused monitoring layer that computes a mode recommendation with evidence, because operators must be able to understand why a mode is being suggested. The system should start in assisted activation, where operators confirm mode changes, and it should only allow automatic activation for low-risk modes after performance and false-trigger rates are understood. The team should ensure that when comms are lost, controllers fall back to a safe local plan that is consistent with OUTAGE/LOCAL playbooks.

- **Roles**: software/data engineering (fusion logic), IT/network (telemetry reliability), operations (activation), traffic engineering (tuning), QA (testing).
- **Deliverables**: integrated monitoring and mode recommendation system, dashboards, alert rules, and a pilot deployment.
- **Risks**: false activations erode trust; telemetry latency creates late triggers.
- **Acceptance checks**: fused monitoring produces stable recommendations and supports clear evidence for operator decisions.

#### Phase 5: Drills, After-Action Reviews, and Scaling (Weeks 35+ / Ongoing)
The agency should run planned drills (tabletop and controlled “game day” exercises) to rehearse mode activation and recovery, because rare conditions are where procedures fail. After real storms or outages, the team should conduct after-action reviews that compare expected and observed outcomes and that result in specific updates to triggers, plans, and playbooks. The program should scale corridor-by-corridor, prioritizing routes that are critical for emergency response or that experience frequent weather-related disruption.

- **Roles**: program owner (governance), operations (drills and real events), traffic engineering (plan updates), analyst (after-action metrics), emergency management (coordination).
- **Deliverables**: drill reports, updated mode policies, annual plan refresh, and scaling roadmap.
- **Risks**: playbooks become stale; lack of drills reduces readiness.
- **Acceptance checks**: false-trigger rate decreases over time, and recovery behavior becomes faster and more stable.

### Choices
- **Manual activation**: operator-triggered.
- **Semi-auto**: threshold-based recommendations.
- **Full fusion**: automatic with strict guardrails.

## Technical Mechanics

### Key Parameters
- Weather thresholds (visibility, precipitation intensity)
- Traffic thresholds (occupancy, travel time spike)
- Power/comms thresholds (UPS battery, heartbeat)
- Dwell times and recovery criteria

### Guardrails
- Preserve pedestrian minimums and clearance behavior.
- Rate-limit mode switching.
- Always provide rollback to a safe local plan.
- Log reasons and evidence for each mode change.

## MVP Deployment
- One corridor.
- Two modes beyond NORMAL (STORM-SAFE, OUTAGE/LOCAL).
- Assisted activation + drill once per quarter.

## Evaluation
- Mode activation accuracy (true/false activations).
- Safety and delay impacts during storms.
- Recovery time and stability (no flapping).

## References / Standards / Useful Sources
- FHWA Traffic Signal Timing Manual: https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf

---

Cross-links: Related ideas include rare-event practice, delay-tolerant signals, and event-aware timing.
