# 18) Event-Aware Timing: Concert/Stadium/Festival Mode

## Catchy explanation
The city already knows the stadium empties at 22:30. Event-aware timing is the signal system’s ability to switch into “event mode” on purpose—before the surge turns into gridlock.

## What it is (precise)
**Event-aware timing** is an operations + engineering capability to detect predictable demand surges (concerts, stadium events, festivals, permitted street closures, planned road work), then **activate special signal timing plans** that:

- bias service inbound vs outbound (or along signed diversion routes),
- maintain coordination where it matters (and break it where it helps),
- protect pedestrian crossings during crowd surges,
- prevent spillback into upstream intersections,
- and transition back to normal operation via a controlled recovery.

Activation can be calendar-based, threshold-based, or operator-assisted. A “twin” (simulation or faster what-if scoring) ranks candidate event plans, estimates clearance time, and tests recovery steps.

## Why it matters (outcomes)
- **Faster clearance**: reduces post-event congestion and gridlock.
- **Safer crowd movement**: predictable crossing opportunities and fewer ped/vehicle conflicts during surges.
- **Less improvisation**: repeatable playbooks reduce operator stress and late-night heroics.
- **Better coordination**: integrates traffic engineering, TMC ops, venue operators, and public safety.

## Where it works best / where it can fail
### Works best when
- the “event network” is well defined (venue ↔ parking ↔ freeway ramps ↔ transit hubs),
- the agency has a **plan library** and documented authority to activate,
- pedestrian/crowd routes are treated as first-class constraints,
- field control (police/officers, barricades, curb closures) aligns with the signal plan.

### Common failure modes
- **Surge variability**: attendance, weather, or staggered departures shift peaks.
- **Staffing gaps**: off-hours monitoring/response is missing.
- **Misaligned field control**: manual traffic control conflicts with signal phasing.
- **Bad transitions**: switching back to normal too abruptly creates “shock waves.”
- **Curb chaos**: unmanaged pickup/dropoff blocks lanes and breaks progression.

## Minimum viable product (MVP)
- One venue corridor (or event district).
- Two plans: **Outbound Clearance** + **Recovery**.
- Assisted activation (operator confirms).
- Lightweight monitoring dashboard: queues, splits/cycle, ped service status, transit headways.

---

# Implementation-ready onboarding
This section is written as a practical “how we do this here” guide for an agency building event-aware timing.

## 0) Operating definition and scope
**Event-aware timing scope** (for this doc):

- planned special events (PSEs): sports, concerts, festivals, conventions, parades, demonstrations (permitted), etc.
- planned disruptions: permitted work zones and major planned lane/road closures.

**Not in scope** (handled by other playbooks):
- no-notice incidents (crashes, downed lines),
- storm evacuation/emergency incident response.

Reference context:
- FHWA treats planned special events as a known-in-advance class of “incidents” that change demand and sometimes reduce capacity. See FHWA’s PSE handbook pages on traffic management plan elements such as pedestrian access planning and traffic control planning.

## 1) Core concepts (shared vocabulary)
### Plan library
A **plan library** is a versioned set of signal timing plans and associated “when/how to use” notes.

Typical event library components:
- **Ingress plan** (pre-event arrival bias)
- **Egress plan** (post-event outbound bias)
- **Diversion plan** (if certain ramps/streets are closed)
- **Ped surge plan** (if crossings must meter vehicles to protect crowds)
- **Recovery plan** (stepwise return to normal coordination)

NCHRP’s Signal Timing Manual frames time-based operation and special-condition plans as structured timing plan sets that must be implemented, maintained, and monitored as part of agency practice.

### Activation and transitions
Event timing is not just a different set of splits/cycle/offset; it also includes:
- **when to switch**,
- **how to transition** without breaking progression catastrophically,
- and **how to recover**.

### Multimodal constraints (ped + transit + curb)
Event corridors are multimodal by default:
- pedestrians surge and require safe crossings/queuing,
- transit often adds special service and faces crowd surges,
- curb usage shifts (pickup/dropoff, shuttles, deliveries).

