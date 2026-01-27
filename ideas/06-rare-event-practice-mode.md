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
The agency should treat playbooks as living assets by adding an expiry date and review requirement to each one, because infrastructure changes frequently invalidate assumptions. The team should recertify playbooks after major construction, controller firmware changes, new transit routes, or policy updates, and it should maintain a “do not use” flag for expired playbooks that forces review before activation. Over time, the team should expand the library cautiously based on demonstrated operational value rather than attempting to cover every possible scenario.

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

## MVP Deployment
- 3 scenarios (e.g., stadium egress, major detour, storm-safe mode).
- Quarterly tabletop drills.
- Assisted activation + rollback.

## Evaluation
- Activation latency (time from detection to plan start).
- Clearance time and spillback frequency.
- Operator adherence to checklist.
- Post-event update rate (how quickly playbooks are refreshed).

## References / Standards / Useful Sources
- FHWA Traffic Signal Timing Manual: https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf
- FHWA ATSPM: https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm

---

Cross-links: Related ideas include event-aware timing, weather/traffic/power constraints, and delay-tolerant signals.
