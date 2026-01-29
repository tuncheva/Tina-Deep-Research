# 17) Safety-First Signals: Optimize Near-Misses, Not Just Delay

## Catchy Explanation
Instead of waiting for crashes to prove a problem, safety-first signals treat **near-misses** as warnings—and optimize timing to reduce them.

## What it is (precise)
**Safety-first signal operations** treat safety as a first-class objective/constraint by using **surrogate safety measures (SSMs)** such as **TTC (time-to-collision)** and **PET (post-encroachment time)** and other conflict/avoidance proxies derived from:
- **simulation trajectories** (digital-twin rollouts),
- **roadside sensing** (video/radar trajectories where available), and/or
- **connected/probe telemetry** (e.g., hard braking / speed profiles).

Candidate timing alternatives are evaluated (offline or near-real-time) and only deployed if they satisfy **hard safety + policy constraints** (e.g., pedestrian timing, clearance intervals, safe transitions), while demonstrating improved safety surrogate distributions with acceptable mobility/transit impacts.

This aligns with the Safe System principle that **safety is proactive** and that redundancy is crucial: use multiple layers (timing, geometry, speed management, enforcement, post-crash care) rather than expecting one control knob to eliminate risk. ([`USDOT Safe System Approach` ](https://www.transportation.gov/safe-system-approach))

## Benefits
- **Proactive safety**: reduces risk before crashes accumulate.
- **Measurable tradeoffs**: safety impacts can be reported like performance impacts.
- **Targeted interventions**: focuses on high-risk movements and times.
- **Better justification**: supports safety-driven retiming decisions.

## Challenges
- **Validation**: SSMs must be locally validated / calibrated to avoid “false confidence.”
- **Data intensity**: needs trajectories or reliable proxies.
- **Political sensitivity**: safety improvements may increase delay.
- **Bias risk**: incomplete detection can undercount vulnerable users.

---

## Operating model (safety-first control loop)

1) **Detect safety risk regime** (baseline vs rain/night/school dismissal/event egress) using environment + ops signals.
2) **Generate bounded candidates** (timing templates) that are controller-feasible.
3) **Score candidates** using **SSM distributions + mobility + multimodal** metrics.
4) **Apply constraints** (hard rules) and governance approvals.
5) **Deploy with stability guardrails** (dwell time, hysteresis, rollback triggers).
6) **Monitor outcomes** continuously (SSMs, mobility, multimodal equity slices) and revalidate periodically.

---

## 1) How to compute SSMs operationally (data sources + fidelity requirements)

SSMs require **trajectory-like information** at sufficient resolution and with correct signal/geometry context.

