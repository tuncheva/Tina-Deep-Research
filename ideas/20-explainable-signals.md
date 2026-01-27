# 20) Explainable Signals (Auditable Reasons for Timing Changes)

## Catchy Explanation
Like a referee explaining a call, an explainable signal doesn’t just change timing—it records a short, auditable “why” with evidence.

## What it is (precise)
**Explainable signals** attach an auditable explanation to each non-trivial timing decision (plan switch, major split/offset change, mode transition, priority action). Explanations are not free-form storytelling; they are structured **reason codes + evidence** generated at decision time:
- reason code taxonomy (controlled vocabulary)
- key inputs (sensor/health signals, thresholds)
- predicted deltas (twin or model outputs)
- constraints that were binding (e.g., ped mins)
- operator overrides and notes

This supports accountability, faster diagnosis, and governance aligned with risk management frameworks.

## Benefits
- **Trust and accountability**: operators and auditors can see why changes happened.
- **Faster incident diagnosis**: root cause analysis becomes searchable.
- **Safer automation**: discourages “mystery optimization” and supports rollback.
- **Compliance support**: easier to document decision processes.

## Challenges
- **Information overload**: explanations must be brief by default.
- **Post-hoc risk**: explanations must be generated with immutable evidence at decision time.
- **Standardization**: codes should be consistent across vendors and corridors.
- **Privacy**: evidence must avoid exposing personal data.

## Implementation Strategies

### Infrastructure Needs
- **Explainability contract**: which actions require explanations and at what level of detail.
- **Reason code taxonomy**: controlled vocabulary (e.g., 100 safety, 200 efficiency, 300 faults).
- **Append-only decision log**: timestamps, inputs, action, constraints, reason, evidence.
- **Dashboards**: “why did this change at 17:20?” query UI.
- **Twin-backed counterfactuals** (optional): “if we did nothing, spillback risk rises to 60%.”

### Detailed Implementation Plan
#### Phase 1: Define the Explainability Contract and Taxonomy (Weeks 1–4)
The agency should begin by defining which actions require explanations (for example, any plan switch, any split change above a threshold, any mode transition, and any priority action), because explainability must be scoped to be operationally feasible. The team should define a minimum evidence schema that can be generated automatically at decision time, including timestamps, location, current mode, triggering signals, thresholds crossed, and binding constraints, because post-hoc explanations are not defensible. The team should define a controlled reason code taxonomy that is small enough to be usable (for example, 10–50 codes) and should include categories for safety, efficiency, faults, policy compliance, and operator override.

- **Roles**: program owner (scope and governance), operations (usability), traffic engineering (action definitions), data/engineering (schema), privacy/security (evidence constraints).
- **Deliverables**: explainability contract, reason code taxonomy v0, evidence schema, and retention policy.
- **Risks**: taxonomy becomes too large; evidence schema requires data that is not available.
- **Acceptance checks**: contract is signed off and every required evidence field has a defined data source.

#### Phase 2: Implement Append-Only Logging and the Operator “Timeline” UI (Weeks 5–10)
The engineering team should implement an append-only decision log that captures every explained action in a tamper-resistant way (for example, write-once storage or integrity checks), because auditability depends on evidence integrity. The team should build an operator-facing timeline UI that allows searching by intersection, corridor, time range, and reason code, because the primary operational value is being able to answer “what changed and why” quickly. The team should integrate log entries with operator notes and overrides so that human decisions are recorded in the same system.

- **Roles**: software engineer (logging service), data engineer (storage), operations (UI requirements), security (integrity controls), QA (testing).
- **Deliverables**: operational logging pipeline, timeline UI, and access control rules.
- **Risks**: logs become noisy without good filtering; weak access controls expose sensitive operational data.
- **Acceptance checks**: >95% of relevant actions appear in the log with valid schema and the UI supports rapid queries.

#### Phase 3: Evidence Quality, Counterfactuals, and Integration with Decision Systems (Weeks 11–18)
The team should enrich explanations with predicted KPI deltas and binding constraints whenever a decision is driven by a model or a twin, because operators need to see what the system thought would happen. If a digital twin is available, the team can add counterfactual summaries such as “if no change, spillback probability increases,” but it should keep counterfactuals optional and clearly labeled with uncertainty. The team should integrate explainability into each decision-producing subsystem (what-if tools, fast-forward twin, self-healing mode transitions, priority credits) so explanations are consistent rather than implemented as an afterthought.

- **Roles**: modeling team (predicted deltas), software engineer (integration), operations (usability), privacy reviewer (evidence minimization).
- **Deliverables**: evidence-enriched explanation records, counterfactual integration (optional), and updated reason code usage guidance.
- **Risks**: predicted deltas may be misleading if state estimation is weak; counterfactuals can increase compute and complexity.
- **Acceptance checks**: predicted deltas are present for model-driven actions and the system clearly indicates confidence/uncertainty.

#### Phase 4: Governance, Audits, and Continuous Improvement (Ongoing)
The agency should establish a governance process where operators and engineers review samples of decision logs weekly and run completeness audits monthly, because explainability must remain reliable as systems evolve. The team should track override patterns and reason code distributions to identify common failure modes and training needs, and it should refine taxonomies and evidence schema carefully through versioned changes. The agency should also publish internal (and optionally public) summaries that demonstrate accountability without exposing sensitive details.

- **Roles**: audit owner (governance), operations (weekly sampling), engineering (schema maintenance), analyst (trend reporting), privacy/security (review).
- **Deliverables**: audit reports, taxonomy updates (versioned), and periodic governance summaries.
- **Risks**: audits become checkbox exercises; taxonomy drift reduces comparability across time.
- **Acceptance checks**: audit completeness stays above target and operators report reduced time-to-diagnosis for incidents.

### Choices
- **Reason codes only**: minimal overhead.
- **Reason + evidence**: recommended baseline.
- **Narratives + counterfactuals**: highest value but more work.

## Technical Mechanics

### Key Parameters
- Reason code taxonomy size (10–50 codes)
- Evidence schema fields (health, thresholds, predicted deltas)
- Retention policy

### Guardrails
- Immutable evidence at decision time.
- Privacy-preserving evidence (aggregates).
- Explanations must not justify constraint violations.

## MVP Deployment
- 10–20 reason codes.
- Log all plan switches and mode changes.
- One timeline UI + monthly audit.

## Evaluation
- % of changes with valid reason + evidence (target >95%).
- Mean time to diagnose incidents.
- Operator override rate change.

## References / Standards / Useful Sources
- NIST AI RMF 1.0: https://doi.org/10.6028/NIST.AI.100-1
- NIST AI RMF Playbook (PDF): https://airc.nist.gov/docs/AI_RMF_Playbook.pdf

---

Cross-links: Related ideas include operator option menu, what-if button, and self-healing intersections.
