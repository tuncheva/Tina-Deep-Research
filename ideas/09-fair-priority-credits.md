# 09) Fair Priority Credits: Transparent Green-Time Budgeting

## Brief Description
Fair priority credits allocate limited priority interventions to users based on transparent rules.

## Analogical Reference
Like a rewards points system, credits ensure fair use of traffic privileges without overuse.

## Comprehensive Information
This system budgets priority actions for transit or emergencies, using digital twins to predict impacts and maintain equity across neighborhoods.

## Upsides and Downsides

### Positive Aspects on People's Lives in 5 Years
- Equitable Access: Fair distribution prevents priority abuse, benefiting all users.
- Reliable Transit: Better bus on-time performance improves daily commutes.

### Positive Aspects on People's Lives in 15 Years
- Optimized Mobility: AI adjusts credits dynamically for maximum fairness and efficiency.

### Downsides in 5 and 15 Years
- Complexity: Tracking and governance increase costs and technical challenges.
- Equity Debates: Political disputes over priority allocations.

### Hard Things People Will Have to Overcome When Getting Used to It
- Transparency Needs: Ensuring users understand credit usage.
- Measurement Accuracy: Reliable detection of eligible users and impacts.
- Scalability: Managing budgets in large networks.

## What it is (precise)
A priority credits system allocates a limited budget of priority interventions (green extensions, early greens) to eligible users (transit, emergency, bikes) according to transparent rules, preventing overuse and ensuring equity. It defines eligibility, actions, budgets, recovery, and fairness constraints, building on standard TSP/EVP mechanics with governance and auditability. Digital twins simulate tradeoffs like bus reliability vs cross-street delay, forecast cascading impacts, and ensure neighborhood equity.

## Benefits
Promotes fairness and transparency:
- **Equity Enforcement**: Prevents repeated harm to same streets; distributes benefits by corridor/neighborhood.
- **Transparency**: Auditable credits spent; explainable priority decisions.
- **Performance Maintenance**: Sustains transit reliability without unbounded delays.
- **Safety Assurance**: Bounded preemption; recovery to coordination.

## Challenges
Requires robust measurement and governance:
- **Complexity**: Tracking credits and impacts across systems.
- **Measurement Needs**: Accurate detection of requesters and impacts.
- **Political Sensitivity**: Balancing priorities; public acceptance.
- **Scalability**: Managing budgets across large networks.

## Implementation Strategies
### Infrastructure Needs
- Priority request detection (transit AVL, emergency comms).
- Policy engine for credit checks and logging.
- Twin for impact forecasting.
- Audit dashboards for transparency.

### Detailed Implementation Plan
#### Phase 1: Requirements (Weeks 1-4)
- Define eligible classes, budgets, fairness rules with stakeholders.
- Assess current priority systems.
- Team: Policy experts, engineers.
- Budget: $100k.
- Risks: Conflicting interests.
- Timeline: 4 weeks; Deliverable: Policy framework.

#### Phase 2: Development (Weeks 5-16)
- Build policy engine; integrate with controllers.
- Twin simulations for budget tuning.
- Team: Devs, modelers.
- Budget: $300k.
- Risks: Integration issues.
- Timeline: 12 weeks; Deliverable: Functional system.

#### Phase 3: Pilot (Weeks 17-24)
- Deploy on one corridor; monitor credits and impacts.
- Public reports.
- Team: Ops.
- Budget: $150k.
- Risks: Resistance.
- Timeline: 8 weeks; Deliverable: Pilot results.

#### Phase 4: Rollout (Weeks 25-40)
- Scale city-wide; adjust budgets.
- Team: Engineers.
- Budget: $500k.
- Risks: Equity imbalances.
- Timeline: 16 weeks; Deliverable: Operational credits.

#### Phase 5: Optimization (Ongoing)
- Continuous tuning; AI for dynamic allocation.
- Budget: $100k annual.
- Timeline: Continuous.

### Choices
- **Always-On**: Unlimited priority.
- **Threshold-Based**: Only when needed.
- **Credit-Based (This)**: Budgeted and fair.

## Future Impacts and Predictions
Credits will standardize priority, reducing conflicts by 40% in 5 years. In 15 years, AI optimizes allocations dynamically.

### Comparison Tables: Upsides vs Downsides

| Time Horizon | Aspect | Upsides | Downsides |
|--------------|--------|---------|-----------|
| **In 5 Years (Post-Implementation)** | **Equity** | Fair distribution. | Implementation complexity. |
| | **Performance** | Sustained reliability. | Potential reduced gains. |

| Time Horizon | Aspect | Upsides | Downsides |
|--------------|--------|---------|-----------|
| **In 15 Years (Post-Implementation)** | **Equity** | AI fairness. | Tech dependencies. |
| | **Performance** | Optimized priorities. | Over-automation. |

**Hard Things to Overcome (Across Horizons)**:
- Budget Tuning: Dynamic adjustments.
- Transparency: User-friendly reports.
- Measurement: Accurate impact tracking.

## Implementation Costs and Case Studies

### Costs for Implementation
- **Hardware**: Detection - $100k-$200k.
- **Software**: Engine - $150k-$300k.
- **Annual Ops**: Monitoring - $50k.

### Real-World Case Studies
- **NTCIP**: Standards for SCP systems.
- **Aimsun**: Predictive prioritization.
- **ITS America**: TSP treatments.

### Additional Implementation Details
- Stakeholder workshops.
- Public KPIs.

## Technical Mechanics
### Key Parameters
- Credit budgets, recovery rules.

### Coordination Types
- Eligibility checks, action selection.

### Guardrails
- Max delays, equity caps.

## MVP Deployment
- One class on one corridor.

## Evaluation
- Priority events, delay distributions.

---

## Key Terms and Explanations
- **Priority Credits**: Budgeted interventions.
- **TSP**: Transit Signal Priority.
- **Recovery**: Returning to coordination.
- **Equity**: Fair distribution.

---

Cross-links: Related ideas include city rules, safety-first signals, explainable signals.
