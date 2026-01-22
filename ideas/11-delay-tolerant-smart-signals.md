# 11) Delay-Tolerant Smart Signals

## What it is (precise)
Design signal control so it remains **safe and stable when data is late, missing, or wrong**—by continuously checking detector/comms health, degrading gracefully to conservative operation, and keeping a vetted set of fallback timing plans.

The goal is not “perfect optimization,” but **bounded-risk control** under real-world failures.

## Why digital twins matter
A digital twin lets you rehearse failure modes that are hard to test in the field:
- detector stuck-on/stuck-off, noisy counts, missing phase logs,
- communications latency/outages between field and central,
- partial power constraints and cabinet resets.

You can quantify how failures propagate (queues/spillback) and validate that fallback logic maintains safety and acceptable service.

## Easy explanation
When the sensors or network get flaky, the signal system should automatically switch to a safe, conservative mode instead of making bad “smart” decisions.

## What is needed (data & infrastructure)
| Need | Typical options | Notes |
|---|---|---|
| Health monitoring | detector diagnostics, missing-data counters, plausibility checks | Detects stuck calls, impossible flows, and dropouts. |
| Fallback plans | fixed-time plans, all-way flash rules, conservative actuated settings | Pre-vetted per intersection/corridor. |
| Controller logs | high-resolution phase + detector logs | Needed to detect anomalies and audit behavior. |
| Digital twin | micro/meso sim + controller logic emulation | Used to stress-test degraded modes before field rollout. |

## Implementation plan (phased)
### Phase 0 — define failure catalog + safety invariants (1–3 weeks)
- List common failures: detector stuck-on, detector dead, comms outage, time drift, controller reboot.
- Define invariants: pedestrian minimums, clearance intervals, max cycle, red-revert behavior.

### Phase 1 — implement health checks (2–6 weeks)
- Add per-detector quality metrics (uptime %, stuck-on duration, dropouts).
- Add plausibility checks (e.g., volume without occupancy patterns, contradictory calls).

### Phase 2 — graded degradation modes (4–8 weeks)
- Mode A: fully adaptive (best data).
- Mode B: conservative adaptive (reduced aggressiveness; stricter caps).
- Mode C: time-of-day fixed plan.
- Mode D: flash / local isolated mode.

### Phase 3 — twin-based failure drills (ongoing)
- Simulate failures and confirm KPIs remain within bounds.
- Train operators: what triggers degrade/recover and how to override.

## Comparison table (failure handling techniques)
| Technique | What it does | Strength | Weakness |
|---|---|---|---|
| Heartbeats + health scores | detects comms/sensor degradation | simple + measurable | can miss subtle faults |
| Plausibility checks | flags impossible flows/speeds | catches spoofing/bugs | tuning is hard |
| Graded modes | step down control aggressiveness | bounded risk | may degrade performance |
| Pre-approved plan library | safe fallback plans | predictable | needs maintenance |

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Resilience | fewer catastrophic “bad timing” incidents | added logic complexity | keep modes simple; test in twin |
| Safety | graceful fallback preserves constraints | risk of over-conservative operation | define recovery rules; monitor KPIs |
| Operations | clearer troubleshooting | requires good logging/telemetry | standardize logs and dashboards |

## Real-world anchors (what exists today)
A concrete motivation for delay-tolerant design is that intelligent intersections can be attacked or degraded when sensor inputs are manipulated or unreliable. A 2025 UC Irvine thesis studies physical inductive-loop spoofing attacks and shows that such attacks can significantly degrade both tracking metrics and intersection throughput in simulation, emphasizing the need for robust detection health monitoring and safe fallbacks. [UC Irvine thesis: Securing Intelligent Intersections (2025)](https://escholarship.org/content/qt50m2f8c7/qt50m2f8c7.pdf)

## Failure-mode drills (what to simulate in the twin)
| Failure / degradation | How it manifests | What a safe controller should do |
|---|---|---|
| Detector stuck-on | constant call, inflated volume/occupancy | cap extensions; use plausibility; ignore that detector after threshold |
| Detector dead | no calls even with queued traffic | failover to recall/minimum service; switch to fixed plan if widespread |
| Clock drift / time skew | offsets “slide”; coordination breaks | detect drift; resync; log drift events |
| Comms latency | late state arrives after decision | act locally; treat late data as stale; widen safety margins |
| Partial observability | only probe travel times; no counts | use conservative plan selection; avoid per-cycle micromanagement |
| Controller reboot | momentary all-red or default plan | enforce safe startup plan; require explicit restore-to-plan logic |

## Practical guardrails (so degradation is bounded)
- **Oscillation protection**: once you drop into a fallback mode, require a minimum dwell time and a clean bill of health for N minutes before stepping back up.
- **Caps that always apply**: max cycle length, max pedestrian wait, max queue spillback risk (if detected), and no “short greens” below minima.
- **Auditability**: every mode switch should emit a reason code and the health signals that triggered it (pairs naturally with [20) Explainable Signals](../ideas/20-explainable-signals.md)).

## MVP (smallest useful deployment)
- Define the 4 modes (`A` adaptive, `B` conservative, `C` fixed, `D` flash) with clear entry/exit rules.
- Add per-intersection **health scorecards** (detector uptime, comms latency, time drift flags).
- Run in **shadow mode** first: compute recommended mode and compare with operator judgment.
- Implement a one-click “force mode C” operator override with automatic expiry.

## Open questions
- Which signals should be centralized vs local-only under comms loss?
- How can we detect “wrong data” (plausibility) without over-triggering fallbacks?
- What should be the default conservative parameters (caps) by intersection type?

## Evaluation checklist (practical)
- Time to detect failure and enter safe mode
- Queue spillback during degraded operation
- False-activation rate (unnecessary fallbacks)
- Recovery behavior (no oscillation between modes)
- Pedestrian compliance (mins never violated)

## Sources
- https://escholarship.org/content/qt50m2f8c7/qt50m2f8c7.pdf
- https://airc.nist.gov/docs/AI_RMF_Playbook.pdf
