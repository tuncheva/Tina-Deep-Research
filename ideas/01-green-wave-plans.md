# 01) Green-Light “Wave” Plans (Progression / Coordination)

## What it is (precise)
A **green wave** (signal progression) coordinates **cycle length, offsets, and splits** across a corridor so vehicle platoons can traverse multiple intersections with fewer stops at a target speed. Practically, this is the coordination problem: aligning *when* each intersection turns green with *when* platoons arrive.

Real systems use either:
- **pre-timed coordination** (time-of-day plans), or
- **adaptive coordination** (adjusts based on measured demand).

Toronto’s traffic-signal overview page explicitly describes “smart” adaptive systems (SCOOT/SCATS) as systems that determine timings using real-time detector information, which is one common path to sustaining coordination under variable demand. [City of Toronto](https://www.toronto.ca/services-payments/streets-parking-transportation/traffic-management/traffic-signals-street-signs/traffic-signals-in-toronto/types-of-traffic-signal-systems/)

## Why digital twins matter
A traffic **digital twin** supports green-wave design by:
- simulating **corridor platoon dynamics** (arrival dispersion, spillback, turning friction),
- testing alternate offsets/cycle lengths before deployment,
- forecasting whether coordination will *hold* over the next minutes under uncertainty.

Aimsun describes its real-time platform as a “traffic digital twin” that runs forecasts quickly (minutes) and can rank response plans by KPIs; it also references predictive signal plan scheduling and integration with systems such as SCATS/SCOOT via emulators. [Aimsun Live](https://www.aimsun.com/aimsun-live/)

## Easy explanation
Instead of every light making decisions alone, the corridor acts like a single system: lights are timed so cars moving at a reasonable speed keep getting green after green.

## What is needed (data & infrastructure)
| Need | Typical options | Notes |
|---|---|---|
| Traffic detection | loops, radar, video analytics, probe data | For progression you care about **flows + travel times** between intersections. |
| Signal controller connectivity | central system + comms (fiber/cellular) | Required to deploy offset/split updates reliably. |
| Timing plan management | time-of-day plan library or adaptive | Many cities run ~5–10 plans/day. Toronto notes practical use of multiple pre-defined plans in its central system description. [City of Toronto](https://www.toronto.ca/services-payments/streets-parking-transportation/traffic-management/traffic-signals-street-signs/traffic-signals-in-toronto/types-of-traffic-signal-systems/) |
| Digital twin model | corridor micro/meso sim + calibration | Must reflect turning rates, queues, spillback. |

## Implementation plan (phased)
### Phase 0 — baseline + observability (2–6 weeks)
- Define corridor KPIs: travel time, stops/vehicle, queue spillback risk, pedestrian delay.
- Ensure intersections publish reliable phase/timing states and detector health.

### Phase 1 — offline coordination design (4–10 weeks)
- Build an offline corridor model (micro/meso).
- Generate candidate time-of-day coordination plans (cycle/offset/split sets).
- Validate against historical days (weekday peak/off-peak, weekend).

### Phase 2 — “shadow mode” digital twin (4–8 weeks)
- Run the twin live, but **do not actuate** signals; produce recommended offsets.
- Compare forecast vs reality; quantify uncertainty.

### Phase 3 — controlled deployment (4–12 weeks)
- Start with a single corridor window (e.g., AM peak) and tighten guardrails:
  - max pedestrian wait,
  - max side-street queue,
  - min walk time, clearance.

### Phase 4 — adaptive refinement (ongoing)
- Add adaptive triggers (incidents, weather, events) to change speed targets, cycles, and offsets.

## Comparison table (implementation choices)
| Choice | Best when | Tradeoffs | Notes |
|---|---|---|---|
| Time-of-day coordinated plans | demand patterns are predictable | weaker response to incidents | common baseline in many cities; maintain 5–10 plans/day |
| Traffic-responsive plan selection | patterns shift but plan library is good | needs reliable detection | selects among plans; easier than full adaptive |
| Adaptive coordination (SCOOT/SCATS-style) | corridor demand varies strongly | more complexity + calibration | continuously updates splits/offsets |
| Twin-supported “shadow” recommendations | you want safe iteration | needs model + ops workflow | compare predicted vs observed before acting |

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Driver experience | fewer stops, smoother flow | may encourage speeding | set progression speed <= posted speed; enforcement; messaging |
| Corridor throughput | better platoon utilization | can increase delay on cross-streets | cap side-street delay; time-of-day balancing |
| Emissions/noise | fewer accel/decel events | benefits depend on demand and heavy-vehicle share | measure before/after with probe + counts |
| Equity | can improve bus reliability on corridors | “main road wins” perception | publish KPI impacts by neighborhood/mode |
| Operations | simple to explain and audit | sensitive to turning surges, blockages | detection health checks; fallback plans |

## Real-world anchors (what exists today)
- Cities deploy corridor coordination via central traffic control systems and pre-defined plans; Toronto’s description of centralized systems and adaptive systems (SCOOT/SCATS) is a concrete public-agency reference. [City of Toronto](https://www.toronto.ca/services-payments/streets-parking-transportation/traffic-management/traffic-signals-street-signs/traffic-signals-in-toronto/types-of-traffic-signal-systems/)
- FHWA’s (archived) Traffic Signal Timing Manual, Chapter 6 (Coordination), provides practical terminology and mechanics (cycle length, splits, offsets, time-space diagrams, transition modes) and concrete guidance like “signals within 0.5 miles should be coordinated unless operating on different cycle lengths” (citing MUTCD guidance). [FHWA Traffic Signal Timing Manual (archived), Ch. 6 Coordination](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter6.htm)
- The successor reference is the NCHRP Signal Timing Manual, Second Edition (NCHRP Report 812), which includes coordination planning and time-space diagram practice. [NCHRP Report 812 PDF](https://transops.s3.amazonaws.com/uploaded_files/Signal%20Timing%20Manual%20812.pdf)
- Digital-twin products explicitly market real-time predictive evaluation of response plans and “predictive signal plan scheduling.” [Aimsun Live](https://www.aimsun.com/aimsun-live/)

## Practical “twin + coordination” workflow (how to make this deployable)
1. **Start with a deterministic backbone**: pick cycle lengths (e.g., 80/90/100/110/120s) that fit local pedestrian constraints and intersection capacity; avoid “over-long cycles as a panacea” (FHWA notes capacity gains can be modest).
2. **Work in time-space diagrams first** (sanity check): validate progression speed assumptions; check for early-return-to-green artifacts and how they might break progression downstream.
3. **Use the twin for the hard parts**:
   - dispersion and mid-block friction,
   - turn-bay interactions and spillback,
   - transition behavior when switching plans.
4. **Deploy with guardrails**: max side-street delay, max ped wait, and explicit spillback constraints; treat the twin recommendation as a *suggestion* unless it passes constraints.
5. **Continuously retime** using measured travel times: treat offsets as parameters to be re-identified when travel times drift (weather, work zones, seasonal demand).

## MVP (smallest useful deployment)
- Pick **one corridor** (5–12 signals) with stable geometry and good detection.
- Ship **two time-of-day progression plans** (AM peak + PM peak) plus a conservative fallback.
- Add a simple **“progression health” monitor**: travel-time drift, stops/vehicle, and blocked-box events.
- Provide an operator **one-click revert** to the last known-good plan.

## Open questions
- What **target progression speed** reduces stops without encouraging speeding?
- How should the system balance **mainline progression** vs cross-street and pedestrian delay caps?
- What is the best way to detect when coordination is “broken” (probe travel time vs stop counts vs occupancy)?

## Evaluation checklist (practical)
- Travel time distribution (median + p95)
- Stops/vehicle and red-delay
- Side-street and pedestrian delay (max and p95)
- Queue spillback frequency (blocked-box events)

---

## Deep dive (practical): how coordination actually works in controllers
This section adds more “under the hood” detail so the digital-twin recommendations map cleanly to what traffic-signal controllers can implement.

### The three knobs: cycle, split, offset
- **Cycle length**: total time for the coordinated ring (e.g., 90s). Longer cycles can increase capacity but can also increase delay (especially for pedestrians and minor movements). A corridor typically uses a *small set* of cycle lengths by time-of-day.
- **Splits**: how the cycle is divided among phases (effective green). In coordinated operation, certain phases are held to satisfy progression while others “fit” around them.
- **Offset**: how each intersection’s coordinated phase is shifted in time relative to a system reference.

A digital twin’s job is to propose cycle/split/offset combinations that achieve objectives *and* satisfy constraints (ped minimums, clearance, storage limits, etc.).

### Why offsets have “reference points” (what you must define)
Offsets are not abstract; they are measured from a **reference point** in the cycle (often tied to the coordinated phase). If one team measures offset from “start of green” and another from “center of green” you can get correct numbers with incorrect field behavior.

Practical rule: write offsets in the plan library along with the explicit reference definition (e.g., “offset = time from system zero to start of NB through green”).

### Bandwidth vs. reliability
Classic green-wave design targets **bandwidth** (a time window where arrivals receive green) in each direction. In practice, reliability matters as much as theoretical bandwidth:
- mid-block friction (driveways, bus stops),
- turning surges,
- queue spillback.

Digital twins help estimate how often a chosen bandwidth is actually “usable” given dispersion and queues.

### Plan transitions (how coordination breaks in the real world)
When switching from one coordination plan to another (e.g., off-peak → peak), controllers must transition without creating unsafe or confusing indications.

Common transition ideas (implementation varies by controller/system):
- **Dwell**: hold current plan for a minimum time before changing.
- **Short-way / long-way** transitions: adjust cycle timing gradually to land on new offsets.
- **Inhibit transitions during heavy pedestrian demand** if it would violate walk/clearance constraints.

Twin implication: recommendations should be “transition-aware”:
- prefer small incremental offset changes,
- avoid frequent plan flips,
- model transition time as part of the cost (a plan that looks best but requires constant transitions may perform worse overall).

### Detection and progression: stop-bar vs. setback pitfalls
Progression quality often degrades when detection is unreliable or mis-placed:
- if presence detection at the stop-bar is missing/failed, minor movements may gap out incorrectly,
- if setback detection is too close/far, passage time settings may under/over-serve.

Twin implication: incorporate a **detector health score** and fall back to conservative splits when detection is degraded.

### Performance measures that diagnose progression (operator-friendly)
Use a small set of measures operators can understand and that align with engineering practice:
- **Arrivals on green** (AOG): what fraction of arrivals hit green.
- **Stops/vehicle**: a direct quality-of-service indicator for progression.
- **Travel time (median + p95)**: reliability matters.
- **Queue spillback / blocked-box events**: progression is irrelevant if storage is exceeded.

A good “progression health monitor” shows these trends before and after changes.

---

## Deep dive (design): time–space diagrams and the twin
Even if you use a simulation/twin, a time–space diagram is still the fastest sanity check.

1. Pick a **progression speed** (usually near the posted speed; don’t reward speeding).
2. Plot intersection greens as bands in time.
3. Overlay expected platoon trajectories.
4. Identify where the platoon “clips” red.

Twin enhancement: generate time–space diagrams automatically from the recommended plan and show them in the operator UI.

---

## Failure modes + guardrails (corridor-level)
- **Spillback**: downstream queue blocks upstream green. Guardrail: constraint on storage occupancy; detect blocked-box patterns.
- **Side-street starvation**: progression over-serves mainline. Guardrail: max delay / max red for minor movements.
- **Pedestrian harm**: long waits/unsafe crossings. Guardrail: min walk + clearance never violated; max ped wait cap.
- **Speeding incentive**: drivers chase the wave. Guardrail: set progression speed at/below limit; coordinate with enforcement and signage.
- **Plan thrashing**: frequent plan changes. Guardrail: minimum dwell time; hysteresis thresholds.

---

## Operator UX: what the “green-wave” screen should show
Cross-link: see [`ideas/08-operator-option-menu.md`](ideas/08-operator-option-menu.md)

Minimum operator widgets:
- Corridor map with current plan name and cycle length.
- “Progression health” (AOG, stops/veh, travel-time drift).
- Recommended action (if any) and why (e.g., “travel time drift +12% due to rain”).
- One-click revert to last known-good plan.

---

## Related topics
- [`ideas/16-fast-forward-twin-next-5-minutes.md`](ideas/16-fast-forward-twin-next-5-minutes.md) — short-horizon forecasting to keep coordination stable.
- [`ideas/03-real-time-what-if-button.md`](ideas/03-real-time-what-if-button.md) — test offsets/cycles before pushing changes.
- [`ideas/10-stop-line-intent-coordination.md`](ideas/10-stop-line-intent-coordination.md) — stop-line intent and how it affects phase service.
- [`ideas/11-delay-tolerant-smart-signals.md`](ideas/11-delay-tolerant-smart-signals.md) — safe degradation and fallbacks.

## Sources
- https://www.toronto.ca/services-payments/streets-parking-transportation/traffic-management/traffic-signals-street-signs/traffic-signals-in-toronto/types-of-traffic-signal-systems/
- https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter6.htm
- https://transops.s3.amazonaws.com/uploaded_files/Signal%20Timing%20Manual%20812.pdf
- https://www.aimsun.com/aimsun-live/
