# 16) Fast-Forward Twin: Short-Horizon Predictive Control

## Brief Description
Fast-forward twin runs short simulations to select optimal near-term timing actions.

## Analogical Reference
Like previewing movie scenes to choose the best plot, the twin evaluates actions for the best outcome.

## Comprehensive Information
Using model-predictive control, digital twins simulate 1-5 minute horizons, scoring actions on KPIs and actuating the best with rollback options.

## Upsides and Downsides

### Positive Aspects on People's Lives in 5 Years
- Adaptive Efficiency: Better real-time responses reduce delays.
- Safety Assurance: Constraints prevent risky choices.

### Positive Aspects on People's Lives in 15 Years
- Intelligent Cities: Fully predictive control for seamless mobility.

### Downsides in 5 and 15 Years
- Computational Demands: High latency if not optimized.
- Prediction Errors: Poor forecasts could lead to wrong actions.

### Hard Things People Will Have to Overcome When Getting Used to It
- Meeting Latency: Ensuring fast simulations.
- State Accuracy: Reliable initialization from data.
- Handling Uncertainty: Managing prediction risks.

## What it is (precise)
Run short simulations from current state to select optimal near-term timing actions for 1-5 minutes. This uses model-predictive control with digital twins to evaluate alternatives, pick the best based on KPIs like delay, stops, spillback, and safety constraints, and repeat frequently. The twin initializes from live detector data and phase states, simulates candidate actions (split/offset tweaks or plan switches), scores them against weighted objectives and hard limits, then actuates the top choice while maintaining auditability and rollback capabilities for rapid, data-driven adaptation.

## Benefits
Enables real-time adaptation:
- **Precision**: Informed decisions per cycle.
- **Efficiency**: Optimizes near-term flow.
- **Safety**: Enforces constraints.
- **Robustness**: Handles uncertainties.

## Challenges
Requires computational speed:
- **Latency**: Must run quickly.
- **Accuracy**: Good state initialization.
- **Complexity**: Scoring alternatives.
- **Over-Reliance**: If predictions fail.

## Implementation Strategies
### Infrastructure Needs
- Live data: Phase states, detectors.
- Action set: Safe tweaks.
- Scoring: KPI metrics.
- Compute: Edge/cloud for speed.

### Detailed Implementation Plan
#### Phase 1: Action Menu (Weeks 1-6)
- Define safe changes; constraints.
- Team: Engineers.
- Budget: $70k.
- Risks: Unbounded actions.
- Timeline: 6 weeks; Deliverable: Menu.

#### Phase 2: Shadow Rollouts (Weeks 7-14)
- Test forecasts without actuation.
- Team: Devs.
- Budget: $120k.
- Risks: Poor calibration.
- Timeline: 8 weeks; Deliverable: Validations.

#### Phase 3: Controlled Deployment (Weeks 15-26)
- Activate in small areas.
- Team: Ops.
- Budget: $180k.
- Risks: Failures.
- Timeline: 12 weeks; Deliverable: Active control.

#### Phase 4: Refinement (Weeks 27-36)
- Expand actions; add uncertainty.
- Team: Analysts.
- Budget: $140k.
- Risks: Instability.
- Timeline: 10 weeks; Deliverable: Enhanced system.

#### Phase 5: Scaling (Ongoing)
- City-wide; continuous updates.
- Budget: $200k annual.
- Timeline: Continuous.

### Choices
- **Fixed Templates**: Pre-approved.
- **Parameterized**: Flexible tweaks.
- **Hybrid**: Best of both.

## Future Impacts and Predictions
Short-horizon control will improve adaptivity by 50% in 5 years. In 15 years, real-time twins are standard.

### Comparison Tables: Upsides vs Downsides

| Time Horizon | Aspect | Upsides | Downsides |
|--------------|--------|---------|-----------|
| **In 5 Years (Post-Implementation)** | **Adaptivity** | Quick responses. | Compute needs. |
| | **Risk Control** | Constraint enforcement. | Latency risks. |

| Time Horizon | Aspect | Upsides | Downsides |
|--------------|--------|---------|-----------|
| **In 15 Years (Post-Implementation)** | **Adaptivity** | AI optimization. | Over-complexity. |
| | **Risk Control** | Predictive safety. | Dependency. |

**Hard Things to Overcome (Across Horizons)**:
- Runtime: Meet latency budgets.
- State Accuracy: Improve estimation.
- Uncertainty: Bound predictions.

## Implementation Costs and Case Studies

### Costs for Implementation
- **Compute**: Servers - $100k-$200k.
- **Software**: Twin engine - $150k-$300k.
- **Annual Ops**: Maintenance - $60k.

### Real-World Case Studies
- **FHWA**: Adaptive control loops.
- **Traffic Manual**: Model-based evaluation.

### Additional Implementation Details
- SLA for runtimes.
- Audit trails.

## Technical Mechanics
### Key Parameters
- Horizon: 1-5 min.
- Action menu: 3-5 options.

### Coordination Types
- MPC with twin rollouts.

### Guardrails
- Hard constraints, fallbacks.

## MVP Deployment
- One corridor; recommend-only.

## Evaluation
- Prediction error, latency, violations.

---

## Key Terms and Explanations
- **Model-Predictive Control**: Evaluate actions via model.
- **Short-Horizon**: Near-term forecasts.
- **KPI Bundle**: Multi-objective scoring.
- **Rollouts**: Scenario simulations.

---

Cross-links: Related ideas include what-if button, green waves.
