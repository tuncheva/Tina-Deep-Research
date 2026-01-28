# 07) Nature- and Noise-Friendly Signals: Calm Urban Flow

## Catchy Explanation
Think “quiet mode” for streets near parks, hospitals, and residential areas: reduce stop‑and‑go so the city sounds and smells less like traffic.

## What it is (precise)
**Nature- and noise-friendly signals** are timing strategies that explicitly optimize for *smoother trajectories* (fewer stops, less harsh acceleration) and lower emissions/noise in defined **sensitive zones**. Instead of optimizing only average delay, the controller (often twin-assisted) evaluates plans using proxies such as stops/vehicle, speed variance, acceleration event counts, and queue spillover risk, then selects conservative progression/meters that reduce disturbance while keeping safety invariants (pedestrian service, clearance intervals) and preventing displacement of congestion to parallel streets.

## Benefits
- **Lower noise**: fewer stop‑start cycles reduce peak noise events.
- **Lower emissions**: less idling and hard acceleration.
- **Better livability**: improved comfort near sensitive land uses.
- **More predictable driving**: smoother progression reduces frustration.

## Challenges
- **Spillover**: improving one zone can push congestion/noise elsewhere.
- **Speed risk**: progression can encourage speeding if not capped.
- **Measurement**: proxies (stops, acceleration) need validation.
- **Policy tradeoffs**: pedestrians/cross streets may see added delay.

## Implementation Strategies

### Infrastructure Needs
- **Zone definition (GIS)**: polygons for hospitals/parks/residential corridors.
- **Telemetry**: stops/vehicle, speed variance, queue length; ideally probe speeds.
- **Plan library**: “quiet mode” timing plans (night, daytime, event overrides).
- **Twin evaluation**: simulate network spillover effects.
- **Guardrails**: speed caps, max side-street delay, pedestrian service constraints.

### Detailed Implementation Plan

#### Phase 1: Define Sensitive Zones, Objectives, and Guardrails (Weeks 1–4)
The city should start by defining the sensitive areas where reduced disturbance is a legitimate objective, such as hospital frontages, residential arterials, parks, and school zones, and it should do so with community input so the zones reflect real concerns. The engineering team should translate “quieter” and “cleaner” into measurable operational proxies that can be tracked with available data, such as stops per vehicle, arrivals-on-green, speed variance, and red-occupancy spillback frequency, and it should also define pedestrian and side-street constraints so benefits are not achieved by increasing pedestrian delay or starving cross traffic. The team should decide what constitutes a “quiet mode” operating period (for example, nighttime hours or specific times near hospitals) and should define how the system will switch in and out of quiet mode without oscillation.

- **Roles**: traffic engineering (zone and timing policy), community liaison/public health contact (zone justification), operations (mode switching workflow), data analyst (metric definitions).
- **Deliverables**: GIS zone map, KPI bundle, guardrail thresholds (speed cap, ped constraints, side-street caps), and a mode schedule/activation policy.
- **Risks**: zones may be too broad and dilute effect; guardrails may be politically contested.
- **Acceptance checks**: zones and KPIs are approved, and measurement feasibility is confirmed for each KPI.

#### Phase 2: Baseline Measurement and Proxy Validation (Weeks 5–10)
The data team should collect baseline metrics for the selected corridor(s) across multiple weeks, including weekday and weekend patterns, because noise and stop-and-go behavior can vary significantly by schedule. Where possible, the team should validate proxy metrics against any available noise monitors or environmental sensors, and if direct measurements are not available, it should at least confirm that proxy changes correlate with observed field conditions and resident complaints. The team should also identify parallel route and neighborhood impacts to establish a baseline for displacement risk, because improving one corridor can shift congestion and disturbance to another.

- **Roles**: data analyst (baseline computation), traffic engineer (interpretation), operations (field context), environmental/public health partner (if measurement exists).
- **Deliverables**: baseline report, proxy validation notes, and a displacement monitoring plan.
- **Risks**: proxies may not correlate with perceived noise; probe speed coverage may be limited.
- **Acceptance checks**: baseline distributions are stable enough to compare before/after, and displacement monitoring locations are defined.

#### Phase 3: Candidate Timing Plans and Twin Evaluation (Weeks 11–18)
The traffic engineering team should design a small set of candidate timing plans that reduce stop-and-go and harsh acceleration, typically by using progression at or below the posted speed, smoothing offsets, and applying gentle metering where needed to prevent downstream spillback. The team should evaluate these plans in a twin to measure not only corridor KPIs but also network effects, including queue spillback to side streets and diversion pressure on parallel residential streets. The team should document explicit tradeoffs for each plan (for example, “quiet night plan reduces stops but increases side-street delay by X seconds”) so the city can make an informed policy choice.

