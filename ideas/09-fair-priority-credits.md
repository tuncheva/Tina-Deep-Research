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

## Implementation Additions (Implementation-Ready)

### 1) Define budget units (impact-based, not just seconds/events)

A “credit” must represent **scarce intersection capacity and disruption cost**, not just an abstract token. Use a unit that is:
- computable in real time,
- explainable,
- comparable across locations,
- improvable over time as instrumentation improves.

#### Candidate budget units (required visual: unit definition → pros/cons → required data)

| Unit | Definition | Pros | Cons | Best for | Required data |
|---|---|---|---|---|---|
| **Seconds of green change** | `Δgreen_sec` (early green / green extension / red truncation) | simplest; controller-native | ignores who is harmed; ignores coordination damage | MVP; single isolated intersection | controller timing, phase state, basic volume estimates |
| **Priority events** | 1 credit per granted request | simple accounting | very coarse; gaming risk | early pilots; low request volumes | request counts, classification by class/route |
| **Marginal person-delay imposed on others** | estimated `Δ person-delay` on non-priority users | impact-based and fairness-aligned | needs data + estimation | mature ATSPM/probe deployments | detectors/ATSPM, probe travel times, occupancy, basic demand model |
| **Coordination disruption cost** | proxy for recovery: AOG degradation / offset recovery time / extra split failures | captures network harm | needs corridor telemetry | coordinated arterials | ATSPM arrivals-on-green, split failures, offset deviations, corridor topology |

#### Real-time cheap impact estimation (vendor-neutral)
Use a tiered estimator:
1. **Tier 0 (fallback)**: cost = fixed seconds/events per treatment.
2. **Tier 1 (ATSPM-based)**: infer extra red time imposed on opposing phases + estimate queue growth from split failures / occupancy proxies.
3. **Tier 2 (person-delay)**: combine:
   - arrivals/served counts (detectors),
   - probe speeds (segment travel time),
   - simple shockwave/queue model for added delay.

Fallback when data missing:
- assume conservative (higher) harm cost,
- or deny non-critical priority and preserve emergency preemption.

#### Recommended phased approach
- Phase A: **seconds/events** credits + strict caps + robust logging.
- Phase B: add **coordination disruption** proxies on coordinated corridors.
- Phase C: graduate to **marginal person-delay** accounting after monitoring is stable.

**Source alignment**: FHWA describes TSP treatments such as red truncation (early green) and green extension, and notes that priority may be accomplished via extending greens, altering phase sequences, or including special phases without interrupting coordination ([`Traffic Signal Timing Manual: Chapter 9`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm)).

### 2) Fairness constraints: operational definitions, accounting windows, and reporting

“Fairness” becomes operational when you define **harm**, choose **accounting windows**, and enforce **caps**.

#### Define “harm” per movement/area
Choose a small set of harm measures with clear semantics:
- **added delay** (seconds) or **person-delay** (preferred when feasible).
- **max wait** (p95) for pedestrians or minor-street movements.
- **queue spillback minutes** (time queue blocks upstream intersection/crosswalk).
- **coordination degradation**: AOG drop or recovery time.

#### Accounting windows
Use multiple windows to prevent “death by a thousand cuts”:
- **per cycle**: hard cap on how much a single request can disrupt.
- **15-min rolling**: operational fairness (shift-level complaints).
- **peak period/day**: predictable service for non-priority approaches.
- **weekly**: equity reporting and policy review.

#### Fairness rule patterns
- **Per-approach cap**: max harm per approach per window.
- **Rolling corridor cap**: total harm within a geofence.
- **Disparity constraint**: limit difference across neighborhoods/approaches.
- **Protected movements**: hard floors for ped phases near schools/hospitals.

#### Reporting template: “credits statement”
Publish (internally; and externally in summarized form):
- credits spent by corridor/route/time.
- who benefited (class: transit/emergency/freight/bike).
- who “paid” (approaches/areas) and how much harm was imposed.

