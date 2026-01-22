# 06) Rare-Event Practice Mode

## What it is (precise)
Use a traffic digital twin to **rehearse rare, disruptive scenarios** (special events, detours, evacuations, major incidents, power constraints) and pre-build a **playbook** of vetted timing plans, coordination patterns, and operator actions.

This is different from everyday adaptive control: it focuses on **non-recurring congestion** and “operational readiness,” where the objective is safe, predictable response rather than optimal average-day efficiency.

## Why digital twins matter
Rare events are hard to learn from because they don’t happen often, and when they do, they’re chaotic. A twin helps by:
- running ingress/egress or diversion scenarios repeatedly and safely,
- quantifying queue spillback risks and failure modes,
- validating “turn this plan on when X happens” triggers,
- keeping operator procedures consistent across shifts.

## Easy explanation
Like a fire drill: you practice the hard days in simulation, then keep a set of proven “buttons” to press when the real incident happens.

## What is needed (data & infrastructure)
| Need | Typical options | Notes |
|---|---|---|
| Scenario inputs | event schedules, road closures, incident templates, evacuation routes | Include timing uncertainty (start/end variability). |
| Timing plan library | pre-timed plans, coordination patterns, special-event plans | Needs versioning + rollback. |
| Field monitoring | detectors, CCTV, probe speeds, operator reports | Used to confirm the scenario is unfolding and to adjust. |
| Control interface | central signal system + comms | Enables rapid activation and corridor-level coordination. |
| Digital twin | corridor/network micro/meso sim | Must represent diversion routes + queue spillback. |

## Implementation plan (phased)
### Phase 0 — define “rare events” and success metrics (1–3 weeks)
- List event types: stadium ingress/egress, parade routes, major detours, evacuation outbound, blackout/communications loss.
- Define hard safety constraints (pedestrian mins, clearance, no-blocking-critical-crossings).

### Phase 1 — build scenario playbooks offline (4–10 weeks)
- Create scenario templates with demand ranges.
- Produce candidate plans: longer greens for priority movements, temporary one-way coordination patterns, diversion support plans.

### Phase 2 — table-top + shadow tests (4–8 weeks)
- Run the twin against historical “bad days.”
- Produce operator checklists: what to monitor, when to switch, when to revert.

### Phase 3 — controlled drills (ongoing)
- Run periodic “practice days” (night/weekend) where operators activate playbook plans in a safe window.
- Update the playbook based on measured outcomes.

## Comparison table (playbook content)
| Playbook element | What it contains | Why it matters |
|---|---|---|
| Trigger definition | thresholds + who can activate | avoids confusion under stress |
| Plan set | conservative + aggressive + revert | prevents “one plan fits all” |
| Monitoring checklist | KPIs, CCTV views, detector health | ensures plan matches reality |
| Decision tree | switch / hold / revert logic | reduces operator improvisation |
| After-action review | what happened + KPI deltas | continuous improvement |

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Incident response | faster, less improvisation | wrong plan if event differs from assumed | include decision tree + monitoring gates |
| Safety | pre-vetted pedestrian/clearance constraints | over-prioritizes vehicles during chaos | explicit multimodal constraints + max waits |
| Staffing | reduces cognitive load | still needs after-hours coverage | automation for activation + on-call rotation |

