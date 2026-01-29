# 14) Weather + Traffic + Power Together: Integrated Constraints Mode

## Purpose (what this doc is)
This is an implementation-ready onboarding document for a **compound-stressor mode** that helps a traffic-signal system behave safely and predictably when **weather + traffic disruption + power/comms degradation** occur together.

It assumes you already have a functioning signal management stack (controller plans, monitoring, ATSPM/performance analytics, and operator workflows). The output is:

- A small, governed **mode library** with pre-vetted plans
- A **trigger design** that fuses weather/traffic/power signals with hysteresis
- A **storm policy “contract”** (what changes and what never changes)
- A **comms plan** (PACE) for internal ops + external messaging
- **Power-constrained behavior** rules for cabinets/controllers/central
- A **runbook + drills** and a basic state diagram for implementation

## Catchy Explanation
When a storm hits and the grid is unstable, “optimal timing” is not the priority. The signal should switch into a clear, safer mode that respects weather risk and power/comms constraints.

## What it is (precise)
**Weather + traffic + power together** is a mode-selection framework that fuses three constraint families:

1) **Traffic state** (demand surges, queues, spillback risk),
2) **Weather risk** (visibility, friction, flooding risk, snow/ice),
3) **Power/comms status** (UPS on battery, comms degraded, controller isolation).

The system chooses conservative, pre-vetted timing plans (storm-safe, outage/local mode, evacuation bias) that favor predictability and safety over efficiency. Digital twins are used to test triggers, plan transitions, and recovery steps.

## Non-goals / important constraints
- This mode is **not** a replacement for: cabinet safety hardware, conflict monitoring, emergency preemption, or MUTCD compliance.
- This mode is **not** “AI decides timing freely.” It selects from a **library of vetted plans**.
- In power/comms loss, many sites will be forced into **local controller behavior** and/or **flash** depending on equipment and policy.

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

---

## 1) Mode Library (keep it small)
Start with **4–6 modes**. If you create 12 modes, operators will never trust or use them.

Recommended minimum set:

- **NORMAL**: existing operations.
- **STORM-SAFE**: conservative plans prioritizing predictable clearance and pedestrian safety.
- **TRAFFIC-INCIDENT SURGE**: queue/spillback containment when weather is not the driver.
- **POWER-SAVE / COMMS-DEGRADED**: energy-conserving, low-telemetry behavior while maintaining safety.
- **OUTAGE / ISOLATED LOCAL**: controller-only behavior with minimal central dependence.
- **RECOVERY**: stepwise restoration to coordination.

Optional (if your region uses these):

- **EVACUATION BIAS**: route/corridor biasing agreed with emergency management.

### Mode invariants (things that must never break)
- Preserve pedestrian minimums and clearance behavior.
- Preserve safe vehicle change/clearance intervals.
- Rate-limit plan changes (no flapping).
- Always provide rollback to a safe local plan.
- Log reasons and evidence for each mode change.

These align with standard signal timing parameter concepts (minimum green, pedestrian timing, change/clearance intervals). See [`chapter5.htm`](ideas/14-weather-traffic-power-together.md:1) citations below for operational parameter context. In particular, pedestrian walk guidance includes a minimum walk duration guidance and clearance-time calculation practices [`chapter5.htm`](ideas/14-weather-traffic-power-together.md:1).

---

## 2) Inputs & Observability (what you must measure)

### 2.1 Weather signals (authoritative sources)
Minimum:
- NWS/WMO-grade hazard products (watch/warning/advisory tiers)
- Precipitation intensity / type (rain/snow/freezing rain)
- Visibility / fog
- Wind gusts
- Temperature and freeze potential

Use “official hazard products” as **policy triggers** because they are operator-friendly and auditable (e.g., Winter Storm Watch/Warning, Dense Fog Advisory, High Wind Warning). NWS definitions provide clear semantics and timing (watch = possible; warning = occurring/imminent) [`warningsdefined`](ideas/14-weather-traffic-power-together.md:1).

