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

## Implementation Additions (Implementation-Ready)

### 1) “Deployability gating” specification (Allowed/Blocked must be rigorous)
“Deployable” must mean **this option can be safely executed right now on this exact infrastructure**. Treat deployability as a dedicated gating engine, not as a UI label.

#### What deployable means (minimum contract)
An option is **DEPLOYABLE** only if all **hard constraints** pass:

- **Controller capability compatibility**
  - feature support: coordination, phase holds/omits, TSP, preemption interface, plan transition modes.
  - configuration support: phases actually mapped, detectors present where required.

- **Transition feasibility**
  - respects min greens, ped walk/clearance, yellow/red clearance.
  - coordination transitions that won’t create unsafe “instant offsets” or flash states.
  - rate limits: no thrashing (plan switches per corridor per time window).

- **Pedestrian timing + accessibility**
  - ped minimums (min walk + clearance) are preserved.
  - ADA/accessibility policies preserved (e.g., audible/tactile timing; if applicable).

- **Current mode constraints**
  - coordination currently running? flash? manual control? preemption active? degraded detection?
  - if incident preemption or emergency mode is active, only show options allowed under that mode.

- **Safety-critical constraints are non-overridable**
  - clearance intervals, ped minimums, prohibited phase sequences, intergreens.

Options may also have **soft constraints** that produce WARNINGS (deployable but discouraged), such as increased ped delay distribution or high spillback risk.

#### How constraints are authored
Constraints are composed from:
- **Policy contracts** (city rules): ped wait caps, max cycle ranges by TOD, “no degrade” corridors.
- **Controller configuration inputs**: timing parameters, phase map, coordination groups.
- **Intersection metadata**: geometry, approaches, school zone flags, transit priority corridors.

Rule priority / arbitration:
- **Hard fail** → BLOCKED (never shown as selectable)
- **Soft warn** → WARN (shown with risk banner; may require supervisor acknowledgement)

#### How deployability checks are tested and audited
- **Unit tests** for each rule using synthetic intersection configs.
- **Scenario regression suite**: replay recorded controller + detector/probe states; ensure same option yields same gating decision.
- **Known-bad test cases**: must remain blocked forever (e.g., options that violate ped clearance).
- **Change control**: constraints are versioned; changes require approval and release notes.

#### Constraint table (template)
| Constraint | Data required | Check logic (sketch) | Failure message | Operator action |
|---|---|---|---|---|
| Ped min walk/clearance preserved | controller ped timings; phase map | `new_walk>=min && new_clear>=min` | “Blocked: ped minimum would be violated at X.” | Pick another option; escalate to engineer |
| Coordination transition feasible | coordination group; current offset; transition mode | “no immediate jump; transition method supported” | “Blocked: unsupported coordination transition.” | Use plan-change with approved transition |
| Preemption active | preemption status | if active, allow only “monitor/hold safe” set | “Blocked: emergency preemption active.” | Follow incident SOP |
| Data freshness | timestamps; sensor health | if required inputs stale → block or warn | “Blocked: data stale for travel-time input.” | Wait/retry; switch to conservative option |
| Plan switch rate-limit | audit log; last switch time | `now-last_switch >= T_min` | “Blocked: rate limit (next allowed at HH:MM).” | Monitor; use non-switch option |