FHWA’s PSE guidance emphasizes a **pedestrian access plan** near venues, including separating pedestrian flows from vehicles, staffed crossings/closures, and ensuring pedestrian routes and crossings have adequate capacity.

## 2) Roles and responsibilities (RACI)
Create a short “who can do what” matrix so event mode never depends on one person.

Minimum roles:
- **Traffic Engineering Lead (Signal Timing Owner)**: owns plan design, approvals, versioning.
- **TMC/Operations Supervisor (Activation Authority)**: day-of decision, monitoring, rollback.
- **Field Command / Law Enforcement Liaison**: coordinates staffed crossings, closures, and officer control.
- **Venue / City Events Liaison**: calendar, gate times, parking operations, anticipated surges.
- **Transit Agency Liaison**: added service, crowd surges, shuttle staging.
- **Curb/Right-of-Way Coordinator**: pickup/dropoff zones, loading relocation, temporary permits.
- **Data/ATSPM Analyst**: KPIs, after-action review, tuning recommendations.

## 3) Data inputs and systems
### Event and disruption inventory
Maintain a **single source of truth**:
- event name, venue, date/time, expected attendance
- ingress/egress “push” windows (start, peak, tail)
- street closures / permit footprint
- parking and shuttle locations
- transit service changes
- contact list (venue ops, police, transit, DOT)

### Traffic/ped/transit sensing
Use whatever you have, but document coverage gaps:
- detector volumes/occupancies on critical approaches
- travel times (probe data, Bluetooth, etc.)
- queue/spillback indicators
- pedestrian volumes if available (CCTV counts, sensors)

FHWA’s PSE handbook explicitly calls out using monitoring assets like CCTV to observe pedestrian operations at crossings and venue egress points.

### Curb and pickup/dropoff data (if available)
Even without perfect curb data, you need at minimum:
- where rideshare can load,
- where buses/shuttles stage,
- which curb lanes are being repurposed for pedestrians.

SFMTA’s curb strategy describes how curb regulations and loading zones interact with safety, transit reliability, and special events; it also describes tactics such as geofencing for TNC pickup/dropoff and ensuring sufficient loading during special events.

## 4) Plan design patterns (engineering templates)
### 4.1 Ingress (arrival bias)
Goals:
- keep mainline progression toward venue,
- avoid blocking cross streets needed for buses/emergency access,
- preserve minimum ped service.

Common tactics:
- adjust split distribution toward inbound approaches
- keep offsets aligned on main corridors
- protect critical side-street storage (spillback guard)

### 4.2 Egress (departure clearance)
Goals:
- clear venue-adjacent storage and key outbound movements,
- prevent spillback that locks intersections,
- coordinate with staffed crossings or temporary closures.

Common tactics:
- longer cycle with dominant outbound green bands (only where queues justify)
- flush plans near parking garage exits
- gating / metering upstream to avoid downstream blockage

### 4.3 Ped surge protection
FHWA’s PSE pedestrian access planning guidance highlights that heavy pedestrian flow near venue gates can overwhelm existing facilities, requiring temporary walkways, staffed crossings, mid-block crossings, or temporary closures to manage conflicts.

Signal-timing implications:
- ensure pedestrian **minimums and clearance** are maintained
- consider ped scramble / exclusive ped phases where warranted
- coordinate with physical barriers and officer control to keep pedestrian routes from intersecting dangerously

### 4.4 Transit and shuttle accommodation
Transit agencies are pushed into special operations for events.

From APTA’s special event standard: transit agencies should have systematic pre-planning, maintain event records, coordinate with event command posts/ICS, and continuously assess conditions during the event.

Signal-timing implications:
- define if TSP is enabled/disabled during certain windows
- protect shuttle movements and bus-only access roads if used
- coordinate with temporary bus stations and loading areas

### 4.5 Curb/ridehail/loading strategy
Curbside failures can negate signal timing gains.

From SFMTA’s curb strategy:
- consider temporary relocation/replacement of loading zones during special events,
- create/standardize procedures for TNC geofencing requests,
- recognize that unmanaged loading increases double-parking and blocks transit.

