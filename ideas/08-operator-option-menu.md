# 08) Control-Room “Future Previews” (Operator Option Menu)

## What it is (precise)
This idea is an **operator-centered decision support workflow** where the system presents **3–5 vetted control options** (timing plans, offsets, priority policies, incident response actions) along with predicted outcomes and risk flags, rather than making one opaque “optimal” change.

The key requirement: each option must be **safe and deployable** (meets constraints), and the UI must communicate **uncertainty**.

## Why digital twins matter
A digital twin enables:
- rapid **forecasting** of near-future conditions under each option,
- evaluation of KPIs (delay, queue spillback, reliability, safety proxies),
- scenario testing for incidents/events.

Aimsun describes a real-time predictive decision support solution that provides forecasts in minutes and can return response plans ranked by KPIs, which directly aligns with the “menu of options” approach. [Aimsun Live](https://www.aimsun.com/aimsun-live/)

FHWA’s decision support report frames how DSTs integrate into traffic management systems and describes online tools that can help predict outcomes of decisions, which supports building this as a TMC capability (not just an algorithm). [FHWA-HRT-23-071](https://rosap.ntl.bts.gov/view/dot/72448/dot_72448_DS1.pdf)

## Easy explanation
Instead of the system changing signals by itself, it shows operators a short list of safe choices (like “Plan A, B, C”) and what each would likely do to congestion and queues.

## Comparison table (operator menu vs automation)
| Approach | Who decides | Strength | Weakness |
|---|---|---|---|
| Full automation | algorithm | fastest response | trust + audit concerns |
| Operator option menu (this) | operator selects | accountable + human-in-the-loop | requires good UI + training |
| Recommend-only | operator | safest rollout | slower impact |
| Playbooks only | operator | simple | limited flexibility |

## What the option cards contain (recommended)
| Field | Example |
|---|---|
| Option name | “Hold coordination + extend EB green” |
| Deployability | “Allowed now” / “Blocked (ped constraint)” |
| One-sentence reason | “Spillback risk rising on NB approach” |
| KPIs | travel time, stops, max queue, bus reliability |
| Risk flags | spillback risk, fairness impact, low data quality |
| Confidence | High / Medium / Low + reason |
| Rollback | “Revert to baseline plan in 10 min if KPI worsens” |
| Audit link | “View evidence + who approved” |

## Practical guardrails (so operator choice stays safe)
- **Safe envelope**: every card must pass hard constraints (ped mins, clearance, max cycle, max queue/spillback policy).
- **Limited levers**: hide “expert-only” actions unless an incident mode is declared.
- **Anti-flapping**: if operators switch plans frequently, require a minimum dwell time or a supervisor confirmation.
- **Evidence at click-time**: the UI must snapshot the KPIs/health that justified the recommendation (pairs well with [20) Explainable Signals](../ideas/20-explainable-signals.md)).

## Implementation plan (phased)
### Phase 0 — decide what operators can actually deploy
- Inventory controllable actions (timing plan selection, offset changes, priority toggles).
- Define “safe envelope” constraints.

### Phase 1 — build the option generator
- Create templates that produce options from a known library:
  - conservative baseline,
  - performance-oriented,
  - transit-priority,
  - incident containment (spillback prevention).

### Phase 2 — twin-based scoring
- For each option, run short-horizon simulation/forecast (5–30 minutes).
- Compute KPIs + risk flags.

### Phase 3 — operator UI + workflow
- Present 3–5 options with clear tradeoffs.
- Require explicit selection and log decision.

### Phase 4 — learning + auditing
- Compare predicted vs actual outcomes; update model calibration.
- Maintain an audit log of who selected what and why.

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Human factors | reduces black-box automation | decision burden on operators | keep options small; presets |
| Safety | ensures options are constrained | UI mistakes can still happen | deployability checks; confirmations |
| Trust | supports transparency & accountability | uncertainty hard to explain | show confidence + worst-case |

## MVP (smallest useful deployment)
- Implement **option cards** for a single corridor with 3 options: baseline, congestion relief, spillback prevention.
- Require a **deployability gate** (constraints check) that can only return `Allowed` or `Blocked`.
- Log: displayed options, selected option, operator ID, and before/after KPIs.
- Add a **“confidence explainer”** (e.g., “Low: detector health degraded on 2/8 approaches”).

## Open questions
- What is the safest default when confidence is low: “do nothing” or “conservative fallback”?
- How do we avoid choice overload while still covering common incident types?
- What is the best workflow for shared accountability (operator vs algorithm) in audits?

## Evaluation checklist
- Time-to-decision in the control room
- Frequency of unsafe/invalid recommendations (should be zero)
- Prediction accuracy (forecast vs observed)
- Operator adoption and override rate

## Sources
- https://www.aimsun.com/aimsun-live/
- https://rosap.ntl.bts.gov/view/dot/72448/dot_72448_DS1.pdf