**Telemetry alignment**: ATSPM provides high-resolution data and performance measures that support monitoring timing/operations and diagnosing issues (e.g., split failures, coordination, pedestrian delay use cases) ([`Automated Traffic Signal Performance Measures (FHWA-HOP-20-002)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm:1)). Use ATSPM-derived checks where feasible.

### 2) Human factors and UI failure modes (beyond choice overload)
Human factors problems in TMC environments are well-known; FHWA provides TMC human factors guidelines including topics such as operator workload/vigilance, multitasking, minimizing interruptions, keeping operators in the loop, and avoiding decisionmaking biases like confirmation bias and anchoring ([`Human Factors Guidelines for Transportation Management Centers (FHWA-HRT-16-060)`](https://www.fhwa.dot.gov/publications/research/safety/16060/16060.pdf:1)).

#### Risks to explicitly design against
- **Confirmation bias**: selecting the option that fits the operator’s initial hypothesis.
- **Anchoring**: overweighting the top card even when evidence is weak.
- **Automation bias**: over-trust in ranked suggestions; under-trust when system is right.
- **Alarm fatigue**: warnings become background noise.
- **Tunnel vision on one KPI**: optimizing delay while ignoring spillback/ped impacts.
- **Inconsistency across shifts**: different operators interpret cards differently.

#### UI patterns to mitigate (implementation rules)
- **Small option set**: default 3–7 cards; more requires explicit “show more”.
- **Default safe highlight**: visually mark “conservative” option (often #2) to reduce anchoring on the #1 recommendation.
- **Consistent ordering**: safety-first ordering (blocked removed; warnings below).
- **Progressive disclosure**:
  - 10-second summary card: action + top 3 KPI deltas + confidence.
  - details: evidence, constraints evaluated, assumptions.
- **Required acknowledgement** for high-risk warnings.
- **Training mode / playback**: shadow-mode replays with “what you would have clicked” and outcome.

#### Minimum usability acceptance criteria
- Median time-to-decision ≤ target (e.g., 30–60s) for standard incidents.
- Error rate: 0 blocked actions applied; near-zero “wrong corridor” mistakes.
- Operator trust calibration: survey + observed behavior shows both acceptance and appropriate skepticism.

### 3) Accountability model (who is responsible for what; what evidence is stored)
The system must make accountability explicit.

#### Responsibility partitioning
- **Operator responsibility**: situational judgment, partner coordination, final selection, notes.
- **System responsibility**:
  - only present deployable options,
  - display uncertainty and data freshness,
  - provide rollback contract,
  - log faithfully.
- **Engineering/admin responsibility**:
  - constraint rule definitions,
  - plan library quality,
  - model calibration and KPI definitions,
  - training materials.

#### Evidence bundle captured at “click time” (minimum)
Store an immutable record:
- state snapshot (corridor/intersections, mode, current plans, active preemption/flash).
- input health (timestamps, missing sensors, probe coverage).
- option card shown (full text + KPI deltas + confidence).
- constraints evaluated + pass/fail results (hard/soft).
- who clicked, when, and reason/notes.
- rollback plan + exit criteria + max duration.

#### Post-incident review workflow
- Review triggers:
  - any rollback, any safety warning, any partner complaint, any bad outcome.
- Attendees: ops supervisor, traffic engineer, safety/accessibility reviewer, partner liaison (as needed).
- Outputs: tuning tasks, constraint-rule changes, UI copy changes, training updates.

### 4) Predicted vs actual feedback loop (without optimizing what’s easiest to predict)
Use a scorecard that separates prediction accuracy from decision quality.

#### Scorecard structure
- **Predicted vs realized KPIs**:
  - error metrics (MAE/MAPE) by KPI and by context (TOD, incident type).
  - calibration curves for confidence.
- **Decision quality metrics** (domain outcomes):
  - prevented spillback? stabilized queues? protected ped service?
  - time-to-stabilize after action.

#### Guardrails for model drift
- **Freeze windows**: don’t update scoring weights during major event seasons without review.
- **Human review** for any material scoring change.
- **Metric selection bias monitoring**:
  - ensure the model isn’t “getting better” only on easy-to-predict KPIs while harming safety/priority KPIs.

#### Separate change tracks (avoid entanglement)
- Simulation calibration parameters (model layer)
- Scoring weights / KPI bundle definitions (policy layer)
- Constraint rules (safety/compliance layer)

Each track has its own versioning and approvals.

### 5) Integration with incident command and external stakeholders
Operators act in a broader incident management ecosystem.

#### Incident management inputs
- **Declared incident**: police/dispatch feed indicates closure, lanes blocked, control points.
- **Transit operations**: detours, headway protection, temporary bus priority mode.
- **Event/venue ops**: scheduled release windows, pedestrian surges.

#### Communication artifacts
For any applied option, auto-generate an **Action Summary**:
- what changed (plan/splits/offsets/mode), where (geofence), when (timestamp), duration.
- expected impact (top KPIs) + confidence.
- constraints honored (e.g., ped mins protected).

#### Permissions / escalations
- Role-based permissions:
  - operator can apply low-risk options,
  - supervisor required for options with major tradeoffs,
  - engineering approval required for template/library changes.

## MVP Deployment
- One corridor.
- 3 options: “do nothing”, “switch to plan B”, “split tweak template”.
- Shadow mode for 2–4 weeks.

## Evaluation
- Decision time (median/p95).
- Operator adoption rate and override frequency.
- KPI improvements vs baseline.
- Prediction accuracy and rollback rate.

---

## Implementation Checklist
- [ ] Document operator workflows and create action inventory (what is allowed today).
- [ ] Define constraints as policy + controller config + geometry metadata.
- [ ] Implement deployability gating engine (hard/soft rules) with unit + regression tests.
- [ ] Build option generator bounded to safe templates (3–10 candidates max).
- [ ] Implement scoring + confidence, using ATSPM/probe inputs and data freshness checks.
- [ ] Build card UI with progressive disclosure and required acknowledgement for high risk.
- [ ] Implement immutable audit logging with click-time evidence bundle.
- [ ] Run shadow mode and evaluate predicted vs realized scorecards.
- [ ] Train operators using playback scenarios including “system wrong” cases.
- [ ] Establish governance: versioning, approvals, and drift monitoring.

## Operator Workflow / SOP

### SOP 0 — Standard decision loop
1. Confirm the corridor and context (incident/event/normal ops).
2. Review data health and confidence flags.
3. Review top 3 option cards; expand details only as needed.
4. Confirm constraints summary (ped mins, clearance, preemption state).
5. Select option; add a short reason note.
6. Confirm activation (explicit confirmation).
7. Monitor KPIs + constraint alerts for `T_observe`.
8. If goals not met or safety warnings appear: execute rollback contract.
9. Log outcome; flag for review if unusual.

### SOP 1 — When options are BLOCKED
- Do not attempt manual overrides that violate hard constraints.
- Switch to conservative “monitor/hold safe” option.
- Escalate to supervisor/engineer if the situation demands action.

### SOP 2 — Incident-command / preemption active
- Follow incident command protocols.
- Use only incident-approved options; the menu should suppress others.

## Governance & Audit Runbook

### Versioning
- Constraint rules: semver-like versioning; hard-rule changes are MAJOR.
- Option templates: versioned library; retire underperforming templates.
- Scoring weights/KPI bundle: versioned with policy approvals.

### Approval workflow
- Hard constraint changes require safety/accessibility sign-off.
- High-risk option templates require engineering + ops supervisor approval.

### Audit and review
- Maintain immutable logs of:
  - options shown,
  - gating outcomes,
  - selected action,
  - realized outcomes.
- Monthly review:
  - prediction calibration,
  - rollback frequency,
  - operator override reasons,
  - “blocked but needed” incidents (gaps in library).

## Reference Links
- FHWA ATSPM overview and use cases: [`Automated Traffic Signal Performance Measures (FHWA-HOP-20-002)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm:1)
- FHWA TMC human factors guidance: [`Human Factors Guidelines for Transportation Management Centers (FHWA-HRT-16-060)`](https://www.fhwa.dot.gov/publications/research/safety/16060/16060.pdf:1)

## Completion Checklist
- ✅ Deployability gating spec + constraint table: see **“1) Deployability gating”**.
- ✅ Human factors + UI failure modes + acceptance criteria: see **“2) Human factors…”**.
- ✅ Accountability model + click-time evidence bundle + review workflow: see **“3) Accountability model”**.
- ✅ Predicted vs actual feedback loop + drift guardrails + separated change tracks: see **“4) Predicted vs actual feedback loop”**.
- ✅ Integration with incident command and stakeholders + action summaries + permissions: see **“5) Integration…”**.
- ✅ Final required sections added: **Implementation Checklist**, **Operator Workflow / SOP**, **Governance & Audit Runbook**, **Reference Links**, **Completion Checklist**.

---

Cross-links: Related ideas include explainable signals, what-if button, and self-healing.
