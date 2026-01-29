# 01) Green-Wave Plans: Synchronized Traffic Harmony

## Catchy Explanation
Imagine traffic lights syncing like an orchestra, letting people, buses, bikes, freight, and emergency vehicles flow through greens like a smooth ocean wave – without starving cross streets or vulnerable users. That’s a **green wave** when it is treated as a **multimodal service** and not just a car-throughput trick.

## What it is (precise)
A **green wave** (signal progression or coordination) synchronizes traffic lights along a corridor to create "green corridors" where vehicles can travel at a consistent speed without stopping. By aligning **cycle lengths**, **offsets**, and **splits**, it ensures platoons of vehicles encounter green lights in sequence. This minimizes stops, reduces travel times, and stabilizes operations.

An implementation-ready program must:
- Optimize **cycle / split / offset** by time-of-day.
- Embed **hard constraints** for pedestrians, bikes, ADA, side streets, and storage.
- Define **how TSP/EVP can break and then recover coordination**.
- Use a **digital twin + shadow mode** with explicit validation gates before field changes.
- Run under **clear governance** (roles, approvals, versioning, rollback) and a recurring evaluation loop for **equity, safety, and multimodal performance**.

Coordination is an established timing practice and is described in FHWA’s Signal Timing Manual, which covers offsets, cycle length selection, and maintaining effective plans over time ([FHWA Traffic Signal Timing Manual (PDF)](https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf)).

## Benefits
Green waves significantly improve traffic efficiency and environmental sustainability, but studies warn of rebound effects where increased attractiveness may boost car use, negating gains unless paired with modal shifts:
- **Reduced travel times**: 10–30% faster journeys by minimizing stops in coordinated systems.
- **Lower emissions**: 5–40% reductions in CO₂, NOx, and PM from less idling and hard acceleration; big-data systems have shown large corridor-level CO₂ savings.
- **Energy savings**: Fuel reductions of 10–20%; GLOSA-style advisory can save ~15% fuel.
- **Safety improvements**: Fewer stops reduce rear-end collisions and smooth flow for emergency vehicles; better timing supports pedestrians and bikes.
- **Economic gains**: Less congestion saves time, fuel, and fleet costs; payback often 2–5 years.
- **Air quality and noise**: Less stopping/starting reduces local noise and hotspot emissions.
- **Multimodal support**: Buses can keep more even headways; pedestrians see more predictable service.

## Challenges
Implementing green waves faces several hurdles:
- **Infrastructure constraints**: Irregular intersection spacing in cities makes perfect synchronization difficult.
- **Traffic variability**: Pedestrians, cyclists, buses, turns, and varying speeds disrupt platoons.
- **Cross-street delays**: Prioritizing main corridors can starve side streets and pedestrians.
- **Data and technology needs**: Requires reliable detectors, communications, and performance systems.
- **Human factors**: Drivers not adhering to target speeds or gaming the progression.
- **Scalability**: Hard to coordinate in dense, dynamic urban areas and across multiple corridors.

---

## Implementation Strategies

### Infrastructure Needs
- **Traffic detection**: Inductive loops, radar, video analytics, and/or probe data for approach counts, occupancies, speeds, and travel times.
- **Controller observability**: Phase/interval state, detector status, pedestrian calls; high-resolution logs if available.
- **Signal connectivity**: Reliable comms (fiber/cellular) to a central system for plan downloads and monitoring.
- **Timing management**: Plan library support (time-of-day), plus optional traffic-responsive or adaptive logic.
- **Digital twin**: Micro/meso model calibrated to travel times, queues, spillback, and platoon behavior.
- **Performance monitoring**: ATSPM-style dashboards to track arrivals on green, split failures, and corridor travel time reliability ([FHWA ATSPM (landing)](https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm)).

---

## 1) Multimodal progression: objectives, constraints, and design patterns

Coordination decisions should be framed as a **corridor multimodal service design** problem, not only a vehicle progression problem. FHWA’s Signal Timing Manual discusses coordination as part of a broader timing program and emphasizes maintaining plans and considering operational context over time ([FHWA Traffic Signal Timing Manual — Chapter 6 (Coordination)](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter6.htm)).

### 1.1 Objective options (choose 1–3 primary; monitor the rest)

**Default agent choice for this corridor guide**  
- **Primary optimization objectives** (what the system actually tries to optimize):
  1. **Person-based outcomes (person-delay)**
  2. **Transit outcomes**
  3. **Pedestrian outcomes**
- **Monitored (non-primary) objectives** (still measured, with constraints, but not optimized directly):
  - Vehicle progression
  - Bike comfort
  - Freight
  - Emergency response

You can override this default if your policy or context demands a different set, but if you “let the agent choose,” it should assume the above.

