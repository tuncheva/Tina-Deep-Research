# 04) City Rules Built Into Signals: Policy-Driven Traffic Control

## Catchy Explanation
Instead of hoping every timing tweak “remembers” city priorities, you bake the priorities directly into the controller: the signal is allowed to optimize traffic only inside the city’s rulebook.

## What it is (precise)
**City rules built into signals** means encoding policy goals (safety, accessibility, transit reliability, equity, climate/noise goals) as **hard constraints** and **auditable objectives** in the signal control stack. Examples include: maximum pedestrian wait, required leading pedestrian interval (LPI) in defined zones, minimum transit service, school-zone timing rules, or fairness caps to avoid repeatedly harming the same side street. A digital twin is used to test candidate rule sets, quantify tradeoffs, and generate “policy impact statements” before deployment. The system also needs an exception workflow (operator overrides with reason codes) and periodic audits.

## Benefits
- **Policy compliance by design**: rules are enforced consistently, shift-to-shift.
- **Multimodal balance**: explicit support for pedestrians, bikes, transit.
- **Transparency and governance**: decisions map back to an approved “policy contract”.
- **Safer operations**: safety constraints can be made non-negotiable.

## Challenges
- **Tradeoffs become explicit**: some users will experience more delay.
- **Political alignment**: choosing priorities and thresholds is hard.
- **Sensor dependency**: enforcement often needs reliable multimodal detection.
- **Constraint tuning**: too strict → brittle control; too loose → no effect.

## Implementation Strategies

### Infrastructure Needs
- **Policy engine**: representation of constraints/objectives (hard vs soft) and rule sets by context (time-of-day, zone, weather).
- **Multimodal sensing**: pedestrian calls/volumes, bike detection, transit AVL/TSP requests.
- **Controller capability**: ability to implement LPI, TSP, phase holds, plan switching.
- **Twin + evaluation**: simulate rule sets and produce KPI + compliance reports.
- **Audit logs**: reason codes, evidence capture, and override tracking.

### Detailed Implementation Plan
#### Phase 1: Build the Policy Contract (Weeks 1–8)
The city should begin by convening the groups that actually own the policy outcomes, including traffic engineering, transit operations, Vision Zero or safety leadership, accessibility/ADA representatives, legal counsel, IT/security, and a community liaison. The group should translate high-level goals into measurable rules that can be tested, such as maximum pedestrian wait in priority zones, mandatory leading pedestrian interval (LPI) with a minimum duration, bounded transit priority rules, and context-specific constraints during school hours or severe weather. The group should explicitly separate non-negotiable rules (hard constraints) from “optimize within limits” objectives (soft weights), because implementers cannot build a defensible system if the rules are ambiguous. The group should also define an exception policy that specifies who can override a rule, for how long, what reason codes are required, and how overrides are reported, because real operations require controlled flexibility.

- **Roles**: policy owner (city delegate), traffic engineering lead (technical translation), transit operations lead (TSP constraints), legal (governance and defensibility), accessibility reviewer (ped rules), data/analytics lead (measurement plan), community liaison (communications).
- **Deliverables**: signed policy contract (versioned), formal rule definitions and thresholds, compliance test plan, and override governance.
- **Risks**: goals may conflict; thresholds may not be measurable with existing sensors; political pressure may change priorities mid-project.
- **Acceptance checks**: each rule has a metric, threshold, and measurement method; hard constraints are clearly identified; override workflow is approved and auditable.

#### Phase 2: Measurement Baseline and Data Readiness (Weeks 9–16)
The engineering and analytics teams should establish baseline measurement before enforcing any rules, because without baseline distributions it is impossible to defend whether the project improved or harmed outcomes. The team should identify the sensors and data feeds required for each rule (ped calls and actuation logs, bike detection, transit AVL/TSP request logs, travel times, and controller phase state) and should document gaps that must be addressed in procurement or field work. The team should then stand up performance monitoring with ATSPM-style measures and multimodal metrics so compliance and impacts can be tracked continuously rather than by occasional studies. Finally, the team should publish a baseline report that describes existing compliance rates (where applicable), delay distributions for pedestrians and side streets, and any known hotspots that will be used to evaluate the pilot.

- **Roles**: data engineer (pipelines), analyst (baseline), traffic engineer (metric definitions), IT/security (data access control), operations lead (dashboard needs).
- **Deliverables**: baseline compliance/impact report, data quality assessment, and policy dashboards.
- **Risks**: missing data prevents compliance measurement; weak multimodal coverage creates biased outcomes.
- **Acceptance checks**: baseline exists for each priority rule, and any measurement gap has an explicit mitigation plan and timeline.

#### Phase 3: Implement the Rule Engine and Controller Mappings (Weeks 17–28)
The software team should implement a rule representation that is versioned and reviewable (for example, configuration files with approvals) so that changes to city policy are traceable and reversible. The engineering team should build a “policy compiler” layer that maps rule definitions to concrete controller settings and plan behaviors, because most policy is enforced through specific signal timing parameters such as minimum greens, phase sequence restrictions, LPI enablement, and plan switching rules. The team should implement static validation that detects contradictory or impossible rules before deployment, and it should implement runtime monitoring that detects violations or near-violations in the field. The team should also enforce access controls so only authorized users can modify rule sets, and it should log the active rule version and the evidence used for any rule-triggered change.

