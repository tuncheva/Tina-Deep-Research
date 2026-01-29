# 18) Event-Aware Timing: Concert/Stadium/Festival Mode

## Catchy explanation
The city already knows the stadium empties at 22:30. Event-aware timing is the signal system’s ability to switch into **event mode** on purpose—before the surge turns into gridlock—and then unwind safely back to normal.

## What it is (precise)
**Event-aware timing** is an operations + engineering capability to detect predictable demand surges (concerts, stadium events, festivals, permitted street closures, planned road work), then **activate special signal timing plans** that:

- bias service inbound vs outbound (or along signed diversion routes),
- maintain coordination where it matters (and break it where it helps),
- treat **pedestrian and crowd safety as first-class**, not an afterthought,
- integrate transit, curb, and emergency access needs,
- prevent spillback into upstream intersections and cut-through in neighborhoods,
- and transition back to normal operation via a **controlled recovery**.

Activation can be calendar-based, threshold-based, or operator-assisted. A digital twin (or faster what-if scoring) is used to build and validate the plan library, test trigger logic, and design recovery steps.

## Why it matters (outcomes)
- **Faster clearance**: reduces post-event congestion and gridlock.
- **Safer crowd movement**: predictable crossing opportunities and fewer ped/vehicle conflicts during surges.
- **Less improvisation**: repeatable playbooks reduce operator stress and late-night heroics.
- **Better coordination**: integrates traffic engineering, TMC ops, venue operators, transit, curb management, and public safety.

## Where it works best / where it can fail

### Works best when
- the **event network** is well defined (venue ↔ parking ↔ freeway ramps ↔ transit hubs),
- the agency has a **plan library** and documented authority to activate,
- pedestrian/crowd routes are treated as first-class constraints,
- curb/field control (officers, barricades, loading zones) aligns with the signal plan.

### Common failure modes
- **Surge variability**: attendance, weather, or staggered departures shift peak time and shape.
- **Staffing gaps**: off-hours monitoring/response is missing.
- **Misaligned field control**: manual traffic control conflicts with signal phasing.
- **Bad transitions**: switching back to normal too abruptly creates “shock waves.”
- **Curb chaos**: unmanaged pickup/dropoff blocks lanes and breaks progression.

## Minimum viable product (MVP)
- One venue corridor (or compact event district).
- Three core plans: **Ingress**, **Egress**, **Recovery**.
- Assisted activation (operator confirms; twin used offline for design, not per-cycle).
- Lightweight monitoring dashboard: queues, spillback, ped service status, transit headways, key curb hotspots.

---

# Implementation-ready onboarding
This section is written as a practical “how we do this here” guide for an agency building event-aware timing.

## 0) Operating definition and scope

**Event-aware timing scope** (for this doc):

- Planned special events (PSEs): sports, concerts, festivals, conventions, parades, permitted demonstrations.
- Planned disruptions: permitted work zones and major planned lane/road closures that materially affect the event network.

**Not in scope** (handled by other playbooks):
- No-notice incidents (crashes, downed lines, hazmat, etc.).
- Storm evacuation/emergency incident response.

FHWA treats planned special events as a known-in-advance class of “incidents” that change demand and sometimes reduce capacity, with explicit guidance on pedestrian access planning, traffic control, and post-event operations.

---

## 1) Core concepts (shared vocabulary)

### 1.1 Plan library
A **plan library** is a versioned set of signal timing plans and associated “when/how to use” notes.

Minimum components for a stadium/arena:
- **INGRESS plan** (pre-event arrival bias).
- **EGRESS plan** (post-event outbound bias, ped-protective variants).
- **PED SURGE plan** (if crossings must meter vehicles to protect crowds).
- **DIVERSION plan** (if certain ramps/streets are closed for the event).
- **RECOVERY plan** (stepwise return to normal coordination).

Each plan must have:
- A numeric ID + version (vMAJOR.MINOR).
- A description of objectives, constraints, and KPIs.
- Boundaries: **where** the plan applies (core/buffer rings below).

### 1.2 Event phases
We will use three operational phases:
- **Pre-event / ingress**: arrivals building, crowd mostly inbound.
- **Egress / peak release**: venue emptying; ped queues at gates, outbound queues at garages.
- **Post-event / recovery**: demand decaying back toward normal; transition back to normal TOD plans.

