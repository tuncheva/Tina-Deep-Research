# 10) Stop-Line Intent Coordination: Precise Green Allocation

## Brief Description
Stop-line intent coordination synchronizes vehicle stops with signal changes for smoother flows.

## Analogical Reference
Like adjusting a recipe based on available ingredients, signals reallocate green time to waiting vehicles.

## Comprehensive Information
Using lane-level detection from sensors or CV data, signals allocate green time to queued movements, reducing waste and improving responsiveness while maintaining safety.

## Upsides and Downsides

### Positive Aspects on People's Lives in 5 Years
- Efficient Travel: Less wasted time at lights, shorter commutes.
- Adaptive Responses: Better handling of traffic surges.

### Positive Aspects on People's Lives in 15 Years
- Predictive Flows: AI anticipates intents for seamless urban mobility.

### Downsides in 5 and 15 Years
- Detection Errors: Poor data can lead to misallocations and frustrations.
- Coordination Issues: Changes may disrupt corridor flows.

### Hard Things People Will Have to Overcome When Getting Used to It
- Data Accuracy: Ensuring reliable lane detection.
- Balancing Fairness: Avoiding bias toward certain movements.
- Integration Complexity: Combining with existing systems.

## What it is (precise)
Use approach and lane-level intent (left/through/right queues at stop line) from detection or connected vehicles to allocate green time precisely, reducing wasted green on idle movements and serving queued ones. This involves mapping lanes to phases, forecasting queue growth, and reallocating splits dynamically while respecting pedestrian constraints and coordination stability.

## Benefits
Optimizes efficiency and responsiveness:
- **Reduced Waste**: Less green on empty lanes; lower delays.
- **Better Responsiveness**: Adapts to turning surges.
- **Improved Flow**: Enhanced queue management per movement.
- **Resource Savings**: Maximizes infrastructure use.

## Challenges
Requires accurate detection and coordination:
- **Data Quality**: Noisy detectors misallocate green.
- **Coordination Risks**: Myopic changes break corridors.
- **Complexity**: Integrating lane mappings and CV data.
- **Safety**: Must maintain pedestrian services.

## Implementation Strategies
### Infrastructure Needs
- Lane-level detection: Loops, radar, video.
- Movement mapping: Lane-to-phase configs.
- CV interface: SPaT/MAP for richer intent.
- Twin: For spillback testing.

### Detailed Implementation Plan
#### Phase 1: Mapping and Baseline (Weeks 1-4)
- Verify lane/phase mappings; establish KPIs.
- Team: Engineers.
- Budget: $50k.
- Risks: Mapping errors.
- Timeline: 4 weeks; Deliverable: Baseline metrics.

#### Phase 2: Detection Rollout (Weeks 5-12)
- Implement intent from stop-line data.
- Add rules for min/max greens.
- Team: Techs.
- Budget: $150k.
- Risks: Detector issues.
- Timeline: 8 weeks; Deliverable: Intent signals.

#### Phase 3: Twin Evaluation (Weeks 13-20)
- Shadow simulations; validate reallocations.
- Team: Analysts.
- Budget: $100k.
- Risks: Model inaccuracies.
- Timeline: 8 weeks; Deliverable: Validation reports.

#### Phase 4: CV Enrichment (Weeks 21-32)
- Integrate SPaT/MAP where available.
- Fallback to detection-only.
- Team: Devs.
- Budget: $200k.
- Risks: Coverage gaps.
- Timeline: 12 weeks; Deliverable: Enhanced system.

#### Phase 5: Corridor Extension (Ongoing)
- Coordinate with adjacent signals.
- Add incident modes.
- Budget: $300k.
- Timeline: Continuous.

### Choices
- **Detection-Only**: Basic presence.
- **CV-Enhanced**: Richer mapping.
- **Hybrid**: Fallback options.

## Future Impacts and Predictions
Intent coordination will reduce delays by 20% in 5 years with CV adoption. In 15 years, AI predicts intents proactively.

### Comparison Tables: Upsides vs Downsides

| Time Horizon | Aspect | Upsides | Downsides |
|--------------|--------|---------|-----------|
| **In 5 Years (Post-Implementation)** | **Efficiency** | Precise allocation. | Detection errors. |
| | **Reliability** | Surge response. | Coordination breaks. |

| Time Horizon | Aspect | Upsides | Downsides |
|--------------|--------|---------|-----------|
| **In 15 Years (Post-Implementation)** | **Efficiency** | Proactive prediction. | Tech over-reliance. |
| | **Reliability** | AI adaptation. | Cybersecurity. |

**Hard Things to Overcome (Across Horizons)**:
- Data Noise: Quality filters.
- Coordination: Caps on changes.
- Equity: Fair reallocations.

## Implementation Costs and Case Studies

### Costs for Implementation
- **Hardware**: Lane detection - $50k-$100k/intersection.
- **Software**: Mapping engine - $80k-$150k.
- **Annual Ops**: Maintenance - $30k.

### Real-World Case Studies
- **NOCoE SPaT**: Implementation guides for intent mapping.
- **ScienceDirect**: Real-time control methodologies.

### Additional Implementation Details
- Phased by intersection.
- Operator training.

## Technical Mechanics
### Key Parameters
- Wasted green %, queue indicators.

### Coordination Types
- Split reallocations, phase adjustments.

### Guardrails
- Min greens, ped constraints.

## MVP Deployment
- One intersection; 3-5s reallocations.

## Evaluation
- Wasted green, split failures, queues.

---

## Key Terms and Explanations
- **Intent**: Queue presence per lane/movement.
- **SPaT**: Signal Phase and Timing messages.
- **Wasted Green**: Unutilized green time.
- **Reallocation**: Dynamic split changes.

---

Cross-links: Related ideas include green waves, what-if button, fair priority credits.
