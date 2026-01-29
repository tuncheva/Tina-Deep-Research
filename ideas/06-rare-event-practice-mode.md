# 06) Rare-Event Practice Mode: Operational Readiness Drills

## Catchy Explanation
Like fire drills for a city’s streets: you rehearse parades, stadium surges, detours, storms, and outages in the digital twin—*before* they happen—so operators already have a safe playbook.

## What it is (precise)
**Rare-event practice mode** is an operational workflow where agencies use a calibrated digital twin to simulate low-frequency/high-impact scenarios (major crashes, road closures, evacuation routes, special events, comms outages, power constraints), pre-select safe timing plans, and document **activation triggers**, **guardrails**, and **recovery steps**. The output is a versioned **playbook library**: scenario definitions + plan sets + checklists + communications templates. During real events, the control room can activate a plan quickly (manual or assisted), monitor KPIs, and roll back safely. After the event, the team runs an after-action review and updates the playbook.

## Benefits
- **Faster, safer response**: reduces improvisation during chaos.
- **Consistency across shifts**: standardized playbooks reduce operator variance.
- **Training and institutional memory**: drills keep skills fresh.
- **Better interagency coordination**: shared scenario definitions improve operations.

## Challenges
- **Scenario drift**: real events differ; infrastructure changes invalidate playbooks.
- **Resource intensity**: drills and plan maintenance take time.
- **Trigger brittleness**: false activations or late activations if observables are weak.
- **Public communication**: mode changes can confuse drivers if not explained.

## Implementation Strategies

### Infrastructure Needs
- **Scenario templates**: closures, detours, inbound/outbound surges, evacuation bias.
- **Plan library + versioning**: approved plan sets with metadata and expiry checks.
- **Observability**: detectors/probes/CCTV + ATSPM metrics for monitoring.
- **Twin rehearsal environment**: repeatable simulations with failure injection.
- **Operator UI**: “activate plan”, “monitor”, “rollback”, “log notes”.

### Detailed Implementation Plan

#### Phase 1: Scenario Inventory and Success Metrics (Weeks 1–3)
The agency should begin by reviewing incident logs, special event calendars, construction history, and weather/outage records to identify the rare scenarios that actually recur and that create high operational risk. The team should select a manageable set of top scenarios (for example, stadium egress, a major bridge closure detour, severe storm mode, and a downtown parade) and should define success metrics for each scenario, such as maximum queue length at critical approaches, clearance time, pedestrian minimum service, and acceptable spillback frequency. The team should also define operational constraints up front (pedestrian clearance, emergency preemption behavior, maximum plan switching frequency) so playbooks do not get designed around unrealistic assumptions.

- **Roles**: operations lead (scenario selection), traffic engineer (metrics and constraints), emergency management liaison (evacuation/incident priorities), maintenance (field feasibility), communications lead (public messaging inputs).
- **Deliverables**: scenario catalog, KPI bundle per scenario, and a list of critical corridors/intersections for each scenario.
- **Risks**: selecting too many scenarios dilutes effort; metrics may be hard to measure with existing data.
- **Acceptance checks**: each scenario has measurable KPIs, an identified “owner,” and clearly documented constraints.

#### Phase 2: Playbook Drafting and Candidate Plan Design (Weeks 4–12)
The traffic engineering team should create a draft playbook for each scenario that describes the operational intent in plain language (for example, “bias outbound movements for 90 minutes after the event and protect pedestrian service at the transit hub”). The team should design a small set of candidate timing plans per scenario (typically 2–4) that include a conservative option, an aggressive clearance option, and an explicit recovery plan that returns the corridor to normal coordination. The team should document activation triggers that are grounded in observable signals, such as occupancy thresholds, probe travel-time spikes, or an external event start time, and it should include a checklist of field actions (deploy portable message signs, coordinate police control, confirm detour signage) when those actions are required for the signal plan to work.

- **Roles**: traffic engineer (plan design), operations (trigger practicality), data analyst (observable definitions), field staff (signage and on-street actions), interagency partners (police/event management).
- **Deliverables**: initial playbook library (versioned), candidate plan sets with metadata, trigger definitions, and operator checklists.
- **Risks**: triggers can be brittle; plans may solve one corridor but push queues to parallel streets.
- **Acceptance checks**: each playbook includes at least one safe rollback path, a clear maximum duration, and explicit recovery steps.