### 1.3 Multimodal constraints (ped + transit + curb)
Event corridors are multimodal by default:
- Pedestrians surge and require safe crossings/queuing.
- Transit often adds special service and faces crowd surges.
- Curb usage shifts (pickup/dropoff, shuttles, deliveries, micromobility).

FHWA PSE guidance emphasizes a **pedestrian access plan** near venues, including separating pedestrian flows from vehicles, staffed crossings/closures, and ensuring pedestrian routes and crossings have adequate capacity.

---

## 2) Roles and responsibilities (RACI)

Create a short “who can do what” matrix so event mode never depends on one person.

Minimum roles:
- **Traffic Engineering Lead (Signal Timing Owner)**: owns plan design, approvals, versioning.
- **TMC/Operations Supervisor (Activation Authority)**: day-of decision, monitoring, rollback.
- **Field Command / Law Enforcement Liaison**: coordinates staffed crossings, closures, and officer control.
- **Venue / City Events Liaison**: calendar, gate times, parking operations, anticipated surges.
- **Transit Agency Liaison**: added service, crowd surges, shuttle staging.
- **Curb/Right-of-Way Coordinator**: rideshare/loading geofences, micromobility corrals, temporary curb permits.
- **Emergency Services Liaison**: emergency routes, access windows, staging.
- **Data/ATSPM Analyst**: KPIs, after-action review, tuning recommendations.

---

## 3) Pedestrian and crowd safety as first-class

### 3.1 Egress protection variants and timing patterns
Near venue gates, safety-first timing patterns should:
- Favor **predictable ped crossing windows** with adequate WALK and clearance.
- Avoid permissive turn conflicts into heavy ped streams where feasible (protected-only or banned turns during peak egress).
- Prevent vehicle queues blocking crosswalks by controlling release from upstream.

Timing variants:
- **Ped-priority egress plan**: more frequent WALKs and longer WALK on key legs, with bounded vehicle queues.
- **Scramble / exclusive ped phase** where warranted (e.g., main gate plaza).
- **Gate-closure coordination**: when gates close or ped flow drops, revert to less restrictive patterns.

### 3.2 Max ped wait caps near gates
Define explicit **maximum ped wait** near event gates (e.g., ≤ 60–90 s under event mode), lower than system-wide caps if feasible.

Guardrails:
- Hard cap on ped delay at designated “gate crossings.”
- If predicted or observed ped delay exceeds cap:
  - raise priority of ped phases in that ring,
  - log violation and escalate to event command.

### 3.3 Turn-conflict mitigations
Candidate mitigations by location:
- Near gates: prohibit permissive lefts/right-turn-on-red during egress window.
- Provide protected-only lefts where geometry & volumes justify.
- If RTOR cannot be banned physically, add clear RLR/RTOR enforcement signage, enforcement coordination, or temporary physical devices.

### 3.4 Monitoring inputs and KPIs (ped/crowd)
Monitor, at minimum:
- **Ped crossing demand** (counts or proxies) at key legs.
- **Ped queueing/crowding** where cameras are available.
- **Conflict proxies**: vehicles entering crosswalk late, yield compliance at turns (if detection supports).

Ped/crowd KPIs:
- Max ped wait (by gate crossing) vs cap.
- Share of cycles missing ped service when ped demand exists.
- Observed conflicts/complaints at gates.

---

## 4) Multimodal + curb integration (transit, TSP, curb, micromobility, emergency)

### 4.1 Transit surges and detours
Transit elements:
- Special event shuttles.
- Added rail or bus service.
- Detoured routes around closures.

Policy:
- Identify **event-critical transit movements** (e.g., shuttle routes, rail-to-bus transfers) that must be protected.
- Guarantee minimum green/service windows for those routes under egress mode.

### 4.2 TSP policy during egress
Define a clear **TSP during event** policy:
- Is TSP **fully enabled, partially enabled, or disabled** during peak egress?
- Do emergency vehicles receive priority separate from transit?

Examples:
- Keep EVP fully enabled; TSP limited to specific corridors or only when headways are degraded.
- Log interactions between event plans and priority requests (for post-event review).

### 4.3 Curb and rideshare/loading geofences
Coordinate with curb management and TNCs:
- Define **geofenced pickup/dropoff zones** away from the most critical links.
- Restrict pickup geofences on links that must remain moving for transit/emergency.
- Temporarily repurpose curb lanes (e.g., barrier-protected ped space, shuttle loading).

