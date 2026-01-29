# 11) Delay-Tolerant Smart Signals: Resilient Control Under Uncertainty

## Catchy Explanation
When sensors or comms get flaky, a good signal shouldn’t “panic optimize.” It should fall back gracefully and stay safe and predictable until the data is trustworthy again.

## What it is (precise)
**Delay-tolerant smart signals** are traffic-signal control systems designed to operate safely and predictably when inputs are **late, missing, wrong, or untrusted**.

They continuously assess data health (latency, loss, plausibility), run a graded degradation policy (**NORMAL → DEGRADED → FALLBACK/ISOLATED**), and rely on pre-vetted timing plans that preserve invariants (pedestrian minimums/clearances, max cycles, preemption behavior). Digital twins are used to rehearse comms outages and sensor degradation, test transition logic, and quantify performance bounds during degraded operation.

---

## What this is *not* (and how it differs from “self-healing intersections”)
This idea is deliberately narrower than a full “self-healing” intersection concept:

- **Self-healing** focuses on automatically diagnosing failures and initiating corrective actions (maintenance workflows, detector remaps, adaptive reconfiguration) to restore normal performance.
- **Delay-tolerant smart signals** focus on **bounded, safe control when data quality is degraded**, including strict staleness/latency contracts, authority shaping (“how much the cloud is allowed to influence right now”), and recovery hysteresis so the system does not oscillate.

In practice they pair well: delay-tolerant control provides the *safe operating envelope*, while self-healing provides the *repair path*.

---

## Benefits
- **Resilience**: fewer disruptions from data pipeline failures.
- **Safety assurance**: graceful degradation preserves constraints.
- **Operational clarity**: explicit modes + logs make troubleshooting easier.
- **Bounded performance**: predictable behavior under stress.

## Challenges
- **Over-conservatism**: safer modes can increase delay.
- **Tuning**: thresholds must avoid false triggers.
- **Mode oscillation**: need dwell times/hysteresis.
- **Maintenance**: fallback plans must stay current.

---

## Implementation Strategies

### Infrastructure Needs
- **Health monitoring**: latency/heartbeat checks, missing-data rate, stuck detector detection.
- **Plausibility checks**: cross-sensor consistency, bounds on volumes/occupancy.
- **Fallback plan library**: conservative actuated + fixed-time plans.
- **High-resolution logs / ATSPM**: measure failures and impacts.
- **Twin drills**: repeatable failure injection and transition testing.

---

## Staleness Contract (timestamps, time sync, and authority rules)
Delay-tolerant control needs explicit semantics for “how old is too old” and “what is allowed when it’s old.” Every feed should carry:

- **event_time** (when the field event happened)
- **ingest_time** (when the central system observed it)
- **source_id + trust tier** (controller direct vs intermediary vs probe)
- **integrity metadata** (transport channel, auth context, optional signature/checksum where available)

### Freshness categories
Compute:
- `age_event = now - event_time`
- `age_ingest = now - ingest_time` (useful for diagnosing pipeline delay)

Define staleness categories:
- **FRESH**: within budget → normal optimization permitted.
- **DELAYED**: usable only for slow decisions → bounded adjustments.
- **STALE**: informational only → do not use for control.
- **UNTRUSTED**: fails plausibility/integrity expectations → treat as hostile/noisy.

### Authority rules: central vs local
- **Local controller is always authoritative** for safety constraints (ped timing, clearance, preemption behavior).
- **Central/cloud influence is conditional** on freshness and integrity.
- When in doubt, **reduce authority** (plan selection only, then freeze, then local fallback).

### Per-feed contract table (required visual: feed → max age → allowed actions → operator UI message)

