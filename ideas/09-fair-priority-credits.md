# 09) Fair Priority Credits: Transparent Green-Time Budgeting

## Catchy Explanation
Priority (for buses, emergency vehicles, maybe bikes) is like a shared city resource. Credits make it explicit: you can spend priority, but not endlessly, and not always in the same neighborhood.

## What it is (precise)
A **fair priority credits** system allocates a limited budget of priority interventions (early green, green extension, phase insertion, plan switch) to eligible requesters (transit, emergency, freight, bike corridors) under transparent rules. It extends standard TSP/EVP mechanics with **governance, accounting, and equity constraints**:
- *Eligibility*: who can request, how verified.
- *Budget*: credits per route/corridor/time window.
- *Spending rules*: which actions cost what.
- *Recovery*: how the corridor returns to coordination.
- *Fairness caps*: prevent repeatedly harming the same cross streets/areas.

A digital twin is used to tune credit budgets and spending rules by forecasting tradeoffs (bus reliability vs side-street delay, queue spillback risk). All priority actions are logged with reason codes and evidence.

## Benefits
- **Equity and predictability**: avoids ad-hoc “always prioritize this route” behavior.
- **Sustained performance**: budgets prevent unbounded delay growth on cross streets.
- **Transparency**: clear accounting of where/when priority was spent.
- **Safer priority**: bounded interventions reduce risky phase disruptions.

## Challenges
- **Governance complexity**: budgets and fairness rules are political.
- **Measurement and verification**: need reliable requester detection (AVL, EVP, etc.).
- **Edge cases**: conflicting requests and emergency overrides.
- **Scalability**: accounting across a city network must be robust.

## Implementation Strategies

### Infrastructure Needs
- **Requester detection**: transit AVL, emergency vehicle preemption interface, bike corridor definitions.
- **Policy/credit engine**: budgets, spend rules, fairness constraints.
- **Controller integration**: ability to execute priority actions and recover coordination.
- **Twin + evaluation**: simulate credit policies and network impacts.
- **Audit dashboards**: credits spent by route/corridor/time + KPI outcomes.

### Detailed Implementation Plan
#### Phase 1: Policy, Eligibility, and Budget Design (Weeks 1–6)
The city should begin by aligning transit, traffic engineering, and emergency services on which priority requests will be in scope, because EVP typically has absolute priority while TSP often needs bounds and fairness rules. The team should define eligibility and verification mechanisms (for example, transit AVL-based requests, authenticated EVP, and optional bike corridor triggers) and should define what priority actions are allowed at which intersections. The city should then define a credit budget structure that makes sense operationally, such as credits per route per time window, corridor-level caps, and fairness caps that limit added delay to cross streets or neighborhoods. The policy team should document exception handling, including how emergencies override budgets and how conflicting requests are resolved.

- **Roles**: traffic engineering lead (signal impacts), transit operations (route priorities and schedules), emergency services liaison (EVP rules), policy owner (equity and governance), data/analytics lead (measurement plan).
- **Deliverables**: priority credit policy contract (versioned), eligibility rules, budget definitions, fairness caps, and conflict-resolution policy.
- **Risks**: political disagreement over fairness constraints; unclear definition of “equity”; scope creep into too many requester types.
- **Acceptance checks**: budgets and caps are measurable, emergency overrides are explicitly defined, and policy is signed off by stakeholders.

#### Phase 2: Build Credit Engine, Logging, and Controller Integration (Weeks 7–16)
The engineering team should implement a credit accounting service that receives verified priority requests, checks budgets and caps, and decides whether and how to grant priority in the current cycle. The team should implement a rules-based conflict resolver that handles simultaneous requests (for example, two buses, or a bus and freight request) and always yields to emergency vehicle priority where required. The signal integration should be implemented using controller-supported priority mechanisms (early green, green extension, phase insertion, or plan switch) and should include a recovery strategy that returns the corridor to coordination after disruptions. Every granted or denied request should be logged with a reason code, the budget state, and the expected recovery action so the system can be audited.