- **Roles**: software engineer (rule engine), traffic engineer (controller mapping), QA engineer (tests), security engineer (access control), operations lead (override workflow).
- **Deliverables**: rule engine service, rule repository (versioned), compile/mapping layer, unit and integration tests, and audit log schema.
- **Risks**: controller limitations prevent implementing certain rules; rule conflicts produce brittle behavior.
- **Acceptance checks**: rules validate cleanly, a representative set compiles to controller configurations, and tests cover pedestrian and safety invariants.

#### Phase 4: Twin Validation and Stakeholder Review (Weeks 29–40)
The modeling team should calibrate a twin for the pilot area and should simulate the rule set under normal days, event surges, and sensor degradation, because real operations include both demand variability and imperfect detection. The analytics team should generate a policy impact statement that quantifies tradeoffs by approach, time-of-day, and mode, because policy-driven control often intentionally shifts delay to meet safety or equity goals. The stakeholder group should review the results and should confirm that observed tradeoffs are acceptable and aligned with the policy contract, and it should adjust thresholds and weights where needed. Finally, the team should define rollout guardrails and rollback criteria that allow operators to revert to a safe baseline plan if compliance monitoring indicates unexpected outcomes.

- **Roles**: modeler (simulation), analyst (impact statement), policy owner (review and sign-off), community liaison (communications).
- **Deliverables**: simulation report, policy impact statement, rollout/rollback plan, and updated rule set if needed.
- **Risks**: stakeholders may reject explicit tradeoffs; the twin may not match rare pedestrian or transit patterns.
- **Acceptance checks**: stakeholder sign-off is documented, rollback criteria are clear, and operator procedures are ready.

#### Phase 5: Pilot Activation and Public Reporting (Weeks 41–52)
The operations team should begin in assisted mode, where the system monitors compliance and recommends changes but operators confirm enforcement actions, because this builds trust and surfaces process gaps. The team should then activate rule enforcement by pushing compiled configurations to controllers and validating on-street behavior through spot checks and automated compliance measures. The communications team should publish regular, plain-language reporting that explains what improved and what became slower, because transparency is critical when policy-driven systems intentionally re-balance outcomes. After the pilot, the city should conduct an after-action review and should revise thresholds, weights, and measurement methods based on observed results.

- **Roles**: operations (activation), traffic engineering (on-call adjustments), analyst (reporting), communications lead (public outputs), legal/policy owner (governance).
- **Deliverables**: pilot results report, compliance dashboard, revised policy contract version if needed, and operator runbook.
- **Risks**: public controversy; unintended delay shifts to neighborhoods not represented in planning.
- **Acceptance checks**: hard-rule compliance meets target (for example, >95%), overrides are logged with reasons and expirations, and no safety violations are observed.

#### Phase 6: Scale and Governance (Ongoing)
The city should scale policy-driven control by district or corridor templates, because implementing and validating rules requires local phasing and pedestrian timing knowledge that is not uniform across the network. The program should schedule quarterly reviews of compliance and equity distributions and should run an annual audit that verifies rule versions, override patterns, and measurement integrity. The engineering team should maintain a change-management process for “breaking” rule updates and should ensure that rule sets can be rolled back quickly if unintended effects are discovered. Over time, the city should treat the rule registry as part of its transportation governance, similar to how design standards are managed.

- **Roles**: program owner (governance), audit committee (quarterly review), operations (daily monitoring), engineering (maintenance and upgrades).
- **Deliverables**: citywide rule registry, audit reports, expansion roadmap, and change-control procedures.
- **Risks**: policy churn; data quality drift; inconsistent enforcement across controller platforms.
- **Acceptance checks**: audits show stable compliance, changes are versioned and reversible, and equity monitoring is embedded in regular reporting.

### Choices
- **Hard-constraint-first**: safety/accessibility non-negotiable.
- **Weighted objectives**: tuneable priorities with guardrails.
- **Mode-based**: context-specific rule bundles (school hours, storms).

## Technical Mechanics

### Key Parameters
- Constraint thresholds (max ped wait, LPI duration, transit delay caps)
- Objective weights (delay vs stops vs emissions vs equity)
- Override limits (duration, scope)

### Guardrails
- Always enforce pedestrian minimums/clearances.
- Require reason codes for overrides and plan switches.
- Run periodic compliance audits.

## MVP Deployment
- One zone + 3–5 rules.
- Assisted mode with operator override.
- Monthly compliance report.

## Evaluation
- Compliance rate per rule (target >95%).
- Ped delay distribution, transit reliability, side-street delay.
- Safety proxies (conflict measures if available).
- Equity distribution (benefits/harms by neighborhood/corridor).

## References / Standards / Useful Sources
- FHWA Traffic Signal Timing Manual: https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf
- FHWA Leading Pedestrian Interval (LPI) fact sheet: https://highways.dot.gov/sites/fhwa.dot.gov/files/2022-06/04_Leading%20Pedestrian%20Interval_508.pdf
- NIST AI RMF 1.0: https://doi.org/10.6028/NIST.AI.100-1

---

Cross-links: Related ideas include safety-first signals, privacy-friendly learning, and what-if button.