### 3) Anti-gaming / abuse resistance controls (transparent + auditable)

Priority is gameable if requests are cheap and verification is weak.

Controls:
- **Low-value/high-frequency detection**:
  - repeated requests with minimal realized benefit.
  - detect “spam”: high grant rate with low lateness or low passenger benefit.
- **Rate limits and cooldowns**:
  - per vehicle ID/route, per intersection, per corridor.
- **Verification/classification checks**:
  - transit: AVL validation + optionally schedule adherence triggers.
  - emergency: authenticated preemption only; never count against budgets.
  - misclassification safeguards: priority ≠ preemption.
- **Exception policies**:
  - incidents/special events allow temporary budget changes with explicit reason code and audit trail.

### 4) Architecture + real-time feasibility: where the policy engine lives and how it fails safely

#### Placement options

| Option | Where | Pros | Cons | Best fit |
|---|---|---|---|---|
| Edge | controller/cabinet | ultra-low latency; resilient to comms loss | limited compute; harder citywide fairness | isolated intersections; EVP enforcement |
| Corridor edge node | field cabinet/edge server | good latency + corridor context | needs field hardware | coordinated arterials |
| Central (TMC/ATMS) | central server | global budgets + reporting | comms dependency; latency | citywide policy, auditing |

#### Request → policy → controller → accounting → reporting diagram (required visual)

```text
[Priority Request]
  - Source: transit AVL, EVP device, freight fleet, bike corridor trigger
  - Includes: requester_class, route/vehicle_id, location, timestamp, context
        │
        ▼
[Ingress & Verification]
  - Authenticate / validate class (transit / emergency / freight / bike)
  - Classify as priority vs preemption
        │
        ▼
[Policy & Credit Engine]
  - Apply eligibility + anti-gaming checks
  - Evaluate credit budgets + fairness caps
  - Run safety/feasibility gate (ped mins, clearance, mode constraints)
  - Decide: GRANT (with treatment), PARTIAL, or DENY
        │
        ▼
[Controller / Field Action]
  - Apply treatment (early green / green extension / phase insert / none)
  - Start recovery plan to restore coordination
        │
        ▼
[Accounting & State Update]
  - Decrement credits / update harm ledgers
  - Record budget_before / budget_after
  - Tag affected approaches / watch areas
        │
        ▼
[Monitoring & Impact Measurement]
  - Measure realized impacts (transit OTP, delay, queues, spillback)
  - Compare predicted vs realized harm
        │
        ▼
[Reporting & Governance]
  - Internal dashboards (spent / denied / who benefited / who paid)
  - Public-facing summaries (equity slices, corridor reports)
  - Inputs to budget resets and policy updates
```

This diagram is the reference **request → policy engine → controller action → accounting → reporting** flow for the fair priority credit system.

#### Latency and reliability
- What must happen **within a cycle**: eligibility check, hard safety gating, basic budget check.
- What can be delayed: reporting aggregation, weekly equity summaries.

#### Safety/feasibility gating
Reuse a deployability gate concept:
- controller constraints, ped mins, clearance intervals.
- coordination maintenance and recovery.

**Source alignment**: FHWA distinguishes **preemption** (“transfer of normal operation … to a special control mode”) from **signal priority** (altering existing operations to serve a priority vehicle without disrupting coordination). Source: FHWA *Traffic Signal Timing Manual*, Chapter 9 ([`chapter9`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm)).

#### Failure behavior (safe defaults)
- **Comms loss**: revert to controller-native default TSP rules; central budgets stop decrementing; log locally.
- **Policy engine down**: disable non-critical priority; preserve EVP.
- **Stale budgets / untrusted time**: freeze spending; allow only emergency.
- **Data missing**: switch to conservative (higher) cost or deny optional requests.

### 5) Stakeholder governance: who sets budgets, how conflicts are resolved, and how changes are managed

#### Roles / RACI (example)

