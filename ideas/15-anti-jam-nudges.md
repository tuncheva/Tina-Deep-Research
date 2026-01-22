# 15) Anti-Jam “Nudges”

## What it is (precise)
Prevent gridlock by making **small, early timing adjustments** when the system predicts **queue spillback** and “blocked-box” conditions.

The focus is not long-horizon optimization; it’s short-horizon stability control: keep critical intersections from locking up by protecting storage, metering inflows, and flushing key bottlenecks before they collapse.

## Why digital twins matter
Spillback is a network phenomenon: one blocked link can cascade. A twin helps by:
- forecasting near-term queue growth and spillback risk,
- testing whether a small nudge (e.g., 3–8s split shift) prevents collapse,
- verifying that nudges don’t violate pedestrian minimums or harm coordination beyond acceptable bounds.

## Easy explanation
Don’t wait until everything is jammed—when the twin sees a jam forming, it makes tiny timing tweaks to keep intersections from blocking each other.

## What is needed (data & infrastructure)
| Need | Typical options | Notes |
|---|---|---|
| Queue / congestion sensing | advance detectors, probe speeds, video queue estimates | Needs to detect growth before stop-line saturation. |
| Spillback indicators | occupancy near 1.0 on upstream detectors, stopped-speed probes, camera verification | Spillback is often better detected with occupancy and midblock sensing than stop-line counts alone. |
| Control levers | split tweaks, metering minor approaches, coordinated offsets, (rarely) preemption-like flushing | “Nudges” should stay within conservative bounds. |
| Policy constraints | ped mins, clearance, max cycle, keep-transit-reliability constraints | Prevents harmful oscillations.
| Digital twin | corridor/network model with queuing | Must model storage and blocking interactions.

## Implementation plan (phased)
### Phase 0 — define jam signatures + protected assets (2–4 weeks)
- Identify critical links where spillback is unacceptable (rail crossings, bridges, tunnels, hospital access).
- Define measurable jam signals: sustained high occupancy + low speed + growing queues.

### Phase 1 — build nudge playbook (4–8 weeks)
- “Flush” plans: temporarily allocate extra green to clear downstream storage.
- “Meter” plans: cap inflow to saturated areas to prevent blocking.
- “Hold coordination” rules: limit how much offsets/splits can move.

### Phase 2 — shadow-mode twin forecasting (4–8 weeks)
- Run the twin live and compute spillback risk.
- Recommend nudges without actuating; validate forecasts and false alarm rates.

### Phase 3 — controlled activation (4–12 weeks)
- Enable automatic nudges with strict bounds and operator override.
- Add hysteresis to avoid flip-flopping.

### Phase 4 — expand network coverage (ongoing)
- Extend from a few critical corridors to a district.
- Integrate with incident/event modes (idea 06/18).

## Comparison table (nudge types)
| Nudge type | What it does | Best when | Risk |
|---|---|---|---|
| Flush | temporarily favors downstream clearance | storage is near full | starves side streets |
| Meter | reduces inflow to saturated link | downstream blocked | diversion to other routes |
| Hold coordination | limits offset/split changes | corridor progression is critical | less adaptivity |
| Emergency protection | protects a “no spillback” asset | rail crossing/bridge | can increase upstream delay |

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Network stability | prevents gridlock cascades | wrong nudges can shift queues elsewhere | corridor-wide evaluation; protect side streets |
| Responsiveness | small changes can avert major delay | may look “weird” to drivers/operators | explainable logs + simple rules |
| Safety | avoids blocking intersections | may shorten some movements temporarily | enforce minimums; use time limits |

## Real-world anchors (what exists today)
The FHWA Traffic Signal Timing Manual discusses traffic responsive plan selection and adaptive control in the context of incidents and unusual conditions, noting that unexpected changes (incidents, extreme weather, events) can make time-of-day plans suboptimal and that data-driven selection/adaptation can improve operations. It also highlights the importance of detector reliability for adaptive systems and discusses incident/event management as a reason to retime signals to “flush” preferred movements and reduce delay during non-recurring congestion. These concepts directly support the idea of small, early “anti-jam” adjustments driven by real-time sensing and prediction. [FHWA Traffic Signal Timing Manual (Chapter 9)](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm)

## MVP (smallest useful deployment)
- Choose **1–2 critical intersections** where spillback is safety-critical (rail crossing, bridge).
- Implement a conservative detector-based spillback trigger (occupancy high + speed low + duration).
- Allow only 2 actions:
  - **meter inflow** (cap minor green)
  - **flush downstream** (temporary extra green)
- Require cool-down and maximum activation time; log each activation with reason + before/after.

## Open questions
- What is the best spillback signal: midblock occupancy, probe stoppage, or camera queue estimate?
- How do we keep nudges from becoming “permanent bias” that shifts congestion to side streets?
- Should nudges be purely local or coordinated across 2–3 upstream signals?

## Evaluation checklist (practical)
- Spillback frequency and duration (before/after)
- Blocked-box events (count, total minutes)
- Travel time reliability (p95)
- Side-street and pedestrian delay impacts
- False-alarm and oscillation rate of nudge activation

## Sources
- https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm
