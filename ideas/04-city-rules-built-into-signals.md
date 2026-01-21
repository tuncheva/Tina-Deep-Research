# 04) City Rules Built Into Signals

## What it is (precise)
This idea encodes **city policy goals** (e.g., school-zone safety, pedestrian priority, transit priority, quiet-night operations, hospital access routes) as explicit **constraints and objectives** that the signal controller must respect when choosing timings.

Instead of “optimize travel time, then hope the result is acceptable,” the controller solves a *constrained* problem: pick phase plans/timing changes that satisfy minimum pedestrian service, safe phasing rules, and policy priorities.

NACTO explicitly states that the operation of a traffic control system should mirror a city’s policy goals and that signal timing influences safety and mode choice, calling out priority tools like leading pedestrian intervals, bike signal coordination, and transit signal priority. https://nacto.org/publication/urban-street-design-guide/intersection-design-elements/traffic-signals/signalization-principles/

## Why digital twins matter
Policy-constrained control becomes hard to tune by intuition because the system must consider many tradeoffs (delay vs compliance vs safety vs multimodal outcomes). A calibrated digital twin helps by:
- testing candidate “policy rule sets” before field rollout,
- quantifying impacts (ped delay, vehicle delay, compliance proxies),
- exposing unintended consequences (spillback, risky turns, long pedestrian waits).

FHWA’s signal timing guidance emphasizes that design choices aim for safe and efficient operation and that timing choices interact with design elements like phasing and pedestrian timing (including tools like leading pedestrian intervals). https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter4.htm

## Easy explanation
The city sets rules (for example: “kids and pedestrians get enough crossing time” or “buses on this corridor get priority”), and the signal system is not allowed to break them—even if it could reduce car delay.

## What is needed (data & infrastructure)
| Need | Typical options | Notes |
|---|---|---|
| Policy rule definitions | written policy + engineering parameters | Convert policy to measurable constraints (min walk, max cycle, priority weights). |
| Multimodal detection | ped pushbuttons/APS, video/radar, transit AVL, bike detection | Needed to measure whether policy goals are met in practice. |
| Controller feature support | modern controllers / ATMS | Must support needed phasing features (e.g., LPI, TSP), and consistent plan deployment. |
| Digital twin model | intersection/corridor simulation | Used to test policy constraints and tune parameters. |
| Audit + monitoring | logs, KPIs dashboards | Needed to prove compliance and manage exceptions. |

## Implementation plan (phased)
### Phase 0 — define “policy as parameters” (2–6 weeks)
- Pick 2–3 priority corridors or zones (school areas, transit corridors).
- Translate policy into constraints and KPIs (e.g., max ped wait, min walk, minimum LPI usage).

### Phase 1 — baseline measurement (4–8 weeks)
- Measure current delay, pedestrian compliance, and safety proxies.
- Identify where policy is currently violated (e.g., insufficient crossing time, excessive waits).

### Phase 2 — twin-based rule testing (4–10 weeks)
- Implement the policy constraint set in the twin.
- Simulate time-of-day patterns and stress cases (surges, blocked crosswalks, bus bunching).

### Phase 3 — controlled deployment (4–12 weeks)
- Deploy in a limited area/time window.
- Add guardrails: max cycle changes, rollback, and “safe mode” if detection fails.

### Phase 4 — scale + governance (ongoing)
- Publish a simple policy + KPI “contract” for each corridor.
- Expand to more objectives (night noise reduction, emergency response routing).

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Safety + equity | makes safety/multimodal goals enforceable | may increase vehicle delay | communicate objectives; choose corridors where policy benefit is highest |
| Governance | decisions become auditable and explainable | political conflict over priorities | transparent KPIs; public-facing dashboards |
| Operations | reduces ad-hoc overrides | constraint tuning is hard | start small; simulate first; iterate |
| Robustness | avoids “optimize into a corner” | detection failures can violate rules | health checks; conservative defaults; fallback plans |

## Real-world anchors (what exists today)
- NACTO emphasizes aligning signal operation with city policy goals and describes priority tools (e.g., leading pedestrian intervals, bike coordination, transit signal priority). https://nacto.org/publication/urban-street-design-guide/intersection-design-elements/traffic-signals/signalization-principles/
- FHWA’s signal timing guidance documents pedestrian phasing options (including leading pedestrian intervals) and describes how signal design/timing choices affect safety and efficiency. https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter4.htm

## Evaluation checklist (practical)
- Pedestrian: max wait time, compliance proxy, crossing completion rate
- Transit: on-time performance and travel time reliability (if applicable)
- Safety proxies: turning conflicts (if measured), red-light violation proxies
- Vehicles: corridor travel time distribution (median + p95), queue spillback
- Policy compliance: % time rules are met (by time of day)

## Sources
- https://nacto.org/publication/urban-street-design-guide/intersection-design-elements/traffic-signals/signalization-principles/
- https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter4.htm