### 2.2 Traffic signals
Minimum:
- Occupancy or volume (per approach)
- Queue proxy (occupancy + speed drop; or camera-based queue)
- Corridor travel time (probe)
- Spillback detection (stopline blocked; upstream occupancy saturation)

### 2.3 Power & comms signals
Minimum:
- Cabinet power: mains present/absent; UPS on battery; estimated runtime
- Controller health: watchdog resets; MMU status; flash transfer events
- Communications: heartbeat state; packet loss/latency; last good config sync

Controller/cabinet standards include explicit considerations for power interruption behavior and malfunction management (MMU) capabilities; TS 2 also includes flash/transfer-related testing sections and power interruption content headings [`nema-ts-2-2021-contents-and-scopeb71294ae-eb9a-45e2-a501-e5c0672dbd6b.pdf`](ideas/14-weather-traffic-power-together.md:1).

---

## 3) Trigger Design (fusion + hysteresis)
This section is the “how we switch modes” design.

### 3.1 Trigger philosophy
- Prefer **auditable, human-readable** triggers.
- Fuse fast local telemetry (power/comms) with slower external signals (weather products, forecasts).
- Use **multi-signal voting** to reduce false positives.
- Use **dwell times** and **exit criteria** to prevent oscillation.

### 3.2 Trigger primitives
Define primitives with normalized severity levels.

**WeatherSeverity** (0–3):
- 0 = none
- 1 = advisory-level hazards likely to degrade operations (e.g., dense fog advisory; winter weather advisory)
- 2 = warning-level hazard or high confidence watch (e.g., winter storm warning; high wind warning)
- 3 = extreme conditions (e.g., blizzard warning; ice storm warning; flash flood warning)

NWS definitions provide thresholds/meaning for these hazard products and timing expectations [`warningsdefined`](ideas/14-weather-traffic-power-together.md:1).

**TrafficStress** (0–3):
- 0 = normal
- 1 = travel time +25% or moderate queueing
- 2 = travel time +50% or spillback risk on key approaches
- 3 = sustained spillback / gridlock indicators

**PowerCommsStatus** (0–3):
- 0 = normal
- 1 = comms degraded (lossy/high-latency), mains OK
- 2 = UPS on battery OR repeated brownout indicators
- 3 = site isolated / controller failsafe / flash transfer / imminent loss of control

### 3.3 Example fusion logic (implementation-ready)
Implement as a simple, testable rules engine.

**Rule set (baseline):**

- Enter **STORM-SAFE** when:
  - WeatherSeverity ≥ 2 for ≥ 10 minutes, AND
  - (TrafficStress ≥ 1 OR forecast horizon < 6h), AND
  - PowerCommsStatus ≤ 2

- Enter **POWER-SAVE / COMMS-DEGRADED** when:
  - PowerCommsStatus = 1 for ≥ 5 minutes, OR
  - telemetry loss > X% for ≥ 5 minutes

- Enter **OUTAGE / ISOLATED LOCAL** when:
  - PowerCommsStatus ≥ 3 OR
  - controller heartbeat lost for ≥ 2 minutes AND cabinet reports mains absent

- Enter **EVACUATION BIAS** when:
  - Emergency management request is active (digitally signed), AND
  - WeatherSeverity ≥ 1 OR TrafficStress ≥ 2

- Enter **RECOVERY** when:
  - WeatherSeverity ≤ 1 for ≥ 60 minutes, AND
  - TrafficStress ≤ 1 for ≥ 30 minutes, AND
  - PowerCommsStatus = 0 for ≥ 30 minutes

**Hysteresis / anti-flap:**
- Minimum time-in-mode: 20–60 minutes (mode-dependent)
- Cooldown between mode changes: 10 minutes
- Require “exit votes” from 2 independent signals (e.g., weather products + RWIS)

