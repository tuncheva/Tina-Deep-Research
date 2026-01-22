# 17) Safety-First Signals: Optimizing for Near-Misses

## Brief Description
Safety-first signals optimize timing to reduce risky interactions using surrogate safety measures.

## Analogical Reference
Like a lifeguard monitoring swimmers, signals use proxies to prevent near-misses.

## Comprehensive Information
Evaluating plans with digital twins for TTC, PET, and conflicts, signals select options that minimize risks while balancing mobility.

## Upsides and Downsides

### Positive Aspects on People's Lives in 5 Years
- Fewer Accidents: Proactive safety reduces crashes and injuries.
- Informed Decisions: Transparent tradeoffs build trust.

### Positive Aspects on People's Lives in 15 Years
- Predictive Safety: AI anticipates and mitigates risks in real-time.

### Downsides in 5 and 15 Years
- Potential Delays: Safety prioritization may increase wait times.
- Data Challenges: Requires accurate trajectory data.

### Hard Things People Will Have to Overcome When Getting Used to It
- Proxy Accuracy: Validating measures against real crashes.
- Data Collection: Ensuring high-quality input.
- Balancing Needs: Managing safety vs efficiency conflicts.

## What it is (precise)
Safety-first signal control treats safety as a primary objective, optimizing to reduce risky interactions using surrogate safety measures like TTC, PET, and conflict counts. It evaluates timing changes in digital twins, selecting plans that balance safety with mobility. Indian case studies show SSMs reduce conflicts; real-time monitoring prevents near-misses.

## Benefits
Enhances safety and decision-making:
- **Direct Safety Targeting**: Uses proxies to minimize conflicts; SSMs like TTC/PET quantify risks.
- **Transparent Tradeoffs**: Explicit safety vs delay balances; Indian evaluations show reduced near-misses.
- **Proactive Measures**: Real-time systems prevent unsafe scenarios; FHWA algorithms support.

## Challenges
Requires data and validation:
- **Proxy Limitations**: SSMs may not predict all crashes; validation against history needed.
- **Data Intensity**: Needs trajectory data from twins or telemetry.
- **Political Sensitivity**: Increased delays can be controversial.

## Implementation Strategies
### Infrastructure Needs
- Micro-sim twins for SSM computation.
- Detectors for conflict data.
- Policy constraints on safety thresholds.

### Detailed Implementation Plan
#### Phase 1: Requirements (Weeks 1-4)
- Select SSMs; define targets.
- Team: Safety experts.
- Budget: $50k.
- Risks: Incomplete proxies.
- Timeline: 4 weeks; Deliverable: SSM framework.

#### Phase 2: Data Calibration (Weeks 5-12)
- Validate data; calibrate twins.
- Team: Data analysts.
- Budget: $120k.
- Risks: Poor fidelity.
- Timeline: 8 weeks; Deliverable: Calibrated models.

#### Phase 3: Optimization (Weeks 13-20)
- Evaluate changes; produce statements.
- Team: Engineers.
- Budget: $100k.
- Risks: Overfitting.
- Timeline: 8 weeks; Deliverable: Safety plans.

#### Phase 4: Deployment (Weeks 21-32)
- Rollout with monitoring.
- Team: Ops.
- Budget: $150k.
- Risks: False positives.
- Timeline: 12 weeks; Deliverable: Active system.

#### Phase 5: Refinement (Ongoing)
- Adaptive constraints.
- Budget: $80k annual.
- Timeline: Continuous.

### Choices
- **Hard Constraints**: Safety minima.
- **Weighted Objectives**: Balance safety/delay.

## Future Impacts and Predictions
SSMs will reduce crashes by 15-25% in 5 years, with AI predicting risks. In 15 years, full integration minimizes incidents.

### Comparison Tables: Upsides vs Downsides

| Time Horizon | Aspect | Upsides | Downsides |
|--------------|--------|---------|-----------|
| **In 5 Years (Post-Implementation)** | **Safety** | Reduced conflicts. | Proxy accuracy. |
| | **Operations** | Explicit tradeoffs. | Data needs. |

| Time Horizon | Aspect | Upsides | Downsides |
|--------------|--------|---------|-----------|
| **In 15 Years (Post-Implementation)** | **Safety** | AI prevention. | Tech dependencies. |
| | **Operations** | Automated safety. | Over-reliance. |

**Hard Things to Overcome (Across Horizons)**:
- Proxy Validation: Correlate with crashes.
- Data Quality: Ensure trajectory accuracy.
- Tradeoff Acceptance: Public communication.

## Implementation Costs and Case Studies

### Costs for Implementation
- **Software**: Twin/SSM tools - $150k-$300k.
- **Hardware**: Sensors - $50k.
- **Annual Ops**: Audits - $40k.

### Real-World Case Studies
- **FHWA SSMs**: TTC/PET for assessment.
- **Indian Studies**: 33 evaluations reduce conflicts.
- **Real-Time Systems**: Proactive monitoring.

### Additional Implementation Details
- Start with micro-sim.
- Publish impact statements.

## Technical Mechanics
### Key Parameters
- TTC, PET, conflict counts.

### Coordination Types
- Safety threshold enforcement.

### Guardrails
- Conservative defaults.

## MVP Deployment
- One intersection; SSM computation.

## Evaluation
- SSM distributions, crash proxies.

---

## Key Terms and Explanations
- **SSMs**: Surrogate Safety Measures like TTC, PET.
- **TTC**: Time to Collision.
- **PET**: Post-Encroachment Time.

---

Cross-links: Related ideas include city rules, explainable signals.
