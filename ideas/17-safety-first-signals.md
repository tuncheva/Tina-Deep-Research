# 17) Safety-First Signals (Optimize Near-Misses, Not Just Delay)

## What it is (precise)
**Safety-first signal control** treats safety as a first-class objective/constraint, optimizing signals to reduce **risky interactions** (near-misses, hard braking, red-light running likelihood, conflict exposure) rather than optimizing only vehicle delay or throughput.

The key technical move is to use **surrogate safety measures** (SSMs)—quantitative proxies for crash risk—because crashes are rare and slow to measure.

## Why digital twins matter
A digital twin is uniquely useful because it can:
- generate **trajectory-level** outputs (micro-simulation) to compute SSMs,
- run counterfactuals (what if we change permissive lefts / clearance / split?),
- rank candidate timing plans by combined mobility + safety criteria.

FHWA’s report on deriving surrogate safety measures from microscopic simulation explicitly defines/organizes SSMs like **Time To Collision (TTC)** and **Post-Encroachment Time (PET)** and discusses algorithms and simulation outputs needed for intersection safety assessment. This is a strong basis for “safety as an optimization target” in a twin. [FHWA RD-03-050](https://ntlrepository.blob.core.windows.net/lib/38000/38000/38015/FHWA-RD-03-050.pdf)

## Easy explanation
Instead of asking “How fast can we move cars?”, the controller asks “How can we reduce close calls and dangerous moves while still keeping traffic flowing?”

## Comparison table (safety levers vs what they reduce)
| Lever | Reduces | Typical cost | When to use |
|---|---|---|---|
| Longer all-red / clearance | crossing/entry conflicts | small added delay | high-risk geometry, night/rain |
| Protected-only lefts | left/through conflicts | longer cycle or added phases | peak turn surges, crash history |
| LPI / scramble | ped-turn conflicts | vehicle delay, longer cycles | high ped volumes, transit hubs |
| Speed-harmonized progression | rear-end braking waves | slightly longer travel time | corridors with shockwaves |
| Spillback prevention | blocked-box + risky merges | shifts green distribution | constrained storage, rail crossings |

## What safety-first can control (examples)
| Lever | Safety mechanism | Typical tradeoff |
|---|---|---|
| Longer all-red / clearance | reduces intersection-entry conflicts | small added delay |
| Protected-only left turns (peak) | reduces left/through crossing conflicts | can increase cycle length |
| Pedestrian head start / scramble | reduces turning conflicts with peds | vehicle delay |
| Speed harmonization progression | reduces harsh braking, rear-end risk | may reduce max throughput |
| Queue spillback prevention | reduces blocked-box and risky merges | may shift green time |

## Surrogate safety measures (SSMs) to use
FHWA’s SSM report defines and discusses surrogate measures and computational algorithms. Practical starting set:
- **TTC** (time to collision) for rear-end and crossing conflicts
- **PET** (post-encroachment time) for crossing conflicts
- **Hard braking / deceleration rate** proxies
- **Conflict counts** by type (crossing/merging/rear-end)

These are computable from either:
- micro-simulation trajectories (twin), or
- high-frequency detection/connected-vehicle telemetry (where available).

Source: [FHWA RD-03-050](https://ntlrepository.blob.core.windows.net/lib/38000/38000/38015/FHWA-RD-03-050.pdf)

## Implementation plan (phased)
### Phase 0 — pick safety proxies and targets
- Select 2–4 SSMs (TTC/PET + braking).
- Define policy constraints: min pedestrian service, max delay bounds.

### Phase 1 — data + calibration
- Validate detector and signal-state data quality.
- Calibrate the twin so conflicts/queues resemble observed patterns.

### Phase 2 — offline safety optimization
- Evaluate candidate timing changes (left-turn protection changes, clearance updates).
- Produce a “safety impact statement” per change (SSMs + delay changes).

### Phase 3 — controlled deployment
- Deploy in one corridor or one high-crash intersection.
- Monitor before/after SSMs plus operational KPIs.

### Phase 4 — adaptive safety constraints
- Add conditional logic (e.g., rain/night modes increase clearance, reduce permissives).
- Ensure fallbacks to conservative safe plans.

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Safety outcomes | targets risky interactions directly | proxies may not perfectly predict crashes | validate against crash history; conservative thresholds |
| Decision-making | makes tradeoffs explicit | can be politically sensitive (more delay) | person-delay framing; publish impact statements |
| Engineering | structured methods exist | needs trajectory data / detailed models | start with micro-sim + a small set of SSMs |

## MVP (smallest useful deployment)
- Choose **one high-risk intersection** and compute SSMs from micro-sim trajectories (twin).
- Implement **one safety lever** (e.g., protected-only lefts during peak or longer all-red) and quantify tradeoffs.
- Publish a “safety impact statement”: SSM deltas + delay/person-delay deltas.
- Add a weather/night conditional rule (ties to idea 14).

## Open questions
- Which SSMs correlate best with observed crash patterns for this city’s intersections?
- How to avoid “overfitting” to proxies that reduce conflicts in sim but not in reality?
- What is the acceptable mobility tradeoff per unit of safety improvement (policy decision)?

## Evaluation checklist
- SSMs (TTC/PET distributions) pre/post
- Hard-braking rates near intersection
- Delay by approach and pedestrian wait
- Longer-term: crash rate changes (lagging indicator)

## Sources
- https://ntlrepository.blob.core.windows.net/lib/38000/38000/38015/FHWA-RD-03-050.pdf
