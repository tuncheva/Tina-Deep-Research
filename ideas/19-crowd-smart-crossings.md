# 19) Crowd-Smart Crossings: Dynamic Pedestrian Phasing

## Catchy explanation
When a crowd shows up, the intersection should behave like a good event coordinator: temporarily prioritize a **safer, clearer pedestrian crossing pattern** (e.g., LPI or an exclusive pedestrian phase), then return to normal operations when the surge is gone.

## What it is (precise)
**Crowd-smart crossings** dynamically adjust pedestrian phasing when large groups are present by switching among strategies such as:

- **Concurrent crossing** (baseline): pedestrians cross parallel to the vehicle movement.
- **Leading Pedestrian Interval (LPI)**: pedestrians get a head start before turning vehicles (often **3–7 seconds**). See FHWA’s LPI proven safety countermeasure fact sheet. ([FHWA LPI fact sheet](https://highways.dot.gov/sites/fhwa.dot.gov/files/2022-06/04_Leading%20Pedestrian%20Interval_508.pdf:1))
- **Exclusive pedestrian phase (“scramble”)**: all vehicle approaches stop while pedestrians cross (optionally including diagonal crossings). ([LADOT Exclusive Pedestrian Phase guide](https://ladot.lacity.gov/sites/default/files/2022-08/exclusive-pedestrian-phase-policy-design-guide-final-2017.pdf:1))

A crowd is estimated from sensors (video, radar, thermal, lidar, pushbutton analytics), and the system chooses a strategy **under strict safety, accessibility, privacy, and stability constraints**, using analysis/simulation (“twin”) to pre-test impacts on:

- pedestrian-vehicle conflicts,
- pedestrian delay and queue spillback onto sidewalks,
- vehicle delay, queues, and coordination impacts.

## Why it matters (benefits)
- **Reduced turning conflicts:** LPI and exclusive phases can reduce exposure to turning vehicles; FHWA reports crash reductions associated with LPI deployments. ([FHWA LPI fact sheet](https://highways.dot.gov/sites/fhwa.dot.gov/files/2022-06/04_Leading%20Pedestrian%20Interval_508.pdf:1))
- **Better surge handling:** handles bursts without chaos (repeated calls, pedestrians crossing late/against signal).
- **More legible right-of-way:** clearer “your turn” for pedestrians at event peaks.
- **Accessibility alignment:** supports slower and vulnerable pedestrians when timings and APS are configured correctly. ([PROWAG](https://www.access-board.gov/prowag/complete.html:1))
- **Explainability:** easier to justify and communicate during planned events.

## Key constraints (must be true)
This concept must always remain compliant with pedestrian signal meaning and timing rules:

- Pedestrian indications (WALK / flashing DON’T WALK / steady DON’T WALK) meanings and use must follow MUTCD guidance. ([MUTCD 2009 Part 4, Chapter 4E](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm:1))
- Pedestrian timing should be computed using established practice (walk interval + pedestrian change/clearance based on crossing distance and walking speed assumptions). ([FHWA Traffic Signal Timing Manual, Ch. 5](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm:1))
- APS (audible/vibrotactile) and accessible pushbutton/indication requirements must be met. ([PROWAG](https://www.access-board.gov/prowag/complete.html:1))
- Video detection (if used) must operate under a written policy addressing notice, retention, access, and governance. ([DHS VQiPS Policy Considerations](https://www.dhs.gov/sites/default/files/publications/Policy_Considerations_for_the_Use_of_Video_in_Public_Safety_Final_v5.pdf:1))

---

## When to use (site selection)

### Good candidate sites
Crowd-smart crossings are most valuable where surges create real risk and operational instability:

- Transit stations, stadium/arena districts, major venues, campuses.
- Downtown commercial areas with heavy pedestrian turning conflicts.
- Intersections with known **peak-hour bursts** (events, school dismissal, transit arrivals).

### Geometry-based scramble / exclusive pedestrian phase selection
Use an exclusive pedestrian phase when pedestrian-turning conflicts are high and pedestrians are numerous enough to justify the cycle-length tradeoff.

One practical screening approach (from LADOT policy guidance) is to consider an exclusive pedestrian phase if **all** of the following apply ([LADOT Exclusive Pedestrian Phase guide](https://ladot.lacity.gov/sites/default/files/2022-08/exclusive-pedestrian-phase-policy-design-guide-final-2017.pdf:1)):

- pedestrian volumes are high (e.g., on the order of **≥300 pedestrians/hour** in a single crosswalk during the peak hour, or pedestrians are a large fraction of vehicle volume),
- turning volumes are high across multiple legs (e.g., **≥200 turning vehicles/hour per crosswalk**),
- there is a pattern of pedestrian crashes involving turning vehicles (e.g., **≥3 crashes in 3 years**),
- operational constraints (queues, rail preemption, freeway ramp spillback) do not make an all-stop vehicular interval unsafe/unworkable.

### Geometry-based site selection checklist (scramble eligibility and pilot archetypes)

**Geometry-based checklist for scramble suitability** (adapted from typical agency practice and guidance on exclusive phases and accessibility ([LADOT guide](https://ladot.lacity.gov/sites/default/files/2022-08/exclusive-pedestrian-phase-policy-design-guide-final-2017.pdf:1), [PROWAG](https://www.access-board.gov/prowag/complete.html:1))):

- Crosswalks are clearly marked and continuous, with curb ramps aligned to crossings.
- Corner geometry does not encourage very high-speed turns (or turning speeds can be mitigated by design/controls).
- Sight distance for all approaches is adequate for drivers and pedestrians.
- APS equipment can be configured to clearly indicate WALK and clearance intervals for all crossing movements (including diagonal if provided).
- Pedestrian storage areas (corners, medians) are large enough to accommodate expected queues without forcing people into the roadway.
- No critical geometric constraints (e.g., skew angles, complex slip lanes) that would make scramble indications hard to understand without redesign.
- Space for any **turn restriction signing** and curb/approach treatments needed to support exclusive operation.

**Recommended pilot archetypes**

- **Transit hub archetype**: 4-leg intersection at major rail/bus hub with frequent burst arrivals.
- **Event district archetype**: near stadium or arena with predictable ingress/egress windows.
- **Downtown main street archetype**: high pedestrian activity with complex turning patterns.

---

## What it is not

- Not a “permission” to shorten pedestrian clearance to recover cycle length.
- Not a replacement for good geometric design (daylighting, tighter corner radii, protected turns).
- Not “adaptive” if it oscillates every cycle—stability is a safety feature.

---

## System architecture (implementation-ready)

### Components
1. **Sensing & perception**
   - Inputs: pedestrian presence/counts/queues, arrival rate, crosswalk occupancy, turning volumes, optional near-miss/conflict surrogate metrics.
   - Sensors: radar/thermal/lidar/video; pushbutton call logs as a weak signal.
   - Output: `crowd_level` + `confidence` per approach/crosswalk.

2. **Policy & control logic**
   - Mode selection: concurrent → LPI → exclusive ped phase.
   - Guardrails: minimum walk/clearance, max vehicle queues, min dwell time, change-rate limit.

3. **Signal controller interface**
   - Implements phase plans and timing parameter sets.
   - Provides status telemetry: current plan, calls, interval times, failures.

4. **Monitoring, audit, and tuning**
   - Dashboards: mode changes, queue lengths, ped delay distribution, max queue events.
   - Audit logs: who changed thresholds, when, and why.

5. **Governance**
   - Accessibility conformance checks.
   - Privacy program (policy, notice, retention, access control).

### Data contracts (example)

- `crowd_level`: {0=none, 1=light, 2=moderate, 3=surge}
- `confidence`: 0–1 (or categorical: low/med/high)
- `queue_estimate`: pedestrians waiting per corner/crossing
- `arrival_rate`: peds/min by approach
- `occupancy`: % time crosswalk is occupied
- `turning_flow`: vehicles/min by movement

---

## Detection requirements and fallbacks

Dynamic crowd-smart operation depends on **reliable, timely detection**. Drawing from general detection performance practices (e.g., traffic detection guidance in the FHWA Traffic Signal Timing Manual and agency video policy work that emphasizes calibration and QA ([FHWA Timing Manual](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm:1), [DHS VQiPS](https://www.dhs.gov/sites/default/files/publications/Policy_Considerations_for_the_Use_of_Video_in_Public_Safety_Final_v5.pdf:1))):

### What must be detected

Minimum functional requirements:

- **Presence**: is at least one pedestrian waiting at each crosswalk.
- **Queue length**: approximate number of pedestrians waiting (e.g., 0–5, 6–15, >15) or a density proxy.
- **Arrival rate**: pedestrians/min by approach to detect surges.
- **Crosswalk occupancy**: % of cycle crosswalk is occupied (helpful for scramble/LPI tuning).
- **Confidence / data quality**: sensor health, recent calibration, and data freshness.
- **Optional classification**: crowd vs. sparse conditions (and, if available, basic grouping like adult/child mix or mobility devices) **only at aggregate level**, never identity-level.

### Performance targets (to configure and monitor)

Agencies should define targets consistent with how they manage other detection systems, with explicit **precision/recall** and **latency** expectations:

- **Detection latency**: crowd classification available within **1–2 cycles** during surge windows.
- **Recall for surge detection** (fraction of true surge periods correctly identified): target e.g. ≥90% in defined evaluation windows.
- **Precision for surge detection** (fraction of detected surges that are real): target e.g. ≥80% to avoid oscillation and driver confusion.
- **Missed detection rate**: acceptable fraction of surge events that are missed (e.g., target <10% for defined surge levels; agency to set).
- **False surge rate**: acceptable rate of false "surge" flags; set both per-hour and per-event caps.
- **Uptime and data quality**: thresholds for when automation must fall back to fixed behavior (e.g., if confidence is low or stale for >N minutes).

### Uncertainty handling

- Every decision that uses sensor data should incorporate a **confidence score** (e.g., low/medium/high) and **data age**.
- Mode-switch logic should:
  - Require **high-confidence** observations (or corroborating sensors) to escalate to scramble.
  - Allow **medium-confidence** observations to trigger LPI or ped recall only.
  - Treat **low-confidence** or stale data as a reason to *not escalate* and/or revert to baseline.
- For analytics, track detection performance by lighting/weather conditions and time of day to detect systematic degradation.

### Outage and low-confidence fallbacks

When detection is unavailable or unreliable, the system should **fail safe and predictable**, similar to guidance for traditional detection failures (e.g., recall strategies discussed in FHWA timing guidance ([FHWA Timing Manual](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm:1))):

- Use **baseline concurrent timing** with conservative pedestrian timings, or
- Continue running a **pre-approved event plan** with fixed LPI/exclusive phase windows (if the event is scheduled), and
- Notify operators in the explainability UI (tie-in with explainable signals concept).

### Detection → method → failures → fallback table

| Signal / metric                | Primary method                         | Common failure modes                               | Fallback behavior |
|--------------------------------|----------------------------------------|----------------------------------------------------|-------------------|
| Presence at corner             | Video, radar, thermal, pushbutton      | Occlusion, poor lighting, snow, pushbutton stuck  | Ped recall every cycle in event window; conservative WALK/clearance |
| Queue length (bucketed)        | Video analytics, lidar, depth sensors  | Misclassification in dense crowds, glare          | Treat as high crowd if confidence low during known event; or revert to time-of-day event plan |
| Arrival rate (peds/min)        | Video or count-based edge processing   | Missing frames, dropped connection                 | Use rolling average from last reliable window; if stale, downgrade to presence-only logic |
| Crosswalk occupancy            | Video, radar over crosswalk            | Occluded view, heavy turning traffic              | Use queue + arrival thresholds only; do not further compress VEH phases based on occupancy |
| Turning flow (veh/min)         | Loop/radar + controller volumes        | Detector failure, misconfigured channels           | Apply fixed turning assumptions from last validated counts; cap surge logic to protect vehicle queues |
| Confidence / health            | Built from self-checks and QA results  | Silent failure if not implemented                  | Policy: if no confidence metric → treat as low confidence and disable automated surge switching |

---

## Core signal timing rules (grounded in guidance)

### Pedestrian intervals and computations
Use established timing practice:

- **WALK interval**: typically configured to give pedestrians time to start; many agencies use a minimum (often around 7 s in practice, with some contexts using shorter). ([FHWA Traffic Signal Timing Manual, Ch. 5](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm:1))
- **Pedestrian change interval (flashing DON’T WALK)**: computed from crossing distance and walking speed; MUTCD references a common walking speed assumption of **3.5 ft/s** and notes slower speeds may be appropriate at some locations. ([MUTCD 2009 Part 4, Chapter 4E](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm:1))
- **Buffer interval**: MUTCD discusses a buffer interval (commonly **≥3 s**) between the end of pedestrian clearance and the onset of conflicting vehicular green. ([MUTCD 2009 Part 4, Chapter 4E](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm:1))

### LPI parameters

- LPI provides pedestrians a head start before turning vehicles (FHWA describes typical values and safety intent). ([FHWA LPI fact sheet](https://highways.dot.gov/sites/fhwa.dot.gov/files/2022-06/04_Leading%20Pedestrian%20Interval_508.pdf:1))
- MUTCD includes guidance on using an LPI (including minimum durations). ([MUTCD 2009 Part 4, Chapter 4E](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm:1))

### APS and accessible timing

- APS provides audible/vibrotactile indications and must be configured so information remains correct under LPI and exclusive phasing.
- PROWAG provides requirements for APS features and pedestrian signal phase timing based on walking speed assumptions and minimum walk intervals. ([PROWAG](https://www.access-board.gov/prowag/complete.html:1))

---

## Modes and state diagram (required visual)

We assume three primary **operational modes**:

- **BASELINE**: concurrent ped phases, possibly with static LPI where already approved.
- **LPI_SURGE**: dynamic LPI emphasis with ped recall during surge windows.
- **SCRAMBLE_SURGE**: exclusive pedestrian phase (with or without diagonal) for intense surges at suitable sites.

### State diagram: baseline / LPI / scramble with switch conditions, dwell, hysteresis (required visual)

```text
             [ BASELINE ]  (concurrent, no dynamic surge logic)
                  |
      (crowd_level ≥ 2 AND
       detection confidence high
       for ≥ N cycles; turning
       conflicts present)
                  |  ENTER_LPI_SURGE
                  v
            [ LPI_SURGE ]
   (LPI active; ped recall during
        surge window)
                  |
      (crowd_level ≥ 3 AND
       site passes scramble
       geometry & safety
       checklist; queue caps
       at boundaries respected;
       persistence ≥ M cycles)
                  |  ENTER_SCRAMBLE_SURGE
                  v
        [ SCRAMBLE_SURGE ]
  (exclusive ped phase during
      configured surge window)
                  |
   (surge dissipates: crowd_level
    ≤ 1 for T1 minutes AND
    vehicle queues at or below
    caps; dwell in SCRAMBLE
    completed; no active event
    egress override)
                  |  EXIT_SCRAMBLE → LPI_SURGE
                  v
            [ LPI_SURGE ]
                  |
   (crowd_level ≤ 1 for T2
    minutes AND no active
    event window AND stability
    timer complete)
                  |  EXIT_LPI_SURGE → BASELINE
                  v
             [ BASELINE ]

Guardrails:
- Minimum dwell per mode (e.g., ≥K cycles) before allowing transition.
- Hysteresis: higher thresholds to enter than to stay; lower thresholds to exit.
- Max switches/hour (e.g., no more than X mode changes per hour).
- If detection confidence drops mid-state → finish current cycle safely, then
  follow fallback arrow to BASELINE or pre-approved EVENT plan.
```

---

## Mode catalog (required visual: table)

### Mode → prerequisites → benefits → risks → constraints → fallback

| Mode                | Prerequisites | Benefits | Risks | Key constraints | Fallback when not met / failure detected |
|---------------------|--------------|----------|-------|-----------------|-----------------------------------------|
| **BASELINE** (concurrent, no special surge logic beyond standard timing) | Standard geometry; compliant ped timings; no active event window; crowd_level mostly 0–1 | Simple, predictable operation; best for off-peak; preserves progression | May under-serve pedestrians during surges; higher exposure to turning conflicts | Must meet MUTCD/PROWAG timing; APS configured correctly; vehicle queues within planned limits | If baseline cannot safely handle persistent surges, promote site to LPI_SURGE or SCRAMBLE_SURGE per screening; if detection fails, remain in baseline with conservative ped timings |
| **LPI_SURGE** (dynamic LPI + ped recall) | Turning conflicts present; detection capable of identifying surges with sufficient precision/recall; APS support for LPI; geometry acceptable for concurrent crossings | Reduces turning conflicts; improves ped start-up; often manageable impact on progression | If mis-tuned, can lengthen cycle, cause vehicle queues, or confuse users if on/off frequently | Minimum LPI duration per guidance; dwell times and hysteresis; vehicle queue caps; APS and signing clearly indicate behavior | If sensor confidence low or queue caps exceeded, fall back to BASELINE or fixed LPI schedule; if complaints/confusion high, revert and revise thresholds/signing |
| **SCRAMBLE_SURGE** (exclusive ped phase) | Site passes geometry/accessibility checklist; high ped + high turning volumes; documented crash pattern; event or recurrent surge window; clear signing and APS configuration | Maximum reduction in turning conflicts; very clear ped priority during bursts | Increased vehicle delay and potential spillback; progression degradation; risk if signing or APS unclear | Strict ped timing and APS configuration; turn restrictions (e.g., NO TURN ON RED) where needed; caps on frequency/duration; coordination with adjacent network | If boundary queues exceed caps or coordination is severely degraded, step down to LPI_SURGE; if detection fails, use scheduled scramble windows only or revert to LPI/baseline; if accessibility issue found, disable scramble until corrected |
| **EVENT_PLAN** (pre-approved event-specific timing with fixed ped priority windows) | Event schedule loaded; pre-validated plan (with safety/accessibility review); operators trained; coordination with TMC | Predictable behavior for recurring events; easier outreach; avoids ad-hoc tuning | May be misaligned with actual attendance; if used outside true events, may cause unnecessary delay | Activated only within defined event windows; clear governance on when/who can activate; APS and signing consistent | If event cancelled/low attendance, revert to BASELINE or LPI_SURGE after short dwell; if incident conflicts with plan, operators may temporarily disable event timing and use manual control |

---

## Dynamic strategy selection (decision tree)

### Decision goal
Select the *least disruptive* strategy that achieves safety and surge handling while preserving compliance.

### Decision tree (operational)

1. **Is sensing confidence low?**
   - Yes → use **safe default plan** (usually BASELINE concurrent + conservative ped timings; optionally fixed LPI if historically beneficial) and alert operations.
   - No → continue.

2. **Are turning conflicts likely high?** (turning volumes high, multiple legs, poor yielding, crash history)
   - Yes → prefer **LPI_SURGE** as first escalation.

3. **Is there a surge?** (queue and/or arrival rate beyond threshold)
   - No → BASELINE (optionally static LPI if always-on PSC).
   - Yes → continue.

4. **Will LPI alone handle the surge without exceeding vehicle queue caps or destroying progression?**
   - Yes → **LPI_SURGE** mode.
   - No → continue.

5. **Does the site meet exclusive phase suitability?**
   - Use LADOT-style screening (high ped, high turning across multiple legs, crash pattern, no disqualifying constraints). ([LADOT Exclusive Pedestrian Phase guide](https://ladot.lacity.gov/sites/default/files/2022-08/exclusive-pedestrian-phase-policy-design-guide-final-2017.pdf:1))
   - If yes → **SCRAMBLE_SURGE** during surge window (or use EVENT_PLAN with pre-approved scramble intervals).
   - If no → **conservative LPI + ped recall** (serve pedestrians every cycle during surge) and consider geometric/turning restrictions.

6. **Is an event window active?** (see coordination section)
   - If active and crowd_level high near gates, event egress safety overrides otherwise tight progression; use SCRAMBLE_SURGE or EVENT_PLAN if safe/eligible.

---

## Control logic: guardrails and anti-oscillation

### Guardrails (non-negotiable)

- Maintain pedestrian minimums and clearance calculations. ([MUTCD 2009 Part 4, Chapter 4E](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm:1))
- Use established ped timing computation guidance. ([FHWA Traffic Signal Timing Manual, Ch. 5](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm:1))
- In scramble mode, stop conflicting vehicular movements and enforce turning restrictions (e.g., “NO TURN ON RED” where required by local policy). ([LADOT guide](https://ladot.lacity.gov/sites/default/files/2022-08/exclusive-pedestrian-phase-policy-design-guide-final-2017.pdf:1))
- Do **not** reduce ped clearance times to compensate for added LPI or scramble unless geometry changes and accessibility reviews explicitly justify different assumptions.

### Stability features (mode-switch guardrails)

- **Hysteresis:** different thresholds to enter vs. exit surge mode.
- **Minimum dwell time:** keep a chosen strategy for at least N cycles.
- **Rate limit:** max 1 mode change per M minutes.
- **Degradation handling:** if confidence drops mid-surge, finish current cycle safely and fall back per state diagram.
- **Consistency and signage:** ensure that any change in basic operation (e.g., adding scramble) is accompanied by appropriate signing and, if needed, public communications, consistent with MUTCD expectations for clarity and legibility of indications. ([MUTCD Part 1 & 4](https://mutcd.fhwa.dot.gov/htm/2009/part1/part1.htm:1))

---

## Compliance, human factors, and operator explainability

### Compliance and human factors considerations

- **Legibility:**
  - Maintain consistent signal meanings and APS messages across modes.
  - Avoid frequent visible changes in phasing pattern outside of known event windows.

- **Mode-switch guardrails:**
  - Configure **minimum dwell times** (e.g., at least several cycles) before allowing another mode change.
  - Limit number of **mode switches per hour** to avoid confusing users.

- **Signage and communication:**
  - Provide clear static or dynamic signing for exclusive phases (e.g., "All-Way Walk" or equivalent, plus turn restrictions) consistent with MUTCD.[^mutcd1]
  - For recurring event sites, use outreach and consistent messages so users learn the pattern.

- **Compliance monitoring & complaint plan:**
  - Track proxies such as late crossings, violations during DON’T WALK, driver yielding observations, and complaints.
  - Maintain a log of **complaint themes** (e.g., confusing indications, excessive wait, accessibility concerns) and relate them to sites/modes.
  - Periodically conduct **field observation studies** (e.g., once per quarter per pilot) to compare observed compliance with expectations.
  - Use findings to adjust thresholds, signage, and communications.

[^mutcd1]: MUTCD emphasizes clear and uniform meaning for traffic control devices to support road user comprehension and compliance. ([MUTCD 2009 Intro & Part 1](https://mutcd.fhwa.dot.gov/htm/2009/part1/part1.htm:1))

### Reason codes for crossing behavior and mode changes

Integrate with the explainable-signals taxonomy:

- Define reason codes for:
  - surge-driven LPI activation,
  - exclusive phase activation,
  - fallback to baseline due to sensor issues,
  - manual overrides during events.
- Log:
  - **what changed** (mode, timing),
  - **why** (surge, safety pattern, equipment issue),
  - **evidence** (crowd level, confidence, complaints, crash trends).

---

## Accessibility and equity

### APS consistency and vulnerable users

- Ensure APS indications (tones, speech messages, vibrotactile features) stay **synchronized** with WALK and clearance indications for all modes, consistent with PROWAG and related accessibility requirements. ([PROWAG](https://www.access-board.gov/prowag/complete.html:1))
- Verify that APS messaging does not become ambiguous when scramble or LPI is active (e.g., clear messaging for diagonal crossings if provided).

### Bias and equity risks

Potential equity risks to manage:

- Sensors might **under-count** certain groups (e.g., children in crowds, people using mobility devices, groups clustered close to buildings) more than others.
- Event plans may prioritize crossings serving venues over nearby residential streets.

Mitigation practices (aligned with equity considerations in accessibility and planning guidance ([PROWAG](https://www.access-board.gov/prowag/complete.html:1))):

- Evaluate crossing performance **by leg and by time of day**, not just corridor averages.
- Track metrics like:
  - **maximum pedestrian wait time** by crosswalk and period,
  - **clearance compliance** (fraction of users completing crossing during WALK + clearance),
  - **disparities** between legs that primarily serve different communities or land uses.
- Include **field validation** with diverse user groups before and after deployment.

### Field validation walkthrough + stakeholder sign-off

For each pilot site:

1. Conduct a **multi-party field walkthrough** (engineers, operators, accessibility staff, advocacy groups) during non-event and event conditions.
2. Verify:
   - APS correctness and audibility,
   - crossing times for slower pedestrians,
   - crowd queuing on corners (no spillback into street),
   - clarity of indications and signage.
3. Document findings and open issues.
4. Require written **sign-off** (or documented conditions) from:
   - traffic engineering,
   - accessibility/ADA coordinator,
   - where applicable, community liaison.

---

## Privacy-by-design for crowd-smart sensing

Following public-sector video and sensor policy recommendations, focus on **purpose limitation, data minimization, and governance** ([DHS VQiPS](https://www.dhs.gov/sites/default/files/publications/Policy_Considerations_for_the_Use_of_Video_in_Public_Safety_Final_v5.pdf:1)):

### Edge processing and output minimization

- Prefer architectures where cameras or sensors produce **counts/queues/occupancy** rather than raw video streams.
- Only store derived metrics needed for operations and evaluation.
- Keep any necessary raw imagery **on the edge device** where possible, with tightly controlled access.

### Retention defaults

- Short retention for raw imagery (if recorded at all), consistent with agency video policy.
- Longer retention for aggregate metrics and decision logs, aligned with general records schedules for traffic operations.

### "No-go" analytics list and exception approvals

- Explicitly prohibit:
  - identity recognition or tracking of individuals,
  - linking crossing data to personal identifiers,
  - non-transportation uses without separate approval.
- If exceptions are requested (e.g., temporary higher-resolution recording for a safety study), require:
  - documented purpose,
  - time-bounded approval,
  - defined retention and deletion plan.

### Audits and transparency template

- Periodically review sensor deployments for compliance with policy.
- Publish a **plain-language summary** that describes:
  - where crowd-smart crossings are deployed,
  - what is measured (counts, queues, not identities),
  - retention practices,
  - how the public can raise concerns.

These elements align with the policy topics identified in DHS’s public safety video guidance (notice, retention, access, analytics, governance). ([DHS VQiPS](https://www.dhs.gov/sites/default/files/publications/Policy_Considerations_for_the_Use_of_Video_in_Public_Safety_Final_v5.pdf:1))

---

## Metrics and success criteria

### Safety and compliance

- Pedestrian-vehicle conflict surrogates (TTC/PET) and yielding rates.
- Compliance audits (timing, indications, APS correctness).
- Changes in ped crash patterns over multi-year windows.

### Mobility and reliability

- Ped delay distribution during events (p50/p90).
- Vehicle queue length and spillback occurrences.
- Coordination impact (travel-time reliability on corridor).

### Equity and accessibility

- Maximum and average pedestrian wait, by leg and time-of-day.
- Clearance compliance rates, including for crossings with high proportions of children/older adults or mobility-device users.
- Stakeholder-reported issues and complaints, categorized and tracked over time.

### Program sustainability

- Sensor uptime and data quality (by weather/light condition).
- Number of operator overrides and reasons.
- Frequency of low-confidence or fallback operation.

---

## Coordination and event-mode integration

Dynamic pedestrian modes affect cycle length, offsets, and progression. Agencies already consider such tradeoffs when introducing exclusive pedestrian phases or LPI treatments; similar coordination principles apply here ([FHWA Timing Manual](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm:1), [LADOT guide](https://ladot.lacity.gov/sites/default/files/2022-08/exclusive-pedestrian-phase-policy-design-guide-final-2017.pdf:1)).

### Corridor coordination impacts

- **Cycle length**: exclusive phases and long LPI may lengthen the effective cycle or compress green for vehicles.
- **Offsets**: frequent unscheduled exclusive phases can disturb progression; therefore:
  - restrict dynamic scramble to **pre-defined event windows** or
  - design coordination plans that explicitly include scramble phases.

### Recovery rules

- After a surge ends, use **hysteresis** and a short **stabilization period** before rejoining tight progression.
- Limit re-entry into coordination plans to defined boundaries (e.g., nearest cycle start) to avoid mid-cycle disruptions.

### Event vs. crowd safety hierarchy

When **event egress** and **network progression** conflict:

- Safety of pedestrians crossing at the intersection takes precedence over corridor progression, consistent with safety-first planning priorities in federal guidance.
- However, to prevent downstream safety issues (e.g., freeway ramp spillback), the agency may define **caps** on exclusive-mode duration or frequency and coordinate with upstream/downstream signals and traffic management centers.

### Operational decision tree (event conflict)

1. **Is an event window active?**
   - No → only use dynamic LPI/small adjustments that preserve corridor plans.
   - Yes → continue.

2. **Is corner/ped queue above surge threshold?**
   - No → maintain coordination plan (concurrent or scheduled LPI).
   - Yes → evaluate scramble/LPI escalation.

3. **Will using scramble violate critical queue caps on adjacent links (e.g., ramps)?**
   - No → allow scramble within configured frequency/duration limits.
   - Yes →
     - limit scramble duration,
     - prioritize LPI plus ped recall,
     - consider temporary vehicular restrictions (e.g., turn bans) as engineering changes.

4. **During resolution**
   - Gradually taper back to non-event plans; avoid hard switches that break coordination abruptly.

---

## Detailed implementation plan (onboarding playbook)

### Phase 1: Site selection, safety context, and baseline (Weeks 1–4)
Select 1–3 intersections with known pedestrian surges (transit/venue/CBD). Collect baseline:

- turning volumes and yielding behavior,
- ped volumes by crosswalk (including surge windows),
- existing ped timings and compliance with guidance,
- any crash history.

Deliverables:

- Pilot site list + suitability screening for LPI vs. exclusive phase.
- Baseline report with target metrics.

### Phase 2: Detection upgrade and crowd metric pipeline (Weeks 5–12)

Deploy/configure detection capable of estimating queue and arrival rate.

Deliverables:

- Crowd metric feed + confidence scoring.
- QA dashboards and alerting.
- Documented fallback behavior.

Privacy note: if using video analytics, build policy and governance up front (notice, retention, access, analytics scope). ([DHS VQiPS Policy Considerations](https://www.dhs.gov/sites/default/files/publications/Policy_Considerations_for_the_Use_of_Video_in_Public_Safety_Final_v5.pdf:1))

### Phase 3: Strategy selection logic, guardrails, and twin validation (Weeks 13–20)

Define switching thresholds for BASELINE/LPI_SURGE/SCRAMBLE_SURGE; validate with analysis/simulation.

Deliverables:

- Switching logic specification.
- Safety and mobility impact report.

### Phase 4: Pilot deployment with assisted mode and tuning (Weeks 21–32)

Start in **assisted mode** (operator enable/disable during known surge periods). Monitor and tune:

- ped delay and corner crowding,
- vehicle queue caps and spillback,
- stability (oscillation incidents).

Deliverables:

- Pilot report + tuned parameters.
- Operator playbook for event days.

### Phase 5: Integration and expansion (ongoing)

Integrate with event-aware timing and policy-driven modes. Expand only after reliability and governance are proven.

Deliverables:

- Expansion roadmap.
- Periodic audits (safety, accessibility, privacy).

---

## Implementation Checklist

(Engineering + policy checklist for onboarding.)

- [ ] Confirm controller supports LPI and (if needed) exclusive pedestrian phase sequencing.
- [ ] Verify existing ped timings comply with guidance (walk, clearance, buffer). ([MUTCD 2009 Part 4, Chapter 4E](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm:1))
- [ ] Compute/validate ped clearance using distance and walking speed assumption; document exceptions for slower populations. ([FHWA Timing Manual, Ch. 5](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm:1))
- [ ] Validate APS requirements and configurations for all modes (BASELINE/LPI_SURGE/SCRAMBLE_SURGE). ([PROWAG](https://www.access-board.gov/prowag/complete.html:1))
- [ ] Define mode thresholds + hysteresis + dwell times; test on recorded data.
- [ ] Define vehicle queue caps and spillback monitors (including event egress boundaries).
- [ ] Implement safe fallback for low-confidence sensing and document behavior.
- [ ] Implement audit logging for all parameter changes and mode switches, including reason codes.
- [ ] Document detection performance targets (precision/recall/latency) and QA processes.
- [ ] Run tabletop scenario review (sensor outage, false surge, special event, emergency preemption).
- [ ] Obtain stakeholder sign-off on accessibility and privacy plan for each pilot site.

---

## Operations SOP (Runbook)

### Normal operations

- Monitor: sensor health, crowd confidence, mode changes per intersection.
- Review weekly: top “mode-change” locations; oscillation incidents; sites operating frequently in fallback mode.

### Event operations

- **Pre-event:**
  - Load event schedule window.
  - Confirm staffing for monitoring.
  - Confirm signing/communications if exclusive phase changes expected.

- **During event:**
  - Watch queue caps and pedestrian corner crowding.
  - If detection confidence degrades, switch to safe default plan.
  - Monitor for abnormal vehicle queues on critical links (ramps, key arterials).

- **Post-event:**
  - Export a run summary: mode time-in-state, p90 ped delay, max vehicle queue, number of overrides.
  - Capture any operator observations or complaints.

### Incident response (SOP snippets)

- **If oscillation occurs:**
  - Temporarily raise thresholds and/or increase dwell time.
  - Verify confidence scoring; check for glare/occlusion.
  - If persistent, disable dynamic scramble at that site and log reason.

- **If queue spillback risk occurs:**
  - Disable exclusive phase and revert to LPI_SURGE/BASELINE.
  - Consider restricting turns or adding protected-only turns (engineering change).

- **If APS or accessibility issue is reported:**
  - Treat as high-priority safety item.
  - Verify configuration and timing at site; if necessary, revert to simpler operation until fixed.

---

## Accessibility & Privacy Governance Runbook

### Accessibility governance

- Maintain an APS configuration inventory and test plan for each intersection.
- Validate that APS messages/indications remain correct under LPI and exclusive phases (PROWAG requirements for APS features and timing). ([PROWAG](https://www.access-board.gov/prowag/complete.html:1))
- Include accessibility staff and disability advocates in pilot site selection and post-implementation reviews.
- Document a **disparity check** at least annually:
  - compare ped delay and clearance compliance by leg and by time-of-day,
  - highlight legs serving equity-priority communities,
  - record mitigations or follow-up actions.

### Privacy minimization for sensing

- Follow the privacy-by-design practices in **"Privacy-by-design for crowd-smart sensing"** above.
- Maintain a **register of sensor types and locations**, including:
  - purpose,
  - data captured,
  - retention/purging rules,
  - contact point for questions.
- For each pilot, maintain a short **privacy impact note** summarizing:
  - what is measured,
  - what is explicitly not measured (no identities),
  - retention defaults,
  - governance and oversight.

---

## Reference Links

- FHWA Proven Safety Countermeasures: Leading Pedestrian Interval (LPI) fact sheet: https://highways.dot.gov/sites/fhwa.dot.gov/files/2022-06/04_Leading%20Pedestrian%20Interval_508.pdf
- MUTCD 2009 Edition, Part 4, Chapter 4E: Pedestrian Control Features: https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm
- MUTCD 2009 Edition, Part 1: General: https://mutcd.fhwa.dot.gov/htm/2009/part1/part1.htm
- FHWA Traffic Signal Timing Manual (FHWA-HOP-08-024), Chapter 5 (Pedestrian timing): https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm
- U.S. Access Board PROWAG (complete text): https://www.access-board.gov/prowag/complete.html
- LADOT Exclusive Pedestrian Phase Policy & Design Guide (2017): https://ladot.lacity.gov/sites/default/files/2022-08/exclusive-pedestrian-phase-policy-design-guide-final-2017.pdf
- DHS VQiPS Report (2016): Policy Considerations for the Use of Video in Public Safety: https://www.dhs.gov/sites/default/files/publications/Policy_Considerations_for_the_Use_of_Video_in_Public_Safety_Final_v5.pdf

---

## Completion checklist (✅/⚠️)

- ✅ Detection requirements, performance targets (including precision/recall/latency), uncertainty handling, and fallbacks: see **"Detection requirements and fallbacks"**.
- ✅ Compliance + human factors (mode guardrails, signage, reason codes, complaint/compliance measurement plan): see **"Control logic: guardrails and anti-oscillation"** and **"Compliance, human factors, and operator explainability"**.
- ✅ Accessibility + equity validation (APS consistency, bias risks, walkthrough/testing protocol, stakeholder sign-off, disparity checks): see **"Accessibility and equity"** and **"Accessibility & Privacy Governance Runbook"**.
- ✅ Privacy-by-design (edge counting/output minimization, retention defaults, "no-go" list + exception approvals, audit and transparency template): see **"Privacy-by-design for crowd-smart sensing"** and **"Accessibility & Privacy Governance Runbook"**.
- ✅ Geometry-based site selection, scramble-eligible criteria, site screening checklist, pilot archetypes: see **"When to use (site selection)"**.
- ✅ Coordination + event-mode integration (corridor cycle/offset impacts, recovery rules, conflict-resolution hierarchy with event egress): see **"Coordination and event-mode integration"**.
- ✅ Required visuals: state diagram for BASELINE/LPI/SCRAMBLE_SURGE and mode catalog table (Mode → prerequisites → benefits → risks → constraints → fallback): see **"Modes and state diagram"** and **"Mode catalog"**.
- ⚠️ Local agency-specific numeric thresholds (exact queue caps, dwell times, surge definitions) are intentionally left for local calibration: see **"Detection requirements and fallbacks"** and **"Control logic: guardrails and anti-oscillation"**.

---

Cross-links: related ideas include [`ideas/17-safety-first-signals.md`](ideas/17-safety-first-signals.md:1) and [`ideas/18-event-aware-timing.md`](ideas/18-event-aware-timing.md:1).