#### Phase 3: Twin Rehearsal and Failure Injection (Weeks 13–20)
The modeling team should rehearse each playbook in the twin using representative demand patterns and should include realistic constraints such as limited lane storage, turning friction, and pedestrian surges near venues. The team should test failure cases that happen during real events, such as partial detector outages, comms degradation, and unexpected road closures, because playbooks must remain usable under degraded conditions. The team should quantify spillover impacts to nearby neighborhoods and should adjust plans or add guardrails if the playbook simply relocates congestion. The team should also produce a short “expected outcomes” summary for operators that describes what KPIs should improve and what tradeoffs to expect.

- **Roles**: modeler (simulation), traffic engineer (interpret results), analyst (KPI computation), operations (operational realism).
- **Deliverables**: simulation validation report per scenario, spillover analysis, and operator-facing outcome summaries.
- **Risks**: the twin may not capture all on-street behaviors; rehearsal may miss rare edge cases.
- **Acceptance checks**: safety invariants remain satisfied in simulation, and the playbook’s expected tradeoffs are documented and approved.

#### Phase 4: Tabletop Exercises and Shadow Deployments (Weeks 21–28)
The agency should run tabletop exercises that walk operators through detection, activation, monitoring, and recovery steps, because procedural failures are more common than technical failures during rare events. The team should then run shadow deployments during smaller real-world events or planned drills, where the system recommends actions and logs outcomes without actuating, because this step reveals data gaps and workflow friction without causing harm. The team should refine trigger thresholds, checklists, and communication templates based on operator feedback and observed conditions.

- **Roles**: operations (tabletop participation), traffic engineering (facilitation), IT/data (tool support), field staff (on-street actions).
- **Deliverables**: tested procedures, refined trigger thresholds, updated playbooks, and training materials.
- **Risks**: tabletop exercises may not reflect field complexity; shadow mode may miss operator stress effects.
- **Acceptance checks**: operators can execute the playbook steps without ambiguity, and triggers fire appropriately on pilot events.

#### Phase 5: Operational Drills and Governance Cadence (Weeks 29–40)
The agency should establish a recurring drill schedule (for example, quarterly) that rotates across shifts so institutional knowledge is not concentrated in a single team. Each drill should include a structured after-action review that captures what worked, what failed, and what data was missing, and it should result in specific updates to plans, triggers, and checklists. The program owner should maintain a governance process that controls who can approve playbook changes and how those changes are communicated, because ad-hoc edits shortly before an event create risk.

- **Roles**: program owner (governance), operations (execution), engineering (plan updates), analyst (after-action metrics), communications lead (public messaging updates).
- **Deliverables**: operational readiness calendar, after-action reports, versioned playbook updates, and updated runbooks.
- **Risks**: drill fatigue; playbooks may become stale due to construction and controller upgrades.
- **Acceptance checks**: playbooks remain versioned and reviewed, and drills produce measurable improvements in activation time and execution quality.

#### Phase 6: Maintenance, Recertification, and Expiry Controls (Ongoing)
The agency should treat playbooks as living assets by adding an expiry date and review requirement to each one, because infrastructure changes frequently invalidate assumptions. The team should recertify playbooks after major construction, controller firmware changes, new transit routes, or policy updates, and it should maintain a "do not use" flag for expired playbooks that forces review before activation. Over time, the team should expand the library cautiously based on demonstrated operational value rather than attempting to cover every possible scenario.

- **Roles**: program owner (expiry enforcement), traffic engineering (recertification), operations (feedback), IT/data (system maintenance).
- **Deliverables**: playbook expiry system, annual recertification reports, and updated scenario coverage plan.
- **Risks**: expired playbooks may still be used under pressure; maintaining too many playbooks increases burden.
- **Acceptance checks**: no expired playbooks are active, and playbook updates are tied to documented after-action findings.

### Choices
- **Manual activation**: operator-triggered plan selection.
- **Assisted activation**: system recommends plan + operator confirms.
- **Auto-activation**: only for well-instrumented scenarios with strict guardrails.

## Technical Mechanics

### Key Parameters
- Trigger thresholds (volume, occupancy, travel time spikes)
- Plan duration and recovery rules
- Safety constraints (ped mins/clearance)

### Guardrails
- Clear rollback path and time limits.
- Keep pedestrian service minimums and clearance behavior intact.
- Rate-limit plan switching.
- Require logging of activation reason and outcome.

## Implementation Additions (Implementation-Ready)

### 1) Trigger Design Framework (robust, observable, anti-false-activation)

Rare events are chaotic and sensors are imperfect; triggers must be designed to be *observable*, *robust to noise*, and *hard to false-activate*.