### 3.4 Evidence packets (why a mode changed)
Every recommendation/activation should generate an “evidence packet”:
- timestamp, corridor/site scope
- WeatherSeverity inputs (hazard product + raw measurements)
- TrafficStress inputs (KPIs/thresholds)
- PowerCommsStatus inputs (UPS state, comms heartbeat)
- proposed mode + rationale + expected tradeoffs
- operator action (confirm/deny) and reason code

### 3.5 Trigger specification template (required visual)

Use a structured trigger specification so each compound trigger can be reviewed, versioned, and audited.

```markdown
| Field | Description | Example |
|---|---|---|
| Trigger ID | Stable identifier for the trigger bundle | STORM_SAFE_WINTER_01 |
| Name | Human-readable trigger name | "Winter storm warning – corridor storm-safe activation" |
| Scope | Network scope affected | Corridor X (MP 0.0–7.5); sites A–M |
| Trusted signals | Inputs and their sources | WeatherSeverity (NWS WSW); RWIS visibility; TrafficStress (TTI & queue proxy); PowerCommsStatus (UPS + heartbeat) |
| Enter condition | Boolean condition to enter mode (including dwell) | WeatherSeverity ≥ 2 for ≥ 10 min AND (TrafficStress ≥ 1 OR forecast horizon < 6h) AND PowerCommsStatus ≤ 2 |
| Exit condition | Boolean condition to leave mode | WeatherSeverity ≤ 1 for 60 min AND TrafficStress ≤ 1 for 30 min AND PowerCommsStatus = 0 for 30 min |
| Hysteresis & persistence | Dwell times, cooldown, exit votes | Min time-in-mode 30 min; 10 min cooldown; 2 independent exit votes (NWS + RWIS) |
| Granularity | Spatial and temporal granularity of evaluation | Evaluate every 5 min per corridor; roll up to regional view for comms |
| Safety invariants | Invariants that must hold if trigger fires | No reduction in ped minimums; no untested plans; max 1 plan change per 20 min |
| Logging fields | Required evidence in logs | All input values, evaluation result, operator decision, reason code, plan IDs before/after |
| Versioning | How this trigger spec is versioned and retired | v1.0 pilot; v1.1 after first season AAR; retired when replaced by v2.0 |
```

---

## 4) Storm Policy Contract (what changes; what never changes)
Treat this as an internal “contract” between engineering, operations, emergency management, and public comms.

### 4.1 Contract goals
- Ensure safety constraints are never violated.
- Ensure changes are **predictable**, **bounded**, and **reversible**.
- Establish what can be automated vs. requires explicit authorization.

### 4.2 Contract: allowed changes by mode
**STORM-SAFE (allowed):**
- Conservative splits; longer all-red only if policy allows and capacity impact accepted.
- Remove permissive movements that become risky with low visibility (policy-dependent).
- Increase pedestrian protection (no reduction of minimums; consider longer walk in school/elderly zones).

Pedestrian walk and clearance considerations should follow established guidance and local policy; MUTCD-related walk minimums and clearance-time computation practices are summarized in FHWA timing guidance [`chapter5.htm`](ideas/14-weather-traffic-power-together.md:1).

**POWER-SAVE / COMMS-DEGRADED (allowed):**
- Disable nonessential telemetry.
- Prefer stable local actuation / fixed-time patterns.
- Reduce optimization compute frequency.

**OUTAGE / ISOLATED LOCAL (allowed):**
- Force controller to a pre-vetted local plan set; eliminate central dependencies.

### 4.3 Contract: prohibited changes (hard no)
- Any change that reduces pedestrian minimums below adopted policy.
- Any change that invalidates change/clearance interval safety.
- Any untested plan pushed to field during an event.

