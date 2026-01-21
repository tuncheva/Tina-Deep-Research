# 06) Rare-Event Practice Mode

## What it is (precise)
Use a traffic digital twin to **rehearse rare, disruptive scenarios** (special events, detours, evacuations, major incidents, power constraints) and pre-build a **playbook** of vetted timing plans, coordination patterns, and operator actions.

This is different from everyday adaptive control: it focuses on **non-recurring congestion** and “operational readiness,” where the objective is safe, predictable response rather than optimal average-day efficiency.

## Why digital twins matter
Rare events are hard to learn from because they don’t happen often, and when they do, they’re chaotic. A twin helps by:
- running ingress/egress or diversion scenarios repeatedly and safely,
- quantifying queue spillback risks and failure modes,
- validating “turn this plan on when X happens” triggers,
- keeping operator procedures consistent across shifts.

## Easy explanation
Like a fire drill: you practice the hard days in simulation, then keep a set of proven “buttons” to press when the real incident happens.

## What is needed (data & infrastructure)
| Need | Typical options | Notes |
|---|---|---|
| Scenario inputs | event schedules, road closures, incident templates, evacuation routes | Include timing uncertainty (start/end variability). |
| Timing plan library | pre-timed plans, coordination patterns, special-event plans | Needs versioning + rollback. |
| Field monitoring | detectors, CCTV, probe speeds, operator reports | Used to confirm the scenario is unfolding and to adjust. |
| Control interface | central signal system + comms | Enables rapid activation and corridor-level coordination. |
| Digital twin | corridor/network micro/meso sim | Must represent diversion routes + queue spillback. |

## Implementation plan (phased)
### Phase 0 — define “rare events” and success metrics (1–3 weeks)
- List event types: stadium ingress/egress, parade routes, major detours, evacuation outbound, blackout/communications loss.
- Define hard safety constraints (pedestrian mins, clearance, no-blocking-critical-crossings).

### Phase 1 — build scenario playbooks offline (4–10 weeks)
- Create scenario templates with demand ranges.
- Produce candidate plans: longer greens for priority movements, temporary one-way coordination patterns, diversion support plans.

### Phase 2 — table-top + shadow tests (4–8 weeks)
- Run the twin against historical “bad days.”
- Produce operator checklists: what to monitor, when to switch, when to revert.

### Phase 3 — controlled drills (ongoing)
- Run periodic “practice days” (night/weekend) where operators activate playbook plans in a safe window.
- Update the playbook based on measured outcomes.

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Incident response | faster, less improvisation | wrong plan if event differs from assumed | include decision tree + monitoring gates |
| Safety | pre-vetted pedestrian/clearance constraints | over-prioritizes vehicles during chaos | explicit multimodal constraints + max waits |
| Staffing | reduces cognitive load | still needs after-hours coverage | automation for activation + on-call rotation |

## Real-world anchors (what exists today)
WSDOT describes **planned event or incident signal timing** as synchronizing groups of signals to favor traffic entering/exiting venues, and also using timing plans to divert traffic around incidents or support evacuations; it notes agencies monitor conditions in real time and often turn plans on/off manually (with detectors sometimes enabling automatic control). That is essentially the operational “playbook” concept this idea formalizes inside a digital twin rehearsal workflow. [WSDOT TSMO: Planned event or incident signal timing](https://tsmowa.org/category/intelligent-transportation-systems/planned-event-or-incident-signal-timing)

FHWA’s planned special events handbook notes agencies can download and implement **special signal timing plans** for event ingress/egress via closed-loop or centrally controlled systems, and that adaptive systems can adjust to event-generated flows. This supports the feasibility of maintaining dedicated event plans and coordinating them from operations centers. [FHWA: Managing Travel for Planned Special Events (Chapter 4)](https://ops.fhwa.dot.gov/publications/fhwaop04010/chapter4_04.htm)

## Evaluation checklist (practical)
- Time-to-activate correct plan (minutes)
- Queue spillback onto critical links (count, duration)
- Travel time reliability on designated ingress/egress routes
- Pedestrian service (max waits near venues)
- Operator workload (alerts acknowledged, manual interventions)

## Sources
- https://tsmowa.org/category/intelligent-transportation-systems/planned-event-or-incident-signal-timing
- https://ops.fhwa.dot.gov/publications/fhwaop04010/chapter4_04.htm
