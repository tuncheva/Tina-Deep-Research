# 05) Adapt to Local Driving Style (With Guardrails)

## What it is (precise)
Calibrate signal timing to **local, measurable driving and discharge behavior**—notably **start-up lost time** and **saturation flow/headways**—so splits, cycle lengths, and clearance intervals better match how queues actually discharge at each intersection.

In practice, this is a “local calibration layer” sitting above standard timing logic: the controller still follows engineering constraints, but its parameters reflect observed local conditions rather than generic defaults.

## Why digital twins matter
A traffic digital twin helps because these local parameters strongly affect predicted queues/delay, and the twin can:
- continuously re-estimate discharge parameters from field telemetry,
- test whether a re-calibration improves corridor KPIs without increasing safety risk,
- validate changes in “shadow mode” before pushing them live.

## Easy explanation
Two intersections can look the same on paper but behave differently: people accelerate differently, trucks are more common, and queues discharge at different rates. The twin learns those differences and tunes the timings, while still enforcing safety rules.

## What is needed (data & infrastructure)
| Need | Typical options | Notes |
|---|---|---|
| Discharge measurements | stop-bar detection (loops/radar/video), high-resolution controller logs | Needed to estimate queue discharge headways and start-up lost time. |
| Baseline signal timing method | conventional timing (HCM-style), time-of-day plans, optimization tool | The “learning” updates parameters; it doesn’t replace the whole method. |
| Guardrails / safety constraints | minimum greens, pedestrian mins, clearance intervals, max cycle, speed targets | Prevents “learning” aggressive or unsafe behavior. |
| Digital twin model | micro/meso simulation with calibration | Used to test sensitivity and avoid overfitting to noisy data. |

## Implementation plan (phased)
### Phase 0 — baseline + instrument (2–6 weeks)
- Ensure high-resolution phase + detector data is available per intersection.
- Define KPIs: queue spillback, control delay, stops, pedestrian delay.

### Phase 1 — estimate local discharge parameters (2–8 weeks)
- Estimate **start-up lost time** and saturation headways per approach/lane group.
- Identify segments where parameters drift by time-of-day (school release, freight peaks).

### Phase 2 — shadow-mode calibration (4–8 weeks)
- Run twin live and compare forecasts with/without local calibration.
- Keep conservative bounds (e.g., only allow small changes per week).

### Phase 3 — controlled rollout (4–12 weeks)
- Apply changes to a small set of intersections.
- Monitor safety proxies (hard braking, near-miss indicators if available) and pedestrian service.

### Phase 4 — continuous learning with guardrails (ongoing)
- Re-estimate parameters periodically; revert if detector health degrades.
- Publish guardrail settings and change logs for auditability.

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Operations realism | predictions match local queues better | overfits to noisy detections | require confidence thresholds; shadow mode |
| Throughput / delay | better split allocation where discharge differs | can “optimize” toward aggressive driving norms | enforce speed/clearance constraints; cap parameter shifts |
| Maintainability | parameters update automatically | harder to explain than fixed defaults | produce simple reports (before/after parameters + KPIs) |

## Real-world anchors (what exists today)
The FHWA Traffic Signal Timing Manual describes **start-up lost time** as the additional time consumed by the first queued vehicles reacting/accelerating at the start of green, and notes it is commonly assumed to be about **2 seconds** in practice. It also defines **saturation flow rate** and describes how discharge stabilizes after the first few vehicles. These are exactly the parameters a “local driving style” calibration layer would estimate and tune. [FHWA Traffic Signal Timing Manual (Chapter 3)](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter3.htm)

## Evaluation checklist (practical)
- Queue spillback frequency and max queue length (p95)
- Approach delay and travel time reliability (p50/p95)
- Stops/vehicle
- Pedestrian delay distribution and max wait
- Detector health (missing data %, stuck-on occupancy)

## Sources
- https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter3.htm