- **Vehicle progression** (monitored): maximize arrivals on green (AoG), minimize stops, reduce travel time **subject to** primary objectives and constraints.
- **Person-based outcomes (PRIMARY)**: minimize **person-delay** rather than vehicle-delay by weighting vehicles by occupancy and prioritizing high-capacity modes.
- **Reliability** (aligned with primary objectives): reduce p95 travel time and variability (corridor reliability and bus on-time performance).
- **Transit outcomes (PRIMARY)**: reduce bus running time and improve schedule adherence via TSP constraints and recovery strategies ([FHWA Signal Timing Manual — Chapter 9 (Priority & Preemption)](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm)).
- **Pedestrian outcomes (PRIMARY)**: reduce maximum wait, meet accessibility needs, and maintain consistent service at crossings.
- **Bike comfort** (monitored): align progression speeds with safe/comfortable cycling speeds and ensure cyclist detection/recall behavior works.
- **Freight** (monitored): reduce travel time and delay for heavy vehicles (often via peak freight time-of-day plans rather than always-on priority).
- **Emergency response** (monitored with hard requirements): maintain EVP readiness and safe transition/recovery behavior ([FHWA Signal Timing Manual — Chapter 9](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm)).

### 1.2 Constraints (examples you can operationalize)
- **Pedestrian maximum wait cap**: define a max wait (e.g., 90–120 s at key crossings) and compute it as the longest time between “ped call registered” and “WALK served,” using controller event logs when available. “Effective signal timing plans” explicitly include pedestrian timing and operations as part of the timing process ([FHWA Traffic Signal Timing Manual — Chapter 5 (Basic timing)](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm)).
- **Ped minimums and clearance**: treat minimum WALK + clearance (FDW) as hard constraints and validate them during plan design ([FHWA Traffic Signal Timing Manual (PDF)](https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf)).
- **ADA / accessibility**: require that accessible pedestrian signals (APS) and pushbutton behavior remain correct after any phasing changes (acceptance test in the field).
- **Side-street delay cap**: set a cap for average or p95 side-street delay (or max queue) by time-of-day to prevent “mainline wins, side streets lose.”
- **Queue storage protection**: treat available storage as a hard constraint; add “spillback risk” thresholds to block offsets/splits that cause blocking.
- **Speed management**: require progression speeds at or below target operating speeds; disallow any plan that requires > posted + small margin for reliable progression.
- **Transit and freight windows**: specify hours where transit or freight objectives override minor travel-time gains for general traffic.

### 1.3 Cross-street protection design patterns
- **Split guards**: set minimum green time for cross-street phases and constrain split reductions per cycle so that progression tuning cannot starve minor movements.
- **Queue-aware gating**: when minor-street queue exceeds a threshold (detector occupancy or video queue estimate), temporarily relax progression objective and allocate recovery green.
- **No-block-the-box / spillback handling**: use queue detectors or downstream occupancy proxies to prevent releasing platoons into blocked downstream links.
- **Protected turn bays**: ensure bay lengths and splits prevent queues from backing into through lanes; treat bay spillback as a hard failure.
- **Progression speed selection**: set the progression speed to the **posted** or a **target operating speed** that does not incentivize speeding; treat “speeding signal” complaints as a formal safety risk (see Safety section).

### 1.4 Corridor typologies (what changes)
- **Downtown grid**: prioritize pedestrian delay caps, shorter cycles, and frequent service; allow weaker vehicle progression but ensure consistency and safe turning.
- **Arterial with long spacing**: classic green-wave candidate; focus on reliable platoon travel time and spillback controls at bottlenecks.
- **Suburban arterial with heavy turns/driveways**: progression is fragile; use more time-of-day plans and protect turn bays/queue storage.

### 1.5 Objective/constraint → metric → data → enforcement layer (required table)

| Objective / constraint            | Primary metric(s)                                              | Data source(s)                                       | Enforcement / control layer                                              |
|----------------------------------|----------------------------------------------------------------|------------------------------------------------------|---------------------------------------------------------------------------|
| Vehicle progression              | AoG (%), stops/veh, corridor median/p95 travel time           | ATSPM logs, probe travel times                       | Time-of-day plans (cycle/split/offset), adaptive parameters              |
| Person-delay minimization       | Person-delay by movement/mode, person-throughput              | Counts × occupancy factors, APC, manual surveys      | Optimization objective in plan design; adaptive weight settings          |
| Pedestrian maximum wait cap     | Max wait, wait distribution, % calls within target            | Ped pushbutton logs, high-res events, video         | Hard constraint in timing design; automated alarms if violated           |
| Bike comfort / progression      | Bike travel time, progression quality, speed compliance       | Bike counters, app/GPS traces, probe speeds         | Progression speed selection; bike-priority offsets at key crossings      |
| Transit reliability             | Bus running time, on-time performance, TSP grant rate         | AVL/APC, TSP logs                                    | TSP policy (conditionality, windows), headway/schedule-based priority    |
| Freight efficiency              | Heavy-vehicle delay and travel time                           | Classified counts, freight GPS, weigh-in-motion     | Freight-oriented plans in peak windows; protected queues at ramps        |
| Emergency response              | Response time, preemption success, recovery time to coord.    | EVP logs, CAD timestamps, ATSPM                     | EVP parameter settings; post-preemption recovery and lockouts            |
| Side-street delay cap           | p95 delay / queue length by approach                          | Detector occupancy, video queues, probe data        | Split floors for minor movements; max-green and recall settings          |
| Queue storage protection        | Max queue vs storage, spillback event count                   | Downstream occupancy, video, probe speeds           | No-release logic when downstream blocked; offset and split guardrails    |
| Emissions / noise exposure      | Emissions estimates, idle time, acceleration; noise level     | Probe speed profiles, emission models, noise meters | Scenario screening in twin; constraints on cycle length, progression     |
| Safety / speeding / RLR risk    | 85th % speed, high-speed arrivals-on-red, near-miss counts    | Speed probes, radar, video surrogate safety, ATSPM  | Progression speed selection, enforcement coordination, phasing/clearance |

