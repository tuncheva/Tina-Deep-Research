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

## Evaluation checklist
- Number of fallback events/week and mean time to recovery
- False positive rate (fallback entered with healthy sensors)
- Safety proxy checks: red-clearance compliance, ped service compliance
- Operational impact: delay increase during fallback vs baseline

## Sources
- https://arxiv.org/abs/2109.10863
- https://www.ntcip.org/file/2018/11/NTCIP1211-v0224j.pdf