### 4.4 Authority matrix
Define who may request/approve each action:
- Operator on duty: STORM-SAFE, POWER-SAVE
- Traffic engineer on call: corridor-wide plan swaps, clearance parameter changes
- Emergency management liaison: EVACUATION BIAS request
- IT/network on call: comms isolation / VPN changes
- Maintenance: cabinet power work, generator/UPS deployment

Make this table explicit in your agency SOP (appendix below).

---

## 5) Comms Plan (PACE) for “out-of-the-ordinary” operations
Use a PACE model (Primary / Alternate / Contingency / Emergency) and practice it.

CISA provides a PACE planning resource intended to establish and practice PACE communications for critical comms in unusual situations [`leveraging-pace-plan-emergency-communications-ecosystem`](ideas/14-weather-traffic-power-together.md:1).

### 5.1 PACE template (fill this in)
**Primary**: Normal ATMS network + dispatch/ops chat + ticketing
- Who: TMC operators, on-call engineers, maintenance supervisor
- What: dashboards, event timelines, plan activation approvals

**Alternate**: Secondary network path + cellular + voice bridge
- What changes: reduced telemetry; prioritized alerts only

**Contingency**: Radio / satellite / pre-established mutual-aid channels
- What changes: corridor-level actions only; no per-intersection tuning

**Emergency**: In-person runner + printed runbook + local cabinet access
- What changes: field crews execute local plan cards and status reporting via voice-only

### 5.2 Message discipline
- Use standard message types: *Advisory*, *Action required*, *Status update*, *All-clear*.
- Keep a single event timeline log.

### 5.3 Public communications triggers
Public comms should activate if:
- widespread flash is expected/occurring
- evacuation bias is active
- major routes are intentionally re-timed

---

## 6) Power-constrained behavior (what the system must do)
Power constraints change what’s possible more than any other stressor.

### 6.1 Design assumptions
- Sites may be on mains, UPS, generator, or fully de-energized.
- Comms may fail before power fails.
- Controllers/cabinets have safety hardware paths (MMU, flash transfer relays) that may dominate behavior.

Traffic controller assembly standards address controller power interruption considerations and malfunction management (MMU) functions; TS 2 content explicitly covers power interruption tests and MMU test sections (conflict monitoring, minimum change/clearance monitoring, etc.) [`nema-ts-2-2021-contents-and-scopeb71294ae-eb9a-45e2-a501-e5c0672dbd6b.pdf`](ideas/14-weather-traffic-power-together.md:1).

### 6.2 System behavior by power state
**Mains OK**
- Full mode logic available.

**UPS on battery (estimated runtime known)**
- Enter POWER-SAVE.
- Reduce nonessential compute and comms.
- Prefer “stable” plans to reduce frequent actuation/coordination churn.
- Pre-stage a local fallback plan if battery drops below a threshold.

**Brownouts / unstable supply**
- Prioritize controller safety behavior.
- Increase logging of resets and voltage alarms.

**Mains absent + comms absent (site isolated)**
- OUTAGE/ISOLATED LOCAL behavior only.
- Controllers run local plan cards; field crews follow cabinet procedure.

### 6.3 DER / microgrid interaction (optional, if applicable)
If intersections are backed by inverter-based DER (solar+storage), grid events may include ride-through behavior and abnormal voltage/frequency conditions.

NREL’s overview of IEEE 1547-2018 highlights modern DER expectations such as **ride-through of abnormal voltage/frequency** and interoperability concepts, useful when coordinating with utilities/microgrids during disturbances [`75436.pdf`](ideas/14-weather-traffic-power-together.md:1).

---

## 7) Storm safety: field & roadway considerations
This mode must reduce additional risk to motorists, pedestrians, and field crews.

### 7.1 Road-user safety behaviors
In STORM-SAFE:
- Emphasize predictability; avoid frequent “clever” adjustments.
- Consider removing risky permissive behavior during low visibility.
- Maintain pedestrian service and avoid creating long waits that encourage unsafe crossings.

