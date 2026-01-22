# 20) Explainable Signals (Auditable Reasons for Timing Changes)

## Brief Description
Explainable signals provide auditable reasons for timing changes, enhancing transparency and trust.

## Analogical Reference
Like a referee explaining a call in a game, signals justify their decisions with evidence.

## Comprehensive Information

Explainable Signals ensure every significant timing change in traffic lights comes with an auditable reason code and evidence, grounded in NIST's AI Risk Management Framework for accountability and transparency. This framework positions Explainable and Interpretable as key trustworthiness traits, aligning with organizational policies for monitoring, auditing, and change management. For instance, instead of a signal simply switching to red, it might log: "Reason Code 104: Pedestrian Surge Detected. Evidence: Sensor count increased by 150% in 2 minutes. Predicted Outcome: Reduces near-miss incidents by 40% without exceeding max green time."

Digital twins enhance this by providing counterfactual explanations, predicting outcomes like spillback risks or queue reductions. For example, a twin might simulate: "If we maintain current cycle, spillback probability rises to 60% in 8 minutes, causing gridlock." This makes decisions grounded in data, improving operator trust and public acceptance.

Research from Nature shows integrating XAI with YOLOv5 in autonomous vehicles improves safety and trust, achieving 99% accuracy with 1% miss rate by explaining object detections. Wiley's work on explainable reinforcement learning for traffic signals reduces complexity in RL models, making them more adoptable by providing insights into policy choices, such as why a particular action was taken to optimize flow.

ArXiv studies reveal XAI identifies key indicators like Longest Stop Duration and Total Jam Distance for anomaly detection in traffic flows, highlighting challenges like transitional data issues where post-hack recovery phases mimic normal traffic, and stealth attacks in low-volume conditions that evade detection due to lack of congestion patterns.

Implementation involves phased rollouts: defining contracts (e.g., which changes require explanation), logging (append-only decision logs), twin-backed counterfactuals (computing deltas like "Plan B reduces p95 queue by 35%"), and governance (periodic audits). Levels range from basic codes (e.g., "101: Detector Failure") to narrative provenance (full stories with operator notes).

Infrastructure needs include decision logs (storing timestamps, inputs, overrides), taxonomies (controlled vocabularies for reasons), evidence capture (KPIs, thresholds, sensor health), and review tools (dashboards for "why did cycle change at 17:20?").

From scraped sources, explainable RL frameworks optimize traffic flow with interpretable decisions, outperforming traditional methods by 5% in accuracy through better policy understanding. This builds on AI RMF Playbook's emphasis on governance, including documented policies, monitoring, and change management.

Real-World Examples include cities using explainable systems for incident response, where operators see reasons like "Emergency Vehicle Priority: Ambulance detected, green extended by 15s to avoid delay." Case studies from FHWA show reduced operator overrides by 25% with explanations, as decisions become more justifiable.

Technical Details involve XAI techniques like occlusion sensitivity (masking inputs to see impact) and LIME (explaining local predictions), applied to CNNs for traffic prediction. For instance, in a model misclassifying hacked signals, XAI reveals focus on non-congestion features, prompting retraining.

Future Scenarios in 5 years: Signals explain changes via apps, reducing confusion. In 15 years: Full AI integration with human oversight, where signals self-explain to regulators for compliance.

Challenges include verbosity (too much text overwhelming operators) mitigated by one-sentence summaries, and post-hoc risks (falsified explanations) addressed by immutable evidence generation at decision time.

Benefits extend to compliance with regulations like GDPR for data transparency, and improved cybersecurity by making anomalies explainable.

## Upsides and Downsides

### Positive Aspects on People's Lives in 5 Years
- Enhanced Safety: Reduced accidents by up to 20% through better compliance and understanding, saving lives and injuries.
- Efficiency: Faster incident resolution and smoother traffic flow, reducing daily commute times and stress.
- Trust Building: Increased public confidence in automated systems, leading to wider adoption and less frustration at intersections.

### Positive Aspects on People's Lives in 15 Years
- Equitable Mobility: Signals optimize for sustainability, equity, and integration with autonomous vehicles, creating fairer access to cities and economic opportunities.
- Economic Savings: Billions in losses from congestion avoided, boosting productivity and quality of life.
- Ethical AI Integration: Fully transparent systems ensure human oversight, preventing biases and promoting societal benefits like reduced emissions.

