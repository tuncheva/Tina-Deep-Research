# 12) Privacy-Friendly Learning (Improve Signals Without Tracking People)

## What it is (precise)
**Privacy-friendly learning** improves signal timing and traffic management decisions using data and algorithms that avoid collecting or retaining **identifying or linkable individual trajectories**.

In practice, this means:
- prefer **aggregated** measurements (counts, occupancy, travel-time summaries),
- minimize raw data retention (especially video/trajectories),
- and/or apply **privacy-preserving computation** (on-device processing, privacy noise, secure aggregation).

## Why digital twins matter
A digital twin can reduce privacy risk because it can:
- use aggregated inputs to simulate system behavior without storing person-level traces,
- support “what-if” evaluation in the model rather than continuous invasive sensing,
- quantify the privacy–performance tradeoff (how much accuracy is lost when only using aggregates).

Digital-twin ATSC work frames the twin as a two-way, real-time data exchange representation of the controller/process, often demonstrated with micro-simulation (e.g., SUMO). A twin can be fed with **counts and signal states** rather than persistent identity-linked trajectories. [Dasgupta et al., arXiv](https://arxiv.org/abs/2109.10863)

## Easy explanation
You can make signals smarter using “how many cars and how long queues are,” without needing to know exactly who each driver is or where they went all day.

## Privacy risks to avoid (common in ITS)
| Risk | Example | Safer alternative |
|---|---|---|
| Re-identification | storing full trajectories over time | aggregate in short windows; discard IDs |
| Sensitive inference | home/work pattern extraction | only corridor travel times, not OD chains |
| Over-retention | long video storage | edge analytics + discard raw frames |
| Data sharing leakage | sharing raw probe data | publish only aggregates with thresholds |

## Practical privacy-first architecture
### Data minimization
- Keep only what you need for control: **approach counts, occupancy, phase states, travel times**.
- Set retention: raw sensor streams minutes–hours, aggregates days–months.

### Edge processing
- Process video at the cabinet/edge to produce counts and near-miss proxies; avoid uploading video.

### Access control + auditing
- Role-based access for operations vs research.
- Audit logs for any export.

## Implementation plan (phased)
### Phase 0 — privacy requirements
- Define the “no-go” list: faces/plates, full trajectories, persistent identifiers.
- Define retention schedules and redaction policies.

### Phase 1 — aggregated data pipeline
- Ensure controller + detectors emit safe aggregates.
- Build quality checks (missing data, sensor drift) without needing raw identities.

### Phase 2 — twin-based optimization
- Calibrate the twin using aggregates.
- Use the twin for evaluation of timing changes and predictive operations.

### Phase 3 — privacy-preserving enhancements
- Add on-device learning or privacy-preserving aggregation if sharing across agencies/vendors.

## Comparison table (privacy approaches)
| Approach | What you keep | Privacy strength | Operational impact |
|---|---|---|---|
| Aggregates only | counts, occupancy, travel-time summaries | high | may reduce fidelity |
| Edge analytics | counts + events, discard raw video | high | needs edge compute |
| Differential privacy | noisy aggregates | very high for sharing | accuracy tradeoffs |
| Secure aggregation | encrypted sharing of aggregates | high | implementation complexity |

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Privacy & trust | less surveillance risk | may reduce accuracy vs full trajectories | focus on key aggregates; periodic calibration |
| Compliance | easier to align with privacy laws/policies | governance overhead | standard retention + audit playbooks |
| Cost | less storage/bandwidth | edge compute cost | right-size edge hardware |

## MVP (smallest useful deployment)
- Publish a **data inventory**: signals collected, retention windows, and who has access.
- Move video analytics to **edge counting** (no raw video retention) at 1–2 pilot intersections.
- Calibrate the twin using **aggregates only** (counts, occupancy, phase states, travel-time summaries).
- Add an “export gate”: any data export requires a ticket + audit log entry.

## Open questions
- What is the minimum data needed for 1–5 minute control decisions with acceptable accuracy?
- Which privacy technique fits best: retention minimization, secure aggregation, or differential privacy?
- How to validate that aggregates can’t be used to re-identify individuals (especially in low-volume areas)?

## Evaluation checklist
- Data inventory (what is collected, for how long)
- Control performance: delay, reliability, spillback risk
- Privacy tests: re-identification risk assessment, access audit results

## Sources
- https://arxiv.org/abs/2109.10863