#### Trigger types
- **Scheduled triggers (known events)**: event calendar/permit start–end time, stadium egress window, parade route schedule.
- **Detected triggers (incident patterns)**: probe travel-time spike, detector occupancy/volume pattern, queue spillback proxies, sudden bus bunching.
- **Declared triggers (partner notification)**: police/Fire/EMS/emergency management declares incident, detour, evacuation, venue operations calls "release".
- **Operator-confirmed triggers**: any automated recommendation remains "pending" until an operator confirms.

#### Decision tree: activate → operate → unwind (required visual)

```text
[IDLE]
  ↓  (trigger observed: scheduled / detected / declared)
[PENDING]
  - System computes confidence and checks persistence
  - Operator reviews evidence + scope
    ↓ if confidence < threshold OR sensors unhealthy
    [REJECT]
      - Log reason (data gap / false trigger / out of scope)
      - Return to [IDLE]
    ↓ if confidence ≥ threshold AND operator confirms
[ACTIVATE]
  - Select playbook + plan option (conservative/agg)
  - Notify partners; execute field actions
    ↓
[OPERATE]
  - Monitor KPIs + safety constraints
  - Adhere to rate limits; adjust only within playbook bounds
    ↓ if exit criteria met (KPIs back to normal / time window over)
[UNWIND]
  - Execute recovery plan (return to normal coord / post-event mode)
  - Confirm ped mins and preemption behavior
  - Notify partners of mode change
    ↓
[AFTER-ACTION]
  - Capture metrics + timeline
  - Run AAR; propose playbook updates
    ↓
[IDLE]
```

This decision tree is the core **activate → operate → unwind** flow that all rare-event playbooks must follow.

#### Robust trigger design rules
Use a state-machine with confidence and persistence:

- **Multi-signal confirmation** (avoid single-sensor activation):
  - Example: *probe travel-time spike* + *downstream occupancy rising* + *controller coordination state indicates plan X is eligible*.
- **Confidence scoring**: compute `confidence ∈ [0,1]` from weighted evidence; require a threshold to move states.
- **Minimum persistence**: condition must hold for `T_persist` (e.g., 3–5 min) before activation.
- **Hysteresis**: activation threshold `θ_on` > deactivation threshold `θ_off` to avoid oscillation.
- **Debounce / rate limiting**: no more than 1 mode change per corridor per `T_debounce` (e.g., 10–15 min), plus global "circuit breaker".
- **Thresholds vs anomaly detection**:
  - Use **thresholds** where baseline is stable and operational intent is clear (e.g., occupancy > 25% for 5 min at a known bottleneck).
  - Use **anomaly detection** where day-to-day variance is large or you want *earlier warning* (e.g., travel-time z-score vs typical day-of-week/time-of-day). Require stricter operator confirmation for anomaly triggers.

#### False activation handling
- **Pending state**: all detected triggers first enter `PENDING` and require operator confirmation unless the scenario is explicitly approved for auto-activation.
- **Auto-expire**: pending recommendations expire after `T_pending_max` (e.g., 10 min) unless confirmed.
- **Rollback contract**: every activation must have explicit exit criteria and an "undo" plan; enforce a max duration (`T_max`) after which the system re-prompts for renew/rollback.
- **Logging + postmortem**:
  - Log signals, computed confidence, who confirmed, and observed outcome.
  - Track false positives/negatives and tune triggers using after-action review.

#### Trigger specification template (copy/paste)

```yaml
trigger_id: "PSE-EGRESS-01"
scenario: "Stadium egress outbound bias"
trigger_type: [scheduled|detected|declared|operator_confirmed]

activation:
  data_sources:
    - probe_travel_time: provider=X, corridor=..., metric=p95
    - detector_occupancy: intersections=[...], approach=...
    - controller_state: central_system=..., plan_eligibility=...
    - partner_declaration: channel="CAD/ICS/phone"
  logic:
    - rule: "probe_tt_zscore >= 2.0"
    - rule: "occ >= 0.25"
    - combine: "(rule1 AND rule2) OR partner_declaration"
  persistence:
    must_hold_for: "5m"
    debounce: "15m"
    hysteresis:
      on: 0.80
      off: 0.55
  confidence:
    method: "weighted_evidence"
    weights:
      probe: 0.5
      detectors: 0.3
      partner: 0.2

workflow:
  state_machine: [IDLE, PENDING, ACTIVE, STABILIZING, RECOVERY]
  operator_confirmation_required: true
  escalation_path:
    - notify: "TMC supervisor"
    - notify: "Police liaison"
    - notify: "PIO"

activation_footprint:
  geofence:
    corridors: ["A St", "B Ave"]
    intersections: [101,102,103]
  plan_set:
    - plan_id: "EGR-OUT-AGG"
    - plan_id: "EGR-OUT-CONS"

exit_criteria:
  conditions:
    - "probe_tt_zscore < 0.5 for 15m"
    - "queues below spillback threshold"
  max_duration: "120m"
  fallback_if_unknown: "rollback_to_normal_coordination"

rollback:
  rollback_plan_id: "NORMAL-AM"  # or an explicit recovery plan
  rollback_steps:
    - "restore coordination"
    - "verify ped mins enabled"
    - "notify partners and log"
```