- **Roles**: software engineer (credit engine), traffic engineer (controller mapping), IT/security (authentication and access), operations (workflow), QA engineer (test scenarios).
- **Deliverables**: working credit engine, controller integration adapters, audit log schema, and operational dashboards.
- **Risks**: controller limitations vary by vendor; incorrect recovery logic can break progression; logging gaps undermine trust.
- **Acceptance checks**: priority actions execute reliably in a test environment, recovery behavior is validated, and logs contain sufficient information for after-action review.

#### Phase 3: Twin-Based Tuning and Pilot Corridor Deployment (Weeks 17–28)
The team should use a calibrated twin to tune credit budgets and action “costs,” because the goal is to improve transit reliability without creating unacceptable side-street delay or spillback. The pilot should start on a single transit corridor with a small number of intersections where TSP is already common, because this reduces operational complexity. The city should run shadow mode first to measure how often requests would be granted under the proposed budgets, and it should then run assisted activation where operators can override during unexpected conditions. During the pilot, the team should measure transit travel time reliability, headway regularity, cross-street delay distributions, and spillback or blocked-box frequency.

- **Roles**: transit operations (pilot route), traffic engineering (pilot corridor), analyst (evaluation), operations (activation), modeler (tuning).
- **Deliverables**: tuned budgets and costs, pilot evaluation report, and recommended policy adjustments.
- **Risks**: bus bunching dynamics can confound results; pilot construction or incidents can bias evaluation.
- **Acceptance checks**: transit reliability improves without violating fairness caps, and denied requests are explainable and accepted by stakeholders.

#### Phase 4: Scaling and Public Reporting (Weeks 29–44)
The program should scale to additional routes and corridors using templates for budgets, costs, and fairness caps, because scaling without standardization increases political and operational friction. The city should implement public reporting that summarizes where priority was spent, what reliability benefits were achieved, and what cross-street impacts occurred, because transparency is the point of the credit approach. The team should also establish a governance cadence where transit and traffic engineering jointly review budgets and adjust them based on measured outcomes.

- **Roles**: program owner (governance), transit and traffic engineering leads (joint review), analyst (reporting), operations (deployment support).
- **Deliverables**: multi-corridor deployment, periodic “priority equity” report, and updated budget policies.
- **Risks**: inequitable distribution if budgets follow political influence; scaling increases integration complexity.
- **Acceptance checks**: reporting is consistent and auditable, and scaling does not create recurring corridor instability.

#### Phase 5: Optimization and Event Modes (Ongoing)
The city should maintain a continuous improvement loop that adjusts budgets and costs based on observed outcomes, such as increasing credits where reliability gains are high and cross-street impacts are low. The program should also define event modes (for example, major downtown events) where budgets and caps temporarily change under documented governance rules, because demand patterns during events differ significantly. Over time, the city can incorporate more predictive logic, but it should keep the credit accounting and fairness constraints simple enough to be explainable.

- **Roles**: program owner (policy updates), data/analytics (monitoring), traffic engineering (signal impacts), transit operations (service impacts).
- **Deliverables**: updated budget schedules, event mode policies, and periodic performance reviews.
- **Risks**: dynamic budgets can be controversial; overly complex rules reduce trust.
- **Acceptance checks**: changes are versioned and reversible, and performance reviews demonstrate sustained benefits.

### Choices
- **Unlimited**: always honor priority (simple, but can be unfair).
- **Threshold-based**: only when late/delayed.
- **Credit-based**: budgeted and auditable (this idea).

## Technical Mechanics

### Key Parameters
- Credit budgets (by corridor/route/time)
- Action costs (seconds of green extension, plan switch cost)
- Fairness caps (max delay added per cross-street)
- Recovery rules (how coordination is restored)

### Guardrails
- Emergency overrides must still work.
- Cap maximum disruption per cycle/time window.
- Require audit log entries for each spend.

## MVP Deployment
- One transit route on one corridor.
- Two actions (green extension, early green).
- Simple daily budget + fairness cap.

## Evaluation
- Transit on-time performance and travel time reliability.
- Cross-street delay distribution.
- Credits spent per period and “credit starvation” events.
- Spillback/blocked-box frequency.

## References / Standards / Useful Sources
- NTCIP 1211 (Signal Control and Prioritization): https://www.ntcip.org/file/2018/11/NTCIP1211-v0224j.pdf
- FHWA Traffic Signal Timing Manual: https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf

---

Cross-links: Related ideas include city rules, safety-first signals, and explainable signals.