Signal-timing implications:
- treat “pickup/dropoff lane blocked” as a scenario with its own response (diversion, temporary phase changes, or officer dispatch)
- pre-define fallback offsets if curb blockage destroys progression

## 5) Triggering and deactivation logic
### 5.1 Trigger types
Use layered triggers (calendar + confirmation):
- **Calendar trigger**: event time window “arming.”
- **Traffic confirmation**: occupancy, travel time, queue length.
- **Ped/crowd confirmation**: CCTV observations, crowd marshals.
- **Operator confirmation**: final “go” to activate.

Why layered triggers: calendar-only activation is frequently early/late; confirmation reduces false positives.

### 5.2 Guardrails
- maintain pedestrian minimums/clearance at all times
- rate-limit plan switching (no flapping)
- require a rollback plan and event log
- define maximum duration and “stop conditions”

### 5.3 Recovery
Recovery is an engineered process, not an afterthought:
- step down cycle length / return offsets gradually
- monitor queue dissipation
- confirm ped service returns to normal constraints

NCHRP’s Signal Timing Manual emphasizes implementation/maintenance and monitoring as part of signal timing practice; for special conditions, agencies need procedures to implement and return to normal operation.

## 6) Validation (“twin”) and acceptance gates
### 6.1 Scenario set
Validate across:
- high/medium/low attendance
- fast vs slow departure (weather-dependent)
- concurrent roadway restrictions
- transit surge scenario
- curb blockage scenario

### 6.2 KPIs (pick a small set)
- clearance time to return to near-normal speeds
- maximum queue length / spillback events
- ped delay and crowding proxies at key crossings
- bus travel time reliability / headway adherence
- number of plan switches and manual interventions

### 6.3 Acceptance gate (before production)
A plan is production-ready only if:
- it meets KPI targets across scenarios,
- it includes a recovery plan,
- it has a documented “when to use” and “when to abort,”
- police/field control assumptions are documented.

## 7) After-action review (AAR) loop
Do not skip this—AAR is how you turn one-off heroics into a program.

AAR agenda:
- what we expected vs what happened
- any near-misses (ped safety, blocked crosswalks)
- plan performance (KPIs)
- staffing and comms issues
- updates: plan library, triggers, curb/tc plan, training

NOCoE’s PSE best practices describe a planning cycle that includes initial planning, feasibility study, traffic management plan, implementation, day-of operations, and post-event activities including debriefing and documenting measures of effectiveness.

---

# Practical templates (copy/paste)

## Template A: Event plan one-pager
**Event**: 

**Venue / location**: 

**Date / time**: 

**Expected attendance**: 

**Primary corridors**:
- 

**Special constraints**:
- Ped surge crossings:
- Transit shuttle routes:
- Closures / detours:
- Curb pickup/dropoff:

**Activation**
- Arm time:
- Confirm triggers:
- Operator authority:

**Plans**
- Ingress plan ID:
- Egress plan ID:
- Recovery plan ID:

**Abort / rollback**
- Conditions:
- Steps:

**KPIs to watch**
- 

## Template B: Day-of decision log (minimum)
- timestamp
- plan activated (ID + version)
- reason (calendar/traffic/ped)
- field conditions (closures, officer control)
- notable issues (spillback, curb blockage)
- interventions (manual overrides)
- time of recovery start/end

---

# Implementation Checklist
- [ ] Identify top 1–3 priority venues/event districts.
- [ ] Build event inventory + contact list + authority matrix.
- [ ] Map event network: ingress/egress routes, parking, transit hubs, ped gates.
- [ ] Define KPIs and minimum safety constraints (especially pedestrian).
- [ ] Build initial plan library: ingress/egress/recovery.
- [ ] Define curb and pickup/dropoff strategy for each event network.
- [ ] Define trigger logic (calendar + confirmation) and guardrails.
- [ ] Validate in twin / scenario scoring.
- [ ] Run pilot events in assisted mode.
- [ ] Run AAR after each pilot; update plans/triggers/checklists.
- [ ] Establish governance for versioning, approvals, and periodic refresh.

---