## Real-world anchors (what exists today)
WSDOT describes **planned event or incident signal timing** as synchronizing groups of signals to favor traffic entering/exiting venues, and also using timing plans to divert traffic around incidents or support evacuations; it notes agencies monitor conditions in real time and often turn plans on/off manually (with detectors sometimes enabling automatic control). That is essentially the operational “playbook” concept this idea formalizes inside a digital twin rehearsal workflow. [WSDOT TSMO: Planned event or incident signal timing](https://tsmowa.org/category/intelligent-transportation-systems/planned-event-or-incident-signal-timing)

FHWA’s planned special events handbook notes agencies can download and implement **special signal timing plans** for event ingress/egress via closed-loop or centrally controlled systems, and that adaptive systems can adjust to event-generated flows. This supports the feasibility of maintaining dedicated event plans and coordinating them from operations centers. [FHWA: Managing Travel for Planned Special Events (Chapter 4)](https://ops.fhwa.dot.gov/publications/fhwaop04010/chapter4_04.htm)

## MVP (smallest useful deployment)
- Create **3 scenario playbooks**:
  - planned venue egress
  - major detour
  - comms/power degraded operation
- For each, maintain **two timing plans** (conservative + aggressive) and a revert plan.
- Run a **quarterly table-top drill** using last year’s “bad day” data in the twin.
- Ship operator checklists with a single-page “when to switch / when to revert” decision tree.

## Open questions
- Which triggers are reliable enough for partial automation (volume thresholds, parking occupancy, 911 CAD alerts)?
- How do we keep playbooks current when lane geometry and construction change?
- What is the right cadence for drills without disrupting normal operations?

## Evaluation checklist (practical)
- Time-to-activate correct plan (minutes)
- Queue spillback onto critical links (count, duration)
- Travel time reliability on designated ingress/egress routes
- Pedestrian service (max waits near venues)
- Operator workload (alerts acknowledged, manual interventions)

---

## Deep dive (design): what a “playbook” actually contains
A real playbook is not “one plan.” It is a **set of plans + triggers + checklists**.

### Minimum artifacts per scenario
1. **Scenario definition**
   - what is happening (closure/event/evacuation)
   - expected demand ranges and uncertainty
   - impacted intersections and detour routes
2. **Plan set**
   - conservative plan (safer under uncertainty)
   - aggressive plan (higher throughput, higher risk)
   - revert plan (last known good)
3. **Activation checklist**
   - required confirmations (CCTV, probe speeds, field reports)
   - which intersections must be switched together
4. **Monitoring checklist**
   - spillback points to watch
   - pedestrian hot spots
   - transit disruptions
5. **Decision tree**
   - “if queues exceed X, switch to Y”
   - “if detector health degraded, lock to conservative plan”
6. **After-action review template**
   - what worked / failed
   - KPI deltas
   - plan tuning tasks

---

## Deep dive (twin workflow): scenario rehearsal loop
A practical rehearsal loop:
1. **Generate scenarios** (demand ranges + timing uncertainty).
2. **Run the twin** with conservative and aggressive plan sets.
3. **Stress test**:
   - detector degraded / comms loss,
   - rain/snow friction,
   - unexpected closure extension.
4. **Pick triggers** that are observable and robust.
5. **Publish the playbook** and train operators.

Cross-link: see [`ideas/03-real-time-what-if-button.md`](ideas/03-real-time-what-if-button.md) for fast evaluation mechanics; playbooks can become the candidate action set.

---

## Deep dive (triggers): avoid brittle automation
Triggers should be:
- **observable**: based on telemetry you reliably have (probe speeds, counts, CCTV confirmation),
- **stable**: not sensitive to noise,
- **actionable**: maps to a specific plan + duration.

Examples:
- “p95 travel time on ingress route > 1.6× baseline for 5 minutes → activate ingress plan A.”
- “blocked-box events detected at intersection X in 2+ consecutive cycles → activate metering preset.”

Avoid:
- single-detector thresholds with no health context.

Cross-link: see [`ideas/02-self-healing-intersections.md`](ideas/02-self-healing-intersections.md).

---

## Deep dive (operations): training, drills, and muscle memory
Practice mode should produce **muscle memory**.

Operational pattern:
- table-top drill quarterly,
- “live drill” yearly in a safe window,
- refresh playbook when geometry changes.

Artifacts for training:
- one-page scenario cheat sheet,
- “first 5 minutes” checklist,
- escalation list (who to call).

---

## Failure modes + mitigations
| Failure mode | Example | Mitigation |
|---|---|---|
| Scenario drift | event ends later than expected | extend plan duration; re-evaluate every 10–15 min |
| Wrong plan activated | detour is different than assumed | decision tree + revert plan |
| Coordination breaks | isolated changes cause corridor conflicts | switch as a group; use pre-vetted offsets |
| Telemetry gaps | CCTV down; detectors failing | fallback to conservative plan; operator confirmation |

---

## Metrics (readiness)
- Time to detect scenario and activate playbook
- % of events where the “right” plan was selected on first try
- Spillback minutes on critical links
- Pedestrian max wait near venues
- Number of playbook updates after each event season

---

## Operator UI: the “playbook panel”
Cross-link: see [`ideas/08-operator-option-menu.md`](ideas/08-operator-option-menu.md)

UI should show:
- scenario list + current status (not active / likely / active)
- recommended playbook action + confidence
- “switch group” preview (which intersections will change)
- revert button + expiry timer
- monitoring checklist (links to camera presets)

---

## Related topics
- [`ideas/18-event-aware-timing.md`](ideas/18-event-aware-timing.md) — event detection and timing changes.
- [`ideas/01-green-wave-plans.md`](ideas/01-green-wave-plans.md) — special coordination patterns for ingress/egress.
- [`ideas/11-delay-tolerant-smart-signals.md`](ideas/11-delay-tolerant-smart-signals.md) — safe degradation when comms/power is constrained.

## Sources
- https://tsmowa.org/category/intelligent-transportation-systems/planned-event-or-incident-signal-timing
- https://ops.fhwa.dot.gov/publications/fhwaop04010/chapter4_04.htm