Pedestrian walk and clearance computations and minimum walk guidance are documented in FHWA signal timing guidance [`chapter5.htm`](ideas/14-weather-traffic-power-together.md:1).

### 7.2 Field crew safety
- Do not dispatch crews into wind/ice conditions above locally defined thresholds.
- Require check-in/out and buddy system for cabinet work during storms.
- Use PACE comms for crew safety (voice confirmation when digital is out).

### 7.3 Flooding / ice / visibility
Use hazard products as strong triggers:
- Dense fog advisory criteria and warning/advisory semantics are defined by NWS products [`warningsdefined`](ideas/14-weather-traffic-power-together.md:1).

---

## 8) Implementation Strategies

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

---

## 9) Integration architecture

### 9.1 Components
- **Telemetry adapters**: weather products + RWIS + ATMS cabinet telemetry + traffic detectors.
- **Fusion & rules engine**: computes (WeatherSeverity, TrafficStress, PowerCommsStatus), produces recommendation.
- **Plan library service**: versioned plans by corridor/site, with mode tags.
- **Orchestration**: pushes plan changes; verifies acknowledgments.
- **Audit log**: evidence packets + operator decisions.
- **Dashboards**: operator views by corridor and site.

### 9.2 State diagram (required visual: Normal → Storm → Power-constrained → Comms-loss → Recovery)

This is the canonical storm/outage state diagram. It abstracts to the five states the program must support, while still mapping to the internal mode library.

```text
            [ NORMAL ]
                |
     (Storm trigger: WeatherSeverity ≥ 2,
      TrafficStress ≥ 1, power/comms OK,
      dwell satisfied)
                v
          [ STORM MODE ]
        (e.g., STORM-SAFE,
         EVACUATION BIAS overlay)
                |
     (PowerCommsStatus ≥ 1,
      UPS on battery or comms
      significantly degraded)
                v
 [ POWER-CONSTRAINED MODE ]
    (e.g., POWER-SAVE /
     COMMS-DEGRADED)
                |
     (Loss of comms and/or
      mains power beyond
      thresholds)
                v
    [ COMMS-LOSS / OUTAGE ]
      (OUTAGE / ISOLATED LOCAL;
       local plans and MMU dominate)
                |
     (Exit criteria satisfied:
      hazards cleared, power and
      comms stable for dwell
      intervals, KPIs within
      bounds, operator approval)
                v
           [ RECOVERY ]
                |
     (Stability verified and
      operator closes event)
                v
            [ NORMAL ]
```

Map to concrete modes:
- **NORMAL** → existing operations.
- **STORM MODE** → STORM-SAFE (plus optional EVACUATION BIAS, if requested).
- **POWER-CONSTRAINED MODE** → POWER-SAVE / COMMS-DEGRADED.
- **COMMS-LOSS / OUTAGE** → OUTAGE / ISOLATED LOCAL (controller-only, MMU-governed behavior).
- **RECOVERY** → RECOVERY plans that step back toward NORMAL with dwell and monitoring.

### 9.3 Integration notes (field realities)
- Plan pushes must be **idempotent** and tolerate partial failure.
- If central cannot reach a site, the site must still behave safely.
- Controllers must have a “local card” for each mode that can run without comms.

---

## 10) Drills & exercises (how you prove it works)
Do not ship this without drills.

### 10.1 Drill types
- **Tabletop**: walk through triggers, authorities, comms PACE.
- **Simulation drill**: twin-based storm + comms loss + queue surge.
- **Game day**: controlled plan swap in a pilot corridor.

### 10.2 Minimum drill scenarios
1) Winter storm watch → warning; travel time +50%; comms stable.
2) Dense fog advisory; moderate queues; operator denies auto STORM-SAFE (test override).
3) UPS-on-battery events across 10% of corridor; comms degrading.
4) Full comms outage; controllers isolated local for 2 hours; recovery ramp.

