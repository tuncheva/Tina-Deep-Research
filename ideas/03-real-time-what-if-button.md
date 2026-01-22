# 03) Real-Time What-If Button: Predictive Traffic Decisions

## Brief Description
The real-time what-if button allows operators to simulate signal changes virtually before applying them, using digital twins for safe decision-making.

## Analogical Reference
Like a flight simulator for pilots testing maneuvers without real risks, the what-if button lets traffic operators test timing changes safely.

## Comprehensive Information
This tool integrates with digital twins to run quick simulations of traffic signal adjustments, comparing outcomes like queue lengths and delays. It ensures operators make informed choices, reducing errors and enhancing safety. Based on standards from FHWA and implementations like Econolite's virtual simulations, it supports real-time decision-making with guardrails to prevent unsafe actions.

## Upsides and Downsides

### Positive Aspects on People's Lives in 5 Years
- Better Traffic Flow: Reduced congestion through informed decisions, saving time for commuters.
- Safety Improvements: Fewer accidents from poor timing choices, protecting lives.

### Positive Aspects on People's Lives in 15 Years
- Smart City Integration: Fully automated with AI, leading to optimized urban mobility and reduced emissions.

### Downsides in 5 and 15 Years
- Resource Intensive: High computational costs and potential delays if simulations are slow.
- Over-Reliance Risk: Operators might ignore real-world factors due to trust in simulations.

### Hard Things People Will Have to Overcome When Getting Used to It
- Building Trust: Initially, operators may doubt simulation accuracy, requiring training.
- Managing Latency: Ensuring fast results for urgent decisions.
- Handling Uncertainty: Interpreting probabilistic outcomes correctly.

## Benefits
The button enhances decision-making and safety:
- **Informed Choices**: Tests actions virtually, reducing risky guesses.
- **Faster Responses**: Speeds up incident handling with data-driven recommendations.
- **Auditability**: Logs tested options for accountability.
- **Performance Gains**: Improves congestion management with minimal compute cost.

## Challenges
Requires robust modeling and trust:
- **Model Accuracy**: Poor calibration leads to bad recommendations.
- **Latency**: Simulations must be fast for real-time use.
- **Operator Trust**: Risk of over-reliance or ignoring uncertainty.
- **Compute Resources**: Running parallel scenarios demands optimization.

## Implementation Strategies
### Infrastructure Needs
- **Live State Estimation**: Real-time detector counts, occupancy, controller phases, probe travel times.
- **Simulation Engine**: Micro/meso twin models running faster-than-real-time.
- **Action Generator**: Library of bounded changes (splits ±5s, offsets ±3s).
- **Guardrails**: Hard constraints (ped mins), rollback mechanisms, staleness checks.

### Detailed Implementation Plan
#### Phase 1: Foundation Building (Weeks 1-8)
- **Team Assembly**: Recruit traffic engineers, data scientists, UI designers, operators.
- **Requirements Gathering**: Define use cases (incident response, peak adjustments); set latency targets (<30s).
- **Data Infrastructure**: Upgrade controllers for state exposure; integrate probe data.
- **Budget**: $200k-$500k for data pipelines.
- **Risks**: Data quality issues; validate sources.
- **Timeline**: 8 weeks; Deliverable: Technical specs.

#### Phase 2: Twin Development (Weeks 9-20)
- **Model Selection**: Choose SUMO or custom engine for corridor simulation.
- **Calibration**: Use 3-6 months historical data to match queues, travel times.
- **Action Library**: Develop 10-20 candidate actions with safety checks.
- **Integration**: Connect twin to live data feeds; test snapshot accuracy.
- **Team**: Modelers, devs.
- **Budget**: $300k-$700k.
- **Risks**: Calibration errors; iterative testing.
- **Timeline**: 12 weeks; Deliverable: Validated twin.

#### Phase 3: Testing and Validation (Weeks 21-28)
- **Shadow Simulations**: Run continuous rollouts without actuation; compare to reality.
- **UI Development**: Build operator dashboard with recommendations, tradeoffs, confidence.
- **Guardrail Implementation**: Add rollback (auto-revert after 10 min), staleness rules.
- **Operator Feedback**: Pilot with 2-3 operators for usability.
- **Team**: UI team, testers.
- **Budget**: $150k.
- **Risks**: UI confusion; simplify.
- **Timeline**: 8 weeks; Deliverable: Tested prototype.