---

## 2) Equity and fairness framework for corridor coordination

A corridor green wave is a policy choice about **who gets delay, noise, emissions, and safety risk**. Make the fairness definition explicit and publish before/after reporting.

### 2.1 Fairness definitions that fit signal timing
- **Distributional fairness (delay burden)**: constrain how much delay can increase on any approach/neighborhood when mainline travel time improves.
- **Person-throughput fairness**: evaluate outcomes by person-delay rather than vehicle-delay.
- **Exposure fairness**: track where idling/acceleration (noise and emissions) are reduced or increased; ensure burdens are not shifted onto vulnerable neighborhoods.
- **Safety-risk fairness**: ensure near-miss proxies and speeding do not worsen near schools, transit stops, or high-pedestrian areas.

### 2.2 Metrics
For each corridor, compute by **time-of-day** and **direction**:
- **Person-delay** (by mode, movement, and geography): person-minutes of delay; percent change vs baseline.
- **Neighborhood delay burden**: sum of person-delay on approaches belonging to a neighborhood or community district.
- **Exposure metrics**:
  - Emissions: estimated grams of CO₂/NOx/PM by segment and neighborhood.
  - Noise: change in equivalent continuous sound level (Leq) if measured; or proxy from stop/accel counts.
- **Equity gap indicators**:
  - Compare changes in delay, emissions, and safety proxies between priority areas (e.g., EJ communities, school areas) and non-priority areas.

### 2.3 Practical measurement and reporting (corridor scorecard template)
Create a **corridor scorecard** for each time-of-day plan, published as a 1–2 page artifact.

**By mode**
- Auto: median/p95 travel time; stops/veh.
- Transit: bus running time, on-time performance, TSP requests, TSP grants, missed priority.
- Pedestrian: delay distributions; max wait; % crossings with APS functioning.
- Bike: travel time (if available), comfort index (e.g., share of time in comfortable speed band).
- Freight: heavy-vehicle travel time and queue time.

**By movement/geography**
- Mainline vs side-street delay; turning vs through movements.
- Segments near schools, transit stops, or EJ priority areas.

**By safety proxies**
- Speed distributions (85th percentile and extreme-speed share).
- Red-light running proxies (high-speed arrivals-on-red, violation counts where available).
- Surrogate safety metrics (TTC, PET) where video or detailed simulation exists.