### 1.1 SSMs from simulation trajectories (digital twin)
Microsimulation can provide detailed per-vehicle state over short time steps (position, speed, acceleration, lane) and model actuated signal operations; accurate modeling of signal operations is a requirement if you want to derive surrogate safety measures from simulation outputs. ([`FHWA-RD-03-050 (microsimulation overview)`](https://www.fhwa.dot.gov/publications/research/safety/03050/03.cfm))

Implementation pattern:
- Run a calibrated microsim (or fast surrogate) with controller logic consistent with field settings.
- Export per-vehicle trajectories (x/y or lane+distance, speed, acceleration) at small time steps.
- Compute TTC/PET/conflicts from relative motion and conflict-zone geometry.

Fidelity notes:
- The FHWA report emphasizes that evaluating surrogate measures relies on **frequent state updates** (sub-second time scales for some proximity measures) and that the simulation must allow that fidelity. ([`FHWA-RD-03-050 (time step fidelity)`](https://www.fhwa.dot.gov/publications/research/safety/03050/03.cfm))
- Simulation models often assume “safe” driver behavior; surrogate derivation must acknowledge that crashes are not explicitly modeled. ([`FHWA-RD-03-050 (limitations)`](https://www.fhwa.dot.gov/publications/research/safety/03050/03.cfm))

### 1.2 SSMs from roadside sensing (video / radar / lidar)
Roadside sensing can produce trajectories or conflict proxies without running a twin.

Operational options:
- **Trajectory extraction**: multi-object tracking → per-object trajectory → conflict metrics.
- **Proxy scoring** when full trajectories aren’t robust:
  - hard-braking events in approach lanes,
  - red-entry detections (stop bar crossings during red),
  - turning yield compliance proxies.

Limitations (must be documented):
- Occlusion, glare, precipitation, and nighttime conditions can degrade tracking.
- Vulnerable road users (VRUs) are disproportionately affected by detection failures → equity risk.

### 1.3 SSMs from connected/probe telemetry (hard braking, speed profiles)
Probe/CV telemetry can support surrogate safety programs even where roadside sensing is limited.

Operational proxies:
- Hard braking and high deceleration events near intersections.
- Speed distributions (e.g., percent of vehicles exceeding context speed in approach).

Privacy guardrails:
- Use aggregation (space/time bins) and avoid raw trajectories where possible.
- Maintain data minimization and access controls (who can see what, for what purpose).

### 1.4 Minimum viable fidelity requirements (practical)
Regardless of source, you need:
- **Time synchronization**: sensors and controller clocks must be aligned (or corrected) so that conflict time ordering is meaningful.
- **Movement classification**: each trajectory/proxy must be mapped to movement (through/left/right, ped crossing leg).
- **Conflict-zone geometry**: consistent definitions of where paths overlap.
- **Sampling rate / update period**: sufficient to resolve near-miss dynamics (higher for TTC-like measures).

From microsimulation specifically, the FHWA review highlights that many surrogate measures require detailed vehicle–vehicle interaction information that is not always exposed to end users; APIs/output configurability matter materially for implementability. ([`FHWA-RD-03-050 (data extraction)`](https://www.fhwa.dot.gov/publications/research/safety/03050/03.cfm))

### 1.5 Required visual: SSM → definition → data sources → fidelity → failure modes → suitable decisions

This table is the **required visual** mapping each SSM/proxy to its definition, data sources, fidelity needs, failure modes, and what kinds of decisions it is suitable to drive.

```markdown
| SSM / proxy | Operational definition (short) | Data source options | Required fidelity (minimum viable) | Known failure modes | Suitable decisions |
|---|---|---|---|---|---|
| TTC (time-to-collision) | Time until collision if current relative motion continues | Micro-sim trajectories; roadside trajectories | High-frequency state updates (≤0.2–0.5 s); accurate x/y or lane+distance and speed/accel; synchronized with signal state | Unstable with noisy tracking; sensitive to time step; mis-specified conflict zones; blind to VRUs if not tracked | Comparing candidate phasing/sequence changes in simulation or offline analysis; ranking safety of major retiming options; NOT for real-time per-cycle actuation without very high-fidelity sensing |
| PET (post-encroachment time) | Time between one user leaving conflict area and another entering | Roadside trajectories; micro-sim | Correct conflict-zone geometry; accurate timestamps (≤0.1–0.2 s); robust object ID over conflict zone | Poor geometry mapping; missed detections (esp. VRUs); trajectory ID swaps | Evaluating effects of protected-only vs permissive turns; assessing LPIs or turn restrictions; site-level before/after studies rather than per-cycle controls |
| Conflict count (by type/severity) | Count of interactions below TTC/PET thresholds, binned by conflict type | Micro-sim; roadside trajectories | Consistent thresholds; stable classification rules; adequate sample size per period | Threshold drift; site-to-site comparability issues; undercounting where data sparse | Program-level screening and prioritization; tracking trends after major policy changes; not precise enough for fine-grained optimization alone |
| Hard braking events | Count/rate of deceleration events above threshold near stop bar or in conflict approach | Probe/CV telemetry; roadside trajectories | Consistent decel definition; location accuracy to attribute to intersection or movement; enough fleet penetration | Device/vehicle bias; weather/grade effects; fleet composition bias; privacy limitations | Network-wide speed/approach aggressiveness monitoring; flagging candidate safety problem sites; cross-checking other SSM trends; not a standalone justification for complex phasing changes |
| Red-entry / RLR proxy | Stop-bar crossing during red interval (RLR proxy) | Controller state + detection; video analytics | Accurate phase state timestamps; precise stop-bar detection; time sync between detection and controller | Stop-bar detector failures; time-sync errors; ambiguity around clearance vs entry | Identifying locations needing red-interval enforcement, all-red extensions, or protected phasing; monitoring RLR trends after timing/geometry changes |
| Speed distribution in approaches | Percentiles of approach speed vs context speed (by time-of-day) | Probe/CV; roadside speed sensors | Stable geofencing; enough samples per bucket; calibrated speed sensors | Sample bias; work-zone/event bias; probe penetration variation | Selecting progression speeds; justifying nighttime speed management or traffic calming; screening for contexts where higher-speed permissive turns are unacceptable |
```

---

## 2) Local validation: SSM ↔ crash correlation and transferability program

SSMs are useful only if you can defend that they track meaningful safety outcomes in your context.

The NCHRP guidance on using surrogate/alternative measures stresses that establishing quantitative linkages between surrogates and crashes has been challenging, and that practitioners still use surrogates even without established quantitative linkages—so interpretation and study design matter. ([`NCHRP Web-Only Document 369 (surrogate measures background)`](https://www.tesc.psu.edu/assets/docs/Estimating-Effectiveness-of-Safety-Treatments-in-the-Absence-of-Crash-Data.pdf))

### 2.1 Validation goals
- **Correlation**: do high-SSM-risk locations/time periods align with crash hot spots and conflict complaints?
- **Sensitivity**: do SSM distributions move in expected direction after known safety treatments?
- **Transferability**: can one set of thresholds/weights work across similar intersection archetypes?

### 2.2 Program design (implementable)

**A) Segment the city into archetypes**
- Intersection form: 4-leg vs skewed, channelized rights, protected-only vs protected/permitted lefts.
- Mode context: near schools, high ped volumes, bike facilities, transit priority corridors.
- Time regimes: peak vs off-peak; night vs day; rain/snow vs dry.

**B) Build a validation dataset**
- Crash history (with severity) + near-miss complaints.
- SSM time series (TTC/PET/conflicts, hard braking, RLR proxies) by intersection/time bucket.
- Exposure measures (volumes by mode, turning volumes, pedestrian crossings).

**C) Study design outline**
- Baseline window: 6–24 months (as available) of SSM + crash/complaint history.
- Intervention window: matched duration post-change.
- Controls:
  - control intersections with similar context but no treatment,
  - time-of-day and seasonal controls.
- Regression-to-the-mean handling (practical):
  - avoid selecting only the worst recent crash spikes; include multi-year context,
  - use controls and time-series comparisons rather than only before/after at treated sites.

**D) Acceptance criteria for adopting SSMs as decision targets**
Minimum thresholds your program should meet before using SSMs to drive automated or high-frequency decisions:
- Repeatable hotspot alignment: top-N SSM risk sites overlap meaningfully with known safety concern sites.
- Measurement stability: SSM estimates are stable under normal sensor health conditions.
- Bias assessment: documented VRU detection bias and mitigation plan.

### 2.3 Revalidation cadence and ownership
- **Monthly (ops):** sensor health + sanity checks; review outlier days.
- **Quarterly (signals + safety):** site archetype scorecards; adjust thresholds cautiously.
- **Annually (Vision Zero lead):** formal revalidation report; decide whether SSM targets/constraints change.

### 2.4 Required visual: safety-first lifecycle diagram

This is the **required diagram**: the lifecycle of a safety-first timing package from baseline to iteration.

```text
   [ BASELINE OPERATIONS ]
           |
           | 1) Build SSM + exposure baseline
           v
   [ MEASURE & ANALYZE ]
   - Compute SSMs (TTC, PET, proxies)
   - Identify high-risk archetypes
           |
           | 2) Design candidate safety timings
           v
   [ DESIGN & SIMULATE ]
   - Use twin / offline SSM analysis
   - Check controller feasibility
           |
           | 3) VALIDATE SSMs LOCALLY
           |   - Correlate with crashes by
           |     archetype / TOD / weather
           v
   [ VALIDATED SSM MODEL ]
           |
           | 4) SHADOW MODE
           |   - Score candidates but do not
           |     actuate
           v
   [ SHADOW DEPLOYMENT ]
           |
           | 5) CONTROLLED FIELD DEPLOY
           |   - Limited sites
           |   - Safety Impact Statement
           v
   [ CONTROLLED DEPLOYMENT ]
           |
           | 6) MONITOR & EVALUATE
           |   - SSM deltas
           |   - equity slices
           |   - rollback triggers
           v
   [ MONITOR & UPDATE ]
           |
           | 7) UPDATE / SCALE / RETIRE
           |   - Update templates
           |   - Expand to new sites
           |   - Retire underperformers
           v
   [ PROGRAM PORTFOLIO ]
           ^
           | (feeds back into)
           |
   [ MEASURE & ANALYZE ]
```

---

## 3) Equity and multimodal safety operationalization (beyond vehicle conflicts)

Safety-first signals must cover **pedestrians, bicyclists, transit riders**, not only vehicle–vehicle conflicts.

### 3.1 Multimodal SSMs and exposure metrics
Examples (pick a small defensible set):
- **Ped turning-conflict proxies**: low PET between turning vehicles and pedestrians in crosswalk.
- **Yield compliance proxies**: percent of turns yielding when ped present (requires detection).
- **Speed-at-crossing**: approach speed distribution during ped phases.
- **Exposure-weighted risk**: (crossing volume × turning volume × speed proxy) to avoid “low counts = safe.”

LPI as a deployable timing safety lever:
- FHWA describes LPI as giving pedestrians a **3–7 second** head start before vehicles receive a green, improving visibility and reducing conflicts. ([`FHWA Leading Pedestrian Interval (PSC)`](https://highways.dot.gov/safety/proven-safety-countermeasures/leading-pedestrian-interval))

### 3.2 Equity framing
Use Safe System principles (shared responsibility, proactive safety, humans are vulnerable) to justify explicit prioritization of severe injury risk reduction over minor delay improvements. ([`USDOT Safe System Approach` ](https://www.transportation.gov/safe-system-approach))

Operationalize equity:
- Report SSM deltas by:
  - neighborhood equity bucket,
  - user type (ped, bike, vehicle),
  - time-of-day (including nighttime).
- Require that safety gains are not concentrated only in already well-served corridors.

### 3.3 Reporting template (before/after by geography, mode, severity proxy)

**Safety Equity Report (per quarter / per program release):**
- Version + dates + corridors covered.
- For each neighborhood bucket:
  - exposure (peds/day, bikes/day, vehicles/day),
  - top 5 intersections by severe-risk proxy,
  - before/after deltas:
    - TTC/PET severe tail metrics (e.g., share below threshold),
    - turning–ped conflict proxies,
    - hard braking rate,
    - red-entry proxy rate,
  - narrative: “what changed and why.”
- VRU detection completeness stats (missingness by hour/weather).

---

## 4) Real-time “adaptive safety constraints” (triggers, dependencies, guardrails)

Safety-first signals are most useful when they can switch into **pre-approved safety modes** when conditions increase risk.

### 4.1 Triggers (examples)
- **Weather/visibility:** rain/snow, low visibility, nighttime.
- **School dismissal windows:** scheduled and geofenced.
- **Event egress:** stadium/concert egress; pedestrian surges.
- **Incident/crash indicators:** sudden speed drops, blocked lanes, emergency response presence.

### 4.2 Guardrails against “false safe” behavior
If the data is missing or unhealthy, do not pretend safety improved.

Guardrails:
- If ped detection fails (or ped tracking missingness exceeds threshold) → switch to conservative mode and treat VRU risk as **high uncertainty**.
- If comms loss / time sync loss → freeze to last known safe plan; require operator confirmation before re-entering adaptive mode.
- If conflict metric spikes unexpectedly after a timing change → rollback.

### 4.3 Example safety mode templates (deployable)

```markdown
| Mode | When used | Timing actions (bounded templates) | Key constraints | Exit rules |
|---|---|---|---|---|
| RAIN_SAFE | Rain / low visibility | Enable protected-only lefts where feasible; reduce progression speed; extend clearance conservatively | Keep ped walk+clearance; don’t exceed max cycle | Dwell ≥ N minutes; exit after rain clears for M minutes |
| SCHOOL_PEAK | School dismissal windows | Enable LPI (3–7 s) on legs with heavy ped; restrict permissive turns | Ensure accessible ped timing and max wait policy | Exit at scheduled end + cooldown ([`FHWA Leading Pedestrian Interval (PSC)`](https://highways.dot.gov/safety/proven-safety-countermeasures/leading-pedestrian-interval)) |
| NIGHT_CALM | Nighttime | Reduce green that encourages high approach speeds; favor smoother progression | Keep minimum service for side streets | Exit at TOD boundary + dwell |
| EVENT_EGRESS | Event release | Dedicate longer ped service; restrict turn conflicts | Never violate ped clearance | Exit when ped surge drops below threshold |
```

### 4.4 Avoiding overreaction / oscillation
Use the same stability tools as other adaptive systems:
- Trigger **persistence** (require condition for N consecutive windows).
- **Hysteresis** (separate enter/exit thresholds).
- **Minimum dwell time** (stay in a safety mode long enough to matter).

---

## 5) Tradeoff accounting + decision ownership (safety vs mobility vs transit)

### 5.1 Hard constraints vs soft objectives
**Hard constraints (must never be violated):**
- Ped walk + clearance intervals and accessible pedestrian signal requirements.
- Safe plan transitions (no unsafe ring/phase transitions).
- Explicit max ped wait caps (agency policy).

**Soft objectives (trade explicitly):**
- Vehicle delay, stops, travel time reliability.
- Transit reliability (headways, OTP).
- Progression quality.

### 5.2 Multi-objective accounting (how to make it operational)
Create a scorecard per candidate:
- Safety: severe tail of TTC/PET, conflict count severity bins, hard braking rate, red-entry proxy.
- Mobility: person-delay (not just vehicle delay), max queue, spillback risk.
- Transit: bus delay/headway disruption.

Use a “do-nothing baseline” candidate and only accept a change if safety improves meaningfully or if required by a safety mode.

### 5.3 Governance (who sets tradeoff policy)
Safe System emphasizes shared responsibility and proactive safety; translate that into named owners for policy. ([`USDOT Safe System Approach` ](https://www.transportation.gov/safe-system-approach))

Recommended governance roles:
- **Vision Zero / safety program lead:** sets severity priorities and acceptable risk thresholds.
- **Signals engineer:** defines feasible timing templates and controller constraints.
- **Transit agency:** defines transit reliability constraints.
- **Emergency services:** defines emergency response constraints.
- **Data/privacy lead:** approves data use and retention.

### 5.4 Safety Impact Statement template (required for any deployment)

Use this for each deployed timing package (pilot or network release):

```text
Safety Impact Statement

1) Change summary
- Corridor / intersections:
- Timing actions (templates):
- Dates / times active:
- Trigger mode (if adaptive):

2) Expected safety effect (surrogates)
- Primary SSM targets (e.g., TTC tail, PET tail):
- Expected direction + magnitude (from evaluation):
- Data sources used (sim/video/probe) + known limitations:

3) Expected mobility + multimodal impacts
- Person-delay delta:
- Transit reliability delta:
- Ped delay / max wait:

4) Equity / distributional impacts
- Neighborhood buckets impacted:
- VRU exposure changes:

5) Approvals
- Safety lead:
- Signals engineer:
- Transit:
- Emergency services (if applicable):
- Date:

6) Rollback triggers
- SSM spike thresholds:
- Detector health / missingness thresholds:
- Public complaint / field observation triggers:

7) Monitoring plan
- Metrics + cadence:
- Owner:
```

---

## 6) Boundaries of signal timing: enforcement, geometry, and complementary actions

Timing is powerful but limited:
- Timing can reduce conflicts by changing phase ordering, providing head starts (e.g., LPI), reducing permissive conflicts, and smoothing speed profiles.
- Timing cannot fix unsafe geometry (poor sight distance, high-speed channelized turns) by itself, and it cannot guarantee compliance.

Safe System explicitly frames safety as requiring **redundancy** and multiple layers of protection—not a single operational tactic. ([`USDOT Safe System Approach` ](https://www.transportation.gov/safe-system-approach))

### 6.1 Complementary measures (coordinate with safety/geometry programs)
- **Geometry:** daylighting, tighter radii, protected intersections, leading turn restrictions.
- **Speed management:** speed limit setting, traffic calming, progression speed tuning.
- **Signage/markings:** improved crosswalk markings, turn restriction signs.
- **Enforcement coordination:** targeted enforcement during known risk windows.
- **Education/outreach:** especially near schools and event venues.

### 6.2 What data to share across programs
- High-risk movement/time windows (SSM hotspot slices).
- Exposure metrics (VRUs and turning volumes).
- Before/after safety surrogate deltas after geometry/enforcement changes.

---

## Implementation Checklist
- [ ] Pick 3–6 SSMs/proxies you can compute reliably (include at least one VRU-focused measure).
- [ ] Decide data source strategy (sim trajectories vs roadside sensing vs probe/CV) and document limitations.
- [ ] Build movement mapping and conflict-zone geometry for every intersection in scope.
- [ ] Implement SSM computation with data-quality gates and reproducible configs.
- [ ] Define safety modes + bounded timing templates (e.g., LPI, protected-only lefts) with clear triggers.
- [ ] Implement stability guardrails (persistence, hysteresis, dwell, cooldown).
- [ ] Establish local validation program (archetypes, controls, acceptance criteria).
- [ ] Establish equity reporting slices (neighborhood, user type, time-of-day).
- [ ] Define governance owners and approval workflow; require Safety Impact Statements.
- [ ] Pilot at 1–3 sites; monitor; expand with periodic revalidation.

---

## Safety Evaluation & Monitoring Runbook

### A) Daily / weekly ops checks
1. Verify sensor health and time synchronization; if missingness high → disable adaptive safety mode and run conservative plan.
2. Review red-entry / RLR proxy time series (if available) and hard braking hotspots.
3. Review top intersections by severe SSM tail (e.g., TTC/PET tail).

### B) Per-deployment monitoring (first 2–4 weeks)
1. Compare before vs after:
   - TTC/PET tail share below threshold,
   - turning–ped conflict proxy rate,
   - hard braking rate,
   - red-entry proxy rate.
2. Verify multimodal constraints:
   - ped delay / max wait policy,
   - transit reliability impacts.
3. Check stability:
   - mode flip-flops (enter/exit too often).

### C) Rollback procedure
1. Triggered by:
   - SSM spike beyond threshold,
   - ped detection failure or severe missingness,
   - field observation of unsafe behavior.
2. Actions:
   - revert to last known safe timing package,
   - freeze in conservative mode until review,
   - file incident summary and open investigation ticket.

### D) Quarterly safety review
- Produce Safety Equity Report (section 3.3).
- Re-evaluate thresholds for archetypes.
- Decide whether to expand, pause, or redesign templates.

---

## Governance / Safety Impact Statement Template

Use the Safety Impact Statement template in section 5.4 for every deployed timing package (pilot or production). Treat it as a required governance artifact with signatures and rollback criteria.

---

## Reference Links
- [`FHWA-RD-03-050 Surrogate Safety Measures From Traffic Simulation Models (chapter page)`](https://www.fhwa.dot.gov/publications/research/safety/03050/03.cfm)
- [`FHWA-RD-03-050 PDF` ](https://ntlrepository.blob.core.windows.net/lib/38000/38000/38015/FHWA-RD-03-050.pdf)
- [`FHWA Leading Pedestrian Interval (Proven Safety Countermeasure)`](https://highways.dot.gov/safety/proven-safety-countermeasures/leading-pedestrian-interval)
- [`USDOT Safe System Approach` ](https://www.transportation.gov/safe-system-approach)
- [`FHWA Automated Traffic Signal Performance Measures overview` ](https://ops.fhwa.dot.gov/arterial_mgmt/performance_measures.htm)
- [`NCHRP Web-Only Document 369: Estimating Effectiveness of Safety Treatments in the Absence of Crash Data (PDF)`](https://www.tesc.psu.edu/assets/docs/Estimating-Effectiveness-of-Safety-Treatments-in-the-Absence-of-Crash-Data.pdf)

---

## Completion Checklist
- ✅ **(1) How to compute SSMs operationally (data sources + fidelity requirements)** with **required SSM table incl. suitable decisions**: see **“1) How to compute SSMs operationally”** and **1.5**.
- ✅ **(2) Local validation: SSM ↔ crash correlation + transferability program + lifecycle diagram**: see **“2) Local validation…”** and **2.4 required visual**.
- ✅ **(3) Equity + multimodal safety operationalization + reporting template**: see **“3) Equity and multimodal…”** incl. **3.3**.
- ✅ **(4) Real-time adaptive safety constraints (triggers, guardrails, templates, anti-oscillation)**: see **“4) Real-time ‘adaptive safety constraints’”**.
- ✅ **(5) Tradeoff accounting + decision ownership + Safety Impact Statement template**: see **“5) Tradeoff accounting…”** incl. **5.4**.
- ✅ **(6) Boundaries of signal timing + complementary measures + coordination**: see **“6) Boundaries of signal timing…”**.
- ✅ Added final sections as required: **Implementation Checklist**, **Safety Evaluation & Monitoring Runbook**, **Governance / Safety Impact Statement Template**, **Reference Links**, **Completion Checklist**.

---

Cross-links: Related ideas include city rules built into signals, explainable signals, event-aware timing, and fast-forward twin.
