# 03) Real-Time “What If?” Button

## What it is (precise)
A **real-time “what if?” button** runs several short-horizon simulations (e.g., 1–5 minutes ahead) from the current observed state to compare small signal-timing changes (splits, offsets, phase sequence, coordination settings) before applying one in the field.

A common digital-twin pattern is to create **parallel simulation instances** initialized from the current calibrated state and test different control actions safely. A motorway digital twin paper explicitly describes parallel Digital Twin Instances (DTI) that run faster-than-real-time to evaluate different control strategies before deployment. https://www.sciencedirect.com/science/article/pii/S1474034622003160

## Why digital twins matter
Without a calibrated twin, operators mostly rely on experience because the effects of timing changes are hard to predict in the moment. A twin provides:
- a current-state model (queues, arrivals, spillback),
- fast comparison of candidate actions,
- quantified tradeoffs and uncertainty.

Industry tooling also frames simulation as a way to **test timing updates and adaptive control impacts** before field changes, using a digital-twin environment. https://www.econolite.com/application-areas/virtual-simulation-modeling-and-validation/

## Easy explanation
Before changing the lights, the system “tries a few options in a safe virtual copy” and picks the one that looks best for the next few minutes.

## What is needed (data & infrastructure)
| Need | Typical options | Notes |
|---|---|---|
| Live state estimation | detector counts/occupancy, probe travel times, controller phase states | Must approximate queues and arrivals well enough for 1–5 min predictions. |
| Simulation engine | micro/meso simulation (corridor/area), calibrated parameters | Needs to run faster than real time for multiple scenarios. |
| Candidate action generator | bounded timing tweaks (splits/offsets), templates | Keep changes small to reduce risk and improve predictability. |
| Guardrails | safety + policy constraints | Enforce pedestrian minimums, max queue, max cycle changes, etc. |
| Deployment path | ATMS/central system to controllers | Must support reliable timing plan updates and rollback. |

## Implementation plan (phased)
### Phase 0 — baseline + observability (2–6 weeks)
- Ensure controllers expose phase/timing states and detector health.
- Define near-term KPIs (queue spillback risk, stops, person delay, pedestrian delay).

### Phase 1 — build + calibrate a “nowcast” twin (4–10 weeks)
- Calibrate the model to match observed queues/travel times.
- Validate that 1–5 minute predictions are stable under normal demand.

### Phase 2 — shadow “what-if” (4–8 weeks)
- Run parallel scenarios continuously, but do not actuate.
- Compare predicted vs actual; tune uncertainty bounds.

### Phase 3 — controlled deployment (4–12 weeks)
- Allow only bounded actions (e.g., small split/offset adjustments).
- Add automatic rollback if performance degrades.

### Phase 4 — expand action space (ongoing)
- Add event/incident modes, transit priority interactions, weather modes.

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Safety | can avoid risky “guessing” under stress | bad model → bad action | guardrails + small bounded changes + rollback |
| Operations | faster decision cycles | operator over-trust (“automation bias”) | show uncertainty + require human confirm early |
| Performance | improves near-term congestion handling | compute cost/latency | limit scenarios; optimize sim runtime |
| Governance | decisions are auditable (tested options) | KPIs can be gamed | publish KPI definitions; include safety KPIs |

## Real-world anchors (what exists today)
- Research on real-time traffic digital twins describes minute-resolution synchronization and **parallel simulation instances** for evaluating actions before deploying control strategies. https://www.sciencedirect.com/science/article/pii/S1474034622003160
- Traffic management vendors explicitly describe simulation/digital-twin environments used to test signal timing plan updates and adaptive signal control impacts before field deployment. https://www.econolite.com/application-areas/virtual-simulation-modeling-and-validation/

## Evaluation checklist (practical)
- Queue spillback frequency / blocked-box risk
- Travel time distribution (median + p95)
- Stops/vehicle and hard-braking proxy (if available)
- Pedestrian delay (max + p95)
- Stability: number/size of timing changes per hour

## Sources
- https://www.sciencedirect.com/science/article/pii/S1474034622003160
- https://www.econolite.com/application-areas/virtual-simulation-modeling-and-validation/
