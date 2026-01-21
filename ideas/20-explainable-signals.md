# 20) Explainable Signals (Auditable Reasons for Timing Changes)

## What it is (precise)
“Explainable signals” means every non-trivial timing change (splits/offsets/cycle, phase sequence changes, special modes) ships with a short, auditable **reason code** and supporting facts, e.g.:

- “Prevent spillback on approach A (queue risk > threshold).”
- “Pedestrian surge detected; enabling LPI for 10 minutes.”
- “Detector health degraded; switching to fallback plan.”

This isn’t “explaining ML” in the abstract—it’s making the control system **accountable and transparent** in day-to-day operations, with artifacts operators and auditors can inspect.

NIST’s AI Risk Management Framework explicitly calls out trustworthiness characteristics including **Accountable and Transparent** and **Explainable and Interpretable**. [NIST AI RMF 1.0 (NIST AI 100-1)](https://doi.org/10.6028/NIST.AI.100-1)

## Why digital twins matter
A digital twin can generate explanations that are grounded in counterfactuals:

- “If we keep the current plan, the twin predicts spillback probability rises from 10% → 60% in 8 minutes.”
- “Switching to Plan B reduces p95 queue by 35% while respecting max pedestrian wait.”

This makes explanations *operational* (what metric/constraint drove the change) rather than purely technical.

## Easy explanation
The system doesn’t just change the lights; it also tells you (in one sentence) why it changed them, what evidence it used, and when it will reconsider.

## What is needed (data & infrastructure)
| Need | Typical options | Notes |
|---|---|---|
| A decision log | central system logging (append-only) | Store: before/after plan, timestamp, operator override, inputs used. |
| Reason-code taxonomy | small controlled vocabulary | Keep it simple (10–30 reasons) so it’s usable. |
| Evidence capture | KPIs + thresholds + sensor health | Store the minimal facts that justify the reason. |
| Guardrails & constraints | policy rules + safety minima | If a change violates a guardrail, block it and log. |
| Review tooling | dashboards + diffs | Support audits: “why did cycle length change yesterday at 17:20?” |

## Implementation plan (phased)
### Phase 0 — define the explainability contract (2–4 weeks)
- Define which actions require an explanation (cycle/split/offset, mode switch, fallback, override).
- Define reason codes + required evidence fields per reason.

### Phase 1 — logging + dashboards (4–8 weeks)
- Implement an append-only decision log.
- Add operator UI: last change, reason, evidence, and “revert” option.

### Phase 2 — twin-backed counterfactual explanations (6–12 weeks)
- For each recommended change, compute:
  - predicted KPI deltas,
  - constraint satisfaction report,
  - predicted risk flags (spillback, max delay exceedance).

### Phase 3 — governance + audits (ongoing)
- Periodic sampling: review a random set of changes and verify evidence quality.
- Track operator overrides and retrain/tune thresholds when overrides cluster.

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Accountability | easier to trust and approve automation | too much text becomes noise | reason codes + one-sentence summary + expandable details |
| Safety | encourages explicit constraints | “good story” could hide bad control | include immutable evidence fields + KPI deltas |
| Ops efficiency | faster troubleshooting | logging overhead | sample rates for high-frequency signals; store aggregates |
| Compliance | aligns with transparency expectations | unclear ownership of audits | assign roles + review cadence |

## Real-world anchors (what exists today)
- NIST AI RMF positions **Accountable & Transparent** and **Explainable & Interpretable** as core trustworthiness characteristics for AI systems. [NIST AI RMF 1.0](https://doi.org/10.6028/NIST.AI.100-1)

## Evaluation checklist (practical)
- % of changes with valid reason + evidence
- Mean time to diagnose incidents (before/after)
- Operator override rate (and reasons)
- Audit findings: missing/incorrect evidence fields

## Sources
- https://doi.org/10.6028/NIST.AI.100-1