### 10.3 After-action review template
- What triggered, when, and why?
- Were there oscillations?
- Did any site violate invariants?
- What was the public-facing impact?
- What should be changed (triggers, plans, training)?

---

## 11) Technical mechanics (parameters & guardrails)

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

Key operational parameters for phase intervals, pedestrian timing, and change/clearance intervals are described in FHWA timing guidance [`chapter5.htm`](ideas/14-weather-traffic-power-together.md:1).

---

## 12) MVP Deployment
- One corridor.
- Two modes beyond NORMAL (STORM-SAFE, OUTAGE/LOCAL).
- Assisted activation + drill once per quarter.

## 13) Evaluation
- Mode activation accuracy (true/false activations).
- Safety and delay impacts during storms.
- Recovery time and stability (no flapping).

---

# Appendix A — Storm/Outage Operations Runbook (SOP)
This is the operator-facing SOP. Print it.

## A1. Event start: “Storm threat identified”
1) Confirm hazard products and time window (NWS watch/warning/advisory) [`warningsdefined`](ideas/14-weather-traffic-power-together.md:1).
2) Start event log (single timeline).
3) Verify staffing + on-call contacts.
4) Verify PACE comms status (Primary + Alternate ready) [`leveraging-pace-plan-emergency-communications-ecosystem`](ideas/14-weather-traffic-power-together.md:1).

## A2. Pre-staging (2–24 hours before)
- Confirm STORM-SAFE plan library version + rollback version.
- Push “pre-stage” configs to controllers (so they can run locally if isolated).
- Identify critical intersections for emergency response routes.
- Confirm UPS telemetry and maintenance readiness.

## A3. Activation steps (assisted mode change)
1) Review evidence packet.
2) Confirm target scope: corridor / sub-corridor / intersections.
3) Announce internal action message: “Activating STORM-SAFE for Corridor X.”
4) Activate mode.
5) Verify acknowledgments (>= 95% within 10 minutes). For non-acks, mark as “likely local.”

## A4. During event: monitoring cadence
- Every 15 minutes: check spillback hotspots, flash sites, UPS-on-battery list.
- Every 60 minutes: check criteria for escalation or recovery.

## A5. Escalation: comms degradation
When comms loss exceeds threshold:
- Switch to PACE Alternate/Contingency.
- Stop per-intersection tuning; focus on corridor-level actions.

## A6. Escalation: power loss / UPS on battery
When UPS battery event occurs:
- Enter POWER-SAVE for affected sites.
- If runtime < threshold: dispatch maintenance (if safe) or pre-approve isolation.

## A7. Full outage / flash
- Confirm safety messaging to public.
- Coordinate with police/field traffic control if required.

## A8. Recovery
1) Confirm hazards have cleared using exit criteria.
2) Switch to RECOVERY plans (stepwise coordination restoration).
3) Watch for rebound congestion.
4) Close event log; schedule AAR within 72 hours.

---

# Appendix B — Governance & Communications Runbook

## B1. Activation authority matrix (fill in)
| Action | Initiator | Approver | Notes |
|---|---|---|---|
| Activate STORM-SAFE | TMC Operator | Shift Supervisor | Assisted activation recommended |
| Activate POWER-SAVE | TMC Operator | Shift Supervisor | Auto allowed after pilot if safe |
| Declare OUTAGE/LOCAL | System | Shift Supervisor | Based on heartbeat + power telemetry |
| Activate EVACUATION BIAS | Emergency Mgmt | Traffic Ops Director | Requires signed request |
| Change clearance parameters | Traffic Engineer | Traffic Ops Director | No changes mid-event unless safety-critical |

## B2. Comms (PACE) owner checklist
- Maintain the PACE contact roster.
- Maintain bridge numbers / radio channel list.
- Run quarterly PACE drill [`leveraging-pace-plan-emergency-communications-ecosystem`](ideas/14-weather-traffic-power-together.md:1).