#### Trigger spec table + example triggers (required visual)

This table operationalizes the YAML template into a quick-reference **trigger spec chart**.

| Trigger ID | Scenario | Type | Key data sources | Core activation logic | Persistence / hysteresis | Operator role | Example exit criteria |
|---|---|---|---|---|---|---|---|
| PSE-EGRESS-01 | Stadium egress outbound bias | Scheduled + detected | Event schedule; probe TT on outbound corridor; detector occupancy at bottleneck intersections | `(event_window == true) AND (probe_tt_zscore ≥ 2.0) AND (occ ≥ 0.25)` OR partner declaration | `must_hold_for = 5 min`; `debounce = 15 min`; `θ_on = 0.80`, `θ_off = 0.55` | Confirm PENDING; select conservative vs aggressive plan; notify partners | `probe_tt_zscore < 0.5 for 15 min` AND queues below spillback threshold OR hard `T_max = 120 min` |
| DETOUR-MAJOR-01 | Major detour for bridge closure | Declared + detected | CAD/incident feed; lane-closure flags; probe TT on signed detour; local counts on parallel streets | `declared_closure == true` AND (detour_tt_zscore ≥ 1.5 OR closure_flag == true)` | `must_hold_for = 3 min`; `debounce = 20 min`; lower `θ_off` to stay in mode until stable | Confirm activation; verify detour signage; coordinate with police for control points | Lanes restored AND `detour_tt_zscore < 0.5` for 20 min; neighborhood counts under cut-through threshold |
| TRANSIT-OUTAGE-01 | Rail outage → bus bridge detours | Declared + detected | Transit control declaration; AVL/RTPI bus delays; headway adherence | `transit_declared == true AND bus_headway_cv ≥ threshold` | `must_hold_for = 10 min`; `debounce = 30 min`; anomaly-based entry; operator must confirm | Confirm bus detour routes; ensure bus-priority corridor enabled | Rail service restored AND bus headways stable for 30 min OR `T_max = 240 min` |
| EVAC-ROUTE-01 | Evacuation route clearance | Declared | Incident command declaration; optional probe/queue indicators | `evacuation_declared == true` (no auto-detected option) | Immediate activation on declaration; no auto-off; must be explicitly released | Execute evacuation playbook only on direction of incident command; maintain hard constraints | Command releases evacuation; staged rollback per playbook; verification of normal ops before idle |

**Source alignment**: FHWA emphasizes that planned special events benefit from advanced information (location/time/duration/demand) and that advanced planning and coordination enable operational strategies, traffic control plans, protocols, and procedures shared with stakeholders ([`Planned Special Events Preparedness - FHWA`](https://ops.fhwa.dot.gov/tim/preparedness/pse/index.htm:1)).

### 2) Interagency Coordination + Communications (first-class playbook component)

Planned special events and major incidents require agencies and stakeholders who "normally don’t work together" to coordinate before/during/after the event, with shared procedures and real-time information exchange ([`Planned Special Events Preparedness - FHWA`](https://ops.fhwa.dot.gov/tim/preparedness/pse/index.htm:1)).

#### Partner map (typical) and roles

| Partner | What they control | What signals ops needs from them | What they need from signals ops |
|---|---|---|---|
| Police / traffic enforcement | intersections manually controlled, lane closures, cones, on-street direction | control points, planned closures, release/hold of traffic, incident updates | recommended route priorities, timing plan status, bottlenecks |
| Transit agency | detours, stop closures, headway management | detour routes, priority needs, service disruptions | bus-priority corridors, expected delays, signal plan changes |
| Venue / event ops | gates, parking release timing, crowd operations | release schedule, parking utilization, pedestrian surge alerts | ingress/egress route messaging, expected clearance times |
| Emergency management | incident command coordination, evacuations | evacuation orders, staging areas, critical routes | route clearance status, congestion hot spots |
| PIO (public information) | public messaging | message approval, cadence, key safety messages | accurate travel conditions, recommended routes |
| Towing/recovery | incident clearance | estimated clearance times, lane availability | access routes, coordination windows |
| Maintenance / field crews | signs, barricades, ITS devices | device status, ability to deploy | where/when to deploy assets |

#### RACI for activation + comms + field actions

| Task | TMC Operator | TMC Supervisor | Traffic Engineer On-Call | Police Liaison | Transit Liaison | PIO | Field Maintenance |
|---|---|---|---|---|---|---|---|
| Detect / receive trigger | R | A | C | C | C | I | I |
| Confirm activation | R | A | C | C | C | I | I |
| Select plan (from playbook) | R | A | C | C | C | I | I |
| Implement field actions (cones, signs) | I | C | C | R/A | C | I | R |
| Issue partner notifications | R | A | C | R | R | R/A | I |
| Publish public messages (511/social/DMS) | I | C | C | I | I | R/A | I |
| Monitor + adjust (bounded) | R | A | C | C | C | I | C |
| Deactivate + recovery | R | A | C | C | C | I | C |

#### Communication plan (minimum viable)
- **Notification channels** (redundant): CAD/dispatch interface, dedicated incident bridge line, SMS/call-down, email distro, chat channel.
- **Call-down list**: role-based (not person-based); includes alternates per shift.
- **SitRep cadence**: e.g., every 15 min during escalation, then every 30 min once stable; ad-hoc for major changes.
- **Public comms integration**:
  - 511 updates, web banner, social media, and if applicable roadway messaging (CMS/DMS) aligned with on-street control.
- **Data sharing expectations**:
  - *From partners → signals ops*: closures, staging, detours, release times, evacuation boundaries.
  - *From signals ops → partners*: corridor status, plan activation/deactivation timestamps, expected outcomes, constraints (ped mins, emergency routes).

**Source alignment**: WSDOT’s TSMO description of planned event/incident signal timing emphasizes coordination among event managers, traffic agencies, public transportation agencies, and law enforcement, and the use of a "common operating picture" for shared situational awareness ([`Planned event or incident signal timing | TSMO | WSDOT`](https://tsmowa.org/category/intelligent-transportation-systems/planned-event-or-incident-signal-timing:1)).

### 3) Equity + multimodal priority regimes during rare events

Rare-event modes shift priorities. Make the shift explicit, pre-approved, and measurable.

#### Priority regime by event type (policy table)

| Event type | Primary objective | Typical priority order | Notes |
|---|---|---|---|
| Venue egress/ingress | life safety + pedestrian egress + prevent spillback | Ped safety/egress → transit access → emergency routes → general traffic | Protect crossings near venues/transit hubs first, then manage vehicle queues. |
| Incident diversion | keep network stable and avoid secondary crashes | emergency routes → diversion corridors → neighborhood protection | Avoid pushing cut-through into sensitive neighborhoods where feasible. |
| Transit disruption (rail outage) | maintain bus throughput and headways | transit corridors → ped access → emergency routes → general traffic | Pre-define detour signal priority corridors and stop relocations. |
| Evacuation / emergency | life safety + clearance time | evacuation routes → emergency response → ped safety → general traffic | May require contra-flow, staging routes; coordinate with emergency management. |
| Severe weather / degraded conditions | safety + maintain minimum mobility | safety constraints (ped mins, red clearance) → emergency routes → key arterials | Avoid frequent plan switching during unstable comms/power. |

#### Constraints/objectives examples (implementation-ready)
- **Pedestrian protection**: enforce minimum walk intervals and conservative clearance near venues/schools; add "no permissive turn on flash" policies where required.
- **Neighborhood cut-through protection** (when diversion is needed): cap green splits on residential collectors; keep speed/volume within acceptable limits; prefer signed diversion routes.
- **Bus-priority corridors**: grant conditional TSP (transit signal priority) on designated detours; protect headways by limiting max red for bus approaches.
- **Emergency/hospital access**: maintain "never degrade" routes and intersections; treat as hard constraint in playbooks.

#### Metrics (by mode + geography)
- **Person-delay** (not just vehicle delay): estimate by mode share and occupancy assumptions.
- **Pedestrian wait / crossings served**: max wait and compliance proxies.
- **Bus on-time performance (OTP)**: headway adherence and travel-time reliability on detours.
- **Neighborhood cut-through proxies**: counts on local streets, turning movement changes, speed percentiles.
- **Safety proxies**: max queue spillback into intersections, red-light compliance stress proxies, conflict indicators if available.

### 4) Playbook lifecycle governance: version control, validation, retirement

FHWA notes that improving planned special event practice often requires incorporating techniques into day-to-day policies and procedures and committing resources to support recommended activities ([`Planned Special Events Preparedness - FHWA`](https://ops.fhwa.dot.gov/tim/preparedness/pse/index.htm:1)). Treat playbooks as controlled operational artifacts.

#### Versioning scheme (semver-like)
Use `MAJOR.MINOR.PATCH` for each playbook:
- **MAJOR**: changes to intent or safety constraints (requires formal approval + training notice).
- **MINOR**: changes to timing plans/thresholds within approved intent (engineering + ops approval).
- **PATCH**: typo, contact list update, metadata-only changes.

Metadata per playbook:
- `owner` (role), `approvers` (roles), `effective_date`, `expiry_date`, `last_validated`, `dependencies` (geometry, routes, controller firmware), `risk_level` (A/B/C), `auto_activation_allowed` (bool).

#### Approval workflow
1. Draft change (engineer or ops) → PR-style review.
2. Safety/constraints review (traffic engineer + ops supervisor).
3. Partner review if comms/field actions change (police/transit/PIO).
4. Publish + training notice + update call-down lists.

#### Revalidation triggers (when to re-certify)
- Construction/geometry changes, lane repurposing, new detours.
- Signal retiming, coordination changes, controller firmware/system upgrades.
- Transit route/detour changes.
- Seasonal demand shifts or a trend break (events changing size).

#### Validation methods
- **Twin re-simulation**: rerun baseline + scenario with updated network.
- **Tabletop review**: walk through triggers, comms, field actions.
- **Field observation checklist** (during a smaller comparable event): verify detectors, comms latency, queue storage, pedestrian volumes.

#### Retirement policy
- Mark as **deprecated** with reason and replacement.
- Archive immutable snapshots for audit.
- Maintain an audit trail of activations and outcomes; expired playbooks cannot be activated without supervisor override.

#### Maintenance burden planning
- Staff time budget per quarter: validation runs + call-down list refresh + comms template refresh.
- Backlog management: label issues as `safety`, `ops friction`, `data gap`, `partner dependency`.

### 5) Boundary with the "Real-Time What-If Button"

Define operating modes to prevent unsafe improvisation.

- **Static playbook buttons**: pre-approved plan sets + checklists + comms templates. Intended for *fast, safe execution*.
- **Dynamic playbooks**: still bounded and pre-approved *candidate sets*, but selection among candidates is informed by quick evaluation (e.g., simulation/what-if scoring) under constraints.

#### Decision logic (when to use which)

```text
IF life-safety / evacuation / police-directed control active:
  use STATIC playbook only (or manual per incident command)
