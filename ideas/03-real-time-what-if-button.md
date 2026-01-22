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

## Comparison table (how to run the “what-if”)
| Pattern | How it works | Pros | Cons |
|---|---|---|---|
| Operator-triggered button | operator runs scenarios on demand | low compute cost | misses fast-moving incidents |
| Continuous rolling evaluation | always simulating a small action set | always ready | higher compute + monitoring needs |
| Recommend-only (shadow) | shows best option but doesn’t actuate | safest for calibration | benefits delayed |
| Closed-loop (auto) | system applies action with rollback | fastest response | requires strong guardrails + trust |

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

## Operator-ready decision output (what the button should show)
A “what-if” tool is only useful if the output is *actionable* in 10–30 seconds.

| Output | Example | Why it matters |
|---|---|---|
| Recommended action | “Apply Plan B for 15 min” | removes guesswork |
| One-sentence reason | “Spillback risk on approach A > 0.6” | aligns with operator mental model |
| Evidence snapshot | current queue estimate + trend | helps trust the recommendation |
| Counterfactual delta | “p95 queue −35%, ped delay +6s” | explicit tradeoff |
| Guardrail check | “max ped wait OK; max cycle OK” | prevents unsafe changes |
| Confidence | “medium (sensors degraded)” | prevents over-trust |

## Practical guardrails (to keep what-if safe)
- **Bound the action space**: allow only a small, pre-approved set of actions (e.g., ±5s splits, select among vetted plans).
- **Rollback by default**: every action has an expiry and an automatic revert if KPIs drift.
- **Staleness rules**: if state estimation is older than N seconds or detector health is poor, restrict to conservative actions.
- **Audit log**: store the candidate set, ranking, and the reason for the chosen action (pairs well with [20) Explainable Signals](../ideas/20-explainable-signals.md)).

## MVP (smallest useful deployment)
- Provide operators a **single corridor “sandbox”** with 3 candidate actions:
  - baseline (do nothing)
  - +5s mainline green
  - “anti-spillback” metering preset
- Run **parallel 5-minute rollouts** every 1–2 minutes and show the top-3 ranked options.
- Require **operator confirmation** and include an automatic rollback timer.

## Open questions
- How do we initialize the twin state robustly (queue estimation) when detectors are sparse?
- What is the right tradeoff between **compute latency** and decision quality (how many scenarios, how often)?
- Which KPIs should be “hard constraints” vs “soft objectives” for operator choice?

## Evaluation checklist (practical)
- Queue spillback frequency / blocked-box risk
- Travel time distribution (median + p95)
- Stops/vehicle and hard-braking proxy (if available)
- Pedestrian delay (max + p95)
- Stability: number/size of timing changes per hour

---

## Deep dive (engineering): what the button actually runs
The “what-if” button is not just a simulator; it is a **decision pipeline**.

### Step 1: snapshot the current state (“nowcast”)
Inputs that must be consistent at a timestamp:
- controller states: active phase/ring, time-in-phase, coordination plan,
- detector/actuation states: calls, occupancies, counts (with health),
- probe travel times / speeds (if available).

The twin turns this into a state estimate:
- queue lengths by approach,
- arrival profiles for the next minutes,
- spillback probability bounds.

If you cannot make a stable snapshot, you should not run aggressive actions; instead restrict to conservative presets.

### Step 2: generate the candidate action set (small + safe)
Treat actions like “medication doses,” not free-form optimization.

Candidate families:
- split nudges: ±(2–8)s on key phases (respecting min greens/peds),
- offset nudges: ±(1–6)s corridor shift,
- metering preset: cap release from upstream when downstream storage is low,
- phase service mode toggle: recall/minimums in degraded detection cases.

### Step 3: run parallel rollouts (faster-than-real-time)
Parallel Digital Twin Instances (DTIs) run the candidate set and output KPIs.

Practical constraints:
- time horizon: 2–10 minutes (too long becomes unstable and sensitive),
- cadence: every 30–120s,
- number of scenarios: 3–20 (latency budget).

### Step 4: score with hard constraints + soft objectives
Use a **two-stage decision rule**:
1. Filter out candidates that violate constraints (max ped wait, storage, safety caps).
2. Rank the remaining by a weighted score (person delay, spillback risk, transit delay, etc.).

Cross-link: see [`ideas/04-city-rules-built-into-signals.md`](ideas/04-city-rules-built-into-signals.md) for encoding constraints.

### Step 5: actuation + rollback contract
Every recommended action should ship with:
- expected benefits and risks,
- expiry time,
- rollback trigger thresholds,
- operator accountability (who approved).

---

## Deep dive (UX): the “10-second card”
Operators need *one card* that answers: what, why, risk.

Card template:
- **Action**: Apply “Plan B (anti-spillback)” for 10 minutes.
- **Why**: Downstream storage predicted to exceed 85% within 3 minutes.
- **Tradeoff**: +8s ped delay (p95) near stopline; −30% blocked-box risk.
- **Confidence**: Medium (video detector noise).
- **Safety check**: OK (ped mins satisfied; max cycle change within cap).

Cross-link: see [`ideas/20-explainable-signals.md`](ideas/20-explainable-signals.md).

---

## Deep dive (failure modes): when the button hurts you
| Failure mode | What you see | Guardrail |
|---|---|---|
| Stale snapshot | sim says “all good” but field worsens | staleness cutoff; conservative presets |
| Model mismatch | predicted queue drops; actual queue grows | continuous calibration; shadow mode |
| Automation bias | operators accept blindly | show uncertainty; require confirm early |
| Thrashing | frequent small changes cause instability | minimum dwell; change-rate caps |

---

## Metrics (product + engineering)
- Median decision latency (snapshot→recommendation)
- Recommendation acceptance rate (by operator)
- Rollback rate and rollback causes
- Prediction error (queue/travel time) on 1–5 min horizon
- Stability: plan changes/hour and average delta size

---

## Operator integration
Cross-link: see [`ideas/08-operator-option-menu.md`](ideas/08-operator-option-menu.md)

Recommended integration:
- “What-if” button is available only when health is `healthy` or `degraded` with low risk.
- Present the top-3 candidates + “do nothing” as an explicit baseline.
- Store the full candidate set and the scoring breakdown for audits.

---

## Related topics
- [`ideas/16-fast-forward-twin-next-5-minutes.md`](ideas/16-fast-forward-twin-next-5-minutes.md) — forecasting framework that powers short-horizon evaluation.
- [`ideas/01-green-wave-plans.md`](ideas/01-green-wave-plans.md) — applying what-if to offsets/cycles and coordination stability.
- [`ideas/02-self-healing-intersections.md`](ideas/02-self-healing-intersections.md) — restricting actions under degraded health.
- [`ideas/04-city-rules-built-into-signals.md`](ideas/04-city-rules-built-into-signals.md) — constraints and policy objective encoding.

## Sources
- https://www.sciencedirect.com/science/article/pii/S1474034622003160
- https://www.econolite.com/application-areas/virtual-simulation-modeling-and-validation/
