# 20) Explainable Signals (Auditable Reasons for Timing Changes)

## What it is (precise)
“Explainable signals” means every non-trivial timing change (splits/offsets/cycle, phase sequence changes, special modes) ships with a short, auditable **reason code** and supporting facts, e.g.:

- “Prevent spillback on approach A (queue risk > threshold).”
- “Pedestrian surge detected; enabling LPI for 10 minutes.”
- “Detector health degraded; switching to fallback plan.”

This isn’t “explaining ML” in the abstract—it’s making the control system **accountable and transparent** in day-to-day operations, with artifacts operators and auditors can inspect.

NIST’s AI Risk Management Framework explicitly calls out trustworthiness characteristics including **Accountable and Transparent** and **Explainable and Interpretable**. [NIST AI RMF 1.0 (NIST AI 100-1)](https://doi.org/10.6028/NIST.AI.100-1)

NIST’s companion *AI RMF Playbook* also emphasizes organizational policies and procedures that establish **monitoring, auditing, review**, and **change management** for AI systems—useful framing for “explainable signals” as a governance + ops practice, not just UX. [NIST AI RMF Playbook (PDF)](https://airc.nist.gov/docs/AI_RMF_Playbook.pdf)

## Why digital twins matter
A digital twin can generate explanations that are grounded in counterfactuals:

- “If we keep the current plan, the twin predicts spillback probability rises from 10% → 60% in 8 minutes.”
- “Switching to Plan B reduces p95 queue by 35% while respecting max pedestrian wait.”

This makes explanations *operational* (what metric/constraint drove the change) rather than purely technical.

## Easy explanation
The system doesn’t just change the lights; it also tells you (in one sentence) why it changed them, what evidence it used, and when it will reconsider.

## Comparison table (levels of explainability)
| Level | What you log | Pros | Cons |
|---|---|---|---|
| L1: reason codes only | reason code + timestamp | very cheap; consistent | may lack context |
| L2: reason + evidence | code + thresholds + KPI deltas | supports audits | more data management |
| L3: counterfactuals | reason + evidence + twin deltas (A vs B) | strongest trust | compute + storage cost |
| L4: narrative + provenance | L3 + operator notes + approvals | governance-ready | risk of verbosity |

## What is needed (data & infrastructure)
| Need | Typical options | Notes |
|---|---|---|
| A decision log | central system logging (append-only) | Store: before/after plan, timestamp, operator override, inputs used. |
| Reason-code taxonomy | small controlled vocabulary | Keep it simple (10–30 reasons) so it’s usable. |
| Evidence capture | KPIs + thresholds + sensor health | Store the minimal facts that justify the reason. |
| Guardrails & constraints | policy rules + safety minima | If a change violates a guardrail, block it and log. |
| Review tooling | dashboards + diffs | Support audits: “why did cycle length change yesterday at 17:20?” |
| Governance plumbing | policy + audit cadence | Borrow from AI RMF Playbook concepts: documented policies, monitoring/auditing, and change management. [NIST AI RMF Playbook (PDF)](https://airc.nist.gov/docs/AI_RMF_Playbook.pdf) |

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

## Comparison table (what to explain)
| Decision type | Example | Minimum explanation | Why |
|---|---|---|---|
| Mode switch | storm-safe mode enabled | reason code + trigger | high impact |
| Plan selection | switched to Plan B | reason + expected KPI delta | auditability |
| Split/offset tweak | +5s mainline green | evidence thresholds | debugging |
| Fallback | detector failed → fixed plan | health evidence + duration | safety assurance |

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Accountability | easier to trust and approve automation | too much text becomes noise | reason codes + one-sentence summary + expandable details |
| Safety | encourages explicit constraints | “good story” could hide bad control | include immutable evidence fields + KPI deltas |
| Ops efficiency | faster troubleshooting | logging overhead | sample rates for high-frequency signals; store aggregates |
| Compliance | aligns with transparency expectations | unclear ownership of audits | assign roles + review cadence |

## Real-world anchors (what exists today)
- NIST AI RMF positions **Accountable & Transparent** and **Explainable & Interpretable** as core trustworthiness characteristics for AI systems. [NIST AI RMF 1.0](https://doi.org/10.6028/NIST.AI.100-1)

## MVP (smallest useful deployment)
- Create a reason-code taxonomy (10–20 codes) and a required evidence schema per code.
- Log only **“non-trivial” changes** at first (mode switches, cycle changes, plan selections).
- Provide a one-screen “last 10 changes” feed with reason + evidence + revert.
- Run a monthly audit sample (e.g., 50 events) for evidence completeness.

## Open questions
- What is the minimal evidence set that remains useful without becoming a logging burden?
- How do we align reason codes across vendors/controllers for portability?
- How to prevent post-hoc “storytelling” (ensure evidence is immutable and generated at decision time)?

## Evaluation checklist (practical)
- % of changes with valid reason + evidence
- Mean time to diagnose incidents (before/after)
- Operator override rate (and reasons)
- Audit findings: missing/incorrect evidence fields

## Sources
- https://doi.org/10.6028/NIST.AI.100-1
- https://airc.nist.gov/docs/AI_RMF_Playbook.pdf