ELSE IF event is known + rehearsed + sensors degraded:
  use STATIC playbook (conservative option)
ELSE IF event is uncertain BUT within a rehearsed scenario family:
  use DYNAMIC bounded evaluation (choose among pre-approved candidates)
ELSE:
  do NOT improvise new actions; fall back to normal + manual incident management
```

#### Auditability + safety
- **Bounded actions only**: what-if can only choose among a whitelisted plan set.
- **Rollback contracts**: every action has explicit rollback plan and max duration.
- **Operator sign-off**: dynamic recommendations require operator confirmation.
- **Event log**: decision rationale recorded (signals + scoring + human confirmation).

### 6) Drill program design (tabletop → functional → full-scale) with evaluation

FHWA explicitly provides "Tabletop Exercise Instructions for Planned Events and Unplanned Incidents/Emergencies" as a preparedness resource ([`Planned Special Events Preparedness - FHWA`](https://ops.fhwa.dot.gov/tim/preparedness/pse/index.htm:1)). WSDOT also highlights real-time monitoring and after-hours staffing needs for turning plans on/off and adjusting to actual conditions ([`Planned event or incident signal timing | TSMO | WSDOT`](https://tsmowa.org/category/intelligent-transportation-systems/planned-event-or-incident-signal-timing:1)).

#### Drill types and cadence
- **Tabletop (monthly/quarterly)**: comms + decision walkthrough; no system actuation.
- **Functional (quarterly)**: simulation-only run in the twin + control-room workflow rehearsal (activate/monitor/rollback) using recorded or synthetic data.
- **Controlled field drill (1–2x/year)**: limited-scope corridor with agreed constraints (off-peak or coordinated event); may use shadow mode first.
- **Full-scale (as feasible)**: integrated with a real event where a playbook is intentionally exercised and evaluated.

#### Success criteria (example rubric)

| Dimension | Target | Measure |
|---|---|---|
| Time-to-activate | ≤ 10 min from trigger to plan active | timestamps (trigger → confirm → implement) |
| Time-to-stabilize | e.g., 30–45 min | queue metrics, travel-time recovery |
| Comms clarity | no missed notifications; SitRep cadence met | call logs, partner feedback |
| Safety constraint compliance | 0 violations | ped mins enforced, max queue not blocking crossings |
| Rollback correctness | rollback completed within 5 min | audit logs + field confirmation |
| Operator workload | manageable | operator feedback + number of manual interventions |

#### After-Action Review (AAR) template

```markdown
# After-Action Review (AAR) — Rare-Event Practice Mode