Timing implications:
- Where curb lanes are repurposed for ped space: adjust detection, phases, or plans to match the changed geometry.
- Where geofenced loading is allowed: monitor blockage and, if persistent, move to alternate plan or escalate curb enforcement.

### 4.4 Micromobility corrals and crossings
If scooters/bikes are prevalent:
- Provide designated **micromobility corrals** to avoid ad-hoc parking in sight lines and curb ramps.
- Consider specialized crossing expectations at multi-use trails that intersect event flows.

### 4.5 Emergency access
- Identify **emergency lanes/routes** to and from the venue and hospitals.
- Ensure egress plans **never block or starve** these movements.
- Explicit coordination with incident command: how event plans and emergency response override or interact.

---

## 5) Spatial boundary: core/buffer/diversion rings

### 5.1 Required visual: control perimeter rings diagram

This ASCII map is the **required visual** for the control perimeter concept.

```text
                  [ OUTER / DIVERSION RING ]
    (corridors for rerouting through-traffic; cut-through monitoring)

             +---------------------------------------------+
             |                                             |
             |   [ BUFFER RING ]                          |
             |   (key approaches & exits feeding venue)   |
             |       o  o  o  signalized intersections    |
             |                                             |
             |        +-----------------------------+      |
             |        |      [ CORE RING ]          |      |
             |        |   streets immediately       |      |
             |        |   surrounding venue, gates  |      |
             |        |   + ped plazas & garages    |      |
             |        +-----------------------------+      |
             |                                             |
             +---------------------------------------------+

Legend:
- CORE: intersections directly adjacent to venue, gates, parking garage exits.
- BUFFER: next layer of intersections and ramps feeding the core.
- DIVERSION: outer ring used for reroutes; monitor for spillback and cut-through.
```

### 5.2 Monitored boundary links
At each ring boundary:
- Identify **boundary links** where spillback/cut-through is unacceptable:
  - Core→Buffer: prevent queues from blocking gates, crosswalks, or bus stops.
  - Buffer→Diversion: prevent event queues from overwhelming neighborhood or arterial streets.

Monitor:
- Queue length vs storage.
- Percent time “near spillback” at critical links.
- Volume/cut-through spikes on neighborhood streets (if data allows).

### 5.3 Escalation actions
If boundary KPIs exceed thresholds:
- **Within core**: reduce inflow from buffer, lengthen ped phases, consider closing certain vehicular turns with field control.
- **Buffer to diversion**: activate diversion signs/plans; raise priority for through traffic on alternate routes.
- If neighborhood cut-through spikes: coordinate with enforcement and consider further restrictions or speed management on residential streets.

---

## 6) Trigger fusion design (activation & deactivation)

### 6.1 Trigger types
Use layered triggers to avoid premature/late activation:
- **Scheduled**: calendar/venue schedule with anticipated ingress/egress windows.
- **Declared**: explicit go/no-go from event command (e.g., “Game is expected to end around 21:50”).
- **Detected**: data-based indicators (vehicle and ped):
  - detector occupancy and queues on key links,
  - travel-time increases,
  - ped counts/queue proxies near gates.
- **Operator-confirmed**: TMC operator validates evidence, checks field reports, and triggers activation.

### 6.2 Trigger fusion logic (confidence, persistence, hysteresis)

Principles:
- Require **agreement between scheduled/declared + detected** where possible.
- Use **persistence** and **hysteresis** to prevent flapping.

Example activation rule for egress plan:
- Scheduled egress window: 21:45–23:15.
- Declared “game in final quarter” signal: true.
- Detected: outbound link A occupancy ≥ 0.8 and queue near spillback for ≥ 5 minutes OR ped volume at main gate above threshold for ≥ 3 cycles.
- Operator confirms that crowd release has started.

Deactivation rule (for recovery):
- Detected: queues on core links back below threshold for ≥ 10–15 minutes.
- Ped volumes near gates back to “normal” band.
- Travel times within X% of baseline TOD.
- Minimum **dwell time** in egress plan met.

### 6.3 Sensor failure handling
If key sensors fail (vehicle or ped):
- **Do not assume safety** improved.
- Enter **conservative event mode** using worst-case assumptions until:
  - manual field check confirms conditions, or
  - sufficient alternative sensing (CCTV, probe data) is available.
- Log missingness and decisions for AAR.

### 6.4 Trigger specification template (copy/paste)