- **Roles**: traffic engineer (plan design), modeler (twin scenarios), analyst (KPI scoring), safety/accessibility reviewer (pedestrian impacts).
- **Deliverables**: ranked plan set with tradeoff notes, simulation report, and recommended activation windows.
- **Risks**: progression can unintentionally encourage speeding; plans can create upstream queuing if downstream capacity is limited.
- **Acceptance checks**: candidate plans satisfy pedestrian minimums and side-street caps in simulation, and progression speed remains at/below the posted limit.

#### Phase 4: Pilot Rollout and Operational Monitoring (Weeks 19–28)
The operations team should deploy the selected “quiet mode” plan in assisted mode first, where operators confirm plan switching, because sensitive-zone changes can draw public attention and should be reversible. The city should coordinate communications if driver expectations will change (for example, night progression behavior) and should monitor compliance and safety proxies during the pilot. The team should review daily ATSPM measures and the noise/emissions proxies, and it should respond to spillover by adjusting metering, cycle length, or plan windows rather than abandoning the concept.

- **Roles**: operations (activation and monitoring), traffic engineering (on-call tuning), communications lead (public messaging), analyst (weekly reporting), maintenance (field checks).
- **Deliverables**: live pilot deployment, monitoring dashboards, and a pilot report with before/after comparisons.
- **Risks**: resident complaints if spillover increases elsewhere; safety risk if speeding increases.
- **Acceptance checks**: targeted proxies improve without violating constraints, and spillover on parallel routes remains within agreed limits.

#### Phase 5: Retiming, Reporting, and Expansion (Ongoing)
The agency should treat quiet-mode timing as an ongoing program with seasonal retiming and periodic reporting, because sensitive-zone conditions change with construction, school schedules, and traffic growth. The team should publish clear summaries for the community that explain what improved and what tradeoffs were accepted, and it should only expand to additional zones after confirming that displacement can be monitored and managed. Over time, the city can incorporate better measurement (noise sensors, emissions modeling) to improve the fidelity of the objective.

- **Roles**: program owner (governance), traffic engineering (retiming), operations (daily monitoring), analyst (reporting), community liaison (feedback loop).
- **Deliverables**: periodic community report, updated plan versions, and expansion criteria.
- **Risks**: program goals can drift; measurement limitations can undermine credibility.
- **Acceptance checks**: periodic reviews show sustained proxy improvement and no recurring safety regressions.

### Choices
- **Progression-focused**: speed‑limit green waves to minimize stops.
- **Selective longer cycles**: reduce starts at the cost of longer waits.
- **Quiet-night mode**: stricter disturbance reductions after hours.

## Technical Mechanics

### Key Parameters
- Stops/vehicle, speed variance
- Acceleration event thresholds
- Side-street delay caps

### Guardrails
- Progression speed at/below posted limit.
- Maintain pedestrian minimums and clearance.
- Monitor parallel routes to prevent displacement.

## Implementation Additions (Implementation-Ready)

### 1) Measurement plan: validate proxies against actual noise (proxy-to-outcome calibration)
A quiet-signal program needs *outcome* measurement (noise) and *operational* measurement (traffic proxies), with a documented calibration so proxy improvements are credible.

#### Noise outcome metrics (what to measure)
- **A-weighted sound level (dBA)**:
  - **`LAeq,T`** (equivalent continuous sound level over period `T`) for before/after comparisons.
  - **`Lmax`** (maximum A-weighted level) to capture peak disturbance events (engine revs, motorcycles, hard accelerations).
- **Community outcome metrics**:
  - **complaint rate** (311/911/non-emergency + city feedback channels), normalized by time-of-day.
  - **perception surveys** (optional): short pulse surveys near pilot zones for perceived disturbance and sleep disruption.