| Data feed | Max age (FRESH) | Max age (DELAYED) | Allowed actions when DELAYED | If STALE/UNTRUSTED… | Operator message |
|---|---:|---:|---|---|---|
| Controller status / phase state | ≤ 2 s | ≤ 10 s | Only corridor plan *selection* (no per-cycle nudges) | Freeze central influence; rely on local plan | “Comms lag; holding local timing.” |
| Detector calls / counts | ≤ 2–5 s | ≤ 30 s | Adjust splits within bounded range + rate limit | Ignore; switch to conservative actuated/fixed | “Detectors stale; using safe plan.” |
| Probe travel time | ≤ 30 s | ≤ 5 min | Only strategic: choose among pre-vetted patterns | Ignore for control; keep last good plan | “Probe delayed; no optimization.” |
| Time sync health | ≤ 1 s drift | ≤ 5 s drift | No transitions requiring tight offsets | Disable coordination transitions; prefer dwell | “Clock drift; coordination degraded.” |

Notes:
- These numbers are **starting points**; tune using observed comms latency distributions.
- The goal is to prevent “false confidence,” especially around coordination transitions.

---

## Latency budgets by control type (what can tolerate delay)
Not all actions are equally sensitive to delay. The Signal Timing Manual emphasizes stability in coordinated operation and documents transition methods used to return to coordination, which should be treated as higher-risk under uncertain timing/latency. See coordination and transition guidance in FHWA Signal Timing Manual Chapter 6 ([`FHWA Signal Timing Manual — Chapter 6`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter6.htm:1)).

Use a simple budget table:

| Control action | Needs “fresh” state? | Typical budget | Why |
|---|---:|---:|---|
| Select among pre-approved timing patterns (plan-of-the-day) | No (tolerates delay) | 30 s–5 min | Slow decision; safe if plans are vetted |
| Adjust split/offset within bounds | Yes | 2–10 s | Wrong/stale state can break coordination/queues |
| Phase holds / force-offs / special transitions | Yes (strict) | < 2 s | Operations sensitive; avoid under uncertainty |
| Preemption coordination behavior | Yes (strict) | real-time | Must preserve preemption invariants |

---

## Flash vs fixed safe operation (decision tree)
“Flash” is sometimes used as an operational mode, but it is not a universal safe fallback; agencies must follow local policy and ensure pedestrian and safety impacts are acceptable. See FHWA Signal Timing Manual Chapter 7 for context on timing plan development and considerations ([`FHWA Signal Timing Manual — Chapter 7`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter7.htm:1)).

Use a decision tree rather than a blanket rule:

1. **Is the controller executing a verified local plan with ped minimums/clearances and preemption behavior preserved?**
   - Yes → prefer **local FALLBACK plan** (actuated/fixed) over flash.
   - No → go to 2.
2. **Is the intersection configured and approved for safe flash operation (policy, signage, visibility, ped accommodation)?**
   - Yes → flash is an option *if* pedestrian service is not required (or is handled by approved procedures).
   - No → go to 3.
3. **Run a conservative fixed-time plan locally** (even if coordination is lost) until the system can re-validate inputs.

Operational caveat: if you cannot guarantee safe flash configuration and pedestrian treatment, do not treat flash as a “safe by default” mode.

---

## Recovery + anti-oscillation: state machine + reason codes
Frequent switching is undesirable. FHWA’s coordination guidance emphasizes stability and documents transition methods used to return to coordination ([`FHWA Signal Timing Manual — Chapter 6`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter6.htm:1)).

Implement a minimal, auditable state machine with hysteresis.

### Mode state machine diagram (required visual: modes with staleness gates)

```text
                 +---------+
   good feeds    | NORMAL  |
 (FRESH/healthy) +---------+
        ^             |
        |             | critical feed DELAYED > X
        |             v
        |        +-----------+
        |        | DEGRADED  |
        |        +-----------+
        |           |      \
        |           |       \ critical feed STALE/UNTRUSTED > Y
        |           |        v
        |           |   +-----------+
        |           |   | FALLBACK  |<-----------------+
        |           |   +-----------+                  |
        |           |        |                         |
        |  comms ok |        | comms lost              |
        |  time sync|        v                         |
        |   healthy |   +-----------+                  |
        |           |   | ISOLATED  |                  |
        |           |   +-----------+                  |
        |           |        |                         |
        |           +--------+                         |
        |      (good_for >= recovery_window           |
        |       & dwell met)                          |
        |                  v                          |
        |            +----------------+               |
        +------------| RECOVERY_VERIFY |--------------+
                     +----------------+
                           |
             good_for >= recovery_window
                           v
                        +---------+
                        | NORMAL  |
                        +---------+
```