## B3. Public information coordination
- Define what triggers a public advisory.
- Provide a template: “Signals may be in flash; treat intersections as all-way stop.”
- Publish “recovery status” updates when large corridors return.

---

# Appendix C — Implementation Checklist (engineering)

## C1. Data & telemetry
- [ ] Weather hazard product feed integrated (watch/warning/advisory) [`warningsdefined`](ideas/14-weather-traffic-power-together.md:1)
- [ ] RWIS/visibility/precip feed integrated (or proxy)
- [ ] Travel-time and queue proxies available
- [ ] Cabinet UPS and mains status telemetry integrated
- [ ] Controller heartbeat + last-config-sync tracked

## C2. Plan library
- [ ] STORM-SAFE plans created and validated
- [ ] OUTAGE/LOCAL plans created (controller-only viable)
- [ ] RECOVERY plans defined (stepwise restoration)
- [ ] Versioning and rollback process in place

## C3. Switching engine
- [ ] Rule engine implemented with dwell/hysteresis
- [ ] Evidence packet logging implemented
- [ ] Assisted activation UI implemented
- [ ] Idempotent plan push + ack tracking implemented

## C4. Operations readiness
- [ ] SOP printed and trained
- [ ] PACE plan documented and drilled [`leveraging-pace-plan-emergency-communications-ecosystem`](ideas/14-weather-traffic-power-together.md:1)
- [ ] After-action review process scheduled

---

# Appendix D — Reference links (authoritative, capped)
1) FHWA Traffic Signal Timing Manual (Chapter 5: basic timing; pedestrian and clearance intervals): https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm [`chapter5.htm`](ideas/14-weather-traffic-power-together.md:1)
2) NOAA/NWS Watch/Warning/Advisory definitions (hazard semantics and criteria): https://www.weather.gov/lwx/warningsdefined [`warningsdefined`](ideas/14-weather-traffic-power-together.md:1)
3) CISA: Leveraging the PACE Plan into the Emergency Communications Ecosystem: https://www.cisa.gov/resources-tools/resources/leveraging-pace-plan-emergency-communications-ecosystem [`leveraging-pace-plan-emergency-communications-ecosystem`](ideas/14-weather-traffic-power-together.md:1)
4) NEMA TS 2 (traffic controller assemblies; power interruption + MMU scope/sections): https://www.nema.org/docs/default-source/standards-document-library/nema-ts-2-2021-contents-and-scopeb71294ae-eb9a-45e2-a501-e5c0672dbd6b.pdf?sfvrsn=ef98a362_3 [`nema-ts-2-2021-contents-and-scopeb71294ae-eb9a-45e2-a501-e5c0672dbd6b.pdf`](ideas/14-weather-traffic-power-together.md:1)
5) NREL overview: Highlights of IEEE 1547-2018 (DER ride-through/interoperability context for backup power/microgrids): https://docs.nrel.gov/docs/fy20osti/75436.pdf [`75436.pdf`](ideas/14-weather-traffic-power-together.md:1)

---

# Appendix E — Completion checklist (ship criteria)
- [ ] Mode library is ≤ 6 modes and each has an owner
- [ ] Trigger bundles defined with dwell + exit criteria
- [ ] Storm policy contract signed off by Ops + Engineering + Emergency Mgmt
- [ ] PACE comms plan documented and exercised [`leveraging-pace-plan-emergency-communications-ecosystem`](ideas/14-weather-traffic-power-together.md:1)
- [ ] Power-constrained behaviors documented and tested in pilot
- [ ] State diagram matches implemented switching logic
- [ ] Plans validated in twin including transitions and recovery
- [ ] Runbook printed; operators trained; at least one tabletop drill completed
- [ ] Evidence packet logs verified in a simulated event

---

Cross-links: Related ideas include rare-event practice, delay-tolerant signals, and event-aware timing.