#### Phase 4: Deployment and Training (Weeks 29-40)
- **Phased Rollout**: Deploy to one corridor; monitor for 1 month.
- **Operator Training**: Hands-on sessions on button usage, interpreting outputs.
- **Monitoring Setup**: Dashboards for latency, acceptance rates, rollbacks.
- **Incident Response**: Test with simulated events.
- **Team**: Trainers, monitors.
- **Budget**: $100k.
- **Risks**: Over-trust; include uncertainty metrics.
- **Timeline**: 12 weeks; Deliverable: Deployed system.

#### Phase 5: Scaling and Enhancement (Weeks 41-60+)
- **City-Wide Expansion**: Roll out to all corridors with performance tuning.
- **AI Integration**: Add predictive triggers for auto-initiation.
- **Continuous Improvement**: Use logs for model refinement; add new actions.
- **Evaluation**: Quarterly benefits reports.
- **Budget**: $500k+.
- **Risks**: Scaling latency; optimize compute.
- **Timeline**: 20+ weeks; Deliverable: Full-scale operation.

### Choices
- **Operator-Triggered**: On-demand, low cost but misses incidents.
- **Continuous**: Always ready, higher resources.
- **Shadow**: Safe for learning.
- **Auto**: Fast but needs strong guards.

## Future Impacts and Predictions
In 5 years, what-if tools will be ubiquitous, cutting decision times by 50%. By 15 years, AI will automate selections with human oversight.

### Comparison Tables: Upsides vs Downsides

| Time Horizon | Aspect | Upsides | Downsides |
|--------------|--------|---------|-----------|
| **In 5 Years (Post-Implementation)** | **Safety & Decisions** | Avoids risky actions; auditable. | Model errors if not calibrated. |
| | **Operations** | Faster incident response. | Compute latency issues. |
| | **Public Impact** | Smoother traffic, less congestion. | Initial trust-building needed. |

| Time Horizon | Aspect | Upsides | Downsides |
|--------------|--------|---------|-----------|
| **In 15 Years (Post-Implementation)** | **Safety & Decisions** | AI-driven automation; near-perfect predictions. | Cyber risks to simulations. |
| | **Operations** | Fully integrated with smart cities. | Over-automation reduces human skills. |

**Hard Things to Overcome (Across Horizons)**:
- State Initialization: Robust queue estimation.
- Latency vs Quality: Balance scenarios and speed.
- Human Factors: Combat automation bias.

## Implementation Costs and Case Studies

### Costs for Implementation
- **Software**: Twin calibration and sim engine - $50k-$150k; faster-than-real-time for 3-20 scenarios.
- **Hardware**: Enhanced data pipelines - $20k-$50k; supports probe data and occupancy.
- **Annual Ops**: Compute for simulations - $10k-$30k; optimizes latency for 10-30s decisions.
- **ROI**: Reduces operational errors by 20-40%, with benefits in congestion management.

### Real-World Case Studies
- **Motorway Twins**: Parallel DTIs run for 1-5 min horizons, evaluating actions before deployment.
- **Econolite**: Virtual simulation validates adaptive controls, preventing risky changes.
- **FHWA**: Automated traffic signal performance measures integrate what-if for better outcomes.

### Additional Implementation Details
- Phased rollout with operator training.
- Integration with health checks for reliability.

## Technical Mechanics
### Key Parameters
- **Horizon**: 1–5 minutes for predictions.
- **Candidates**: Small, safe action sets.

### Coordination Types
- Parallel DTIs for rollouts.
- Two-stage scoring: Constraints then objectives.

### Guardrails
- Bounded actions, rollback, staleness rules.

## MVP Deployment
- Single corridor with 3 actions, parallel rollouts, confirmation required.

## Evaluation
- Decision latency, acceptance rate, rollback frequency, prediction error.

---

## Key Terms and Explanations
- **What-If Button**: Tool for simulating traffic actions virtually.
- **Parallel DTIs**: Duplicate twins running scenarios.
- **Nowcast**: Current state snapshot for simulation.
- **Rollback**: Auto-revert if KPIs worsen.
- **Automation Bias**: Over-trusting automated suggestions.

---

Cross-links: Related ideas include fast-forward twin, green waves, self-healing intersections, and city rules.
