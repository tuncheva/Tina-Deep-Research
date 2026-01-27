# 16) Fast-Forward Twin (Next 5 Minutes): Short-Horizon Predictive Control

## Catchy Explanation
It’s like fast-forwarding the city by a few minutes—many times in parallel—to see which timing choice avoids the worst queues, then applying the safest best option.

## What it is (precise)
A **fast-forward twin** is a short-horizon model-predictive control (MPC) loop using a digital twin that repeatedly:
1) initializes from the current field state,
2) simulates multiple candidate actions over a 1–5 minute horizon,
3) scores them on a KPI bundle (delay, stops, p95 queues, spillback risk, ped delay, safety proxies),
4) selects an action within hard constraints,
5) applies it with rollback protections.

It differs from a one-off what-if tool by being a **continuous control loop** with runtime SLAs, strict action bounds, and explicit safety envelopes.

## Benefits
- **High responsiveness**: adapts to near-term demand shifts.
- **Constraint-aware**: selects actions that respect safety and policy invariants.
- **Stability control**: can reduce spillback cascades with early interventions.
- **Auditability**: logs evaluated rollouts and selected action.

## Challenges
- **Compute + latency**: must run fast enough every cycle/window.
- **State estimation**: twin initialization errors compound quickly.
- **Overfitting to the horizon**: myopic optimization can harm longer-term flow.
- **Operational risk**: requires strong rollback and supervision.

## Implementation Strategies

### Infrastructure Needs
- **Live state feeds**: phases, detector aggregates, ped calls, probe travel times.
- **Candidate action set**: bounded plan switches and split/offset templates.
- **Fast simulation / surrogate**: accelerated micro-sim or hybrid predictor.
- **Constraint engine**: hard limits + policy rules.
- **Observability**: ATSPM metrics and health monitoring.

### Detailed Implementation Plan
#### Phase 1: Define the Action Menu, Replan Interval, and Safety Envelope (Weeks 1–6)
The agency should begin by defining a small set of actions that are safe, implementable, and reversible in real controllers, because MPC is only as safe as its action space. The traffic engineer should define bounds for split and offset changes, specify which plan IDs can be selected, and define how often the controller is allowed to change plans or parameters (replan interval). The team should encode non-negotiable constraints, including pedestrian minimums and clearance, maximum cycle rules where applicable, corridor coordination caps, and any city policy constraints, so the MPC loop cannot violate them even if predicted delay improves.

- **Roles**: traffic engineering (action bounds and constraints), operations (workflow and supervision), software engineer (constraint engine), safety/accessibility reviewer (ped constraints).
- **Deliverables**: action menu (versioned), constraint specification, and a replan/actuation policy.
- **Risks**: too many actions increase risk; too few actions reduce benefit.
- **Acceptance checks**: all actions are implementable on target controllers and all constraints are testable.

#### Phase 2: Build Continuous Shadow Rollouts and Benchmark Runtime (Weeks 7–14)
The engineering team should run continuous rollouts in shadow mode, where the system initializes from field state, evaluates candidates, and logs the selected action without actuating, because this provides real-time validation without risk. The team should benchmark runtime distributions (p50/p95) under realistic candidate counts and should tune the simulation approach (micro, meso, or hybrid) until the system meets the required SLA. The team should also measure prediction accuracy for short horizons and should identify when the state estimation confidence is too low to produce actionable results.

- **Roles**: modeling engineer (twin and runtime tuning), infrastructure engineer (performance), data engineer (state inputs), analyst (accuracy evaluation).
- **Deliverables**: shadow validation report, runtime benchmark report, and confidence/staleness policy.
- **Risks**: micro-simulation may be too slow; poor state estimation yields unreliable predictions.
- **Acceptance checks**: runtime SLA is met in shadow mode and predictions are within agreed error bounds for key KPIs.

#### Phase 3: Controlled Deployment with Assisted Actuation and Rollback (Weeks 15–26)
The agency should enable assisted actuation first, where operators approve the chosen action, because continuous control loops require trust and clear supervision. The team should implement rollback triggers that revert to a known safe plan if KPIs degrade beyond thresholds, if detector health drops, or if oscillation is detected. The team should also enforce rate limits and minimum dwell times to avoid rapid oscillations that confuse drivers and degrade progression.

- **Roles**: operations (approval and monitoring), traffic engineering (on-call tuning), software engineer (rollback and rate limits), maintenance (field issues).
- **Deliverables**: active pilot, rollback policies, operator runbook, and actuation logs.
- **Risks**: overly sensitive rollback can prevent benefits; insufficient rollback can create operational risk.
- **Acceptance checks**: operators can supervise without excessive workload, rollbacks work reliably, and constraints are never violated.

#### Phase 4: Add Robustness to Uncertainty and “Safe Regret” Scoring (Weeks 27–36)
The modeling team should improve robustness by evaluating candidates under multiple plausible demand and noise samples or by incorporating uncertainty directly into scoring, because the system must avoid fragile “wins” that fail when inputs are slightly wrong. The scoring policy should prefer actions with bounded downside (safe regret) rather than purely maximizing predicted improvement, and it should incorporate explicit penalties for destabilizing corridor coordination or increasing spillback risk. The team should validate that robustness improvements reduce rollback rates and improve operator trust.

- **Roles**: modeler/data scientist (robust scoring), traffic engineer (policy weights), analyst (evaluation), operations (feedback).
- **Deliverables**: robust MPC scoring module, updated evaluation reports, and tuned guardrails.
- **Risks**: robust scoring may become too conservative; uncertainty modeling may be complex to maintain.
- **Acceptance checks**: rollback frequency decreases, and performance improvements remain stable across varying days.

#### Phase 5: Scaling, Governance, and Maintenance (Ongoing)
The agency should scale the fast-forward twin corridor-by-corridor using templates for action menus and constraints, but it should recalibrate the twin and constraints for each corridor because phasing, pedestrian requirements, and geometry differ. The program should implement governance for model updates, action template changes, and KPI weight changes so modifications are reviewed, versioned, and reversible. The team should continuously monitor model drift and should revalidate performance after construction, detection upgrades, or controller firmware changes.

- **Roles**: program owner (governance), modeling team (calibration), traffic engineering (constraints and plans), operations (monitoring), IT/data (pipeline maintenance).
- **Deliverables**: scaling roadmap, corridor onboarding checklist, periodic audits, and updated model versions.
- **Risks**: scaling increases compute and monitoring burden; drift can undermine trust.
- **Acceptance checks**: each new corridor passes a shadow validation gate before actuation is enabled.

### Choices
- **Fixed templates**: simplest and safest.
- **Parameterized tweaks**: more flexible but riskier.
- **Hybrid**: templates + small parameter tuning.

## Technical Mechanics

### Key Parameters
- Horizon (1–5 minutes)
- Replan interval (e.g., every cycle or every 30–60s)
- Candidate count (N)
- Runtime SLA (p95)

### Guardrails
- Hard constraints (ped service/clearance) are non-negotiable.
- Rate-limit and revert plan changes.
- Require audit logs for each actuation.

## MVP Deployment
- One corridor.
- 3 candidate actions.
- Recommend-only for 2–4 weeks, then assisted activation.

## Evaluation
- Prediction error and runtime distribution.
- KPI improvements vs baseline (delay, p95 queue, split failures).
- Rollback frequency and reasons.

## References / Standards / Useful Sources
- FHWA ATSPM: https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm

---

Cross-links: Related ideas include what-if button, anti-jam nudges, and green waves.