| Decision | Signals Ops | Transit Agency | Emergency Services | Freight Program | Equity Office | City Leadership |
|---|---|---|---|---|---|---|
| Define eligible classes | R | C | C | C | C | A |
| Set transit budgets | C | R | I | I | C | A |
| Define EVP policy | I | I | R/A | I | C | C |
| Approve fairness constraints | R | C | C | C | R/A | A |
| Publish reports | R | C | I | I | C | A |

#### Budget-setting process
- initial allocation: pilot-based with transparent assumptions.
- seasonal adjustments: school year, construction, demand shifts.
- incident overrides: time-bounded with explicit approvals.

#### Conflict arbitration framework (default)
1. **Safety/legal minima**
2. **Emergency preemption**
3. **Transit commitments** (headway/OTP) within caps
4. **Multimodal/equity constraints**
5. **Efficiency** (delay minimization)

#### Change control + transparency
- policy versioning (semver-like) + release notes.
- public-facing summaries of what changed and why.

**Source alignment**: FHWA’s environmental justice definition includes adverse effects such as congestion and denial/reduction/delay of benefits, and defines “disproportionately high and adverse” impacts on minority/low-income populations ([`FHWA Order 6640.23`](https://highways.dot.gov/laws-regulations/directives/orders/664023:1)). Use this to justify “no repeat harm” and disparity constraints.

### 6) Concrete implementation artifacts (deployable)

#### Machine-readable policy example (YAML)

```yaml
policy_id: "PRIORITY-CREDITS-01"
version: "1.0.0"
credit_unit:
  type: "seconds_green"   # later: "person_delay" | "coord_disruption"

classes:
  emergency:
    eligibility: { auth: "evp_authenticated" }
    treatments_allowed: ["preemption"]
    budget: { type: "unlimited" }
    override: true

  transit:
    eligibility:
      auth: "avl_signed"
      conditions:
        - "behind_schedule >= 60s"  # optional; policy choice
    treatments_allowed: ["early_green", "green_extension"]
    budgets:
      - scope: { route: "Route_10", window: "weekday_peak_am" }
        amount: 300   # seconds of green change
      - scope: { corridor: "MainSt", window: "weekday_peak_pm" }
        amount: 600
    fairness:
      per_approach_harm_cap:
        window: "15m"
        max_added_delay_sec: 120
      protected_movements:
        - { movement: "ped_school_crossing", min_service: "never_degrade" }
    anti_abuse:
      cooldown_sec: 120
      rate_limit:
        per_vehicle_per_15m: 6
        per_intersection_per_15m: 20

  freight:
    eligibility: { auth: "fleet_signed" }
    treatments_allowed: ["green_extension"]
    budgets:
      - scope: { corridor: "IndustrialAve", window: "weekday_midday" }
        amount: 120

exception_modes:
  incident:
    activation: { source: "TMC_declared" }
    temporary_budget_multiplier: 2.0
    duration_max: "2h"
    audit_required: true

logging:
  required_fields:
    - request_id
    - timestamp
    - intersection_id
    - requester_class
    - requester_id
    - eligibility_result
    - budget_before
    - cost_estimate
    - decision
    - treatment_applied
    - recovery_action
    - predicted_harm
    - realized_harm
    - operator_notes
```

#### Event/log schema (audit trail)

Events (append-only):
- `request_received`
- `eligibility_decided`
- `credit_checked`
- `treatment_applied`
- `recovery_completed`
- `impact_measured`

Each event includes: state snapshot, timing plan/mode, data freshness, and reason codes.

#### KPI set for monitoring success
- transit travel time reliability + headway adherence.
- cross-street delay distributions (p50/p95) and ped delay where relevant.
- coordination recovery time (post priority/preemption).
- credit spend rates, denied-request rates, cooldown hits.
- equity slices: impacts and benefits by neighborhood/time-of-day.

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
- NTCIP 1211 (Signal Control and Prioritization): [`NTCIP 1211`](https://www.ntcip.org/file/2018/11/NTCIP1211-v0224j.pdf)
- FHWA Traffic Signal Timing Manual (TSP + Preemption overview): [`chapter9`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm)
- FHWA Order 6640.23 (Environmental Justice definitions): [`FHWA Order 6640.23`](https://highways.dot.gov/laws-regulations/directives/orders/664023)

---

## Implementation Checklist
- [ ] Define requester classes, eligibility verification, and allowed treatments per intersection.
- [ ] Choose initial credit unit (seconds/events) and define conversion tables per treatment.
- [ ] Define fairness caps (per approach/corridor) and accounting windows (cycle/15m/day/week).
- [ ] Implement policy engine with safe defaults (EVP always works; fail-safe on comms loss).
- [ ] Implement controller adapters for early green/green extension and recovery logic.
- [ ] Implement anti-abuse controls (cooldowns, rate limits, verification checks).
- [ ] Implement audit log schema + dashboards (spent/denied/why + impacts).
- [ ] Run shadow mode, validate estimator tiers, then pilot in assisted mode.
- [ ] Publish internal “credits statement”; iterate budgets and caps through governance cadence.

## Operations Runbook (SOP)

### SOP 0 — Normal operations
1. Receive priority request (AVL/PRG) with requester metadata.
2. Verify eligibility and classify request (priority vs preemption).
3. Run safety/feasibility gating (ped mins, clearance, mode constraints).
4. Evaluate credit availability and fairness caps for the relevant window.
5. Select treatment (early green / green extension / none) and apply.
6. Monitor recovery to coordination; log treatment and recovery completion.
7. Measure realized impacts; update spend ledgers.

### SOP 1 — Emergency preemption
- Always grant authenticated EVP.
- Log the preemption, and track recovery/transition time.
- Do not decrement transit/freight budgets.

### SOP 2 — Incident / event override
- Require declared incident/event activation.
- Apply temporary budget multipliers only within duration bounds.
- Record approvals and reason codes.

### SOP 3 — Failure modes
- Comms loss: revert to controller-local default rules; preserve EVP.
- Engine down / time untrusted: disable non-critical priority.

## Governance & Change-Control Runbook

### Policy versioning
- `MAJOR`: changes to eligibility, fairness caps, or arbitration order.
- `MINOR`: budget adjustments within the same policy structure.
- `PATCH`: logging/reporting metadata updates.

### Review cadence
- weekly ops review: hotspots, denied requests, recovery problems.
- monthly policy review: equity slices, corridor performance, anti-abuse signals.
- seasonal recalibration: route changes, construction, schedule changes.

### Dispute resolution and transparency
- Publish aggregate reports (spend + outcomes) and equity slices.
- Keep security-sensitive details (e.g., EVP identifiers) internal.

## Reference Links
- [`chapter9`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm)
- [`NTCIP 1211`](https://www.ntcip.org/file/2018/11/NTCIP1211-v0224j.pdf)
- [`FHWA Order 6640.23`](https://highways.dot.gov/laws-regulations/directives/orders/664023)

## Completion Checklist
- ✅ Budget **unit** table (definition → pros/cons → required data) + phased approach: see **“1) Define budget units”**.
- ✅ Fairness constraints + windows + “credits statement” reporting: see **“2) Fairness constraints…”**.
- ✅ Anti-gaming/abuse resistance: see **“3) Anti-gaming…”**.
- ✅ Architecture placement + **request → policy engine → controller action → accounting → reporting diagram** + safe failure behavior: see **“4) Architecture…”**.
- ✅ Stakeholder governance + arbitration + change control: see **“5) Stakeholder governance…”**.
- ✅ Concrete artifacts (policy YAML, log schema, KPIs): see **“6) Concrete implementation artifacts”**.
- ✅ Final required sections present: **Implementation Checklist**, **Operations Runbook (SOP)**, **Governance & Change-Control Runbook**, **Reference Links**, **Completion Checklist**.

---

Cross-links: Related ideas include city rules, safety-first signals, and explainable signals.