```text
Trigger ID: EVT_STADIUM_EGRESS_V1

Scope:
- Venue: Stadium X
- Rings: CORE + BUFFER
- Affected plans: EGRESS_PLAN_X, PED_SURGE_PLAN_X

Inputs:
- Scheduled window: [21:30, 23:30]
- Declared signal: "game_in_final_phase" (from venue/command)
- Detected:
  - core_outbound_occ >= 0.80 on links {L1, L2}
  - ped_gate_volume >= P_thresh at gates {G1, G2}
- Operator confirmation: required (TMC supervisor)

Activation logic:
- IF time ∈ window AND declared==true AND
   ((core_outbound_occ >= 0.80 for ≥ 5 min) OR (ped_gate_volume >= P_thresh for ≥ 3 cycles))
  AND operator_confirms
  THEN ACTIVATE EGRESS_PLAN_X + PED_SURGE_PLAN_X

Deactivation logic (recovery start):
- IF time ≥ 30 min after activation AND
   core_outbound_occ < 0.60 for ≥ 10 min AND
   ped_gate_volume back to baseline band AND
   travel_time_corridor <= 1.3× baseline
  THEN START RECOVERY_PLAN_X

Fallbacks:
- IF key detectors missing > M minutes AND CCTV confirms egress underway:
    - allow manual activation with reason code "sensor_miss".

Logging:
- timestamps, inputs, activation reason, operator ID, plan IDs.
```

---

## 7) Robust return-to-normal (recovery)

### 7.1 Dissipation detection
Recovery should not start until:
- Queues at **core and buffer** have receded below thresholds.
- Ped flows at gates have dropped to near-normal levels.
- Transit headways and travel times have stabilized.

Use multiple signals (vehicles + peds + transit) to decide.

### 7.2 Staged unwind
Avoid going directly from high-bias egress plans to normal TOD plans.

Example sequence:
1. **Phase 1 – EGRESS**: strong outbound bias + strong ped protection.
2. **Phase 2 – SOFT EGRESS**: reduce outbound bias; reintroduce more balanced cross-street service; relax some turn restrictions.
3. **Phase 3 – RECOVERY**: shorten cycles; return offsets toward TOD; maintain ped protections where still needed.
4. **Phase 4 – NORMAL**: return to TOD plan once KPIs and conditions stable.

### 7.3 Anti-flap and dwell
- Minimum dwell times in each phase (e.g., ≥ 20–30 min) unless **safety** requires earlier change.
- Hysteresis: deactivation thresholds stricter than activation thresholds.
- Flip-flop guard: if multiple transitions happen in quick succession, revert to SAFE_RECOVERY and require operator review.

### 7.4 Rollback contract and edge cases

Rollback contract:
- Every event plan must have:
  - a **named safe fallback plan** (e.g., base TOD or conservative recovery),
  - a rollback command path (central or field),
  - confirmation/verification steps.

Edge cases:
- **Overtime / extended event**: keep egress plans longer but re-check staffing and ped caps.
- **Incident during egress**: may require switching to incident playbook; pause recovery.
- **Sudden weather (e.g., heavy rain)**: optionally flip into combined **RAIN_SAFE + EGRESS** with more conservative peds and speeds (see weather/safety ideas).

---

## 8) Governance: pre-approval, versioning, audits, training, AAR

### 8.1 Pre-approval workflow and sign-offs
Event plan changes must follow a **Plan-Approval Runbook** (see below):
- Draft by Traffic Engineering (timing + network schematic).
- Safety/accessibility review (ped and VRU constraints, max ped waits).
- Transit and curb review (bus reliability, loading zones, geofences).
- Emergency services review (access and staging).
- Final sign-off from signal program owner and, if relevant, Vision Zero/safety lead.

### 8.2 Versioning and revalidation triggers
Track versions of:
- Plan library (INGRESS/EGRESS/PED_SURGE/DIVERSION/RECOVERY).
- Trigger specifications.
- Curb and closure assumptions.

Revalidate when:
- Nearby geometry or operations change (e.g., new garage, lane changes, new bus routes).
- Curb regulations or loading zones change.
- Repeat AARs show systematic issues (e.g., new ped hotspots, recurring spillback).

### 8.3 Training and drills
- Tabletop drills before each season (simulate event flows, decisions, communication).
- At least one **live drill** for high-priority venues each year (under lower-stakes events).
- Training for operators on:
  - plan library and triggers,
  - what they can override and how,
  - how to log exceptions and issues.

### 8.4 AAR template (expanded)