- Staleness gates:
  - FRESH/healthy feeds keep the system in `NORMAL`.
  - Critical feeds in `DELAYED` move to `DEGRADED` after `X` seconds.
  - Critical feeds in `STALE/UNTRUSTED` move to `FALLBACK` or `ISOLATED`.
  - Only after a **recovery window** of good health and minimum dwell can the system re-enter `NORMAL` via `RECOVERY_VERIFY`.

### Timers (starting defaults)
- `min_dwell_degraded = 10 min`
- `min_dwell_fallback = 30 min`
- `recovery_good_window = 15 min` (must be continuously “good”)
- `lockout_after_flap = 60 min` (if > N mode flips/hour)

### Reason codes (loggable, searchable)
- `COMMS_HEARTBEAT_STALE`
- `DETECTOR_STUCK_OR_CHATTER`
- `PHASE_STATE_UNKNOWN`
- `CLOCK_DRIFT`
- `PLAUSIBILITY_FAIL`
- `INTEGRITY_FAIL`

### Pseudocode (core transition logic)

```text
if state == NORMAL:
  if critical_feed in DELAYED for >X:
    enter(DEGRADED, reason=COMMS_HEARTBEAT_STALE)
  if critical_feed in STALE or UNTRUSTED for >Y:
    enter(FALLBACK, reason=PLAUSIBILITY_FAIL)

if state == DEGRADED:
  if critical_feed in STALE or UNTRUSTED for >Y:
    enter(FALLBACK, reason=...)
  if good_for >= recovery_good_window and dwell >= min_dwell_degraded:
    enter(NORMAL)

if state in {FALLBACK, ISOLATED}:
  if comms_restored and time_sync_ok and good_for >= recovery_good_window and dwell >= min_dwell_fallback:
    enter(RECOVERY_VERIFY)

if state == RECOVERY_VERIFY:
  if any_watchdog_trips:
    enter(FALLBACK)
  if good_for >= recovery_good_window:
    enter(NORMAL)

anti_oscillation:
  if flips_last_hour > N:
    lockout(lockout_after_flap)
```

---

## Plausibility checks (including spoofing/integrity) + forensic logging
Plausibility checks should be layered, cheap, and designed for **low false positives** (because false triggers create unnecessary fallback).

### Checks (examples)
- **Physics/rate bounds**: occupancy and volume cannot jump beyond feasible rates between samples.
- **Cross-sensor consistency**: phase state must be consistent with detector calls (persistent calls with no service can indicate stuck detectors).
- **Redundancy checks**: compare controller events vs. high-resolution logs to detect missing segments.
- **Low-demand window anomalies**: use off-peak windows to detect “impossible” actuation rates.

ATSPM guidance includes concrete “watchdog-style” checks that can be repurposed as triggers.

Examples from FHWA ATSPM Use Cases (Chapter 4):
- Flag a **possible communications issue** if an approach has “less than 500 records” per day in high-resolution data ([`FHWA ATSPM Use Cases — Chapter 4`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm:1)).
- Flag a **split failure pattern** if a movement has **>90% force-offs/max-outs between 1–5 AM** ([`FHWA ATSPM Use Cases — Chapter 4`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm:1)).
- Flag a **stuck pedestrian button** if there are **>200 pedestrian actuations between 1–5 AM** ([`FHWA ATSPM Use Cases — Chapter 4`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm:1)).

### How checks bound control actions
- Failed plausibility → classify feed as **UNTRUSTED**.
- UNTRUSTED feeds are removed from control decisions; system transitions to `DEGRADED` or `FALLBACK` depending on criticality.
- Critical actions are limited to plan selection or local fallback until the feed passes verification.

