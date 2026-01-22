# 19) Crowd-Smart Crossings: Dynamic Pedestrian Phasing

## Brief Description
Crowd-smart crossings adjust pedestrian phasing dynamically for large groups using LPI or scramble phases.

## Analogical Reference
Like a conductor adapting to orchestra size, signals adjust phases for crowd size.

## Comprehensive Information
Detecting crowd sizes via sensors, signals switch to leading pedestrian intervals or exclusive phases, simulated in twins for safety.

## Upsides and Downsides

### Positive Aspects on People's Lives in 5 Years
- Safer Crossings: Reduced conflicts for pedestrians and vehicles.
- Accessible Mobility: Better handling of surges for all users.

### Positive Aspects on People's Lives in 15 Years
- Intelligent Ped Management: AI predicts and optimizes phasing automatically.

### Downsides in 5 and 15 Years
- Vehicle Delays: Increased wait times for cars during ped phases.
- Sensor Challenges: Reliability in varying conditions.

### Hard Things People Will Have to Overcome When Getting Used to It
- Detection Accuracy: Multi-sensor fusion for crowds.
- Threshold Adjustments: Location-specific tuning.
- Public Education: Understanding dynamic changes.

## What it is (precise)
Crowd-smart crossings adjust pedestrian phasing in real time for large groups, switching between LPI, lagging intervals, or exclusive phases. Digital twins forecast crowd arrivals, simulate impacts on conflicts and queues, selecting strategies with safety constraints. FHWA describes LPI (3-7s head start) and scramble phases for reduced conflicts, with tradeoffs in cycle length.

## Benefits
Enhances pedestrian safety and flow:
- **Conflict Reduction**: LPI/scramble minimize turn-ped collisions; FHWA PSC shows visibility gains.
- **Surge Handling**: Adapts to crowds without delays; reduces repeated button presses.
- **Accessibility**: Prioritizes vulnerable walkers; enforces min clearance.
- **Efficiency**: Balances ped/vehicle needs; twin simulations optimize.

## Challenges
Requires accurate detection and tuning:
- **Sensor Reliability**: Weather/lighting affect detection; need robust methods.
- **Mode Selection**: Choosing LPI vs scramble based on volumes.
- **Vehicle Impacts**: Increased delays; need max queue caps.
- **Integration**: With event modes; avoid oscillations.

## Implementation Strategies
### Infrastructure Needs
- Ped detection: Video, radar, thermal for queue/flow.
- Controller flexibility: Configurable LPI/scramble.
- Twin for simulations.

### Detailed Implementation Plan
#### Phase 1: Site Selection (Weeks 1-4)
- Identify surge-prone crossings.
- Team: Planners.
- Budget: $40k.
- Risks: Over-selection.
- Timeline: 4 weeks; Deliverable: Site list.

#### Phase 2: Detection Upgrade (Weeks 5-12)
- Implement crowd metrics.
- Team: Engineers.
- Budget: $120k.
- Risks: Detection failures.
- Timeline: 8 weeks; Deliverable: Observability.

#### Phase 3: Strategy Deployment (Weeks 13-20)
- Add safe mode menu.
- Team: Devs.
- Budget: $100k.
- Risks: Unsafe switches.
- Timeline: 8 weeks; Deliverable: Modes.

#### Phase 4: Twin Integration (Weeks 21-32)
- Shadow recommendations.
- Team: Analysts.
- Budget: $150k.
- Risks: Poor predictions.
- Timeline: 12 weeks; Deliverable: Smart switching.

#### Phase 5: Tuning (Ongoing)
- Adjust thresholds; audits.
- Budget: $60k annual.
- Timeline: Continuous.

### Choices
- **Concurrent**: Baseline.
- **LPI**: Head start.
- **Scramble**: Exclusive ped.

## Future Impacts and Predictions
Crowd-smart will reduce ped conflicts by 25% in 5 years. In 15 years, AI predicts surges instantly.

### Comparison Tables: Upsides vs Downsides

| Time Horizon | Aspect | Upsides | Downsides |
|--------------|--------|---------|-----------|
| **In 5 Years (Post-Implementation)** | **Safety** | Fewer conflicts. | Detection issues. |
| | **Flow** | Surge adaptation. | Vehicle delays. |

| Time Horizon | Aspect | Upsides | Downsides |
|--------------|--------|---------|-----------|
| **In 15 Years (Post-Implementation)** | **Safety** | Proactive phasing. | Tech dependencies. |
| | **Flow** | AI optimization. | Over-complexity. |

**Hard Things to Overcome (Across Horizons)**:
- Detection in Conditions: Multi-sensor fusion.
- Threshold Tuning: Location-specific.
- Public Acceptance: Signage and education.

## Implementation Costs and Case Studies

### Costs for Implementation
- **Hardware**: Ped sensors - $70k-$140k.
- **Software**: Mode logic - $100k-$200k.
- **Annual Ops**: Maintenance - $40k.

### Real-World Case Studies
- **FHWA LPI**: Proven safety countermeasure.
- **Scramble Examples**: Reduced conflicts in high-ped areas.

### Additional Implementation Details
- MUTCD compliance.
- Crowd management integration.

## Technical Mechanics
### Key Parameters
- Ped queue length, arrival rate.

### Coordination Types
- Phasing switches with constraints.

### Guardrails
- Min clearance, max queues.

## MVP Deployment
- One crossing; LPI mode.

## Evaluation
- Conflict proxies, delays.

---

## Key Terms and Explanations
- **LPI**: Leading Pedestrian Interval.
- **Scramble**: Exclusive ped phase.
- **Crowd Metrics**: Queue/flow estimates.

---

Cross-links: Related ideas include safety-first signals, city rules.