```text
Planned Special Event AAR — Venue X, Date Y

1) Event summary
- Type, attendance (est.), weather, unusual factors
- Plans used (IDs + versions) for ingress, egress, recovery

2) Operational performance
- Clearance time vs expected
- Max queue lengths at core and buffer intersections
- Spillback events (where, how often)
- Transit KPIs (headways, travel time)

3) Pedestrian and crowd safety
- Ped max waits at gate crossings vs caps
- Observed conflicts or complaints (turn vs ped, blocked crosswalks)
- Any crowd management issues (queuing, overflow)

4) Curb and multimodal
- Pickup/dropoff behavior (double-parking, blockages)
- Bus/shuttle loading performance
- Micromobility issues (parking, conflict points)

5) Triggering and recovery
- When did we arm / activate / recover?
- Did triggers fire at the right times (too early/late)?
- Any flapping or confusing transitions?

6) Exceptions and manual interventions
- Manual overrides (who, why, impact)
- Deviations from playbook

7) Recommendations
- Plan library changes
- Trigger threshold changes
- Curb / geometry / enforcement coordination items
- Training / staffing changes

8) Approvals and follow-up
- Owners for each change
- Target completion dates
```

---

## 9) Required visual: Event phase → objectives → plan variants → constraints → KPIs

This is the **required table** connecting event phases to objectives, plan variants, constraints, and KPIs.

```markdown
| Event phase | Primary objectives | Plan variants (examples) | Key constraints | KPIs to monitor |
|---|---|---|---|---|
| Pre-event / ingress | Get people to venue without early gridlock; protect transit and ped routes | INGRESS_MAIN (mainline toward venue); INGRESS_TRANSIT (extra bus lanes); INGRESS_PEAK (higher cycle length for peak) | Maintain ped minimums; preserve transit access routes; no spillback onto freeway ramps | Approach travel times to venue; queue length at key ramps; ped wait near gates; bus on-time performance |
| Egress / peak release | Clear venue-adjacent storage; protect ped surges at gates; avoid spillback into neighborhoods | EGRESS_STRONG (outbound bias + ped surge protection); EGRESS_BALANCED (moderate bias); EGRESS_WET (with rain-safe speeds and more ped time) | Max ped wait caps at gate crossings; protected-only or restricted turns where needed; emergency and transit access preserved; boundary queues below storage | Time to clear parking garages; max queue lengths at core/buffer; ped delay at gates; number of spillback events; transit headway adherence |
| Post-event / recovery | Return to normal without creating shock waves; restore fairness to side streets | RECOVERY_SHORT (shorter cycles); RECOVERY_STAGED (offset and split ramping); NORMAL_TOD (final state) | Dwell times between transitions; no sudden ped service degradation; maintain acceptable bus performance | Time to return to baseline travel times; plan transition count; ped delay distribution returning to normal; complaints/observations |
```

---

## 10) Implementation Checklist

- [ ] Identify top 1–3 priority venues/event districts.
- [ ] Build event inventory + contact list + authority matrix.
- [ ] Map event network: ingress/egress routes, parking, transit hubs, ped gates, curb/loading zones.
- [ ] Define ped/crowd safety constraints (max ped waits, protected crossings, no-turn zones).
- [ ] Build initial plan library: INGRESS/EGRESS/PED_SURGE/DIVERSION/RECOVERY.
- [ ] Define transit, TSP, curb, micromobility, and emergency access policies per event network.
- [ ] Design and document rings (CORE/BUFFER/DIVERSION) and boundary links with KPIs.
- [ ] Define trigger fusion logic (scheduled/declared/detected/operator-confirmed) and guardrails.
- [ ] Implement trigger specs in config (using the template) and integrate with monitoring.
- [ ] Validate plans in a twin / scenario scoring environment (including curb and transit scenarios).
- [ ] Run pilot events in assisted mode; capture full logs.
- [ ] Run AAR after each pilot; update plans/triggers/curb strategies.
- [ ] Establish governance for versioning, approvals, revalidation, and training.

---

## 11) Event Operations SOP

### Pre-event (T-7d to T-1d)
- Confirm event time window, expected attendance, and street/curb closures.
- Confirm transit added service and shuttle staging.
- Confirm pedestrian routing + staffed crossings plan (who controls which crossings, when).
- Confirm curb pickup/dropoff plan + any TNC geofencing requests and micromobility corrals.
- Pre-stage monitoring views: critical intersections, crossings, travel time links, boundary links.
- Load/verify plan IDs in controller/central system; verify rollback availability.
- Brief TMC operators, field staff, and liaisons on plan variants and triggers.