# Event Ops Runbook
## Pre-event (T-7d to T-1d)
- Confirm event time window, expected attendance, and any street closures.
- Confirm transit added service and shuttle staging.
- Confirm pedestrian routing + staffed crossings plan.
- Confirm curb pickup/dropoff plan + any TNC geofencing requests.
- Pre-stage monitoring views: critical intersections, crossings, travel time links.
- Load/verify plan IDs in controller/central system; verify rollback availability.

## Day-of (T-2h to T+2h)
- Arm event window (calendar).
- Confirm triggers (traffic + ped/crowd indicators).
- Activate plan (record in log).
- Monitor:
  - spillback at critical intersections
  - pedestrian queues/crossing compliance
  - transit performance (if applicable)
  - curb blockage and illegal stopping near venue
- Intervene only via defined levers (priority order):
  1) dispatch field control / adjust curb operations,
  2) apply alternate timing plan,
  3) targeted manual override (document).

## Recovery (when stop conditions met)
- Start recovery plan.
- Confirm stabilization (queues dissipating, travel times improving).
- Return to normal TOD plan.
- Close log and schedule AAR.

---

# Governance / Plan-Approval Runbook
## Artifacts under change control
- plan library (timing sheets, controller databases, offsets)
- triggers and thresholds
- event inventory and contact list
- curb/closure assumptions (traffic control plan dependencies)

## Approval workflow (minimum)
- Draft (Traffic Engineering)
- Safety/accessibility review (ped constraints)
- Ops review (usability + on-call feasibility)
- Stakeholder sign-off (field command + transit + venue liaison as needed)
- Publish (versioned release + rollback plan)

## Review cadence
- after every pilot event
- quarterly for top venues
- immediately after geometry/construction/curb regulation changes

---

# Reference links (sources used)
Keep sources to 5–6 total.

1. FHWA, *Managing Travel for Planned Special Events*, Pedestrian Access Plan section (Chapter 6, p. 5 of 9): https://ops.fhwa.dot.gov/publications/fhwaop04010/chapter6_05.htm
2. NCHRP Report 812, *Signal Timing Manual, Second Edition* (TRB, 2015): https://onlinepubs.trb.org/onlinepubs/nchrp/nchrp_rpt_812.pdf
3. NOCoE, *Best Practices: Planned Special Event Management* (2025) (PDF): https://transportationops.org/system/files/uploaded_files/2025-03/NOCOE%20Best%20Practices%20Report%20-%20Planned%20Special%20Event%20Management.pdf
4. APTA-SS-SEM-S-003-08 Rev.1, *Security and Emergency Management Aspects of Special Event Service* (2013): https://www.apta.com/wp-content/uploads/Standards_Documents/APTA-SS-SEM-S-003-08.pdf
5. SFMTA, *Curb Management Strategy* (Feb 2020) (PDF): https://www.sfmta.com/sites/default/files/reports-and-documents/2020/02/curb_management_strategy_report.pdf

---

# Completion Checklist (✅/⚠️)
- ✅ Clear definition + scope exists: see [What it is (precise)](ideas/18-event-aware-timing.md:6)
- ✅ MVP is defined: see [Minimum viable product (MVP)](ideas/18-event-aware-timing.md:44)
- ✅ Roles/RACI included: see [Roles and responsibilities (RACI)](ideas/18-event-aware-timing.md:83)
- ✅ Triggering, guardrails, recovery covered: see [Triggering and deactivation logic](ideas/18-event-aware-timing.md:199)
- ✅ Event operations steps exist: see [Event Ops Runbook](ideas/18-event-aware-timing.md:315)
- ✅ Governance/approvals exist: see [Governance / Plan-Approval Runbook](ideas/18-event-aware-timing.md:365)
- ✅ Implementation checklist exists: see [Implementation Checklist](ideas/18-event-aware-timing.md:278)
- ✅ References are provided (≤6): see [Reference links (sources used)](ideas/18-event-aware-timing.md:401)
- ⚠️ Source citations are “section-level” only: convert to line-item inline citations if you require strict academic-style quoting.

---

Cross-links: Related ideas include rare-event practice mode and crowd-smart crossings.
