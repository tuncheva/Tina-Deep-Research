# 16) Fast-Forward Twin (Next 5 Minutes)

## What it is (precise)
Run many short-horizon simulations (seconds to a few minutes) from the **current live state** to choose the best near-term signal timing actions for the next 1–5 minutes.

This is essentially “model-predictive control with a digital twin”: evaluate candidate actions, pick the best, repeat frequently.

## Why digital twins matter
This idea is the digital twin itself used as a control loop:
- represent current queues/phase states,
- forecast outcomes of alternative split/offset/phase decisions,
- choose the action that best meets a KPI bundle (delay, stops, spillback risk, safety constraints).

Short horizons keep it computationally feasible and reduce sensitivity to long-term demand uncertainty.

## Easy explanation
Every few minutes the twin “fast-forwards” reality under a few safe options and picks the option that looks best.

## What is needed (data & infrastructure)
| Need | Typical options | Notes |
|---|---|---|
| Live state ingest | controller phase states, detector data, queue estimates | Must initialize the twin from current conditions. |
| Candidate action set | a small menu of safe timing tweaks | Avoids an unbounded search.
| KPI scoring | delay, stops, spillback risk, ped delay, safety caps | Must include hard constraints.
| Compute + latency budget | edge server, cloud, or TMC compute | Needs predictable runtime.
| Deployment interface | central system push or local controller API/NTCIP | Must be reliable and auditable.

## Implementation plan (phased)
### Phase 0 — establish “safe action menu” (2–6 weeks)
- Define allowable changes per cycle (split +/- seconds, offset shift bounds).
- Hard constraints: ped mins, clearance, max cycle, max side-street delay.

### Phase 1 — shadow-mode rollout (4–8 weeks)
- Run fast-forward rollouts but do not actuate.
- Compare predicted vs observed KPIs; calibrate queue estimation.

### Phase 2 — controlled deployment (4–12 weeks)
- Start with one corridor or a small network.
- Only allow changes when confidence is high; otherwise keep current plan.

### Phase 3 — continuous refinement (ongoing)
- Expand action menu and incorporate incident/event triggers.
- Add robustness: evaluate “best” under uncertainty bands (demand, detector noise).

## Comparison table (action menu design)
| Menu style | What it includes | Strength | Weakness |
|---|---|---|---|
| Fixed templates | 3–5 pre-approved actions | safe + explainable | limited optimality |
| Parameterized tweaks | bounded split/offset changes | more flexible | harder to verify |
| Mode switches | plan selection (baseline/incident) | robust | coarse control |
| Hybrid | templates + small tweaks | best of both | complexity |

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Adaptivity | responds quickly to changing queues | requires good state estimation | invest in detection + calibration |
| Risk control | can enforce constraints per rollout | computational failures/latency | strict runtime budget; fallback plan |
| Explainability | chooses from a known action set | may miss global optimum | broaden menu gradually |

## Real-world anchors (what exists today)
The FHWA Traffic Signal Timing Manual describes adaptive traffic signal control as a loop that (1) collects real-time data, (2) evaluates alternative strategies on a model of traffic behavior, (3) implements the “best” strategy, and (4) repeats continuously. That is the same control pattern this idea applies, with an explicit “fast-forward” digital twin to score short-horizon alternatives. [FHWA Traffic Signal Timing Manual (Chapter 9)](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm)

## MVP (smallest useful deployment)
- Pick a corridor and run rollouts with a **fixed, small action menu** (3–5 actions).
- Enforce a strict compute SLA (e.g., **p95 < 30s**) and fall back to baseline when missed.
- Start in **recommend-only** mode with a daily report of “what it would have done”.
- Add robustness scoring: choose actions that perform well under ± demand and detector-noise bands.

## Open questions
- How often should we re-plan: every cycle, every 2–5 minutes, or event-triggered?
- What is the safest interface for applying changes (plan select vs per-cycle split tweaks)?
- Which uncertainty sources dominate (demand, queues, turning rates, detector noise) and how to bound them?

## Evaluation checklist (practical)
- Prediction error (queue, delay) vs observed
- Runtime/latency distribution (p95)
- Safety constraint violations (should be zero)
- Travel time reliability improvement (p95)
- Operator override rate and reasons

## Sources
- https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter9.htm
