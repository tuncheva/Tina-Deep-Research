# 13) Red-Team the Signals

## What it is (precise)
Continuously **stress-test signal control and its data inputs** (detectors, comms, connected-vehicle feeds) in a digital twin to identify vulnerabilities, quantify impacts, and design safe mitigations and fallbacks.

This is a structured “red team” program: attack simulation + detection + response playbooks.

## Why digital twins matter
You can’t safely run spoofing/jamming tests on a live city network. A twin enables:
- repeatable adversarial scenarios (sensor spoofing, false calls, missing data),
- measuring impacts on queues, safety proxies, and throughput,
- testing mitigations (sensor fusion, plausibility checks, conservative modes).

## Easy explanation
Hack your own signals in simulation so you can fix the weak points before someone else finds them.

## What is needed (data & infrastructure)
| Need | Typical options | Notes |
|---|---|---|
| Threat model | detector spoofing, CV data spoofing, comms compromise | Keep it practical: what could happen to your deployments. |
| Twin fidelity | controller logic + detection modeling | Needs realistic detector behaviors and failure injection. |
| Monitoring | anomaly detection on detector health + flows | Used to detect attacks and trigger fallbacks. |
| Response modes | degrade-to-safe plans, isolation, alerts | Must be fast and operator-auditable. |

## Implementation plan (phased)
### Phase 0 — define critical assets + worst outcomes (1–3 weeks)
- Identify what must not fail: pedestrian phases, clearance intervals, emergency routes.
- Define unacceptable outcomes: spillback blocking rail crossings, gridlock, safety-critical phase violations.

### Phase 1 — build attack library (4–8 weeks)
- Loop spoofing (false presence), stuck-on calls, suppressed detections.
- CV spoofing / false requests (where relevant).
- Comms delay/blackout.

### Phase 2 — mitigation design in twin (4–10 weeks)
- Add plausibility checks, multi-sensor cross-validation.
- Add graded degradation modes (see idea 11).

### Phase 3 — operationalize (ongoing)
- Run periodic red-team drills in the twin.
- Track “mean time to detect” and “mean time to safe mode”.

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Security posture | reduces surprise failures | requires specialized skills | start with simple attack cases |
| Resilience | mitigations are tested before field deployment | twin may miss real-world quirks | validate with limited lab/HIL testing |
| Governance | creates auditable risk register | may surface uncomfortable issues | treat as continuous improvement |

## Real-world anchors (what exists today)
A 2025 UC Irvine thesis explicitly studies attacks on physical inductive loop detectors (ILDs) via magnetic loop spoofing, using a simulation framework and reporting measurable degradation in tracking metrics and intersection throughput under attack. This is direct evidence that realistic, physical sensor attacks can materially impact intersection performance—exactly the kind of scenario a red-team-in-the-twin program is meant to rehearse and defend against. [UC Irvine thesis: Securing Intelligent Intersections (2025)](https://escholarship.org/content/qt50m2f8c7/qt50m2f8c7.pdf)

## Evaluation checklist (practical)
- Coverage of attack library (top risks modeled)
- Detection accuracy (false positives/negatives)
- Time to safe mode and recovery time
- Performance under attack (throughput, spillback)
- Operator clarity (alerts are actionable)

## Sources
- https://escholarship.org/content/qt50m2f8c7/qt50m2f8c7.pdf
