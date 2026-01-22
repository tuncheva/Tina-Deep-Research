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

## Comparison table (what to calibrate)
| Parameter | What it captures | Common measurement | Risk if wrong |
|---|---|---|---|
| Start-up lost time | first vehicles reacting/accelerating | high-res stop-bar logs | under/over-allocates green |
| Saturation headway / flow | steady discharge rate | stop-bar counts over sustained queue | split allocation errors |
| Heavy vehicle factor | truck/bus impact on discharge | classification + headways | systematic bias by time-of-day |
| Turn movement rates | lane group demand mix | counts/video turning movements | wrong phase demand |

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Operations realism | predictions match local queues better | overfits to noisy detections | require confidence thresholds; shadow mode |
| Throughput / delay | better split allocation where discharge differs | can “optimize” toward aggressive driving norms | enforce speed/clearance constraints; cap parameter shifts |
| Maintainability | parameters update automatically | harder to explain than fixed defaults | produce simple reports (before/after parameters + KPIs) |

## Real-world anchors (what exists today)
The FHWA Traffic Signal Timing Manual describes **start-up lost time** as the additional time consumed by the first queued vehicles reacting/accelerating at the start of green, and notes it is commonly assumed to be about **2 seconds** in practice. It also defines **saturation flow rate** and describes how discharge stabilizes after the first few vehicles. These are exactly the parameters a “local driving style” calibration layer would estimate and tune. [FHWA Traffic Signal Timing Manual (Chapter 3)](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter3.htm)

## MVP (smallest useful deployment)
- Select **3–5 intersections** with good stop-bar detection.
- Estimate and publish weekly: **start-up lost time** and **saturation headway** per approach.
- Allow only **small parameter updates** (e.g., ±5–10% per month) with automatic rollback if KPIs worsen.
- Keep a visible “guardrails” panel: ped minimums, clearance, max cycle, speed target.

## Open questions
- How do we separate “driving style” from geometry effects (grade, sight distance) and vehicle mix?
- What confidence tests should gate parameter updates (sample size, detector health, seasonality)?
- How do we prevent feedback loops (changed timing changes observed headways)?

## Evaluation checklist (practical)
- Queue spillback frequency and max queue length (p95)
- Approach delay and travel time reliability (p50/p95)
- Stops/vehicle
- Pedestrian delay distribution and max wait
- Detector health (missing data %, stuck-on occupancy)

---

## Deep dive (traffic engineering): what “driving style” means in signals
Signals don’t care about psychology; they care about measurable discharge behavior.

### Key measurable components
1. **Start-up lost time**: the initial “wake-up” time at start of green.
2. **Saturation headway**: steady headway once discharge stabilizes.
3. **Heavy-vehicle share**: trucks/buses change acceleration and headways.
4. **Lane discipline**: late merges, blocking, and noncompliance increase effective lost time.

If you calibrate these, you’re not “rewarding aggressiveness”; you’re making the model match reality, and then enforcing *separate* safety/policy caps.

---

## Deep dive (data): extracting discharge parameters from high-resolution logs
High-resolution controller data (phase events + detector actuations) enables repeatable estimation.

### A simple estimation recipe
For each approach/lane group during queued discharge:
1. Identify the **start of green** timestamp.
2. Use stop-bar detections to capture vehicle departures.
3. Compute headways for vehicles 4–10 (avoid first few vehicles).
4. Estimate:
   - start-up lost time = (time to first few departures) − (N × saturation headway)
   - saturation headway = median(headway[4..10])

Quality gates:
- require a minimum sample size,
- exclude cycles with downstream blocking/spillback,
- exclude cycles with detector health issues.

---

## Deep dive (twin): calibrate, then validate with counterfactuals
Calibration isn’t the same as improvement.

Twin workflow:
- baseline model uses generic parameters,
- calibrated model uses local headways/lost time,
- evaluate both on historical days.

If the calibrated model predicts reality better but KPIs don’t improve, it may still be worth it because **future what-if decisions** become more accurate.

Cross-link: see [`ideas/03-real-time-what-if-button.md`](ideas/03-real-time-what-if-button.md).

---

## Guardrails (non-negotiables)
Local calibration must never:
- shorten pedestrian walk/clearance below minimum,
- encourage higher progression speed than posted,
- reduce yellow/all-red below standards,
- exceed max cycle caps in policy zones.

Cross-link: see [`ideas/04-city-rules-built-into-signals.md`](ideas/04-city-rules-built-into-signals.md) and [`ideas/17-safety-first-signals.md`](ideas/17-safety-first-signals.md).

---

## Failure modes and mitigations
| Failure mode | Example | Mitigation |
|---|---|---|
| Overfitting | camera mis-detection changes headways | confidence gates + shadow mode |
| Feedback loop | changed splits change observed headways | freeze updates for N weeks post-change |
| Bias by vehicle mix | trucks at certain hours | segment by time-of-day and classify vehicles |
| Hidden geometry effects | grade, sight distance | stratify by approach; include geometry metadata |

---

## Metrics (calibration health)
- Prediction error (queue / travel time) before vs after calibration
- Drift in estimated headways (weekly)
- % of cycles used for estimation (after quality gates)
- Number of automatic rollbacks triggered

---

## Operator UX: “calibration report” page
A simple report makes this explainable:
- parameter changes (before → after),
- why change was made (sample size, confidence),
- KPI impact estimate (from twin),
- rollback threshold.

Cross-link: see [`ideas/20-explainable-signals.md`](ideas/20-explainable-signals.md).

---

## Related topics
- [`ideas/01-green-wave-plans.md`](ideas/01-green-wave-plans.md) — progression plans depend on travel time; calibration keeps offsets honest.
- [`ideas/03-real-time-what-if-button.md`](ideas/03-real-time-what-if-button.md) — uses calibrated parameters for faster, safer decisions.
- [`ideas/12-privacy-friendly-learning.md`](ideas/12-privacy-friendly-learning.md) — privacy when using probe trajectories.

## Sources
- https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter3.htm
