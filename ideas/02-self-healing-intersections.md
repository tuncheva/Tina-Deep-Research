# 02) Self-Healing Intersections (Fail-Safe Detection + Fallback Timing)

## What it is (precise)
A **self-healing intersection** is a signalized intersection that can **detect abnormal/failed inputs** (detectors, comms, controller state) and **degrade gracefully** to a known-safe operating mode (e.g., fixed-time plan, conservative actuated plan, protected-only turns) until health is restored.

In practice, this is “fault detection + safe fallback,” applied to traffic control. The goal is not to be optimal during failures; the goal is to stay **safe, predictable, and legally compliant**, while minimizing disruption.

## Why digital twins matter
A digital twin helps because it can:
- run **what-if safety checks** on fallback plans (will queues spill back into upstream intersections?),
- maintain a **model-based estimate** of queues when sensing fails (within guardrails),
- act as a **test harness** for failure scenarios (stuck-on detectors, missing occupancy, comms loss).

Digital-twin ATSC literature highlights a twin as a two-way, real-time data exchange representation of the controller/process, and uses micro-simulation (e.g., SUMO) to evaluate adaptive behaviors; this supports the idea of rehearsing detection failures and safe fallback logic. [Dasgupta et al., arXiv](https://arxiv.org/abs/2109.10863)

## Easy explanation
If the signal “realizes it’s blind or confused,” it switches to a simple, safe plan that still lets everyone move, rather than making risky decisions based on bad data.

## Comparison table (graded degradation strategy)
| Mode | Trigger | Control behavior | Goal |
|---|---|---|---|
| Normal | healthy detectors + comms | adaptive/optimized control | performance |
| Conservative | partial degradation | tighter caps, fewer changes, conservative recalls | bounded-risk control |
| Fallback plan | sensors failed or inconsistent | fixed-time / approved plan library | predictable service |
| Local isolated | comms loss | local controller logic only | safe autonomy |

## Failure modes to handle (examples)
| Category | Example symptom | Typical cause |
|---|---|---|
| Detector stuck-on | constant occupancy/call | damaged loop, camera zone error |
| Detector stuck-off | zero counts all day | power/cable fault, misalignment |
| Impossible speeds/flows | travel times implausible | timestamp drift, data fusion bug |
| Comms drop | controller stops reporting | fiber/cellular outage |
| Controller state mismatch | phases not as commanded | cabinet/controller malfunction |

## What “safe fallback” looks like
Typical fallback library (choose 1–3 per intersection):
- **Fixed-time plan** (time-of-day) with conservative clearance.
- **Basic actuated plan** with bounded extensions.
- **Protected-only left turns** (if permissives become risky under uncertainty).
- **All-red extension** under certain abnormal conflicts (use sparingly).

## Implementation plan (phased)
### Phase 0 — define safety & compliance constraints
- Confirm legal constraints: min walk times, yellow/all-red, coordination requirements.
- Define “do no harm” thresholds: maximum queue spillback probability, max ped wait, max cycle.

## Failure-mode drills (what to simulate in the twin)
| Failure / degradation | How it manifests | What a safe controller should do |
|---|---|---|
| Detector stuck-on | constant call, inflated occupancy | cap extensions; apply stuck-on timeout; ignore offending channel |
| Detector stuck-off | no actuation with queued traffic | enable recall/minimum; switch to pre-timed split if widespread |
| Comms loss (central) | no remote commands / no telemetry | run local isolated; log “central blind” events and durations |
| Time skew | offsets drift; coordination breaks | detect drift; resync; freeze offsets during instability |
| State mismatch | controller not in commanded plan | fail-safe revert to last known-good; require operator ack if repeated |

## Practical guardrails (so “self-healing” doesn’t flap)
- **Hysteresis**: enter fallback quickly; exit fallback only after stable health for N minutes.
- **Oscillation detection**: if mode switches happen repeatedly, hold in a conservative plan and alert operators.
- **Always-on caps**: max cycle length, max ped wait, min greens/clearances.
- **Audit log**: emit reason code + implicated channels + duration (pairs well with [20) Explainable Signals](../ideas/20-explainable-signals.md)).

### Phase 1 — health monitoring
- Implement detector health KPIs (stuck-on/off tests, variance tests, sanity bounds).
- Implement comms/controller heartbeat checks.

### Phase 2 — fallback plan library
- Create, test, and approve fallback plans for each intersection and corridor.
- Encode clear rules for *entering* fallback and *exiting* fallback (hysteresis to avoid flapping).

### Phase 3 — digital twin validation
- Use the twin to simulate failure scenarios and confirm:
  - no unsafe phase sequences,
  - queues don’t cause blocked-box cascades,
  - pedestrian service remains within policy bounds.

### Phase 4 — operations + governance
- Add operator UI for health status and current mode.
- Log every fallback event, reason, duration, and observed impacts.

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Safety | reduces risk when inputs are wrong | over-triggering degrades service | thresholds + hysteresis + periodic audits |
| Reliability | keeps intersection functional during outages | may create long queues in peaks | multi-level fallback; corridor-aware selection |
| Maintainability | surfaces failing sensors fast | requires ongoing tuning | monthly health dashboards; automated alerts |
| Public trust | more predictable than erratic signals | can feel “broken” during fallback | messaging + visible status for operators |