Use ATSPM measures to standardize operational reporting; FHWA’s ATSPM guide includes common measures and use cases such as split failures and occupancy-based measures ([FHWA ATSPM — use cases (Ch.4)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

### 2.4 Equity policy statements you can encode
- “No approach shall experience more than **X seconds increase in p95 delay** during peak periods relative to baseline, unless that approach is explicitly designated as lower-priority and this is documented.”
- “Pedestrian max wait at designated crossings shall not exceed **Y seconds** in any plan; violations trigger automatic review and potential rollback.”
- “Transit on-time performance shall **not degrade**; if it does, the plan must be adjusted or TSP rules changed.”
- “Corridor retiming shall **not increase emissions or noise burdens** in identified EJ or sensitive areas beyond agreed thresholds without mitigation.”

### 2.5 Equity reporting template (one-page outline)
For each corridor and plan set:
1. **Context**: map showing corridor, neighborhoods, sensitive areas (schools, hospitals, EJ zones).
2. **Baseline vs after**: small table of key metrics (person-delay, emissions proxy, safety proxies) for each neighborhood.
3. **Winners and losers**: short text summary of who gained/ lost delay and exposure.
4. **Mitigations**: list any changes made in response (more crossings, lower progression speed, added APS, transit adjustments).
5. **Public release**: timestamp, contact person, plan version ID.

---

## 3) Calibration and validation protocol (digital twin + shadow mode)

A corridor twin must be calibrated and validated before it is used to justify field timing changes. The goal is not “perfect prediction,” but **bounded error**, explicit uncertainty, and **formal acceptance gates**.

### 3.1 Step-by-step calibration plan
1. **Inventory and map**: geometry, lane counts, storage lengths, turn bay lengths, bus stop locations, school crossings, pedestrian volumes, and all phase/interval definitions.
2. **Demand and turning movement inputs**: build time-of-day demand profiles and turning proportions; document missing turning counts and how they are inferred.
3. **Signal control replication**: replicate coordination mode, offsets, actuated behavior, recalls, max greens, and pedestrian service rules.
4. **Behavioral parameters**: calibrate saturation flow, start-up lost time, lane-change/turn friction, and speed distribution.
5. **Oversaturation and spillback**: explicitly calibrate queue spillback and blocking behavior at known bottlenecks.

Calibration guidance and the importance of documentation are emphasized in simulation calibration references such as the JRC calibration guide ([JRC — The Calibration of Traffic Simulation Models (PDF)](https://publications.jrc.ec.europa.eu/repository/bitstream/JRC68403/lbna25188enn.pdf)).

### 3.2 Validation metrics (shadow-mode gates)
Validate at multiple levels:
- **Travel time**: median and p95 corridor travel time (probe data or Bluetooth/Wi‑Fi travel time).
- **Queue length / spillback**: max queue at critical approaches and spillback event frequency.
- **Arrivals on green**: match AoG patterns directionally by segment (field ATSPM vs simulated AoG).
- **Split failures**: match occurrence rates for critical phases; ATSPM defines split failures and related occupancy ratios used to identify unmet demand ([FHWA ATSPM (PDF)](https://ops.fhwa.dot.gov/publications/fhwahop20002/fhwahop20002.pdf)).
- **Detector health**: ensure field detector QA is stable before trusting validation results (otherwise you are validating against measurement error).

### 3.3 Suggested acceptance thresholds (go/no-go gates)
Use explicit gates, then tighten as the program matures:
- **Travel time error**: median and p95 within **10–15%** on key segments before using the twin to approve timing changes.
- **Queue validation**: reproduce the *presence/absence* of spillback at known hot spots and match peak queue order-of-magnitude.
- **Event reproduction**: if split failures occur in field peaks, the model should reproduce them under baseline timing; otherwise, it may be underestimating demand.
- **Pedestrian service**: simulated ped delay and service rates must not under-estimate max waits compared to observed.

These gates complement (not replace) standard signal timing plan maintenance practices described by FHWA, including ongoing monitoring and retiming triggers ([FHWA Traffic Signal Timing Manual — Chapter 8 (Maintain effective timing)](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter8.htm)).

### 3.4 Shadow-mode protocol
1. **Shadow build**: run the new plan set in the twin using observed demand, logging expected KPIs by day.
2. **Field mirror**: compute the same KPIs from field data under existing timing; ensure feed quality.
3. **Comparison window**: use at least 2–4 weeks (all relevant seasons/events if possible).
4. **Gate checks**:
   - If expected improvements exceed **X%** but shadow comparison shows poor alignment or unstable metrics, treat as **data/model issue**, fix, and rerun.
   - If constraints are violated in shadow (e.g., predicted side-street or ped delay exceeds caps), revise plans before any field deployment.
5. **Sign-off**: only when shadow-mode KPIs and constraints are within agreed tolerances does the plan become eligible for live pilot.

---

## 4) Interaction with Transit Signal Priority (TSP) and Emergency Vehicle Preemption (EVP)

TSP and EVP are not edge-cases; they are constraints that can disrupt offsets and degrade progression. FHWA’s Signal Timing Manual describes both priority and preemption and notes that priority algorithms modify green allocation within constraints such as coordination settings and maximum green ([FHWA Traffic Signal Timing Manual — Chapter 9](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm)).

### 4.1 TSP/EVP interaction policy
Adopt a **written corridor policy** that covers at least:
- **Hierarchy of objectives**: safety → EVP → transit reliability → general progression.
- **Where TSP applies**: list specific approaches/segments and time windows.
- **Allowed disruption**: max shift in coordinated phase end time per cycle (e.g., ≤ N seconds).
- **Recovery expectations**: max recovery time to return to planned coordination (e.g., ≤ M cycles) after TSP or EVP.
- **Equity and accessibility**: TSP cannot violate pedestrian/ADA constraints; EVP follows pre-defined safe sequences.

### 4.2 How TSP disrupts offsets and what to do
- **Disruption mechanism**: green extensions, early green, or phase insertions can shift the coordinated phase end time, which changes effective offsets.
- **Mitigation policies**:
  - **Conditional priority** (e.g., only late buses or headway-based rules): reduces disruption frequency and improves regularity (common practice described in TSP literature and agency handbooks; see the “conditional vs unconditional” framing in TSP materials such as [A Sophisticated Transit Signal Priority System – How it Works (PDF)](http://conf.tac-atc.ca/english/resourcecentre/readingroom/conference/conf2006/docs/s009/stewart1.pdf)).
  - **Priority windows**: only grant TSP during parts of the cycle where offset disruption is bounded.
  - **Max disruption limits**: cap green extension seconds per cycle and cap consecutive priority actions.
  - **Transit equity checks**: ensure TSP usage does not systematically shift delay to specific vulnerable crossings.

### 4.3 EVP (preemption) disruption and recovery
- **Disruption mechanism**: preemption interrupts normal operations to serve emergency vehicles; offsets are effectively broken for that cycle.
- **Recovery strategies**:
  - Use controller recovery modes to return to coordination safely and predictably (field-tested via drill scenarios).
  - Log each preemption event and measure **recovery time to coordination**.
  - Ensure preemption sequences maintain ped safety, including adequate clearance if pedestrians are in the crosswalk.

### 4.4 Implementation patterns to keep progression intact
- **Compensation logic**: after priority/preemption, gradually restore offsets (avoid sudden jumps that create unexpected stops).
- **Coordination boundaries**: define which intersections participate in coordination during heavy TSP/EVP activity (e.g., truncating the coordinated section when a hospital zone has frequent preemptions).
- **Transit- and EVP-aware twin**: simulate plausible TSP/EVP event rates in the twin to verify that progression remains acceptable under real-world priority loads.

### 4.5 Monitoring requirements
Use consistent monitoring dashboards:
- **Priority call frequency** by route and time-of-day.
- **TSP benefit**: change in bus running time and on-time performance.
- **Progression degradation**: changes in AoG, stops/vehicle, and travel time reliability during heavy TSP usage.
- **Recovery time**: time to return to normal AoG distribution post-TSP/EVP event.

---

## 5) Operational governance: roles, workflows, versioning, and rollback

Coordination succeeds when treated as a **program** with staffing, change control, and monitoring—not a one-off project. FHWA provides staffing guidance for traffic signal operations and maintenance and discusses maintaining timing plans as an ongoing activity ([FHWA Staffing Guidelines (PDF)](https://ops.fhwa.dot.gov/publications/fhwahop09006/fhwahop09006.pdf), [FHWA Signal Timing Manual — Chapter 8](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter8.htm)).

### 5.1 Recommended RACI-style roles (example)
- **Accountable**:
  - Traffic engineering program owner (approves plan releases and constraints).
- **Responsible**:
  - Timing engineer/modeler (plan design, twin analysis).
  - Operations/TMC lead (activation, monitoring, incident response).
  - Signal maintenance lead (field readiness, detector health, controller updates).
  - IT/network lead (comms, time sync, cybersecurity, access control).
  - Transit agency liaison (TSP policies and operations, bus operations feedback).
- **Consulted**:
  - Safety office / Vision Zero team.
  - ADA/accessibility coordinator.
  - Emergency services (EVP requirements and drills).
  - Environmental/equity office (equity review of scorecards).
- **Informed**:
  - Public information officer.
  - Corridor stakeholders and community boards.

### 5.2 Change management workflow
1. **Plan proposal**
   - Timing engineer drafts new or revised time-of-day plans, with a change description and rationale.
   - Twin scenarios and shadow-mode comparisons are attached.

2. **Constraint and equity review**
   - Safety and accessibility reviewers confirm ped/ADA, speed, and conflict constraints.
   - Equity/evironment reviewers check burden shifts using the corridor scorecard.

3. **TSP/EVP review**
   - Transit liaison and emergency services confirm priority and preemption policies, including recovery behavior.

4. **Approval gates** (documented checklist)
   - **Twin validation gate**: twin meets validation tolerances.
   - **Shadow-mode gate**: KPIs and constraints behave as expected over a multi-week period.
   - **Equity/safety gate**: equity metrics and safety proxies meet committed thresholds.
   - **Ops readiness gate**: monitoring dashboards, alarms, and operator runbooks are ready.

5. **Versioning and deployment**
   - Each plan gets a **version ID** (corridor + date + revision number).
   - Metadata recorded: corridor, intersections, cycle/split/offset set, constraints, approvers, and date.
   - Deployment window is scheduled (low-risk period when possible) with a named operator.

6. **Rollback playbook**
   - For each plan set, define:
     - **Rollback trigger conditions** (e.g., sustained spillback, ped max-wait violations, crash cluster, major TSP disruption, significant public complaints).
     - **Rollback path** (previous version, or revert to free or basic TOD plan).
     - **Rollback operator authority** (who can push the button without waiting for a meeting).

7. **Field feedback loop**
   - Complaints and field observations are logged as tickets tied to plan version IDs.
   - Monthly review closes or escalates tickets; escalations can lead to retiming or policy changes.

### 5.3 Staffing cadence
- **Daily/weekly**: detector health and ATSPM alarms review; investigate split failures, spillback, and serious anomalies.
- **Monthly**: KPI scorecards, equity check, plan drift review, complaints triage.
- **Seasonal / construction**: retiming and revalidation; temporary plans for work zones, special events, and school calendars.

---

## 6) Safety evaluation and mitigations (beyond ped timing)

Coordination can improve safety by reducing stop-and-go, but it can also **incentivize speeding**, affect red-light running, and change conflict patterns. Safety must be part of the go/no-go decision, not an afterthought.

### 6.1 What to measure
- **Speed distributions** (not just average):
  - 50th, 85th percentile speeds by segment and time-of-day.
  - Share of vehicles above **posted + Δ** (e.g., +10 km/h) and extreme speeding share.
- **Red-light running proxies**:
  - High-speed arrivals on red or during clearance.
  - Violation counts from enforcement or video where available.
- **Turning conflicts / near-misses**:
  - Surrogate safety measures such as TTC and PET from trajectories or simulation; FHWA documents surrogate safety measures derived from microscopic models ([FHWA — Surrogate Safety Measures From Traffic Simulation Models (PDF)](https://ntlrepository.blob.core.windows.net/lib/38000/38000/38015/FHWA-RD-03-050.pdf)).
- **Rear-end risk proxies**:
  - Hard braking events (if probe/telematics data is available).
  - High red occupancy with high approach volumes.
- **Vulnerable road users (VRUs)**:
  - Conflicts between turning traffic and pedestrians/cyclists at key crossings.
  - Yielding behavior at permissive turns.

### 6.2 Safety gates and countermeasures
Treat the following as **explicit gates** for plan activation and expansion:
- If 85th percentile speeds **rise above target** or extreme-speed share increases, then:
  - Reduce progression speed (target speed) and adjust offsets.
  - Consider shorter cycles and stronger progression “bands” that reward steady speed at or below the limit.
  - Coordinate with speed management/enforcement teams; consider enforcement or design changes (speed feedback signs, road diet, curb extensions) where feasible.
- If high-speed arrivals-on-red or RLR violations increase:
  - Review clearance intervals, yellow/red timings, and progression offsets.
  - Re-check visibility, signing, and enforcement.
- If near-miss conflicts at VRU locations increase:
  - Consider protected or leading ped phases, left-turn protection, or reduced turning volumes.
  - Adjust splits and offsets to reduce pressure on drivers to “beat the red.”

### 6.3 Instrumentation options
- **Speed probes**: connected vehicle probe speeds or third-party probe datasets.
- **Radar**: spot speed by segment and time-of-day.
- **Video analytics**: turning conflicts and near-misses at key intersections.
- **Field audits**: short, structured on-street observations to validate analytics.

---

## 7) Detailed Implementation Plan (programmatic)

### Phase 1: Program Setup and Corridor Selection (Weeks 1–6)
The agency establishes a corridor retiming team that includes traffic engineering, signal maintenance, operations, IT/networking, transit liaison, and a data/analytics contact so decisions are not blocked later by ownership questions. The team defines corridor-level outcomes (e.g., reduced median travel time, improved reliability, reduced stops per vehicle, improved arrivals-on-green) and explicit constraints for side streets and pedestrians so the project does not succeed by shifting delay to more vulnerable users.

A pilot corridor is selected with 5–12 signals where geometry is stable, spacing supports progression at the posted speed, and detection and communications can reasonably be made reliable. The team produces a baseline report from at least several weeks of data (counts, high-resolution controller logs if available, and probe travel times where available) and documents “special conditions” (school dismissal, freight peaks, recurring incident locations).

- **Roles**: traffic engineer (lead), signal maintenance supervisor (field readiness), TMC operations lead (activation/monitoring), IT/network lead (comms/security), data analyst (baseline and KPIs), transit liaison (TSP policy, bus ops), public information officer (communications if needed).
- **Deliverables**: project charter, corridor map and movement inventory, baseline KPI & equity report, initial risk register, and agreed operational constraints (pedestrian minimums/clearance, side-street delay caps, emissions/safety boundaries).
- **Risks**: corridor is not realistically coordinatable; baseline data incomplete; community acceptance low if perceived as “speeding up traffic.”
- **Acceptance checks**: baseline KPIs reproducible; corridor selection documented with constraints; conservative fallback strategy agreed before any field changes.

### Phase 2: Detection, Communications, and Data Readiness (Weeks 7–14)
The agency closes detection gaps that prevent reliable travel-time and arrivals-on-green measurement by repairing loops, adding radar, validating video zones, and ensuring stop-line presence where actuated operation depends on it. IT hardens communications (plan downloads, time sync, and event logging) and documents minimum uptime targets and escalation paths. The data team implements a pipeline that stores controller events, detector actuation data, TSP/EVP events, and probe travel times in consistent intersection and movement identifiers.

Detector QA (stuck-on/off, chattering, flatline checks) is run before any retiming; failures are fixed so the system does not calibrate on bad data.

- **Roles**: signal technicians (detection work), IT/network engineer (connectivity and security), data engineer (pipelines and schemas), traffic engineer (movement mapping), operations (monitoring requirements).
- **Deliverables**: detector QA report, comms readiness report, operational data feeds, intersection/movement registry, basic health dashboards.
- **Risks**: detector noise produces misleading travel times; comms dropouts cause stale state; retrofits delayed by procurement.
- **Acceptance checks**: data feeds meet completeness thresholds (e.g., >95% during peaks); time sync stable; dashboards show plausible volumes/occupancies.

### Phase 3: Build and Calibrate the Corridor Twin (Weeks 15–26)
The modeling team builds a corridor model including lanes, turn bays, signal phasing, pedestrian timings, bus stops, and realistic speed behavior—progression quality depends heavily on turning friction and side-street discharge. Demands and turning proportions are calibrated by time-of-day; operational parameters such as saturation flow and start-up lost time are estimated using high-resolution controller logs where possible.

The model is validated against field measurements by matching not only average travel times but also reliability (p95 travel time), queue patterns, arrivals-on-green, and, where possible, basic safety proxies (speed distributions, near-miss counts using surrogate safety tools).

- **Roles**: traffic modeler (twin build), data analyst (calibration datasets), traffic engineer (validation criteria), operations representative (operational realism).
- **Deliverables**: calibrated twin, validation report, parameter registry, and a “known limitations” memo that defines when the twin should not be trusted.
- **Risks**: overfitting to a short calibration window; poor representation of pedestrian or transit interactions; missing turning data.
- **Acceptance checks**: twin meets validation tolerances on key segments (e.g., median and p95 travel times within 10–15% of observations), reproduces observed queue peaks and spillback patterns, and passes safety proxy spot-checks.

### Phase 4: Time-of-Day Plan Design and Constraint Review (Weeks 27–34)
The engineering team generates a small library of time-of-day plans (AM peak, midday, PM peak, off-peak) using standard coordination practice (cycle length, splits, offsets) while explicitly preserving pedestrian minimums and avoiding progression targets that incentivize speeding. Candidate plans are tested in the twin to ensure progression does not create systematic spillback or unacceptable side-street delay and that transit and freight windows are respected.

Each plan is packaged as a **change-control artifact** that includes a before/after KPI estimate, constraints satisfied (including equity and safety gates), and a rollback plan so that operators can revert quickly if field behavior diverges from simulation.

- **Roles**: traffic engineer (plan design), safety/accessibility reviewer (ped timing and VRU checks), equity/environment reviewer, operations lead (operational constraints), transit liaison (TSP integration), communications contact (if public messaging is needed).
- **Deliverables**: versioned plan library, constraint compliance report, documented rollback paths, and an operator-facing summary of when/how to use each plan.
- **Risks**: side-street starvation; pedestrian service violations; progression that rewards speeding; unexpected TSP/EVP interactions.
- **Acceptance checks**: pedestrian timings verified; side-street delay caps met in simulation; TSP/EVP scenarios tested; each plan includes a tested rollback procedure.

### Phase 5: Pilot Deployment (Shadow → Assisted → Active) (Weeks 35–48)
The agency first runs the new plans in **shadow mode**, computing expected corridor metrics and comparing them to existing operation without changing the field. This detects data quality problems and model drift early.

The operations team then activates the plans in an **assisted manner** (e.g., enabling AM peak coordination first) while maintaining a clear manual override and immediate revert path. During activation, the team monitors ATSPM measures daily (arrivals on green, split failures, red occupancy, travel time reliability) plus equity and safety indicators (person-delay distribution, speed distributions, RLR proxies) and responds to anomalies with targeted adjustments—not uncontrolled “tuning by feel.”

- **Roles**: operations (activation and monitoring), traffic engineering (on-call retiming), analyst (daily KPI review), maintenance (hot fixes), transit liaison (monitoring TSP impacts).
- **Deliverables**: pilot report with before/after KPIs, operator runbook, alarm thresholds, and a decision log of changes made during the pilot.
- **Risks**: public complaints; unexpected queues due to construction or seasonal changes; over-reliance on the twin; unanticipated transit or EVP behavior.
- **Acceptance checks**: KPI improvements observed without violating constraints; operators can revert within minutes; monitoring alerts tested and validated.

### Phase 6: Operations and Continuous Optimization (Weeks 49+ / Ongoing)
Coordination is treated as an **ongoing program**. The team establishes a monthly cadence to review detector QA, comms uptime, KPI drift, equity impacts, and safety metrics, and retimes seasonally or when drift triggers indicate that plans are no longer effective.

As the program scales to adjacent corridors, the agency reviews cross-corridor equity so improvement is not concentrated only on major arterials while side streets and pedestrian networks degrade.

- **Roles**: traffic engineering (retiming ownership), operations (daily monitoring), IT (system upkeep), analyst (monthly reporting), equity/safety leads (periodic reviews).
- **Deliverables**: monthly KPI and equity/safety report, updated plan library versions, maintenance work orders, expansion roadmap.
- **Risks**: calibration drift; equipment aging; shifting demand due to land-use changes; policy drift away from equity and safety commitments.
- **Acceptance checks**: KPI regressions trigger documented corrective action; plan versioning and audit logs are maintained; detector/comms health remains within SLA; equity gaps do not systematically widen.

---

## 8) Corridor coordination workflow diagram (conceptual)

Use this as a conceptual diagram to communicate end-to-end flow. Implement in your own tooling as a process map or swimlane diagram.

```text
[Design & Objectives]
   ↓
[Data & Twin Build]
   ↓
[Candidate Plans (cycle/split/offset + TSP/EVP rules)]
   ↓  (simulate)
[Digital Twin Evaluation]
   ↓  (if validation & constraints OK)
[Shadow Mode]
   ↓  (compare expected vs actual KPIs, equity, safety)
   ↓  (gates passed?)
[Governance Approval]
   ↓
[Pilot Deployment]
   ↓  (monitor KPIs, equity, safety, TSP/EVP, complaints)
   ↓  (rollback if triggers)
[Full Deployment]
   ↓
[Continuous Monitoring & Retiming Loop]
   ↺ (feeds back into Design & Objectives)
```

You can convert this into a swimlane diagram separating responsibilities for **Engineering**, **Operations/TMC**, **Maintenance/IT**, **Transit/EVP**, and **Equity/Safety**.

---

## 9) Choices

- **Pre-timed** for predictable demand and simple progression.
- **Traffic-responsive** for repeated but less predictable patterns.
- **Adaptive (SCOOT/SCATS-style)** for highly variable traffic.
- **Twin-supported** for safe experimentation and policy testing.

---

## 10) MVP Deployment

- One corridor (5–12 signals) with good detection and communications.
- 2–3 time-of-day plans plus a clearly defined fallback.
- Health monitor tracking: travel times, arrivals on green, side-street/ped delay, safety proxies, TSP/EVP behavior.
- Operator revert option tested in drills.

---

## 11) Evaluation

- **Travel time distributions** (median, p95) by direction, mode, and time-of-day.
- **Stops/vehicle and red delays**.
- **Side-street/ped delays** and compliance with caps.
- **Queue spillback events** and storage utilization.
- **Person-delay and equity metrics** by neighborhood and mode.
- **Safety proxies** (speeds, RLR proxies, near-misses) before and after.
- **Transit reliability** and TSP usage impacts.

---

## 12) Implementation Checklist (operator/agency facing)

Use this as a pre-launch and scaling checklist for a corridor green-wave program.

- [ ] Define corridor objectives, including at least one **person/multimodal** objective, and explicit constraints (ped max wait, side-street delay/queue caps, emissions/safety boundaries).
- [ ] Map corridor typology, sensitive locations (schools, hospitals, EJ areas), transit routes, freight routes, bike routes, and key pedestrian crossings.
- [ ] Create a **multimodal corridor scorecard** template (by mode, movement, time-of-day, geography, equity).
- [ ] Repair and validate detection and comms; implement detector QA and time-sync checks.
- [ ] Stand up ATSPM or equivalent reporting; monitor split failures and occupancy-based measures.
- [ ] Build and calibrate the digital twin; validate travel time, queue/spillback patterns, and basic safety proxies before approving timing changes.
- [ ] Define **validation gates** (twin validation, shadow-mode agreement, equity and safety thresholds) and document acceptance criteria.
- [ ] Design time-of-day plans with safe progression speeds; verify ped/ADA behavior in both model and field spot checks.
- [ ] Define written **TSP/EVP policies** (conditional priority, max disruption limits, recovery rules) and test them in the twin.
- [ ] Run **shadow mode** and compare expected vs observed metrics for multiple weeks before activation.
- [ ] Establish an approval workflow with RACI, sign-offs, and version IDs for each plan set.
- [ ] Deploy with assisted activation and a **tested rollback plan**; conduct rollback drills.
- [ ] Monitor safety proxies (speed distributions, RLR proxies, near-miss metrics where available) and require safety sign-off for continuation or expansion.
- [ ] Publish corridor scorecards and equity/safety summaries; track community feedback and complaints by plan version.
- [ ] Establish change control (versioned plan library, approval gates, decision logs) and a monthly review cadence.

---

## 13) Reference Links

- [FHWA Traffic Signal Timing Manual (PDF)](https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf)
- [FHWA Traffic Signal Timing Manual — Chapter 5 (Basic timing)](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm)
- [FHWA Traffic Signal Timing Manual — Chapter 6 (Coordination)](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter6.htm)
- [FHWA Traffic Signal Timing Manual — Chapter 8 (Maintain effective timing)](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter8.htm)
- [FHWA Traffic Signal Timing Manual — Chapter 9 (Priority & Preemption)](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm)
- [FHWA ATSPM (landing)](https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm)
- [FHWA ATSPM — use cases (Ch.4)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)
- [FHWA ATSPM (PDF)](https://ops.fhwa.dot.gov/publications/fhwahop20002/fhwahop20002.pdf)
- [FHWA Staffing Guidelines (PDF)](https://ops.fhwa.dot.gov/publications/fhwahop09006/fhwahop09006.pdf)
- [JRC — The Calibration of Traffic Simulation Models (PDF)](https://publications.jrc.ec.europa.eu/repository/bitstream/JRC68403/lbna25188enn.pdf)
- [FHWA — Surrogate Safety Measures From Traffic Simulation Models (PDF)](https://ntlrepository.blob.core.windows.net/lib/38000/38000/38015/FHWA-RD-03-050.pdf)
- [A Sophisticated Transit Signal Priority System – How it Works (PDF)](http://conf.tac-atc.ca/english/resourcecentre/readingroom/conference/conf2006/docs/s009/stewart1.pdf)

---

## 14) Key Terms and Explanations

- **Green Wave**: Coordinated traffic signals creating a "wave" of green lights for smoother flow.
- **V2X (Vehicle-to-Everything)**: Communication tech allowing vehicles to interact with infrastructure, other vehicles, and networks.
- **SCOOT/SCATS**: Adaptive traffic control systems (SCOOT: UK; SCATS: Australia) that adjust signals based on real-time data.
- **Cycle Length**: Total time for a complete signal cycle (e.g., red–yellow–green phases).
- **Offsets**: Time shifts in signal phases between intersections to align greens for progression.
- **Splits**: Division of cycle time among phases (e.g., how much green for each direction).
- **Platoon**: Group of vehicles traveling closely together at similar speeds.
- **Digital Twin**: Virtual model simulating real-world traffic systems for testing and optimization.
- **Rebound Effects**: Unintended consequences where efficiency gains increase car use, negating environmental benefits.
- **GLOSA (Green Light Optimal Speed Advisory)**: System advising drivers on speeds to catch green lights.
- **AVs (Autonomous Vehicles)**: Self-driving vehicles that can integrate with traffic systems.
- **AI (Artificial Intelligence)**: Algorithms like RL (Reinforcement Learning) for adaptive decision-making.
- **KPI (Key Performance Indicator)**: Metrics like travel time or emissions to measure system success.
- **Guardrails**: Safety and equity limits (e.g., max pedestrian wait, emissions caps) to prevent adverse effects.
- **MVP (Minimum Viable Product)**: Smallest useful deployment for testing (e.g., one corridor and a minimal plan set).
- **p95**: 95th percentile, indicating worst-case performance (e.g., 95% of trips under this time).

---

Cross-links: Related ideas include real-time what-if buttons, delay-tolerant signals, explainable signals, and safety-first signals.