### Forensic logging (minimum fields)
- `intersection_id`, `corridor_id`
- `state_before`, `state_after`
- `reason_code`
- `event_time`, `ingest_time`, `age_event`, `age_ingest`
- `source_id`, `trust_tier`, `transport_path`
- `raw_values_snapshot` (bounded and sampled)
- `plausibility_rule_id` + evidence
- `operator_override` (who/when/why)

### Security/OT hardening tie-in
For control environments, CISA recommends practices such as maintaining and testing an incident response plan, using a risk-based defense-in-depth approach, and utilizing network segmentation/DMZs, hardening remote access, and avoiding persistent remote connections into control networks ([`CISA ICS Cybersecurity Practices`](https://www.cisa.gov/sites/default/files/publications/Cybersecurity_Best_Practices_for_Industrial_Control_Systems.pdf:1)).

---

## Detailed Implementation Plan

### Phase 1: Failure Catalog and Safety/Service Invariants (Weeks 1–3)
The agency should begin by identifying the data dependencies of its signal operations, including controller polling, detector streams, probe travel time feeds, and any central optimization service, because “delay tolerance” is mostly about knowing what can go wrong. The team should document failure modes such as comms outages, delayed data, partial detector loss, time drift, and implausible values, and it should define unacceptable outcomes, including pedestrian service violations, loss of preemption behavior, and sustained queue spillback. The team should then define invariants that must hold in all modes, such as pedestrian minimums/clearance, maximum cycle constraints where applicable, and a clear safe local plan that can run autonomously.

- **Roles**: traffic engineering (invariants), operations (failure history), IT/network (data paths), maintenance (controller constraints), safety/accessibility reviewer (ped service).
- **Deliverables**: failure catalog, invariants list, and an initial mode policy draft.
- **Risks**: missing a key dependency leads to unhandled failures; invariants may conflict with desired performance.
- **Acceptance checks**: all critical data feeds are inventoried and each has a documented failure response.

### Phase 2: Health Checks, Scoring, and Alerting (Weeks 4–10)
The engineering team should implement health checks for each input stream, including comms heartbeat latency, missing-data rate, detector stuck/chatter checks, and plausibility bounds on occupancy and flow, because delayed or corrupt data must be detected before it drives control decisions. The team should compute a unified health score per intersection and per corridor so that mode selection can be driven by consistent logic rather than ad-hoc alarms. The operations team should receive dashboards and alerting that indicate which component is unhealthy and what the recommended mode change is, because troubleshooting is a core goal of delay-tolerant design.

- **Roles**: data engineer (health metrics), IT/network (heartbeat monitoring), maintenance (detector QA), operations (alert routing), QA (threshold testing).
- **Deliverables**: health scoring service, dashboards, alert rules, and a detector/comms QA report.
- **Risks**: false alarms create unnecessary fallback; weak thresholds miss real failures.
- **Acceptance checks**: health scoring is stable on normal days and detects known failure cases in controlled tests.

### Phase 3: Graded Degradation Modes and Transition Logic (Weeks 11–18)
The software team should implement explicit modes (`NORMAL`, `DEGRADED`, `FALLBACK`, `ISOLATED`) and should define what control actions are permitted in each mode, because the safest response to missing data is often to reduce complexity rather than guess. The transition logic should include dwell times and hysteresis so the system does not oscillate during intermittent comms, and it should require multiple consecutive “good” windows before returning to `NORMAL`. The team should validate the transitions in a twin by injecting delayed data, dropped messages, and detector failures, and it should quantify the performance bounds of each mode so operators know what to expect.

- **Roles**: software engineer (mode controller), traffic engineer (mode behaviors and plans), modeler (failure injection tests), operations (override workflow).
- **Deliverables**: mode controller, transition policies, twin validation report, and operator-facing mode definitions.
- **Risks**: degraded modes may cause unacceptable delay; recovery may be too slow and reduce performance.
- **Acceptance checks**: transitions are explainable and rate-limited, and the system always has a safe fallback plan.

### Phase 4: Shadow Drills, Training, and Assisted Activation (Weeks 19–26)
The agency should run delay-tolerant logic in shadow mode first, where it detects failures and recommends mode changes without actuating, because this provides real-world validation without operational risk. Operators and maintenance staff should be trained on what each mode means, how to override it, and how to troubleshoot common causes such as time sync drift or detector faults. After shadow results are stable, the agency should enable assisted activation where operators confirm mode changes, because this step builds trust and reveals operational edge cases.

- **Roles**: operations (pilot users), maintenance (troubleshooting), engineering (on-call), analyst (evaluation).
- **Deliverables**: readiness report, updated thresholds, training materials, and a runbook for common failure patterns.
- **Risks**: operators may ignore recommendations if early alerts are noisy; shadow mode may not capture stress behavior.
- **Acceptance checks**: false-trigger rate is within target, and operators can reliably interpret and act on mode recommendations.

### Phase 5: Rollout, Audits, and Continuous Maintenance (Ongoing)
The agency should roll out delay-tolerant operation corridor-by-corridor, prioritizing areas where comms reliability is known to be challenging or where detector quality is variable. The program should conduct periodic audits of mode events, including why mode changes occurred and what performance impacts resulted, because auditing is how thresholds and plans remain defensible over time. The team should also maintain fallback plan libraries and update them after construction, timing changes, and controller upgrades so “safe mode” does not become stale.

- **Roles**: program owner (rollout planning), operations (monitoring), traffic engineering (plan maintenance), IT (network improvements), analyst (audits).
- **Deliverables**: rollout roadmap, quarterly mode-event audit reports, updated fallback plan library, and maintenance tickets.
- **Risks**: scaling increases alert volume; stale fallback plans reduce safety and performance.
- **Acceptance checks**: mode events decrease over time as infrastructure improves, and fallback plans remain current and validated.

### Plan maintenance cadence (governance tie-in)
The Signal Timing Manual recommends reviewing signal timing on a regular cycle (often cited as every 3–5 years depending on change rates), which applies to maintaining fallback plan libraries so they remain safe and representative ([`FHWA Signal Timing Manual — Chapter 7`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter7.htm:1)).

---

## Technical Mechanics

### Key Parameters
- Latency thresholds and missing data rate
- Health score boundaries
- Dwell times / hysteresis windows
- Recovery criteria

### Guardrails
- Preserve pedestrian service and clearance invariants.
- Rate-limit mode transitions.
- Require reason codes + evidence logs.

---

## MVP Deployment
- One corridor.
- 3 modes: `NORMAL`, `DEGRADED`, `FALLBACK`.
- Shadow mode first.

## Evaluation
- Time-to-detect data failures.
- False trigger rate.
- Performance bounds during degradation (delay, queues, split failures).

---

## Implementation Checklist
- [ ] Define staleness categories and per-feed latency budgets (table).
- [ ] Define time-sync requirements (max drift) and monitoring.
- [ ] Implement per-feed health checks (latency, missingness, drift) + unified health score.
- [ ] Implement plausibility + integrity checks and UNTRUSTED handling with evidence logs.
- [ ] Implement mode state machine with dwell timers, hysteresis, and lockout.
- [ ] Define safe `FALLBACK` timing plans (actuated/fixed) and verify ped/preemption invariants.
- [ ] Decide flash policy (when allowed, when prohibited) and encode decision tree.
- [ ] Integrate ATSPM/high-resolution logging; define watchdog rules.
- [ ] Run twin failure-injection drills and validate performance bounds.
- [ ] Deploy shadow mode; measure false triggers; tune thresholds.
- [ ] Roll out corridor-by-corridor with quarterly audits.

---

## Operations Runbook (SOP)

### Goal
Keep service predictable while protecting safety invariants.

### 1) Triage dashboard
- Confirm current state (`NORMAL/DEGRADED/FALLBACK/ISOLATED`) and the **reason code**.
- Identify which feed is `DELAYED/STALE/UNTRUSTED`.
- Check whether time sync is healthy.

### 2) Immediate actions
- If `FALLBACK/ISOLATED`: confirm the controller is executing the configured safe plan.
- If flash is active: confirm it matches approved flash policy and site conditions.
- If `INTEGRITY_FAIL` is present: limit remote access and escalate per security procedure.

### 3) Recovery workflow
- Do not force `NORMAL` immediately after comms restore; allow `RECOVERY_VERIFY`.
- If repeated flapping occurs: accept `lockout_after_flap` and open a maintenance/network ticket.

### 4) Escalation
- If ped service complaints or safety issues: treat as priority incident; disable risky transitions.
- If plausibility/spoofing alarms: notify cybersecurity/IT per incident procedures.

---

## Governance / Maintenance Runbook

### Ownership
- Traffic Engineering: defines invariants, fallback plans, and tuning targets.
- Signal Ops: monitors mode events, approves overrides, manages training.
- IT/Network: owns comms uptime, latency SLOs, time sync infrastructure.
- Maintenance: resolves detector/controller hardware faults.
- Security: owns incident response process for suspected integrity issues.

### Change control
- Version fallback plan library; require peer review + field validation before activation.
- Record changes to thresholds/state machine (who/why/test evidence).
- Test patches/configs offline where feasible (mirrors OT practice recommendations) ([`CISA ICS Cybersecurity Practices`](https://www.cisa.gov/sites/default/files/publications/Cybersecurity_Best_Practices_for_Industrial_Control_Systems.pdf:1)).

### Cadence
- Weekly: review top mode-event intersections, watchdog anomalies.
- Monthly: validate fallback plan library against construction/geometry changes.
- Quarterly: audit mode-event logs and false-trigger rates; update thresholds.
- Every 3–5 years (or sooner with change): timing review/retiming cycle ([`FHWA Signal Timing Manual — Chapter 7`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter7.htm:1)).

### KPIs
- Mode-event rate per intersection/corridor (and trend).
- Mean time to detect data failure.
- Mean time to recover to `NORMAL` without flapping.
- Thrash rate: mode flips/hour (lockout triggers).
- False trigger rate (shadow mode and production).
- ATSPM watchdog anomaly counts (comms, split failures, stuck ped) ([`FHWA ATSPM Use Cases — Chapter 4`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm:1)).

---

## Reference Links
- [`FHWA ATSPM (overview)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm)
- [`FHWA ATSPM Use Cases — Chapter 4`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm:1)
- [`FHWA Signal Timing Manual — Chapter 6`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter6.htm:1)
- [`FHWA Signal Timing Manual — Chapter 7`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter7.htm:1)
- [`CISA ICS Cybersecurity Practices`](https://www.cisa.gov/sites/default/files/publications/Cybersecurity_Best_Practices_for_Industrial_Control_Systems.pdf:1)

---

## Completion Checklist
- ✅ Clear differentiation from self-healing and interface boundaries: see **“What this is not…”**.
- ✅ Staleness semantics + latency budgets + authority rules: see **“Staleness Contract”** and **“Latency budgets by control type”**.
- ✅ Flash vs fixed constraints + decision tree: see **“Flash vs fixed safe operation”**.
- ✅ Recovery/anti-oscillation thresholds + **state machine diagram with staleness gates**: see **“Recovery + anti-oscillation…”**.
- ✅ Plausibility checks for wrong/spoofed data + low false-positive design + forensic logging: see **“Plausibility checks…”**.
- ✅ Governance: ownership, review cadence, KPIs: see **“Governance / Maintenance Runbook”**.
- ✅ Required visuals present: **per-feed staleness table** and **mode state machine diagram**.

---

Cross-links: Related ideas include self-healing intersections, explainable signals, and safety-first signals.
