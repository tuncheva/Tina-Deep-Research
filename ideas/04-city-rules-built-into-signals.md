# 04) City Rules Built Into Signals: Policy-Driven Traffic Control

## Brief Description
City rules built into signals encode policy goals as constraints in controllers to ensure fair and safe traffic management.

## Analogical Reference
Like traffic laws enforced by police, these rules are directly integrated into signals to automatically prioritize safety and equity.

## Comprehensive Information
This approach embeds city policies into signal timing algorithms, using constraints for pedestrian waits, transit priority, and bike safety. It relies on multimodal sensors and digital twins for testing, following standards like MUTCD and NACTO guidelines. This ensures compliance with equity goals while optimizing flow, with audits for transparency.

## Upsides and Downsides

### Positive Aspects on People's Lives in 5 Years
- Safer Crossings: Reduced pedestrian accidents through enforced rules, improving community safety.
- Equitable Access: Better transit and bike priority, making cities more inclusive.

### Positive Aspects on People's Lives in 15 Years
- Sustainable Cities: Integrated policies reduce congestion and emissions, enhancing quality of life.

### Downsides in 5 and 15 Years
- Potential Delays: Increased vehicle wait times to prioritize others.
- Implementation Complexity: High costs and tuning challenges.

### Hard Things People Will Have to Overcome When Getting Used to It
- Balancing Priorities: Managing trade-offs between different user groups.
- Political Resistance: Agreeing on policies in diverse communities.
- Sensor Reliability: Ensuring accurate detection for rule enforcement.

## Benefits
Enforces equity and safety:
- **Policy Compliance**: Guarantees rules like ped waits are met.
- **Multimodal Balance**: Prioritizes buses, bikes, walkers.
- **Auditability**: Decisions traceable to policy.
- **Safety Gains**: Reduces conflicts and violations.

## Challenges
Balancing constraints with flow:
- **Tradeoffs**: May increase vehicle delays.
- **Tuning**: Hard to set constraint levels.
- **Detection Needs**: Requires multimodal sensors.
- **Governance**: Political conflicts over priorities.

## Implementation Strategies
### Infrastructure Needs
- **Policy Engine**: Software for encoding rules as constraints/objectives.
- **Multimodal Sensors**: Ped buttons, video, transit AVL, bike detectors.
- **Controller Upgrades**: Support LPI, TSP, dynamic phasing.
- **Twin Simulation**: Corridor models for testing policies.

### Detailed Implementation Plan
#### Phase 1: Policy Development (Weeks 1-6)
- **Stakeholder Workshops**: Involve city planners, safety experts, transit agencies to define priorities (e.g., school zones, bus reliability).
- **Rule Translation**: Convert policies to measurables (max ped wait 30s, LPI required).
- **Legal Alignment**: Ensure compliance with MUTCD, access laws.
- **Budget**: $100k for consulting/engagements.
- **Risks**: Conflicting priorities; iterate.
- **Timeline**: 6 weeks; Deliverable: Policy contract doc.

#### Phase 2: Technical Design (Weeks 7-14)
- **Sensor Deployment**: Install multimodal detection at 5-10 intersections.
- **Policy Engine Build**: Develop software for hard/soft constraints; integrate with controllers.
- **Twin Setup**: Calibrate corridor models; simulate policy scenarios.
- **Testing**: Validate rule enforcement in simulation.
- **Team**: Policy experts, devs.
- **Budget**: $300k-$600k.
- **Risks**: Sensor reliability; pilot.
- **Timeline**: 8 weeks; Deliverable: Functional prototype.

#### Phase 3: Pilot Implementation (Weeks 15-24)
- **Zone Selection**: Choose school/transit corridor for testing.
- **Rule Activation**: Enable 5-10 constraints; monitor compliance.
- **Operator Training**: Teach policy modes, overrides, dashboards.
- **Data Monitoring**: Track ped delays, transit on-time, violations.
- **Team**: Field installers, trainers.
- **Budget**: $150k.
- **Risks**: Resistance; communicate benefits.
- **Timeline**: 10 weeks; Deliverable: Pilot results.

