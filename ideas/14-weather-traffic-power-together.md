# 14) Weather + Traffic + Power Together

## What it is (precise)
Operate signals with an explicit “constraints-aware” mode that fuses:
- **traffic state** (queues, spillback risk),
- **weather risk** (reduced visibility/friction, demand shifts), and
- **power/communications constraints** (battery/UPS limits, cabinet resets, loss of central connectivity),

to select safer timing strategies during storms/outages rather than continuing normal optimization.

## Why digital twins matter
A twin can run scenario rollouts that combine these stressors:
- traffic surges + detours,
- sensor degradation (rain/fog impacts, dropouts),
- partial network connectivity,
- controlled “degraded operation” policies.

This makes it possible to validate storm/outage timing playbooks (and recovery behavior) before deploying them.

## Easy explanation
When weather and power are bad, you don’t want “business as usual” signal logic—you want a storm-safe mode that keeps intersections predictable and prevents gridlock.

## What is needed (data & infrastructure)
| Need | Typical options | Notes |
|---|---|---|
| Weather inputs | road weather stations, forecasts, visibility/rain flags | Used to trigger safer constraints and slower progression targets. |
| Traffic monitoring | detectors, probe speeds, CCTV | Needed to detect spillback and unsafe queuing.
| Power/comms status | cabinet power alarms, UPS status, comms heartbeat | Triggers local control and conservative plans.
| Plan library | storm plans, outage plans, evacuation/diversion plans | Versioned and drill-tested in the twin.
| Digital twin | network model + degraded-mode logic | Used to test combined failures.

## Implementation plan (phased)
### Phase 0 — define storm/outage modes and triggers (2–4 weeks)
- Define triggers: heavy rain/fog, high winds, flood detours, power instability, comms loss.
- Define constraints: maximum cycle, protected ped phases where needed, conservative clearance.

### Phase 1 — build and test playbooks in the twin (4–10 weeks)
- Create plans for:
  - reduced-speed progression,
  - spillback prevention,
  - evacuation outbound bias,
  - local isolated operation when comms drop.

### Phase 2 — deploy monitoring + automatic mode switching (4–12 weeks)
- Add health checks and mode logic (see idea 11).
- Require operator confirmation for major mode switches (unless safety-critical).

### Phase 3 — recovery and audit (ongoing)
- Define stable return-to-normal criteria.
- Publish after-action reports after major events.

## Comparison table (storm/outage operating modes)
| Mode | Trigger examples | Primary objective | Typical costs |
|---|---|---|---|
| Normal | fair weather + stable power | mobility optimization | none |
| Storm-safe | heavy rain/fog | safety + predictability | more delay |
| Power-constrained | UPS/battery limits | minimize cabinet risk | reduced features |
| Comms-loss local | central link down | safe local autonomy | reduced coordination |

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Safety | more predictable control in hazardous conditions | wrong trigger thresholds | tune with after-action data |
| Resilience | handles comms/power degradation | can increase delay | prioritize critical routes; communicate modes |
| Operations | clear playbooks reduce improvisation | requires maintenance of plan library | periodic drills + versioning |

## Real-world anchors (what exists today)
WSDOT notes that planned event or incident signal timing can support diversion around incidents and even regionwide evacuations, and that agencies often monitor conditions in real time and adjust timing accordingly. Weather and power events frequently manifest operationally as incidents/detours plus degraded sensing/communications, making this “fused constraints” mode a natural extension of existing event/incident timing practices. [WSDOT TSMO: Planned event or incident signal timing](https://tsmowa.org/category/intelligent-transportation-systems/planned-event-or-incident-signal-timing)

A 2025 UC Irvine thesis simulates intelligent intersections under varying weather conditions (clear/heavy rain/heavy fog) and studies how sensor fusion performance and intersection throughput can degrade, highlighting why weather should be treated as a first-class input when deciding which control strategies are safe and reliable. [UC Irvine thesis: Securing Intelligent Intersections (2025)](https://escholarship.org/content/qt50m2f8c7/qt50m2f8c7.pdf)

## MVP (smallest useful deployment)
- Add a single **“storm/outage” mode** that can be activated citywide or by district.
- Define a **trigger bundle**: rain/fog severity + comms heartbeat + cabinet power alarms.
- Maintain a versioned library of **storm-safe plans** (slower progression, protected ped focus, spillback prevention).
- Produce after-action “mode timeline” reports for each activation.

## Open questions
- Which weather indicators are reliable enough for automatic activation (forecast vs observed sensors)?
- How should policies differ between power-constrained local operation vs central connectivity available?
- What routes must remain prioritized during storms (hospitals, evacuation, transit) and how is that encoded?

## Evaluation checklist (practical)
- Time-to-switch into storm/outage mode (and back)
- Queue spillback avoidance on critical links
- Pedestrian compliance (mins/clearance)
- Stability under comms loss (no oscillation)
- Post-event recovery time and KPI deltas

## Sources
- https://tsmowa.org/category/intelligent-transportation-systems/planned-event-or-incident-signal-timing
- https://escholarship.org/content/qt50m2f8c7/qt50m2f8c7.pdf
