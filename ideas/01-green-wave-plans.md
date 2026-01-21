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
- Digital-twin products explicitly market real-time predictive evaluation of response plans and “predictive signal plan scheduling.” [Aimsun Live](https://www.aimsun.com/aimsun-live/)

## Evaluation checklist (practical)
- Travel time distribution (median + p95)
- Stops/vehicle and red-delay
- Side-street and pedestrian delay (max and p95)
- Queue spillback frequency (blocked-box events)

## Sources
- https://www.toronto.ca/services-payments/streets-parking-transportation/traffic-management/traffic-signals-street-signs/traffic-signals-in-toronto/types-of-traffic-signal-systems/
- https://www.aimsun.com/aimsun-live/
