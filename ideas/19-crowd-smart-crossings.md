# 19) Crowd-Smart Crossings (Dynamic Pedestrian Phasing)

## What it is (precise)
“Crowd-smart crossings” means adjusting **pedestrian signal phasing** in real time when large groups show up (e.g., after events, at school dismissal, at transit hubs). The control can switch between common pedestrian strategies such as:

- **leading pedestrian intervals (LPI)**,
- **lagging pedestrian intervals**, and
- **exclusive pedestrian phases** (“pedestrian scramble” / “Barnes’ Dance”).

The FHWA Traffic Signal Timing Manual describes these three options, including the tradeoff that an **exclusive pedestrian phase** reduces conflicts but can increase cycle length and delay. [FHWA Signal Timing Manual (Archived), Ch. 4.5](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter4.htm#4.5)

FHWA’s Proven Safety Countermeasures note that an **LPI gives pedestrians 3–7 seconds** head start to increase visibility and reduce conflicts with turning vehicles. [FHWA LPI PSC](https://highways.dot.gov/safety/proven-safety-countermeasures/leading-pedestrian-interval)

## Why digital twins matter
A digital twin helps because the *right* pedestrian strategy depends on the interaction of:

- turn volumes (right/left turns),
- crosswalk occupancy and platoons of pedestrians,
- queue spillback risk (ped-only phase can consume green time), and
- safety objectives (conflict reduction).

The twin can:

- forecast “next 5–15 minutes” crowd arrival from transit/event signals,
- simulate LPI vs exclusive ped phase impacts on vehicle queues, and
- recommend a safe switch plan with constraints (max vehicle queue, max pedestrian wait, minimum clearance).

## Easy explanation
When a big group of people arrives, the signal temporarily changes how it serves pedestrians—either giving them a short head start, or running a quick “all-walk” phase—then returns to normal when the crowd clears.

## What is needed (data & infrastructure)
| Need | Typical options | Notes |
|---|---|---|
| Pedestrian presence + volume estimate | pushbuttons, video analytics, radar/LiDAR, thermal | For “crowd-smart”, you need more than a single call: you need **queue length / flow**.
| Pedestrian timing parameters | controller ped timing, configurable LPI/scramble | Ensure walk + clearance remain MUTCD-compliant.
| Vehicle turning context | stopline detection, turning counts, probe | Needed to judge conflicts and delay impacts.
| Policy constraints | max ped wait, accessibility, school rules | Encode guardrails as hard limits.
| Digital twin + decision logic | micro/meso sim + rule/policy layer | Run in shadow mode first.

## Implementation plan (phased)
### Phase 0 — pick candidate intersections (2–4 weeks)
- Select 2–5 crossings with recurring surges (stadium, station, CBD).
- Baseline: ped delay, turning conflicts, near-miss signals (if available), vehicle queues.

### Phase 1 — add observability (4–8 weeks)
- Add/validate pedestrian detection quality.
- Implement “crowd state” metrics (ped queue length, arrival rate, occupancy).

### Phase 2 — deploy a safe strategy set (4–8 weeks)
- Implement a small menu:
  - baseline (concurrent ped),
  - LPI mode,
  - exclusive pedestrian phase (scramble) mode.
- Add safety/ops constraints:
  - min ped clearance,
  - max side-street queue,
  - maximum cycle length.

### Phase 3 — twin-guided switching (6–12 weeks)
- Run the twin in shadow mode to recommend when to switch strategies.
- Deploy automatic switching only after stability validation.

### Phase 4 — continuous tuning (ongoing)
- Tune thresholds per location and time-of-day (schools vs event nights).
- Publish simple, auditable rules: “if ped queue > X for Y seconds, enable LPI/scramble”.

## Comparison table (ped strategies)
| Strategy | Conflict reduction | Capacity impact | Best when |
|---|---|---|---|
| Concurrent ped | baseline | low | normal demand |
| LPI | medium | low–medium | high turning conflicts |
| Lagging ped | medium | low–medium | site-specific turn patterns |
| Scramble (exclusive ped) | high | higher delay/cycle | very high ped surges |

## Upsides vs downsides
| Aspect | Upside | Downside / risk | Mitigations |
|---|---|---|---|
| Pedestrian safety | LPIs and scrambles reduce turn conflicts | wrong timing can increase exposure or confusion | clear signage, consistent operation, conservative thresholds |
| Pedestrian throughput | handles surges without repeated pushbutton cycles | can increase vehicle delay | cap cycle length; return-to-normal hysteresis |
| Accessibility | can prioritize slower walkers | if miscalibrated, could increase wait | enforce max wait; ensure APS and clearance rules |
| Operations | simple concept to communicate | sensor errors could trigger oscillations | sensor health checks; minimum dwell time per mode |

## Real-world anchors (what exists today)
- FHWA documents LPI/lagging/exclusive pedestrian phases and their tradeoffs, including the “pedestrian scramble” concept. [FHWA Signal Timing Manual (Archived), Ch. 4.5](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter4.htm#4.5)
- FHWA’s Proven Safety Countermeasures describe LPI timing (3–7 seconds lead) and benefits (visibility, fewer conflicts). [FHWA LPI PSC](https://highways.dot.gov/safety/proven-safety-countermeasures/leading-pedestrian-interval)

## MVP (smallest useful deployment)
- Pilot at **one transit hub** intersection with frequent surges.
- Add a simple crowd classifier (low/med/high) from ped queue length + arrival rate.
- Enable only two modes initially: baseline and **LPI mode** (scramble later).
- Enforce max ped wait, min clearance, and a minimum dwell time to avoid oscillation.

## Open questions
- What detection method is reliable in all lighting/weather: thermal, radar, video, or button analytics?
- When is scramble justified vs LPI (turn volumes, conflict history, crowd size)?
- How should crowd-smart interact with event mode (idea 18) and safety-first tuning (idea 17)?

## Evaluation checklist (practical)
- Pedestrian delay (median + p95) and maximum wait
- Turning-vehicle conflicts (proxy: yielding rate, near-miss/vision analytics if available)
- Vehicle queue spillback frequency
- Mode-switch frequency (avoid oscillation)

## Sources
- https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter4.htm#4.5
- https://highways.dot.gov/safety/proven-safety-countermeasures/leading-pedestrian-interval
