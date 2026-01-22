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

## Comparison table (policy encoding approaches)
| Approach | What it looks like | Strengths | Weaknesses |
|---|---|---|---|
| Hard constraints | “max ped wait <= X”, “min walk >= Y” | enforceable; clear audits | can reduce flexibility |
| Weighted objectives | score = delay + safety + transit weight | tunable; smoother tradeoffs | harder governance |
| Rule-based modes | school-hour mode, night-quiet mode | easy to communicate | edge cases proliferate |
| Permit/exception workflow | explicit override with expiry | realistic ops | needs strong governance |

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

## MVP (smallest useful deployment)
- Pick **one policy zone** (e.g., 2–6 intersections around a school or transit corridor).
- Encode **3–5 hard rules** (max pedestrian wait, LPI enabled during school hours, max cycle length, min walk/clearance).
- Add **one “policy override” workflow**: who can override, for how long, and automatic reversion.
- Publish a corridor “policy contract” and a weekly compliance report.

## Open questions
- How should policy translate into constraints: fixed caps, dynamic caps (by time-of-day), or weighted objectives?
- What is the right governance for exceptions (emergency routes, construction, major events)?
- How do we verify compliance when detection is degraded (fallback assumptions)?

## Evaluation checklist (practical)
- Pedestrian: max wait time, compliance proxy, crossing completion rate
- Transit: on-time performance and travel time reliability (if applicable)
- Safety proxies: turning conflicts (if measured), red-light violation proxies
- Vehicles: corridor travel time distribution (median + p95), queue spillback
- Policy compliance: % time rules are met (by time of day)

---

## Deep dive (engineering): turning policy into enforceable constraints
The core challenge is translating political/strategic statements into **measurable, testable** constraints.

### Policy primitives
Think of policy as a set of primitives that can be composed:
- **Minimum service**: “ped wait <= X”, “bike green at least every Y seconds”.
- **Priority weighting**: “bus person-delay is worth 3× car delay”.
- **Forbidden actions**: “no permissive lefts during school hours”.
- **Contextual modes**: “quiet-night mode from 22:00–06:00”.
- **Geofences**: apply rule sets inside zones (school, hospital).

### Constraint types
| Type | Example | Implementation pattern |
|---|---|---|
| Hard safety constraint | min walk/clearance; conflict matrix | enforced at controller level |
| Hard policy constraint | max pedestrian wait; max cycle | enforced in optimizer + controller caps |
| Soft objective | minimize person delay | part of scoring function |
| Procedural constraint | operator approval required | workflow/permissions |

Cross-link: see [`ideas/20-explainable-signals.md`](ideas/20-explainable-signals.md) for how to explain constraint-based decisions.

---

## Deep dive (governance): the “policy contract” per corridor
A corridor policy contract is a one-page document (and a machine-readable schema) that states:
- objectives (e.g., transit reliability, pedestrian safety),
- hard constraints (max ped wait, min LPI),
- allowed tools (LPI, TSP, offset changes),
- monitoring KPIs and reporting cadence,
- exception process (who can override; expiry).

Why it matters: it prevents the system from silently drifting into a different “priority regime.”

---

## Deep dive (twin workflow): test rules before you enforce them
In the twin, run a suite of scenarios:
- weekday peaks vs weekend,
- school dismissal surge,
- bus bunching,
- detection degraded,
- incident detour.

For each scenario:
1. verify constraints are always met,
2. quantify costs (vehicle delay, queue spillback),
3. identify boundary cases where constraints “bind” and drive outcomes.

This creates a **traceable justification**: “This policy set increases car delay by 6–9% but reduces ped max wait by 40% in the school zone.”

---

## Deep dive (operations): exception handling without chaos
Real operations require exceptions (construction, emergency routes).

Exception workflow:
- explicit override type (e.g., “construction detour mode”),
- time-bounded (auto-expiry),
- logged with reason + approver,
- monitored for constraint violations.

Guardrail: forbid “silent overrides” that permanently disable policy constraints.

---

## Metrics (policy + engineering)
- % time each hard constraint is satisfied (by corridor, by hour)
- Max pedestrian wait distribution and tail (p95/p99)
- Person-delay split by mode (walk/bike/transit/car)
- TSP usage count and its impact on cross-street delay
- Safety proxies: turning conflicts (if measured), speed/harsh-braking proxies
- Override frequency and mean override duration

---

## Operator UX: “policy lens” view
Cross-link: see [`ideas/08-operator-option-menu.md`](ideas/08-operator-option-menu.md)

The operator should be able to answer:
- What policy regime is active right now (school / normal / quiet-night)?
- Which constraints are binding?
- What changes are allowed?
- What exception can I request, and for how long?

---

## Related topics
- [`ideas/17-safety-first-signals.md`](ideas/17-safety-first-signals.md) — safety constraints and safe decision ordering.
- [`ideas/12-privacy-friendly-learning.md`](ideas/12-privacy-friendly-learning.md) — privacy when measuring multimodal outcomes.
- [`ideas/03-real-time-what-if-button.md`](ideas/03-real-time-what-if-button.md) — evaluating policy changes with counterfactuals.

## Sources
- https://nacto.org/publication/urban-street-design-guide/intersection-design-elements/traffic-signals/signalization-principles/
- https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter4.htm
