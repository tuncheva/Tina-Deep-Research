# 18) Event-Aware Timing: Concert/Stadium/Festival Mode

## Catchy Explanation
The city already knows the stadium empties at 22:30. Event-aware timing is the signal system’s ability to switch into “event mode” on purpose—before the surge turns into gridlock.

## What it is (precise)
**Event-aware timing** detects predictable demand surges (concerts, stadium events, festivals, planned road work) and activates special timing plans that bias service inbound/outbound, protect pedestrian crossings, and prevent spillback. Activation can be calendar-based, threshold-based, or operator-assisted. Digital twins rank candidate event plans, estimate clearance time, and test recovery steps back to normal coordination.

## Benefits
- **Faster clearance**: reduces post-event congestion and gridlock.
- **Safer crowd movement**: predictable crossing behavior during surges.
- **Less improvisation**: repeatable playbooks reduce operator stress.
- **Better coordination**: integrates traffic management with venues and public safety.

## Challenges
- **Surge variability**: real departure patterns vary by weather and attendance.
- **Staffing needs**: off-hours activations and monitoring.
- **Integration complexity**: temporary traffic control and detours.
- **Smooth recovery**: transitioning back to normal without shock.

## Implementation Strategies

### Infrastructure Needs
- **Event inventory + calendar**: known events, corridors, time windows.
- **Detection**: volume/occupancy thresholds, parking/venue signals if available.
- **Plan library**: inbound/outbound plans + recovery plans.
- **Twin forecasting**: spillback testing and clearance estimation.
- **Operator workflow**: activation, monitoring, rollback.

### Detailed Implementation Plan
#### Phase 1: Event Catalog, Stakeholder Alignment, and Corridor Mapping (Weeks 1–4)
The agency should begin by creating a practical catalog of recurring events and planned disruptions, using venue calendars, city permits, construction schedules, and police or emergency management inputs. The team should map each event to expected ingress and egress routes and should identify critical intersections where spillback frequently occurs, because those locations drive the design of event plans. The team should define KPI targets for each event type, such as maximum clearance time, maximum queue lengths, and pedestrian delay caps, and it should define staffing and authority for activation so event mode does not depend on a single individual.

- **Roles**: operations (activation workflow), traffic engineering (corridor mapping), venue/city events liaison (calendar and operations), police/public safety (traffic control needs), communications (public messaging).
- **Deliverables**: event catalog, corridor maps, KPI targets, and an activation authority matrix.
- **Risks**: events may change schedule or attendance; detours and police control can change traffic patterns.
- **Acceptance checks**: each priority event has a defined corridor and measurable KPIs.

#### Phase 2: Build an Event Plan Library and Recovery Plans (Weeks 5–12)
The traffic engineering team should create a plan library for each event corridor, including inbound bias, outbound bias, and a recovery plan that returns the corridor to normal coordination in steps. Plans should explicitly protect pedestrian service where crowds are expected and should be designed with coordination constraints so the event plan does not simply shift congestion upstream. The team should document each plan with intended use conditions and a rollback procedure, because event operations often occur under pressure and outside normal hours.

- **Roles**: traffic engineer (plan design), safety/accessibility reviewer (ped impacts), operations (usability), venue liaison (crowd patterns).
- **Deliverables**: versioned plan library, recovery plan definitions, and operator checklists.
- **Risks**: event plans may create unexpected side-street congestion; recovery may be too abrupt.
- **Acceptance checks**: each plan has a clear purpose, a maximum duration, and a recovery path.

#### Phase 3: Validation in a Twin and Trigger Design (Weeks 13–20)
The modeling team should validate event plans in a twin across different attendance and departure scenarios, because event demand is uncertain and needs robustness. The team should evaluate spillback risk, clearance time, pedestrian delay, and safety proxies, and it should document tradeoffs so operators understand expected outcomes. The engineering team should implement trigger logic that combines calendar scheduling with threshold confirmation (occupancy, travel time spikes, parking egress indicators), because calendar-only activation is often too early or too late.

- **Roles**: modeler (simulation), analyst (KPI scoring), operations (trigger practicality), traffic engineering (threshold approval).
- **Deliverables**: validation report, tuned triggers, and operator-facing “expected outcome” summaries.
- **Risks**: poor sensor coverage makes triggers unreliable; the twin may not capture unusual crowd behavior.
- **Acceptance checks**: triggers are testable and event plans perform acceptably across multiple simulated scenarios.

#### Phase 4: Pilot Activation During Real Events and After-Action Reviews (Weeks 21–32)
The operations team should pilot event plans during several real events in assisted mode, where operators activate the plan and monitor KPIs with the ability to roll back. The team should coordinate with police and venue staff to ensure that on-street traffic control aligns with signal plans, because conflicts between manual control and automated plans can cause failure. After each event, the team should run an after-action review that compares predicted and observed outcomes and updates triggers, plans, and checklists.

- **Roles**: operations (activation), traffic engineering (on-call tuning), police/public safety (field coordination), analyst (evaluation), venue liaison (event context).
- **Deliverables**: pilot report, updated plan versions, and after-action review notes.
- **Risks**: staffing gaps during late events; unexpected road closures or incidents.
- **Acceptance checks**: clearance time improves without violating pedestrian constraints, and after-action updates are incorporated quickly.

#### Phase 5: Scale and Automate Carefully (Ongoing)
The agency should scale event-aware timing to additional venues using templates and standardized checklists, because consistency reduces operator burden. Automation should be introduced cautiously, starting with calendar-based reminders and recommended triggers rather than full automatic activation, unless the corridor is well instrumented and the risk of false activation is low. The program should maintain governance for plan updates, because event plans must be refreshed when geometry, parking operations, or construction changes.

- **Roles**: program owner (governance), operations (deployment), traffic engineering (plan maintenance), analyst (reporting), venue/city liaison (updates).
- **Deliverables**: expanded event plan library, annual refresh schedule, and periodic performance reports.
- **Risks**: plan library becomes stale; automated activation causes false positives.
- **Acceptance checks**: each venue corridor passes a validation gate before automation increases.

### Choices
- **Pre-scheduled**: calendar-driven.
- **Semi-auto**: detection-assisted recommendations.
- **Manual**: operator-activated (fallback).

## Technical Mechanics

### Key Parameters
- Trigger thresholds (occupancy, travel time spike)
- Plan duration and recovery rules
- Pedestrian safety constraints

### Guardrails
- Maintain pedestrian minimums/clearance.
- Rate-limit plan switching.
- Require rollback plan and logging.

## MVP Deployment
- One venue corridor.
- Two plans (outbound + recovery).
- Assisted activation.

## Evaluation
- Clearance time and spillback frequency.
- Pedestrian delay and safety proxy behavior.
- Operator workload and adherence.

## References / Standards / Useful Sources
- FHWA ATSPM: https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm

---

Cross-links: Related ideas include rare-event practice mode and crowd-smart crossings.