**Source alignment**: FHWA’s noise measurement field guide focuses on planning, preparation, and execution of noise measurement efforts, including on-site data collection steps and checklists as best practice guidance ([`Noise Measurement Field Guide (FHWA-HEP-18-066)`](https://www.fhwa.dot.gov/ENVIRonment/noise/measurement/fhwahep18066.pdf:1)).

#### Proxy metrics (what signals can influence)
Use proxies that are observable and sensitive to timing changes:
- **Stops/vehicle** (and distribution; not just mean).
- **Acceleration events** (count of `a > threshold`), **hard braking** events if you can estimate.
- **Speed variance** and **speed percentile spread** (e.g., `p85 - p15`).
- **Queue spillback frequency** (proxy for honking/driver stress, plus safety risk).
- **Vehicle mix modifiers** (critical for noise):
  - **heavy-vehicle share** (trucks/buses) and **motorcycle share** where available.
  - **grade/surface flags**: steep grades and rough pavement can dominate noise; record as covariates.

#### Calibration protocol (pilot + control, then validate)
1. **Select 2–4 pilot locations** in sensitive zones + **1 control site** with similar geometry and demand.
2. **Collect short-term noise samples** across time-of-day (e.g., AM peak, midday, PM peak, night) and day types (weekday/weekend).
3. **Collect synchronized traffic proxy data** in the same windows:
   - probe speeds, stop events, estimated acceleration events.
   - classification (truck/motorcycle share) where possible.
4. **Fit a proxy-to-noise model** (start simple):
   - `LAeq = β0 + β1*(stops/veh) + β2*(speed variance) + β3*(HV share) + β4*(volume) + ε`
   - validate with holdout windows; report R²/MAE and limitations.
5. **Define a confidence statement** for operational use:
   - “A 10% reduction in stops/veh is associated with ~X dBA change (±Y) under normal vehicle mix; unreliable when HV share > Z% or grade > G%.”

#### Instrumentation and data sources
- **Noise measurement**:
  - temporary, standards-compliant sound level meters (short campaigns) vs.
  - low-cost fixed sensors (useful for trends; validate against a reference meter).
- **Traffic proxies**:
  - probe speed data (segment-level), ATSPM events, controller logs.
  - video classification (helpful for HV/motorcycle share; document accuracy limits).

#### “Proxies are unreliable when…” checklist
Treat proxy improvements as *insufficient evidence* when any apply:
- high/variable **truck share** or significant **motorcycle** presence.
- **steep grade** where engine load/braking dominates.
- rough pavement, construction plates, potholes.
- major diversion/rerouting changes during the pilot.

### 2) Equity / Environmental Justice operationalization + “no harm” spillover governance
Quiet zones can’t be allowed to shift harms (noise, congestion, cut-through) to adjacent neighborhoods. Operationalize “no harm” constraints and publish transparent reporting.

**Source alignment**: FHWA’s environmental justice order defines *adverse effects* to include **noise pollution** and **increased traffic congestion**, and defines “disproportionately high and adverse effects” as those predominantly borne by, or more severe on, minority and/or low-income populations ([`FHWA Order 6640.23`](https://highways.dot.gov/laws-regulations/directives/orders/664023:1)).

#### Spillover governance design
- Define **Sensitive Zone** (where you optimize for calm flow) and **Spillover Watch Areas** (adjacent neighborhoods and parallel routes).
- For each watch area, define:
  - affected populations and sensitive receptors (schools, elder care).
  - baseline distributions for volume/speed/stops and complaint counts.

#### “No harm” constraints (examples)
Set explicit, measurable caps for watch areas during quiet mode:
- `Δvolume_p50 ≤ +5%` and `Δvolume_p95 ≤ +10%` (relative to baseline).
- `Δp85_speed ≤ +2 mph` (or local policy equivalent).
- `Δstops/veh ≤ +10%`.
- `Δcomplaints ≤ +X per week` OR `complaints z-score ≤ 2`.

#### Triggers and response actions
- If any constraint breaches for `T_persist` (e.g., 2 weeks):
  - adjust offsets/cycle/splits to reduce diversion pressure,
  - add **gating** to prevent spillback into neighborhoods,
  - coordinate with curb/routing policy (loading windows, turn restrictions),
  - rollback the quiet plan schedule for the problematic time window.

#### Reporting templates (publishable)
- **Before/After** by neighborhood, time-of-day, and vehicle class:
  - `LAeq`, `Lmax` at pilot points.
  - stops/veh, speed variance, p85 speed.
  - truck share and estimated motorcycle share.
  - complaints normalized by population/blocks.

### 3) Multimodal interactions and constraints (operational guardrails)
Noise-friendly progression changes how all users experience the street. Make constraints explicit.

#### Pedestrian comfort and accessibility
- **Max wait caps** near parks/schools/hospitals (policy-based; e.g., p95 wait ≤ X sec).
- **Minimum walk intervals** and conservative clearance; never degrade below legal minima.
- **School zones**: time-of-day constraints and crossing-guard coordination.

#### Bike comfort / greenway protection
- Reduce stop frequency on designated bike corridors by:
  - progression speed selection that matches comfortable cycling speeds where appropriate.
  - limiting “chop” (frequent short greens) that create repeated stops.

#### Transit / freight interactions
- **TSP interaction**: smoothing can conflict with frequent TSP interrupts.
  - define quiet-mode TSP policy: e.g., only “early green/green extend” within bounds, no cycle truncation.
- **Heavy vehicles dominate noise**:
  - signal timing can reduce stop-start *impulse noise* from trucks/buses but cannot eliminate engine noise; pair with routing/pavement policies.

### 4) Freight / heavy-vehicle routing and complementary policies (integration)
Signal timing is only one lever; for truck noise it is often secondary.

#### When signal timing helps
- corridors where truck stop-start is common (queues at reds, short cycles).
- areas where smoother flow reduces gear changes and hard acceleration events.

#### When signal timing is insufficient
- steep grades (engine load/braking), rough pavement, high motorcycle activity.

#### Complementary measures (program planning level)
- **Truck routes/time windows** near sensitive receptors.
- **Curb management/loading rules** to reduce double-parking and stop-start turbulence.
- **Enforcement** (engine braking restrictions where applicable, speeding enforcement).
- **Pavement treatments/maintenance** (conceptual; coordinate with asset management).

#### Coordination model
- Establish a monthly working group: traffic engineering + operations + freight policy + enforcement + public health/community liaison.
- Treat quiet-mode corridors as “special operating areas” with jointly-owned outcomes.

### 5) Safety plan: speed compliance and conflict monitoring
Progression can incentivize speeding if the progression speed is too high or if drivers “chase greens.” Safety must be monitored and tied to rollback.

#### Speed compliance controls
- **Progression speed selection**: choose a target at or below posted speed; optimize for *compliance* not just travel time.
- **Monitor speed distributions**:
  - track p50/p85 speeds and the share exceeding posted speed by a policy threshold.
- **Compliance triggers**:
  - if p85 increases above threshold for `T_persist`, reduce progression speed (offset adjustments) and/or shorten coordination windows.

#### Safety proxies to track
- red-light running proxies (if detector/video available), split failures.
- turning conflicts / near-miss indicators (video analytics if available; document limitations).
- crash trend monitoring (lagging indicator; use for quarterly governance, not real-time ops).

#### Go/no-go and rollback triggers
- Any sustained speed compliance regression.
- Ped delay cap violations near schools/hospitals.
- Increased queue spillback into crosswalks/intersections.

### 6) Implementation details: zone design, modes, and operations

#### “Quiet/Livability Mode” definition (artifact)
```yaml
mode_id: "QUIET-01"
name: "Quiet/Livability Mode — Park Edge Corridor"
geofence:
  polygon: "gis://zones/park_edge"
  corridors: ["Main St", "Park Ave"]
  intersections: [101, 102, 103]
schedule:
  windows:
    - { days: [Mon, Tue, Wed, Thu], start: "21:00", end: "06:00" }
    - { days: [Sat, Sun], start: "08:00", end: "22:00" }
objectives:
  - "reduce stops/veh"
  - "reduce speed variance"
  - "reduce LAeq at sensor points"
constraints:
  - ped_minimums_enforced: true
  - ped_p95_wait_max: "60s"
  - side_street_p95_delay_max: "75s"
  - p85_speed_max_delta: "+2 mph"
  - spillover_no_harm_caps: "see watch_area_policy"
allowed_actions:
  - "offset tuning within bounds"
  - "cycle length within [80, 110]"
  - "split rebalance within bounds"
  - "gating to prevent downstream spillback"
activation:
  type: "scheduled"
  operator_confirmation_required: true
rollback:
  plan_id: "NORMAL-TOD"
  triggers:
    - "safety regression"
    - "spillover constraint breach"
```

#### Intervention patterns (catalog)
| Pattern | What you change | When it helps | Watch-outs |
|---|---|---|---|
| Progression tuning | offsets + progression speed | reduce stops and harsh accel | can induce speeding if too fast |
| Cycle adjustment | slightly longer cycles | reduce start frequency | increases ped/side-street waits |
| Split rebalance | smooth mainline flow while protecting crossings | stabilize queues | can starve minor street |
| Gentle metering/gating | limit inflow to prevent spillback | protect neighborhoods downstream | can push queue upstream |

#### Operator workflow
1. Confirm scheduled activation (or supervisor-approved manual activation).
2. Check device health: detectors/probes coverage, comms, ped calls.
3. Activate quiet plan; start monitoring dashboard.
4. Monitor: stops/veh, p85 speed, spillover watch areas, ped wait caps.
5. If constraints breach: apply bounded adjustments; if persistent, rollback and log.
6. Weekly review and monthly governance meeting.

#### Acceptance criteria (pilot → scale)
To expand beyond pilot, require:
- **Noise outcome improvement** (LAeq/Lmax at pilot points) *or* validated proxy improvement with documented confidence bounds.
- **No spillover harm** within watch-area caps.
- **No safety regression** (speed compliance + safety proxies) and ped accessibility constraints met.
- Documented community-facing report and governance sign-off.

## MVP Deployment
- One sensitive zone corridor.
- One “quiet plan” + one normal plan.
- Weekly review of spillover metrics.

## Evaluation
- Stops/vehicle and speed variance changes.
- Proxy noise reduction (and measured noise if available).
- Emissions proxy changes.
- Side-street and pedestrian delay distributions.

## References / Standards / Useful Sources
- FHWA Traffic Signal Timing Manual: [`https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf`](https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf)
- FHWA Noise Measurement Field Guide (FHWA-HEP-18-066): [`https://www.fhwa.dot.gov/ENVIRonment/noise/measurement/fhwahep18066.pdf`](https://www.fhwa.dot.gov/ENVIRonment/noise/measurement/fhwahep18066.pdf:1)
- FHWA Order 6640.23 (Environmental Justice; includes noise and congestion as adverse effects): [`https://highways.dot.gov/laws-regulations/directives/orders/664023`](https://highways.dot.gov/laws-regulations/directives/orders/664023:1)

---

## Implementation Checklist
- [ ] Define sensitive zones and spillover watch areas (GIS polygons + affected receptors).
- [ ] Define objectives + constraints (ped wait caps, side-street caps, speed compliance, no-harm caps).
- [ ] Build baseline dataset: proxies + noise samples at pilot/control sites.
- [ ] Calibrate proxy-to-noise relationship and document confidence/limitations.
- [ ] Design 2–3 candidate quiet-mode plan sets (progression/cycle/splits + gating).
- [ ] Twin-evaluate corridor benefits and spillover impacts; choose pilot plan.
- [ ] Deploy in assisted mode with operator confirmation + rollback contract.
- [ ] Run weekly monitoring; publish monthly equity/no-harm report.
- [ ] Gate expansion on outcomes: noise/proxy improvements + no spillover harm + no safety regression.

## Monitoring & Governance Runbook

### Monitoring cadences
- **Daily (ops)**: p85 speeds, stops/veh, split failures, queue spillback, ped service indicators.
- **Weekly (engineering/analyst)**: before/after distributions, spillover watch areas, complaint trends.
- **Monthly (program governance)**: equity/no-harm review, partner coordination (freight/enforcement), plan adjustments.

### Required dashboards
- Zone status: current mode, schedule window, plan ID/version.
- Safety: speed distribution, red-running proxy, queue spillback alerts.
- Livability: stops/veh, accel events, speed variance.
- Equity/no-harm: watch-area caps status and complaint signals.

### Change control
- Version quiet-mode plans and document:
  - objective/constraint changes (major),
  - parameter tuning (minor),
  - reporting/contact edits (patch).
- Any change requires a before/after evaluation note and a rollback plan.

## Reference Links
- [`FHWA Noise Measurement Field Guide (FHWA-HEP-18-066)`](https://www.fhwa.dot.gov/ENVIRonment/noise/measurement/fhwahep18066.pdf:1)
- [`FHWA Order 6640.23 (Environmental Justice)`](https://highways.dot.gov/laws-regulations/directives/orders/664023:1)
- [`FHWA Traffic Signal Timing Manual`](https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf)

## Completion Checklist
- ✅ Measurement plan + proxy calibration: see **“1) Measurement plan”**.
- ✅ Equity/EJ + no-harm spillover governance: see **“2) Equity / Environmental Justice…”**.
- ✅ Multimodal interactions + constraints: see **“3) Multimodal interactions…”**.
- ✅ Freight/heavy-vehicle integration: see **“4) Freight / heavy-vehicle…”**.
- ✅ Safety plan (speed + conflicts + rollback): see **“5) Safety plan…”**.
- ✅ Implementation details (mode definition, patterns, workflow, acceptance): see **“6) Implementation details…”**.
- ✅ Final required sections added: **Implementation Checklist**, **Monitoring & Governance Runbook**, **Reference Links**, **Completion Checklist**.

---

Cross-links: Related ideas include green waves, city rules, and safety-first signals.