## Interoperability / standards anchor
Priority and control systems often interface with multiple entities (requesters, controllers, management centers). Standards like **NTCIP 1211** define management information objects and interfaces for signal control and prioritization, which supports implementing controlled mode switching and monitored status across devices. [NTCIP 1211](https://www.ntcip.org/file/2018/11/NTCIP1211-v0224j.pdf)

## MVP (smallest useful deployment)
- Implement **detector + comms health scoring** with 3 states: `healthy`, `degraded`, `failed`.
- Configure **one approved fallback plan** per intersection (plus a “local isolated” comms-loss mode).
- Add **entry/exit hysteresis** (e.g., require 2–5 minutes stable health before exiting fallback).
- Log every event with: reason, detectors implicated, mode entered, duration, and KPI deltas.

## Open questions
- What are the right thresholds to minimize false positives while still catching dangerous faults early?
- How should fallback selection be **corridor-aware** (so one intersection’s fallback doesn’t break progression badly)?
- Which failure types require operator confirmation vs automatic switching?

## Evaluation checklist
- Number of fallback events/week and mean time to recovery
- False positive rate (fallback entered with healthy sensors)
- Safety proxy checks: red-clearance compliance, ped service compliance
- Operational impact: delay increase during fallback vs baseline

---

## Deep dive (architecture): the “health → mode” state machine
Self-healing is easiest to implement as a **small state machine** with explicit transitions and reason codes.

### States
- `NORMAL`: full adaptive/optimized control allowed.
- `DEGRADED`: control allowed, but only within tighter guardrails (smaller action set; conservative recalls).
- `FALLBACK`: switch to an approved plan from a library.
- `ISOLATED`: comms loss; local controller rules only.

### Transition rules (example)
| From → To | Condition | Notes |
|---|---|---|
| NORMAL → DEGRADED | any critical sensor health < 0.7 for 30s | enter quickly |
| DEGRADED → FALLBACK | health < 0.4 or state mismatch detected | prioritize safety/predictability |
| * → ISOLATED | central comms heartbeat missed for > N seconds | isolate immediately |
| FALLBACK → DEGRADED | health stable > 0.7 for 5–10 min | avoid “flapping” |
| DEGRADED → NORMAL | health stable > 0.9 for 10–30 min | require high confidence |

Twin implication: the twin should be able to simulate **state transitions** (not only “best timing”), because the biggest operational risks come from **mode switching**.

---

## Deep dive (detection): fault detection patterns that work in practice
Use multiple weak tests instead of one brittle rule.

### Basic channel tests
- **Stuck-on**: occupancy/call present for > X% of time across Y minutes.
- **Stuck-off**: zero activations across a window when adjacent channels show activity.
- **Chatter**: too many transitions per minute (video zone noise).
- **Variance collapse**: flow variance drops to near-zero for long periods.

### Cross-sensor consistency tests
- Count vs. occupancy vs. speed: “high occupancy + zero counts” is often wrong.
- Adjacent detectors: approach upstream vs stop-bar should correlate in peaks.
- Probe travel times: if travel time spikes but counts remain flat, suspect sensor error.

### Time integrity tests
- Controller clock skew breaks coordination and event ordering.
- Validate NTP sync; alarm on sustained drift.

---

## Deep dive (fallback design): what makes a fallback plan “safe”
Fallback plans should be designed with *bounded risk* as the objective.

Checklist for each fallback plan:
- Fixed phase sequence is legal and matches field wiring.
- Ped minimums and clearances always satisfied.
- Conservative all-red / change intervals (within standards).
- Side-street starvation prevented via max-red caps.
- Storage protection: minimize blocked-box cascades by throttling upstream release when downstream storage is low.

Corridor nuance: a single intersection fallback can break progression. Maintain at least:
- a **local-only fallback** (safe for that intersection), and
- a **corridor-aware fallback** (keeps coordination coherent at reduced performance).

---

## Deep dive (operations): incident workflow and escalation
Cross-link: see [`ideas/08-operator-option-menu.md`](ideas/08-operator-option-menu.md)

Operator workflow:
1. Alert appears: “Detector 12 stuck-on; entering DEGRADED (auto).”
2. Screen shows: impacted movements, current mode, expected KPI impact.
3. System recommends an action: “Switch to Fallback Plan F2 (protected-only lefts) until repair.”
4. Operator can:
   - accept recommendation,
   - pick an alternate approved plan,
   - request maintenance ticket auto-filled with diagnostics.

Escalation policy:
- repeated mode flips → hold in FALLBACK and page supervisor.
- state mismatch persists → require field check.

---

## Metrics (operator + engineering)
- Mean time to detect and isolate a fault (MTTD)
- Mean time to recovery (MTTR)
- % time in each mode (NORMAL/DEGRADED/FALLBACK/ISOLATED)
- Phase failures / max-outs frequency (proxy for underservice)
- False positive / false negative rate of health triggers

---

## Related topics
- [`ideas/11-delay-tolerant-smart-signals.md`](ideas/11-delay-tolerant-smart-signals.md) — system-wide safe degradation and fallbacks.
- [`ideas/03-real-time-what-if-button.md`](ideas/03-real-time-what-if-button.md) — safely testing fallback selection / changes in shadow mode.
- [`ideas/20-explainable-signals.md`](ideas/20-explainable-signals.md) — reason codes and audit logs for every mode switch.

## Sources
- https://arxiv.org/abs/2109.10863
- https://www.ntcip.org/file/2018/11/NTCIP1211-v0224j.pdf
