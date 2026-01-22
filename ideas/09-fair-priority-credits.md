# 09) Fair Priority Credits (Transparent Green-Time Budgeting)

## What it is (precise)
A **priority credits** system is a governance + control mechanism that allocates a limited “budget” of priority interventions (e.g., green extension, early green, phase insertion) to eligible users (transit, emergency, freight, bikes/pedestrians) according to transparent rules.

Instead of “always give the bus green,” you define:
- **who** can request priority,
- **what** actions the controller is allowed to take,
- **how often** priority can be granted (budget),
- **how impacts are compensated** (recovery),
- and **how equity is enforced** (by corridor, neighborhood, time-of-day).

This idea sits on top of standard priority mechanics: it doesn’t invent TSP/EVP; it adds **fairness accounting + auditability**.

## Why digital twins matter
A digital twin can simulate and forecast the **system-level tradeoffs** of priority budgeting:
- travel-time reliability for buses vs cross-street delay,
- cascading coordination impacts (recovery to coordination),
- neighborhood distribution of benefits/harms.

Aimsun describes ranking response plans by KPIs and “predictive policy-based prioritization,” which aligns with using a policy layer to decide how/when to deploy priority actions. [Aimsun Live](https://www.aimsun.com/aimsun-live/)

## Easy explanation
Priority is like spending money. Credits make it clear how much “green time” you can spend helping buses or emergency vehicles, so one group doesn’t silently take all the benefit.

## How priority works in practice (baseline)
The ITS America TSP overview describes:
- **priority vs preemption** (different goals and operational impacts),
- common **priority treatments** (early green/red truncation, green extension, phase insertion/rotation, actuated transit phase, adaptive/real-time control),
- and the importance of **signal recovery/transition** back to coordination after priority events.

These are the building blocks that a credit/budget policy controls. [ITS America / AZMAG TSP overview (2002)](https://azmag.gov/LinkClick.aspx?fileticket=O8w-C3FPGWk%3D&tabid=527&portalid=0&mid=3809)

## Standards / interoperability anchor
**NTCIP 1211** defines management information for **Signal Control and Prioritization (SCP)** systems and explicitly frames SCP as a system including priority requesters (e.g., transit vehicles), signal controllers, and management centers that configure/monitor. This is a standards basis for implementing priority requests, monitoring state, and auditing outcomes across devices. [NTCIP 1211](https://www.ntcip.org/file/2018/11/NTCIP1211-v0224j.pdf)

## Credit system design (one workable model)
### 1) Eligibility and classes
Define requester classes (example):
- Class E: emergency (preemption allowed)
- Class T: transit (priority allowed)
- Class B: bike corridor (priority allowed)

### 2) Actions allowed (by class)
| Action | Example | Allowed for |
|---|---|---|
| Green extension | keep green a few seconds longer | T, B |
| Early green (red truncation) | shorten conflicting green | T |
| Phase insertion/rotation | insert or reorder a phase | T (limited) |
| Preemption | interrupt normal operation | E |

### 3) Budgets
Budgets are enforced per:
- intersection, corridor, and time-of-day (e.g., peak vs off-peak),
- and optionally per neighborhood (equity).

Example budget units:
- 1 credit = 5 seconds of additional green **or** 1 priority event.

### 4) Fairness constraints
- Max priority events per cycle/hour
- Max added delay per cross-street approach
- Max pedestrian wait
- “No repeat harm” rule: don’t push delay into the same side-street repeatedly

### 5) Recovery rules
Given recovery is critical (TSP guide), define:
- how many cycles to return to coordination,
- how to repay skipped green time.

## Implementation plan (phased)
### Phase 0 — measurement + stakeholder agreement
- Define objectives: bus on-time performance, emergency response time, person delay.
- Agree on fairness metrics and reporting (public transparency).

### Phase 1 — baseline TSP/EVP deployment (if not already present)
- Deploy priority request detection/communications.
- Confirm controller support for requested treatments.

### Phase 2 — add credit/budget layer
- Implement a policy engine that:
  - checks eligibility,
  - checks budgets,
  - selects a permitted priority treatment,
  - logs decision + predicted impacts.

### Phase 3 — digital twin “policy tuning”
- Use the twin to tune budgets and thresholds.
- Validate that recovery maintains coordination stability.

### Phase 4 — audit + iteration
- Publish monthly reports: who got priority, where, and impacts.
- Adjust budgets by corridor performance and equity analysis.

## Comparison table (priority governance models)
| Model | How it allocates priority | Strengths | Weaknesses |
|---|---|---|---|
| Always-on priority | all eligible requests granted | simple; strong bus benefit | can harm cross streets; opaque equity |
| Threshold-based priority | grant only when bus is late / queue exceeds X | reduces unnecessary interventions | still can concentrate harm |
| Credit/budgeted priority (this idea) | limited budget by corridor/time/neighborhood | transparent + tunable equity | needs measurement + reporting |
| Auction/market-like | requests “bid” for priority | can optimize under scarcity | hard to govern; equity risk |

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Transparency | explainable “who gets green and why” | complex to communicate | publish simple rules + dashboards |
| Performance | maintains transit reliability under constraints | may reduce peak bus gains vs always-on | target “late buses only”; corridor budgets |
| Equity | prevents priority from harming same streets repeatedly | depends on good measurement | neighborhood KPIs + twin-based evaluation |
| Safety | preemption remains strict; priority bounded | misconfiguration can break coordination | conservative defaults; staged rollout |

## MVP (smallest useful deployment)
- Implement budgeting for **one class** (transit priority) on **one corridor**.
- Define and publish:
  - max priority events/hour,
  - max added cross-street delay,
  - max pedestrian wait.
- Add a simple monthly “credits statement”: credits spent, denied, and impacts.

## Open questions
- What is the best unit for budgeting: seconds of green, events, or person-delay impact?
- How should credits roll over (or not) across days and time-of-day buckets?
- How do we prevent gaming (e.g., repeated low-value requests) while staying transparent?

## Evaluation checklist
- Bus travel time and reliability (p90, on-time)
- Cross-street delay distribution
- Pedestrian max wait and compliance
- Count of priority events by corridor/neighborhood

## Sources
- https://azmag.gov/LinkClick.aspx?fileticket=O8w-C3FPGWk%3D&tabid=527&portalid=0&mid=3809
- https://www.ntcip.org/file/2018/11/NTCIP1211-v0224j.pdf
- https://www.aimsun.com/aimsun-live/