#### Phase 4: Evaluation and Refinement (Weeks 25-32)
- **Impact Assessment**: Compare KPIs pre/post; survey users.
- **Refinements**: Adjust constraints based on data (e.g., soften if delays too high).
- **Override Protocols**: Implement and test exception workflows.
- **Documentation**: Create compliance reports and governance guides.
- **Team**: Analysts, evaluators.
- **Budget**: $100k.
- **Risks**: Unintended delays; balance.
- **Timeline**: 8 weeks; Deliverable: Refinement plan.

#### Phase 5: Full-Scale Deployment (Weeks 33-52)
- **City-Wide Rollout**: Expand to all intersections; phases by district.
- **Monitoring Systems**: Public dashboards; automated audits.
- **Continuous Governance**: Regular policy reviews; adapt to new goals.
- **Sustainability**: Link to climate plans; monitor equity.
- **Budget**: $1M+ for city-wide.
- **Risks**: Scaling issues; phased approach.
- **Timeline**: 20 weeks; Deliverable: Operational system.

#### Phase 6: Long-Term Maintenance (Ongoing)
- **Updates**: Add new policies (e.g., EV priority); AI for dynamic tuning.
- **Community Feedback**: Surveys on perceived fairness.
- **Budget**: $200k annual.
- **Timeline**: Continuous.

### Choices
- **Hard Constraints**: Enforceable caps.
- **Weighted Objectives**: Tunable scoring.
- **Mode-Based**: Contextual rules.
- **Override Workflow**: Explicit exceptions.

## Future Impacts and Predictions
In 5 years, policy signals will reduce violations by 30%. By 15 years, AI will auto-tune rules for optimal equity.

### Comparison Tables: Upsides vs Downsides

| Time Horizon | Aspect | Upsides | Downsides |
|--------------|--------|---------|-----------|
| **In 5 Years (Post-Implementation)** | **Equity & Safety** | Enforceable priorities; safer crossings. | Increased delays for cars. |
| | **Operations** | Auditable decisions. | Tuning complexity. |
| | **Public Trust** | Transparent policies. | Political debates. |

| Time Horizon | Aspect | Upsides | Downsides |
|--------------|--------|---------|-----------|
| **In 15 Years (Post-Implementation)** | **Equity & Safety** | AI-optimized fairness. | Tech failures bypass rules. |
| | **Operations** | Automated compliance. | Over-constraints limit flexibility. |

**Hard Things to Overcome (Across Horizons)**:
- Constraint Tuning: Dynamic vs fixed.
- Exception Governance: Balanced overrides.
- Verification: Fallback assumptions.

## Implementation Costs and Case Studies

### Costs for Implementation
- **Detection**: Multimodal sensors (ped pushbuttons, video) - $30k-$70k/intersection; supports AVL for transit.
- **Software**: Rule encoding and twin testing - $40k-$100k; includes constraint scoring.
- **Annual Ops**: Monitoring and audits - $15k-$40k; ensures 90%+ compliance reporting.
- **Total**: Pilot zones at $100k-$200k, scaling with benefits in safety and equity.

### Real-World Case Studies
- **NACTO Guidelines**: Leading ped intervals (LPI) and bike coordination reduce conflicts by 20-30%.
- **FHWA**: Signal timing for safety; ped phasing cuts violations; dynamic caps improve flow.
- **ArXiv**: Policy primitives like min service and priorities balance multimodal needs.

### Additional Implementation Details
- Public dashboards for transparency.
- Pilot zones for testing.

## Technical Mechanics
### Key Parameters
- Constraint types: Hard, soft, procedural.

### Coordination Types
- Primitives: Min service, priorities.
- Modes: Contextual rule sets.

### Guardrails
- Health checks, conservative defaults.

## MVP Deployment
- One zone with 3-5 rules, override workflow.

## Evaluation
- Compliance %, ped waits, safety proxies.

---

## Key Terms and Explanations
- **Policy Constraints**: Rules encoded as controller limits.
- **LPI**: Leading Pedestrian Interval for early crossing.
- **TSP**: Transit Signal Priority for buses.
- **Policy Contract**: Documented rules per corridor.
- **Override**: Temporary exception to rules.

---

Cross-links: Related ideas include safety-first signals, privacy-friendly learning, and what-if button.