### Day-of (T-2h to T+2h)
- Arm event window (calendar + schedule triggers).
- Monitor ingress conditions; activate ingress plan if needed.
- As end time nears, monitor:
  - outbound queues at core garages/ramps,
  - ped queues and flows at gates,
  - transit and curb conditions.
- When trigger conditions met and operator confirms, **activate egress/ped-surge plans**; log plan IDs and reasons.
- During egress:
  - monitor spillback at core/buffer intersections,
  - monitor ped queues and conflicts (especially turns vs peds),
  - monitor transit headways and curb blockages,
  - intervene first via curb/field control, then plan changes as defined.

### Recovery
- When dissipation criteria met, start RECOVERY plan (Phases 2–4 above).
- Confirm stabilization (queues dissipating, speeds improving, ped delay normalizing).
- Return to normal TOD plan.
- Close event log and schedule AAR.

---

## 12) Plan-Approval Runbook (Governance)

### 12.1 Artifacts under change control
- Plan library (timing sheets, controller databases, offsets by ring).
- Trigger and threshold specs.
- Event inventory and contact list.
- Curb/closure assumptions (traffic control plan dependencies).

### 12.2 Approval workflow (minimum)
- Draft (Traffic Engineering).
- Safety/accessibility review (ped and VRU constraints, max ped waits).
- Ops review (usability, staffing, and monitoring feasibility).
- Transit and curb review (headways, loading zones, geofences).
- Emergency services sign-off (where applicable).
- Publish (versioned release + rollback plan; update runbooks and training docs).

### 12.3 Review cadence
- After every pilot event and after any major event.
- Quarterly for top venues.
- Immediately after geometry/construction/curb regulation changes affecting the event network.

---

## 13) Reference links (sources used)

Keep sources to 5–6 total.

1. FHWA, *Managing Travel for Planned Special Events*, Pedestrian Access Plan section: https://ops.fhwa.dot.gov/publications/fhwaop04010/chapter6_05.htm
2. NCHRP Report 812, *Signal Timing Manual, Second Edition* (TRB, 2015): https://onlinepubs.trb.org/onlinepubs/nchrp/nchrp_rpt_812.pdf
3. NOCoE, *Best Practices: Planned Special Event Management* (2025) (PDF): https://transportationops.org/system/files/uploaded_files/2025-03/NOCOE%20Best%20Practices%20Report%20-%20Planned%20Special%20Event%20Management.pdf
4. APTA-SS-SEM-S-003-08 Rev.1, *Security and Emergency Management Aspects of Special Event Service* (2013): https://www.apta.com/wp-content/uploads/Standards_Documents/APTA-SS-SEM-S-003-08.pdf
5. SFMTA, *Curb Management Strategy* (2020) (PDF): https://www.sfmta.com/sites/default/files/reports-and-documents/2020/02/curb_management_strategy_report.pdf

---

## 14) Completion Checklist (✅/⚠️)

- ✅ Pedestrian/crowd safety as first-class (egress variants, max ped waits, turn-conflict mitigations, ped KPIs): see **Section 3**.
- ✅ Multimodal + curb integration (transit surges/detours, TSP policy, rideshare/loading geofences, micromobility, emergency access): see **Section 4**.
- ✅ Spatial boundary with control perimeter rings + monitored boundary links, and escalation actions: see **Section 5** including the **required perimeter diagram**.
- ✅ Trigger fusion design (scheduled/declared/detected/operator-confirmed; confidence/persistence/hysteresis; sensor failure handling; trigger spec template): see **Section 6**.
- ✅ Robust return-to-normal: dissipation detection, staged unwind, min dwell/anti-flap, rollback contract, edge cases: see **Section 7**.
- ✅ Governance: pre-approval workflow/sign-offs, versioning/revalidation triggers, audit logs, training/drills, expanded AAR template, exceptions: see **Sections 8 and 12**.
- ✅ Required **Event phase → objectives → plan variants → constraints → KPIs** table: see **Section 9**.
- ✅ End sections present: **Implementation Checklist** (Section 10), **Event Operations SOP** (Section 11), **Plan-Approval Runbook** (Section 12), **Reference Links** (Section 13), **Completion Checklist** (this section).

---

Cross-links: Related ideas include rare-event practice mode, safety-first signals, delay-tolerant smart signals, and anti-jam nudges.