## 1. Summary
- Scenario:
- Date/time:
- Playbook ID/version:
- Activation type: scheduled/detected/declared/operator

## 2. Timeline
- T0 trigger observed:
- T1 pending created:
- T2 operator confirmed:
- T3 plan active:
- T4 stabilized:
- T5 recovery started:
- T6 normal restored:

## 3. What worked / what failed
- Triggers:
- Comms:
- Field actions:
- Plan performance:

## 4. Metrics
- person-delay delta:
- clearance time:
- ped wait / served:
- bus OTP/headway:
- neighborhood cut-through proxies:
- safety proxies:

## 5. Decisions + rationale

## 6. Updates required
- Playbook changes (version bump proposal):
- Data gaps:
- Partner process changes:

## 7. Approvals / sign-off
```

## Operations artifacts (templates and tables)

### Playbook template (what every playbook must contain)

1. **Scenario definition** (who/what/where/when)
2. **Objective + constraints** (safety + multimodal priorities)
3. **Activation triggers** (spec + confidence + persistence)
4. **Plan set** (2–4 candidates + conservative default)
5. **Field actions checklist** (signage, cones, staffing)
6. **Communications package** (partner notification + SitRep cadence + public messages)
7. **Monitoring KPIs + expected outcomes**
8. **Exit criteria + recovery plan**
9. **Rollback contract**
10. **Governance metadata** (owner, version, expiry, approvals)

### Monitoring dashboard minimum (operator view)

- Current mode + time since activation + time to expiry.
- Corridor KPIs (queues, travel times) + "safety constraint" indicators.
- Partner status panel (closures, police control points).
- Recommended next step (stay / adjust within bounds / recover / rollback).

## MVP Deployment

- 3 scenarios (e.g., stadium egress, major detour, storm-safe mode).
- Quarterly tabletop drills.
- Assisted activation + rollback.

## Evaluation

- Activation latency (time from detection to plan start).
- Clearance time and spillback frequency.
- Operator adherence to checklist.
- Post-event update rate (how quickly playbooks are refreshed).

---

## Implementation Checklist

- [ ] Identify top 3–6 rare-event scenarios (recurrence × impact) and assign owners.
- [ ] Define per-scenario objectives, constraints, and multimodal priority regime.
- [ ] Build trigger specs with persistence, hysteresis, and operator-confirm "pending" state.
- [ ] Create plan sets (2–4 each) including conservative default and explicit recovery/rollback.
- [ ] Define partner map, RACI, call-down lists, and SitRep cadence.
- [ ] Create public messaging templates (511/social/DMS as applicable).
- [ ] Validate in twin with failure injection and neighborhood spillover checks.
- [ ] Run tabletop + functional drills; publish AARs and track corrective actions.
- [ ] Establish playbook versioning, approval workflow, expiry, and revalidation triggers.
- [ ] Instrument dashboards + logs for auditability (activations, signals used, outcomes).

## Operations Runbook (SOP)

### SOP 0 — Common steps (all event types)

1. **Detect/declare**: receive trigger (scheduled/detected/declared).
2. **Enter PENDING**: system compiles evidence + confidence; operator reviews.
3. **Confirm scope**: verify geofence/corridor and that required devices/comms are healthy.
4. **Notify partners**: start call-down; open incident bridge; establish SitRep cadence.
5. **Activate plan**: select playbook option (conservative/aggressive) and activate.
6. **Verify safety constraints**: ped mins enabled, preemption behavior correct, rate-limiters active.
7. **Monitor**: watch KPIs and safety indicators; adjust only within playbook bounds.
8. **Recover**: when exit criteria met (or max duration reached), execute recovery plan.
9. **Rollback if needed**: if outcomes degrade or safety constraints threatened.
10. **Log + AAR**: finalize timeline, export metrics, schedule AAR.

### SOP 1 — Planned special event (ingress/egress)

- **Activation**: schedule-based with operator confirmation; verify venue release schedule.
- **Key monitoring**: ped volumes/crossings, queue spillback, transit hub access.
- **Deactivation**: after egress decays + queues stable for persistence window.

### SOP 2 — Major incident diversion

- **Activation**: declared (police/dispatch) + detected (probe/detector).
- **Key monitoring**: diversion route travel time, neighborhood cut-through proxies, emergency routes.
- **Deactivation**: when lanes restored + travel times normalize; coordinate with towing/recovery ETA.

### SOP 3 — Transit disruption (bus detours)

- **Activation**: declared by transit + detected bus bunching.
- **Key monitoring**: bus OTP/headways, detour corridor queues, ped access near stops.
- **Deactivation**: when rail restored and buses return to normal routing.

### SOP 4 — Evacuation / emergency

- **Activation**: declared by emergency management/incident command.
- **Mode rule**: static playbook only; bounded actions; prioritize clearance routes.
- **Deactivation**: only after incident command releases; staged rollback.

### SOP 5 — Degraded operations (comms/power constraints)

- **Activation**: declared by ITS/maintenance or detected comms degradation.
- **Mode rule**: conservative timing plans, minimize plan switching.
- **Deactivation**: after comms stable and device health checks pass.

## Governance / Versioning Runbook

### Ownership

- **Program owner**: accountable for library health, drill calendar, partner coordination.
- **Playbook owner** (per scenario): accountable for accuracy, expiry, validation evidence.
- **On-call approvers**: ops supervisor + traffic engineer + partner sign-off where needed.

### Change management

- Use `MAJOR.MINOR.PATCH` versioning.
- Require review evidence attached (twin run, tabletop notes, field observation).
- Publish release notes; update training artifacts; update call-down lists.

### Recertification + expiry

- Default expiry: 12 months (or shorter for construction-heavy corridors).
- Auto-block activation if expired (with override workflow).
- Recertify upon network/controller/transit/geometry changes.

### Audit + compliance

- Maintain immutable activation logs (who/what/when/why + signals used + outcomes).
- Keep archived versions for audit, especially for incidents with public scrutiny.

## Reference Links

- FHWA Planned Special Events Preparedness: [`https://ops.fhwa.dot.gov/tim/preparedness/pse/index.htm`](https://ops.fhwa.dot.gov/tim/preparedness/pse/index.htm:1)
- WSDOT TSMO — Planned event or incident signal timing: [`https://tsmowa.org/category/intelligent-transportation-systems/planned-event-or-incident-signal-timing`](https://tsmowa.org/category/intelligent-transportation-systems/planned-event-or-incident-signal-timing:1)
- FHWA Traffic Signal Timing Manual (already referenced): [`https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf`](https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf)
- FHWA ATSPM (already referenced): [`https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm`](https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm:1)

## Completion Checklist

- ✅ Trigger design framework added: see **"1) Trigger Design Framework"**.
- ✅ Decision tree (activate → operate → unwind) added: see **"Decision tree: activate → operate → unwind"**.
- ✅ Trigger spec template + example trigger table added: see **"Trigger spec table + example triggers"**.
- ✅ Interagency coordination + comms added: see **"2) Interagency Coordination + Communications"**.
- ✅ Equity + multimodal priority regimes added: see **"3) Equity + multimodal priority regimes"**.
- ✅ Playbook lifecycle governance added: see **"4) Playbook lifecycle governance"** and **"Governance / Versioning Runbook"**.
- ✅ Boundary with Real-Time What-If Button clarified: see **"5) Boundary with the Real-Time What-If Button"**.
- ✅ Drill program design + evaluation added: see **"6) Drill program design"**.
- ✅ Final required sections added: **Implementation Checklist**, **Operations Runbook (SOP)**, **Governance / Versioning Runbook**, **Reference Links**, **Completion Checklist**.

---

Cross-links: Related ideas include event-aware timing, weather/traffic/power constraints, and delay-tolerant signals.
