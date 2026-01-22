# 10) Stop-Line Intent Coordination

## What it is (precise)
Use **approach + lane-level intent** (left/through/right, and queue presence/occupancy at the stop line) to allocate green time more precisely, reducing **wasted green** on movements that have little/no demand and serving movements that are actually queued.

“Intent” can come from classic detection (lane-by-lane stop-line presence/volume), or from connected-vehicle / roadside messages that map approaching vehicles to movements.

## Why digital twins matter
A digital twin helps because intent signals are high-frequency and context-dependent:
- it can forecast near-term queue growth per movement,
- test whether reallocating green causes spillback elsewhere,
- validate that phase changes remain compatible with safety constraints and pedestrian service.

## Easy explanation
If the system knows which lanes are actually backed up (and which aren’t), it can stop giving empty lanes extra green time.

## What is needed (data & infrastructure)
| Need | Typical options | Notes |
|---|---|---|
| Lane-level detection | stop-line loops, radar, video analytics | The most common path to “intent” is simply knowing which lane group is occupied/queued. |
| Movement mapping | lane-to-phase configuration (controller database) | Needed so “left lane occupied” maps to the correct signal phase/service. |
| Connected-vehicle interface (optional) | SPaT/MAP via RSU/V2I hub | Enables richer mapping of vehicles to approaches/lanes when available. |
| Controller connectivity | central system + comms | Required for rapid plan changes and monitoring. |
| Digital twin model | intersection/corridor micro-sim (or meso + intersection logic) | Needs realistic discharge and queue spillback behavior. |

## Implementation plan (phased)
### Phase 0 — baseline + lane/phase map hygiene (2–4 weeks)
- Verify lane group definitions, detector assignments, and phase-to-movement mapping.
- Establish KPIs: wasted green %, split failures, max queue, pedestrian delay.

### Phase 1 — “intent from detection” rollout (4–8 weeks)
- Use stop-line detection to compute per-movement demand/queue indicators.
- Add conservative rules: minimum greens, max recalls, ped minimums, clearance constraints.

### Phase 2 — shadow-mode twin evaluation (4–8 weeks)
- Run twin in parallel to score alternative split allocations.
- Stress-test odd cases: short lanes, spillback, heavy left-turn surges.

### Phase 3 — connected-vehicle enrichment (optional, 2–6+ months)
- Where available, integrate SPaT/MAP infrastructure and use lane/approach mapping to improve intent inference.
- Keep a fallback path: if CV data drops, revert to detector-only.

### Phase 4 — corridor coordination (ongoing)
- Extend to adjacent signals so intent-aware decisions don’t break progression.
- Add event/incident modes to prevent gridlock when intent surges are detected.

## Comparison table (intent sources)
| Intent source | What you get | Strength | Weakness |
|---|---|---|---|
| Stop-line presence/occupancy | who is waiting now | widely available | blind to midblock queue growth |
| Lane-level counts | demand over time | good for split tuning | sensitive to detector errors |
| Video movement classification | rich turning info | flexible; can estimate queues | weather/lighting issues |
| CV/V2X (SPaT/MAP + messages) | approach/lane context + timing | can improve inference | deployment cost; coverage gaps |

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Efficiency | less wasted green, lower delay | detector errors can mis-allocate green | detector health checks; conservative bounds |
| Reliability | responds to turning surges | can harm coordination if too myopic | corridor-level constraints; cap per-cycle changes |
| Deployability | works with standard detection | CV/RSU adds cost/complexity | stage deployment; keep detector-only mode |

## Real-world anchors (what exists today)
The SPaT Challenge implementation guide describes how roadside traffic signal controllers can output SPaT parameters, which can be converted to SAE J2735 SPaT messages, and combined with a static MAP message that describes intersection geometry so vehicles can interpret approach/phase status. It also notes this DSRC broadcast is one-way (helpful for privacy) and identifies minimum field needs like an NTCIP 1202 SPaT output via Ethernet plus software to translate to J2735 and generate MAP/RTCM messages (e.g., via an FHWA V2I Hub approach). This provides a concrete reference architecture for obtaining the signal-state + geometry context that enables richer “intent” mapping when using connected-vehicle data. [NOCoE: SPaT Challenge Implementation Guide](https://www.transportationops.org/spatchallenge/resources/Implementation-Guide)

## MVP (smallest useful deployment)
- Fix lane/phase mapping and compute **wasted green %** for each movement.
- Start with **one intersection** and reallocate at most **3–5 seconds/cycle** based on stop-line occupancy.
- Enforce non-negotiables: ped mins, clearance, min green per movement.
- Add a “coordination friendliness” cap when part of a corridor plan (don’t shift splits too far).

## Open questions
- What is the best intent signal when detection is noisy: presence, count, or queue estimate?
- How do we avoid starving low-volume movements while still reducing wasted green?
- What is the correct interface boundary: does logic run in controller, edge, or central?

## Evaluation checklist (practical)
- Wasted green time (seconds/cycle and %)
- Split failures and max queue length per movement
- Corridor travel time reliability (if coordinated)
- Pedestrian delay and compliance with minimums
- Detector/CV data health (missing rate, stuck-on, latency)

## Sources
- https://www.transportationops.org/spatchallenge/resources/Implementation-Guide
