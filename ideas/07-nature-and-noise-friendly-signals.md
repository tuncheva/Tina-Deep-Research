# 07) Nature- and Noise-Friendly Signals

## What it is (precise)
Treat **noise and disturbance** as explicit objectives in signal timing by reducing **stop-and-go**, hard accelerations, and unnecessary idling—especially near parks, quiet residential streets, and sensitive corridors.

Mechanically, this means prioritizing timing patterns that maximize smooth progression (when safe), minimize stops, and avoid aggressive “catch-up” behavior, while respecting pedestrian service and safety constraints.

## Why digital twins matter
Noise impacts depend on where/when vehicles stop and accelerate, not just average travel time. A digital twin helps by:
- forecasting queues/stops that create repeated acceleration bursts,
- comparing alternative timings on **stops/vehicle** and “smoothness” metrics,
- defining geofenced constraints (quiet zones) and verifying corridor spillover effects.

## Easy explanation
Instead of only optimizing “fastest,” you also optimize “calmest”: fewer start/stop cycles means less noise and a better environment near sensitive places.

## What is needed (data & infrastructure)
| Need | Typical options | Notes |
|---|---|---|
| Signal telemetry | phase logs, offsets/splits, controller status | Required to measure stops and evaluate coordination effects. |
| Traffic observation | detectors, probe speeds, video analytics | Needed to estimate stops/queues and validate predictions. |
| Noise proxy metrics | stops/vehicle, acceleration events, speed variability | Direct noise sensing is optional; proxies often suffice initially. |
| Policy zones | GIS polygons (parks, schools, hospitals, quiet streets) | Applies tighter guardrails locally (max stops, max speed target). |
| Digital twin model | corridor micro/meso sim + calibration | Must capture platoons and queue spillback. |

## Implementation plan (phased)
### Phase 0 — define objectives + quiet zones (1–4 weeks)
- Identify sensitive corridors (parks, hospitals, residential night routes).
- Choose measurable proxies: stops/vehicle, p95 queue, speed variance.

### Phase 1 — baseline measurement (2–6 weeks)
- Measure current stops and delay (before/after by time-of-day).
- Identify intersections creating repeated braking/acceleration waves.

### Phase 2 — shadow-mode twin optimization (4–8 weeks)
- Generate candidate timing plans that reduce stops (without violating pedestrian mins).
- Run twin forecasts and rank by “smoothness + safety + travel time” KPI bundle.

### Phase 3 — rollout with guardrails (4–12 weeks)
- Deploy on a small area and monitor spillover (diverted traffic to side streets).
- Add caps: max cycle length, max pedestrian wait, progression speed <= posted.

### Phase 4 — continuous tuning (ongoing)
- Update plans seasonally (tourism, school schedules, daylight).
- Publish “calmer flow” reports for accountability.

## Comparison table (noise mitigation levers)
| Lever | Primary effect | Risk | Guardrail |
|---|---|---|---|
| Progression smoothing | fewer stops | speeding | progression speed <= posted |
| Longer cycle (selectively) | reduces lost time / stop frequency | longer ped waits | max ped wait cap |
| Metering side streets | reduces mainline braking waves | diversion | spillover monitoring |
| Time-of-day quiet mode | targets night periods | inconsistent driver expectations | clear signage + predictable schedule |

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Environment | fewer acceleration events → less noise/disturbance | may shift congestion/noise elsewhere | corridor-wide evaluation; protect side streets |
| Driver experience | smoother trips, fewer stops | could encourage speeding if poorly set | set progression speed <= posted; enforcement; messaging |
| Multimodal service | can improve bus smoothness | risk of longer pedestrian waits | explicit caps on ped delay; leading ped intervals |

## Real-world anchors (what exists today)
FHWA describes Adaptive Signal Control Technology (ASCT) as adjusting signal timing to changing patterns and lists benefits including **“reduce congestion by creating smoother flow”** and notes ASCT can **reduce emissions** due to improved traffic flow. “Smoother flow” is the operational lever this idea uses as a proxy to reduce noise and disturbance near sensitive areas. [FHWA: Adaptive Signal Control Technology](https://www.fhwa.dot.gov/innovation/everydaycounts/edc-1/asct.cfm)

The FHWA Traffic Signal Timing Manual notes that **stops** are important both for perceived progression quality and because accelerating vehicles can have disproportionately higher emissions than idling, motivating designs that reduce stop-and-go cycles rather than only minimizing average delay. [FHWA Traffic Signal Timing Manual (Chapter 3)](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter3.htm)

## MVP (smallest useful deployment)
- Define **one quiet zone** (night hours or a park-adjacent corridor) with a posted-speed progression target.
- Optimize for **stops/vehicle reduction** first (simple proxy), with strict caps on pedestrian wait.
- Add a “spillover guard”: monitor volumes/speeds on parallel residential streets.
- Publish a monthly “calmer flow” report: stops, speed variance, and complaints.

## Open questions
- Which proxy best tracks perceived noise: stops/vehicle, acceleration events, or speed variance?
- How to prevent diversion that moves noise to nearby side streets?
- Should quiet-zone rules be time-based (night) or event-based (wildlife seasons, hospital peaks)?

## Evaluation checklist (practical)
- Stops/vehicle (corridor and inside quiet zones)
- Speed variability and hard-acceleration proxy counts (if available)
- Travel time distribution (median + p95)
- Pedestrian delay and max wait
- Spillover: volumes and speeds on parallel residential streets

## Sources
- https://www.fhwa.dot.gov/innovation/everydaycounts/edc-1/asct.cfm
- https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter3.htm
