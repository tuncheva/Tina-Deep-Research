# 18) Event-Aware Timing (Concert/Stadium/Festival Mode)

## What it is (precise)
**Event-aware timing** detects or anticipates predictable demand surges (planned events, incidents, evacuations) and activates **special signal timing plans** that favor key inbound/outbound movements for a limited window, with active monitoring and rollback.

This is “planned event or incident signal timing” plus (optionally) automation: detecting the surge and switching plans with minimal operator burden.

## Why digital twins matter
A digital twin supports event-aware timing by:
- forecasting congestion build-up and spillback under different event plans,
- ranking response plans by KPIs (delay, max queue, emissions proxies),
- supporting a control-room workflow where operators choose from safe options.

Aimsun’s real-time platform describes rapid forecasting and response plan ranking, including predictive signal plan scheduling. [Aimsun Live](https://www.aimsun.com/aimsun-live/)

## Easy explanation
When a concert ends, traffic suddenly changes. Event mode is a pre-planned set of signal timings that helps people leave safely and quickly, then returns signals to normal.

## Real-world practice anchor
WSDOT’s TSMO guidance describes **planned event or incident signal timing** as synchronizing groups of signals to favor entering/exiting a venue or divert around incidents/evacuations; it notes goals like increasing green for event movements, real-time monitoring, and manual or detector-based activation. [WSDOT TSMO](https://tsmowa.org/category/intelligent-transportation-systems/planned-event-or-incident-signal-timing)

## What you need (per WSDOT TSMO)
| Category | Typical requirements |
|---|---|
| Equipment | comms to central ops, central signal software, detection, cameras |
| Planning | analysis and pre-programmed plans favoring specific movements |
| Operations | staff to activate/adjust/turn off (or automation with detectors) |
| Coordination | event managers, transit, law enforcement, affected agencies |

Source: [WSDOT TSMO](https://tsmowa.org/category/intelligent-transportation-systems/planned-event-or-incident-signal-timing)

## Implementation plan (phased)
### Phase 0 — pick event types and corridors
- Identify recurring events and affected corridors.
- Define success KPIs: clearance time, max queue, ped safety.

### Phase 1 — build an event plan library
- Create pre/post-event plans: inbound (before), outbound (after).
- Include contingency plans for incidents or road closures.

### Phase 2 — monitoring and activation
- Manual activation via TMC operators initially.
- Add detector-based triggers where feasible (parking occupancy, volume thresholds).

### Phase 3 — twin-based forecasting and option ranking
- Use the twin to test plan robustness (start/end time uncertainty, attendance variability).
- Present operators with 3–5 safe options (e.g., more outbound green vs protect pedestrians).

### Phase 4 — automation with guardrails
- Auto-switch plans with strict constraints and clear rollback rules.

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Operations | faster clearance and less gridlock | requires off-hours staffing | automation + playbooks |
| Safety | reduces chaotic merging and blocked intersections | can increase ped wait | pedestrian constraints; crowd management |
| Reliability | predictable response to known surges | false triggers disrupt normal traffic | multi-signal confirmation; hysteresis |

## Evaluation checklist
- Time to clear event traffic (queue dissipation)
- Spillback/blocked-box frequency
- Pedestrian delay and compliance
- Operator workload and plan activation accuracy

## Sources
- https://tsmowa.org/category/intelligent-transportation-systems/planned-event-or-incident-signal-timing
- https://www.aimsun.com/aimsun-live/
