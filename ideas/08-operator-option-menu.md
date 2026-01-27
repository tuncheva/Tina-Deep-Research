# 08) Operator Option Menu: Human-Centered Decisions

## Catchy Explanation
Instead of throwing operators a hundred charts, the system offers a short “menu” of safe choices—each with a predicted outcome—so humans stay in control without flying blind.

## What it is (precise)
An **operator option menu** is a human-in-the-loop decision support interface where a digital twin (or fast predictor) generates and ranks a small set of **vetted control options** (plan switch, bounded split/offset tweaks, special mode activation). Each option is scored on a KPI bundle (delay, p95 queues, spillback risk, pedestrian delay, safety proxies) and displayed as “cards” with tradeoffs, confidence, and constraints. The menu enforces a **safe envelope** (hard constraints) and includes confirmation, rollback, and audit logging.

## Benefits
- **Accountable decisions**: operators see what’s being optimized and what tradeoffs exist.
- **Safer deployment**: options are pre-checked against constraints.
- **Faster action**: reduces time spent inventing changes under pressure.
- **Training-friendly**: helps new operators learn typical levers and effects.

## Challenges
- **UI overload**: too many options or too much evidence slows response.
- **Trust calibration**: need to communicate uncertainty and model limitations.
- **Option quality**: garbage-in/garbage-out if generator produces weak candidates.
- **Bias and preference drift**: operators may prefer familiar options without evidence.

## Implementation Strategies

### Infrastructure Needs
- **Option library**: parameterized templates for allowed actions.
- **Forecasting**: digital twin rollouts or fast surrogate model.
- **Safety envelope**: constraints (ped mins/clearance, max cycle, coordination caps).
- **Audit logs**: who chose what, when, and why.
- **ATSPM telemetry**: validate outcomes and improve ranking.

### Detailed Implementation Plan
#### Phase 1: Operator Workflow Discovery and Action Inventory (Weeks 1–4)
The agency should begin by observing operators during typical and high-stress situations to understand how decisions are actually made and what information is used, because a menu that does not match real workflow will be ignored. The traffic engineering team should list the actions that operators are allowed to perform today (plan switches, bounded split/offset changes, special mode activation) and should document which actions require engineering approval or field work. The team should translate legal and safety constraints into a “safe envelope,” including pedestrian minimums/clearance, maximum cycle boundaries, and limits on plan switching frequency, because the menu must prevent unsafe or non-compliant actions by design.

- **Roles**: operations lead (workflow owner), traffic engineer (action definitions and constraints), UX/product (user research), safety/accessibility reviewer (ped rules), IT/security (access control requirements).
- **Deliverables**: workflow map, action inventory, constraints specification, and initial menu “card” designs.
- **Risks**: the action inventory may not reflect vendor/controller capabilities; constraints may be incomplete.
- **Acceptance checks**: the list of allowable actions is signed off, and constraints are testable and enforceable.

#### Phase 2: Option Generation and Scoring Service (Weeks 5–12)
The engineering team should implement an option generation service that produces a small, bounded set of candidates (for example, 3–10) rather than an open-ended search, because operators need speed and clarity. The scoring service should evaluate each option on a standard KPI bundle (delay, p95 queue, spillback probability, pedestrian delay, arrivals-on-green) and should also compute a confidence score that reflects input data quality and model limitations. The team should implement strict validation so illegal options are rejected before they reach the operator UI, and it should log both the generated options and the ranking rationale so the system can be audited.

- **Roles**: software engineer (generation/scoring), traffic engineer (KPI definitions and bounds), data engineer (inputs and health checks), QA engineer (test scenarios), operations (feedback on usefulness).
- **Deliverables**: option generation API, scoring/ranking module, confidence reporting, and audit log schema.
- **Risks**: options may be low quality if the generator is poorly constrained; model uncertainty may be hidden.
- **Acceptance checks**: generated options are implementable on controllers, scoring outputs are stable, and confidence is shown when inputs are weak.

#### Phase 3: UI Cards, Confirmation, and Rollback Controls (Weeks 13–20)
The UX team should design the option cards to present the decision in layers, with a clear headline (recommended action and primary KPI deltas) and expandable evidence, because operators need the ability to act quickly without losing access to details. The UI should require explicit confirmation before any change is applied and should show the rollback path and expected recovery behavior, because this is how operators build trust in decision support tools. The team should integrate the UI with existing incident logging and shift notes so decisions are captured in the same systems operators already use.

- **Roles**: UX engineer (interface), operations lead (training and workflows), software engineer (integration), traffic engineer (interpretation), QA (usability testing).
- **Deliverables**: operator UI prototype, confirmation and rollback UX, and integration to incident/shift logging.
- **Risks**: too much information slows decision-making; unclear rollback behavior increases risk.
- **Acceptance checks**: operators can complete standard tasks quickly in drills, and every applied option is logged with a reason and a revert path.

#### Phase 4: Pilot Deployment and Training (Weeks 21–28)
The agency should run the menu in shadow mode first, where it generates and ranks options but does not apply them, because this allows the team to compare predicted outcomes to real outcomes without operational risk. The agency should then move to assisted activation, where operators choose an option and apply it with rollback enabled, and it should track decision time, adoption rate, and override reasons. Training should use realistic scenarios and should include situations where the system is uncertain or wrong, because this reduces automation bias.

- **Roles**: operations (pilot users), traffic engineering (on-call support), analyst (evaluation), maintenance (field issues), product/UX (feedback collection).
- **Deliverables**: pilot report, refined card designs, tuned option generator bounds, and operator training materials.
- **Risks**: operators may distrust the tool if early options perform poorly; pilot incidents can bias results.
- **Acceptance checks**: operators can use the menu without slowing response, and pilot KPIs improve without policy violations.

#### Phase 5: Continuous Learning and Governance (Ongoing)
The program should continuously improve the menu by comparing predicted outcomes to realized outcomes and by retiring option templates that consistently underperform. The team should maintain a change-control process for option templates and KPI weights so updates are reviewed, versioned, and reversible. Over time, the agency can add more sophisticated prediction models, but it should keep the menu concept stable and focused on a small set of safe, high-value actions.

- **Roles**: program owner (governance), data/analytics (model monitoring), traffic engineering (template updates), operations (feedback).
- **Deliverables**: monthly model performance report, versioned option template library, and updated training playbooks.
- **Risks**: ranking drift can encode operator bias; expanding action scope can increase risk.
- **Acceptance checks**: outcome logs show improved prediction accuracy and stable safety compliance.

### Choices
- **Menu**: operator chooses from ranked safe options (this idea).
- **Recommend-only**: show best option but don’t allow applying.
- **Full automation**: algorithm applies best option automatically.

## Technical Mechanics

### Key Parameters
- Candidate count (N)
- KPI bundle (delay, p95 queues, spillback)
- Confidence/uncertainty fields

### Guardrails
- Constrain to safe envelope; reject illegal options.
- Require explicit confirmation and rollback path.
- Log reasons and outcomes.

## MVP Deployment
- One corridor.
- 3 options: “do nothing”, “switch to plan B”, “split tweak template”.
- Shadow mode for 2–4 weeks.

## Evaluation
- Decision time (median/p95).
- Operator adoption rate and override frequency.
- KPI improvements vs baseline.
- Prediction accuracy and rollback rate.

## References / Standards / Useful Sources
- FHWA ATSPM: https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm

---

Cross-links: Related ideas include explainable signals, what-if button, and self-healing.