### Downsides in 5 and 15 Years
- Costs and Overhead: Higher implementation and maintenance expenses, with data privacy risks from logging detailed explanations.
- Over-Reliance and Vulnerabilities: Dependency on AI could lead to system failures or cyberattacks, exacerbating disruptions.
- Job Impacts: In 15 years, potential displacement of human traffic operators, shifting roles to AI oversight.

### Hard Things People Will Have to Overcome When Getting Used to It
- Adaptation to New Interfaces: Operators must learn to interpret reason codes and evidence, requiring training to avoid misinterpretation.
- Standardization Challenges: Aligning codes across vendors and ensuring portability, with risks of post-hoc storytelling if evidence isn't immutable.
- Ethical and Privacy Concerns: Balancing transparency with personal data protection, managing vast data streams, and adapting to dynamic urban changes.

## Implementation Plan (Phased)
### Phase 0 — Define the Explainability Contract (2–4 Weeks)
- Define explainable actions (e.g., cycle changes, mode switches) and reason codes (e.g., 101: Pedestrian Surge, 102: Spillback Risk).
- Specify evidence fields like sensor thresholds and predicted outcomes.
- Involve stakeholders: traffic engineers, safety officers, and data privacy experts to ensure compliance.

### Phase 1 — Logging + Dashboards (4–8 Weeks)
- Implement append-only decision logs capturing before/after states, timestamps, and operator overrides.
- Develop operator UIs with one-screen feeds showing last changes, reasons, and expandable evidence.
- Integrate with existing ATMS for real-time data feeds.

### Phase 2 — Twin-Backed Counterfactuals (6–12 Weeks)
- Compute predicted KPI deltas (e.g., queue reduction by 35%) and risk flags (e.g., spillback probability).
- Use simulations to generate narratives like "Switching plans reduces max delay from 120s to 80s."
- Validate with historical data to ensure accuracy.

### Phase 3 — Governance + Audits (Ongoing)
- Establish review cadences: weekly operator sampling, monthly audits for evidence completeness.
- Track operator overrides to refine thresholds, preventing clustering around poor decisions.
- Integrate with organizational policies from AI RMF Playbook for monitoring and change management.

## Technical Details
- Reason Codes: Controlled vocabulary (e.g., 100-series for safety, 200-series for efficiency).
- Evidence Capture: Immutable fields generated at decision time, including sensor health and twin predictions.
- XAI Integration: Techniques like SHAP for feature importance in ML models, explaining why certain inputs drove the decision.
- Guardrails: Policy rules preventing changes that violate safety minima, with automatic rollbacks.

## Real-World Examples
- In a city pilot, explainable signals reduced public complaints by 30% by providing reasons like "Extended green for transit to maintain schedule."
- FHWA case studies show explainable systems improving incident response times by 25%, as operators trust automated suggestions.
- European deployments use XAI for GDPR compliance, logging explanations without revealing personal data.

## Stakeholder Impacts
- Operators: Gain confidence with clear rationales, reducing stress and improving decision quality.
- Public: Builds trust in smart cities, especially in diverse communities wary of AI.
- Regulators: Easier audits and compliance with transparency laws.
- Vendors: Standardized reason codes facilitate interoperability across controllers.

## Future Scenarios
- In 5 Years: Signals explain changes via public dashboards, reducing confusion during events.
- In 15 Years: AI systems self-generate and verify explanations, integrating with autonomous vehicles for coordinated decisions.
- Potential Risks: Over-explanation leading to information overload; mitigated by customizable detail levels.

## MVP (Smallest Useful Deployment)
- Reason-code taxonomy (10-20 codes) with evidence schema.
- Log non-trivial changes (mode switches, cycle changes) with one-screen feed.
- Monthly audits sampling 50 events for completeness.

## Open Questions
- What is the minimal evidence set that remains useful without creating a logging burden?
- How to align reason codes across vendors for portability and consistency?
- How to prevent post-hoc storytelling by ensuring evidence is immutable and generated at decision time?

## Evaluation Metrics
- Percentage of changes with valid reason + evidence (target: >95%).
- Mean time to diagnose incidents (before/after implementation).
- Operator override rates (and categorized reasons).
- Audit findings: percentage of missing/incorrect evidence fields.
- Public trust surveys: pre/post adoption.

## Costs and ROI
- Development: $100k-$200k for logging and UIs.
- Annual Ops: $50k for audits and maintenance.
- ROI: Payback in 6-12 months via reduced overrides and faster resolutions, with long-term savings in liability reduction.
