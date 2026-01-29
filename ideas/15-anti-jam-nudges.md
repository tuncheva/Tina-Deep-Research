# 15) Anti-Jam Nudges: Proactive Stability Control

## Catchy Explanation
Instead of waiting for gridlock and then reacting, you apply small, safe timing “nudges” early—like gentle steering corrections—so queues don’t cascade into a jam.

## What it is (precise)
**Anti-jam nudges** are bounded, short-horizon signal adjustments triggered by predicted spillback risk. Using detectors/probe data (and often a fast digital twin rollout), the system forecasts near-term queue growth and identifies links at risk of blocking-box or network cascade. It then applies conservative interventions such as:
- **Metering inflow** to saturated links,
- **Flushing downstream storage** to create space,
- **Holding coordination** temporarily to prevent destabilizing changes,
- **Recovering** back to normal timing in a controlled way,

while keeping changes within a safe envelope (e.g., ±3–8s split shifts) and preserving pedestrian service/clearance invariants.

### The key idea: shift objectives as conditions shift
Under oversaturation, classic optimization goals like minimizing delay or maximizing progression can fail because queues are unstable and can grow without bound. Practice guidance emphasizes switching objectives toward **maximizing throughput** where possible and then **queue management**—controlling where queues form so they do the least system damage—when a cure is not possible ([`FHWA “Signal Timing Under Saturated Conditions” (Guidance)`](https://ops.fhwa.dot.gov/publications/fhwahop09008/guidance.htm); [`FHWA “Signal Timing Under Saturated Conditions” (Ch. 2)`](https://ops.fhwa.dot.gov/publications/fhwahop09008/chapter2.htm)).

## Benefits
- **Prevents cascades**: reduces blocked-box and network gridlock events.
- **High leverage**: small changes can avoid big delays.
- **Protects critical routes**: stabilizes key corridors during incidents.
- **Operational clarity**: nudges are limited and reversible.

## Challenges
- **Prediction accuracy**: false positives waste capacity and annoy users.
- **Shifting harm**: metering one approach can create queues elsewhere.
- **Coordination coupling**: interventions can disrupt progression.
- **Explainability**: hard to communicate “why we slowed you down now.”

## Implementation Strategies

### Infrastructure Needs
- **Queue/spillback sensing**: advance detectors, occupancy, probe speeds.
- **Risk model**: heuristics or twin-based short rollouts.
- **Control levers**: split tweaks, plan switches, metering templates.
- **Guardrails**: ped mins, max cycle, rate limits.

### Detailed Implementation Plan
#### Phase 1: Define Jam Signatures and Protected Assets (Weeks 1–4)
The agency should begin by identifying the corridors and intersections where spillback creates network-wide harm, such as downtown grids, bridge approaches, and freeway ramp terminals, because anti-jam control is most valuable where cascades occur. The team should define “jam signatures” using observable measures, such as sustained high occupancy, red-occupancy, blocked-box events, and rapid queue growth at known bottlenecks, and it should determine which links must be protected (for example, keep the hospital route from being blocked). The team should define acceptable tradeoffs, including maximum added delay on metered approaches and minimum pedestrian service, because metering is a deliberate shift of delay.

- **Roles**: traffic engineering (definitions and bounds), operations (field knowledge), data analyst (signature metrics), safety/accessibility reviewer (ped constraints).
- **Deliverables**: jam signature specification, protected-link list, baseline blocked-box and spillback report, and initial guardrails.
- **Risks**: signatures may be too broad and trigger constantly; protected-link priorities may be politically sensitive.
- **Acceptance checks**: signatures can be computed from existing data and thresholds are documented.

#### Phase 2: Build a Nudge Playbook and Safe Control Templates (Weeks 5–12)
The traffic engineering team should create a small playbook of nudges that are implementable with existing controllers, such as metering inflow by reducing green slightly on feeder approaches, flushing downstream by temporarily increasing green to clear storage, and holding coordination stable to avoid oscillations during stress. Each nudge should be bounded (for example, ±3–8 seconds or a limited number of cycles) and should include a defined recovery action that returns to the normal plan. The team should encode constraints that ensure pedestrian minimums and clearance are always preserved and that plan changes are rate-limited.

- **Roles**: traffic engineer (templates and bounds), software engineer (implementation), operations (usability), QA (test cases).
- **Deliverables**: nudge library (versioned), constraint/guardrail specification, and operator-facing descriptions of each nudge.
- **Risks**: nudges may create queues that block upstream intersections; templates may not generalize across sites.
- **Acceptance checks**: each nudge has a clear start condition, maximum duration, and recovery step.

#### Phase 3: Shadow Forecasting and Threshold Validation (Weeks 13–20)
The agency should run the risk model in shadow mode so it predicts spillback without actuating, because this allows tuning of thresholds and confidence logic without affecting traffic. The team should compare predicted jam events to observed blocked-box and spillback measures and should tune thresholds to reduce false positives. The team should add hysteresis and minimum dwell times so nudges do not turn on and off repeatedly during borderline conditions.

- **Roles**: data scientist/modeler (risk model), analyst (validation), operations (review of false positives), traffic engineering (threshold approval).
- **Deliverables**: validated risk model report, tuned thresholds, and a documentation of model limitations.
- **Risks**: prediction quality may be limited by sensor coverage; “ground truth” spillback may be hard to measure.
- **Acceptance checks**: shadow evaluation shows acceptable false positive/negative rates and stable recommendations.

#### Phase 4: Assisted Activation Pilot (Weeks 21–32)
The operations team should pilot nudges in assisted mode first, where the system recommends a nudge and an operator confirms activation, because early deployments require human judgment and trust-building. During the pilot, the team should monitor not only the protected bottleneck but also upstream and parallel routes for displacement, and it should roll back if queues exceed agreed caps. The team should log each nudge activation with the trigger evidence and the observed outcome so the program can learn which templates work.

- **Roles**: operations (activation and monitoring), traffic engineering (on-call tuning), analyst (evaluation), maintenance (field issues).
- **Deliverables**: active pilot, intervention logs, and a pilot evaluation report.
- **Risks**: operator workload increases during peaks; displacement impacts may create complaints.
- **Acceptance checks**: blocked-box frequency decreases at protected links while displacement remains within caps.

#### Phase 5: Expansion and Integration with Other Modes (Ongoing)
The agency should expand nudges to additional corridors where spillback cascades are common and should integrate them with event and weather modes so strategies do not conflict. The team should maintain a change-control process for nudge templates and thresholds, because small parameter changes can have large operational effects. Over time, the agency can increase automation for low-risk nudges if the pilot demonstrates stable performance and low false activation rates.

- **Roles**: program owner (scaling governance), traffic engineering (template updates), operations (monitoring), analytics (continuous evaluation).
- **Deliverables**: rollout roadmap, monthly performance reports, updated template library.
- **Risks**: scaling increases complexity; conflicting modes create unpredictable behavior.
- **Acceptance checks**: performance improvements persist across seasons and the system remains explainable.

### Choices (control “verbs”)
Anti-jam nudges are a small vocabulary of reversible actions.

- **Flush**: clear downstream storage.
- **Meter**: limit inflow to prevent spillback.
- **Hold**: stabilize coordination and avoid thrashing.
- **Recover**: return to baseline timing smoothly and measurably.

## 1) Equity / displacement governance (make “protect side streets” measurable)
Queue management guidance explicitly calls out controlling **where queues form** to avoid “damaging queues” and network impacts ([`FHWA “Signal Timing Under Saturated Conditions” (Guidance)`](https://ops.fhwa.dot.gov/publications/fhwahop09008/guidance.htm)). This section operationalizes that idea into measurable harm definitions and enforceable caps.

### 1.1 Define what “harm” means for nudges
Define harm as **approach- and neighborhood-specific outcomes** that worsen when a nudge is active.

Minimum recommended harm metrics (pick at least 3, keep them consistently reported):

| Harm type | Metric (measurable) | Why it matters |
|---|---:|---|
| Queue spillover | Queue length vs. storage; % time queue is “near spillback” | Prevents queues from blocking upstream intersections (queue-management objective) ([`FHWA “Signal Timing Under Saturated Conditions” (Guidance)`](https://ops.fhwa.dot.gov/publications/fhwahop09008/guidance.htm)) |
| Cycle failure / residual queue | split failures; residual queue persistence | “Residual queues that grow” are a key symptom of oversaturation and trigger for changing strategy ([`FHWA “Signal Timing Under Saturated Conditions” (Ch. 2)`](https://ops.fhwa.dot.gov/publications/fhwahop09008/chapter2.htm)) |
| Delay / max wait | max vehicle delay; max red; person-delay | Avoids repeated concentrated harm on the same approaches / communities |
| Diversion pressure | diversion proxy (side-street volume spike, probe routing changes) | Prevents moving congestion into neighborhoods |
| Complaints & safety concerns | complaint triggers; near-miss proxy; blocked crosswalk/box reports | Ensures legitimacy and responsiveness |

**Note on oversaturated evaluation:** During oversaturated conditions, recommended performance measures include queue lengths, number of cycle failures, and percent time congested; objectives become minimizing oversaturation duration and managing spillback ([`FHWA “Traffic Signal Timing Manual” Ch. 7`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter7.htm)).

### 1.2 Where harm is measured (protected places)
Create a **Protected Assets Register** and map each protected item to signal approaches.

Required protected categories:
- **Protected side streets**: residential collectors/local streets that should not become the system “buffer.”
- **School zones and senior centers**: emphasize safe crossings and predictable waits.
- **Equity-priority neighborhoods**: as defined by the agency (equity index, historically overburdened areas).
- **Bus corridors / high-frequency routes**: transit reliability must not be sacrificed silently.

### 1.3 Guardrails to prevent repeated harm
Use *rate limits* and *rolling caps* that apply per approach and per protected geography.

**Activation caps (per approach):**
- Max nudge activation **minutes per hour** and **minutes per day**.
- Max number of nudge activations per hour.

**Rolling-window caps (stability accounting):**
- “No more than **X** nudges affecting approach **A** in **Y** minutes.”
- “No more than **X** cycles with split reduced below baseline by > **Δsplit**.”

**Lightweight budget / credit accounting:**
- Each approach (or neighborhood bucket) gets a daily “nudge budget” (minutes).
- Each activation spends budget proportional to severity (e.g., Δsplit * duration).
- When budget is near exhausted → require operator confirmation or switch to alternative strategy.

This converts a vague “don’t hurt side streets” norm into enforceable, auditable behavior.

### 1.4 Spillover thresholds and response ladder
When protected assets begin to experience harmful spillover, the system must de-escalate.

**Response ladder (automatable):**
1. **Reduce magnitude** (halve Δsplit / shorten duration).
2. **Change strategy** (meter somewhere else; stop flushing that starves ped/transit).
3. **Pause** nudge mode and enter **Hold/Recover** for N cycles.
4. **Escalate to operator confirmation** (assisted mode) before further actions.

Use explicit thresholds (example template):
- If protected approach queue exceeds **Q_protect_warn** for **T_warn** → step 1.
- If exceeds **Q_protect_stop** for **T_stop** or spillback risk hits “high” → step 3.

### 1.5 Equity / displacement reporting template
Use a standard per-event report so complaints become data.

**Template (fill per activation window):**
- Event ID / corridor / time-of-day / plan
- Trigger signature + confidence
- Protected assets in control region
- **Before vs during vs after** (by approach):
  - queue length / spillback risk
  - max delay / max red
  - split failures
  - bus delay / headway disruption
  - ped delay / denied ped calls
- Equity summary (by neighborhood bucket):
  - # activations impacting bucket
  - minutes in nudge mode
  - top 3 worst-impacted approaches
- Operator notes + reason codes

## 2) Reliable spillback-risk inference (multi-signal confirmation + confidence scoring)
A core risk is acting on noisy sensing. The goal is a trigger framework that is conservative, explainable, and measurable.

### 2.1 Multi-source signals you can combine (with examples)
Use multiple weak signals rather than one “hard” threshold.

**Controller/ATSPM signals (high leverage, widely available):**
- **Split failures**: in ATSPM, split failure is inferred when both **green occupancy ratio (GOR)** and **red occupancy ratio in first 5 seconds (ROR5)** are high (typically ~80% or higher), indicating demand not served within a cycle ([`FHWA ATSPM Use Cases` (Purdue Split Failure Diagram)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).
- **Max-outs / force-offs patterns**: phase termination states can indicate insufficient green or detector issues ([`FHWA ATSPM Use Cases` (Phase Termination Diagram)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

**Timing/retiming guidance signals:**
- **Cycle failures** and **queues spilling back** are operational symptoms used to decide retiming and interventions ([`FHWA “Traffic Signal Timing Manual” Ch. 7`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter7.htm)).

**Optional / if available:**
- Downstream detector occupancy increase while discharge rate drops.
- Probe speed collapse (segment speeds) + growing travel time variance.
- CV queue estimates (if your platform supports them).
- Camera/CCTV verification (operator or automated) for “blocking the box” events.

### 2.2 Confidence scoring framework (reduce false positives)
Treat “spillback risk” as an evidence accumulator with persistence and hysteresis.

**Design principles:**
- **At least 2 independent signals** to trigger a nudge (e.g., split failures + downstream occupancy).
- **Persistence requirement**: evidence must persist for N consecutive cycles or M seconds.
- **Hysteresis**: separate thresholds for enter vs exit.
- **Debounce / cooldown**: after a nudge ends, suppress retrigger for K cycles unless risk is extreme.

**Example scoring (simple, implementable):**
- Maintain `risk_score ∈ [0, 100]` updated each cycle.
- Add points:
  - +25 if split failures detected in last 15 min window (GOR & ROR5 high) ([`FHWA ATSPM Use Cases` (Split Failure)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm))
  - +20 if downstream occupancy is high while discharge is low
  - +20 if probe speeds collapse below threshold for >2 min
  - +15 if residual queue present across ≥2 cycles
- Subtract points:
  - −15 if discharge recovers and ROR5 drops
  - −20 if probe speeds recover above threshold for >3 min

Trigger bands:
- 0–39: normal
- 40–69: watch (shadow recommend only)
- 70–84: assisted recommend
- 85–100: allow automatic nudge if governance budgets allow

### 2.3 Jam signature library (templates)
Practice guidance stresses that different regimes require different objectives and tactics ([`FHWA “Signal Timing Under Saturated Conditions” (Guidance)`](https://ops.fhwa.dot.gov/publications/fhwahop09008/guidance.htm); [`FHWA “Signal Timing Under Saturated Conditions” (Ch. 2)`](https://ops.fhwa.dot.gov/publications/fhwahop09008/chapter2.htm)). Encode this into a library.

**Signature types (minimum set):**
1. **Oversaturation at a critical movement** (growing residual queues)
2. **Downstream bottleneck / storage collapse** (risk of backing through upstream intersection)
3. **Blocking-the-box / intersection blockage** (queues occupy intersection area)
4. **Incident / lane drop signature** (sudden speed + capacity collapse)

**Per-signature spec template (copy/paste):**

```text
Signature name:
Objective: (throughput-maximization | queue-management)
Control region footprint: (intersection only | 2-3 signals | corridor segment)
Inputs:
  - Signal A:
  - Signal B:
  - Optional confirmation:
Trigger thresholds:
  - Condition(s):
  - Persistence: N cycles / M seconds
  - Confidence score threshold:
Enter criteria:
  - risk_score >= ... AND governance_budget_ok
Exit criteria:
  - risk_score <= ... for ...
  - OR protected-asset harm threshold exceeded
Recommended nudges:
  - Primary:
  - Secondary:
Constraints binders:
  - ped hard constraints:
  - transit hard/soft constraints:
Rollback / recover:
  - revert steps:
Logging:
  - required evidence fields:
```

### 2.4 False-alarm measurement plan (precision/recall)
Shadow-mode is essential: guidance recommends proactive monitoring and evaluation loops ([`FHWA “Traffic Signal Timing Manual” Ch. 7`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter7.htm)). Also, ATSPM emphasizes that tools don’t solve problems without interpretation and action ([`FHWA ATSPM Use Cases` (intro)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

Track:
- **Precision**: % of triggers that correspond to confirmed spillback/jam events.
- **Recall**: % of observed spillback/jam events that were predicted.
- Operator confirmations / overrides.
- Detector health flags (avoid “false alarms” caused by bad detection; ATSPM watchdog patterns help) ([`FHWA ATSPM Use Cases` (Watchdog)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

Operational loop:
- Weekly review of top false positives + threshold tuning.
- Monthly KPI: false-alarm rate by intersection and by signature.

## 3) Multimodal constraints (ped/transit are first-class binders)
“Don’t violate pedestrian service/clearance” is necessary but not sufficient. Make ped/transit constraints explicit and measurable.

### 3.1 Hard constraints vs soft objectives
**Hard constraints (must never be violated):**
- Pedestrian walk + clearance intervals as configured (MUTCD-driven, agency policy). Signal timing development guidance treats pedestrian timing as a standards/parameters item and a required minimum split check (are ped times met?) ([`FHWA “Traffic Signal Timing Manual” Ch. 7`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter7.htm)).
- ADA/accessibility requirements (audible/tactile if present; avoid excessive waits at pushbutton crossings).

**Soft objectives (optimize but may trade off with explicit approval):**
- Ped delay target bands (e.g., keep average below X, cap max at Y where feasible).
- Transit performance targets: on-time performance, headway regularity.

ATSPM guidance notes that “excessive pedestrian delay” can increase risk-taking; ped delay is defined as time from pushbutton actuation to walk display and can be used to identify issues ([`FHWA ATSPM Use Cases` (Pedestrian Delay)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

### 3.2 How nudges can harm ped/transit (and mitigations)

| Nudge type | How it can degrade ped/transit | Mitigations |
|---|---|---|
| Meter | Longer reds on side streets can increase ped delay and bus delay on cross streets | (1) hard cap on ped max wait at protected crossings; (2) exclude bus corridors from metering during headway recovery; (3) require “ped recall preserved” during metering windows where demand is high ([`FHWA ATSPM Use Cases` (Ped delay definition)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)) |
| Flush | Over-allocating green to one movement can starve ped phases or create long cycles | Prefer shortest cycle compatible with minimum intervals in congested operations; oversaturation guidance highlights that objectives shift to managing queues and that cycle length tradeoffs are nontrivial ([`FHWA “Signal Timing Under Saturated Conditions” (Guidance)`](https://ops.fhwa.dot.gov/publications/fhwahop09008/guidance.htm)) |
| Hold | Holding coordination can delay plan transitions that might benefit transit priority | Define TSP interaction policy (below); log “TSP blocked by nudge” |
| Recover | Abrupt recovery can create long pedestrian waits or bus bunching if not staged | staged recovery: ramp split changes, enforce cooldown |

### 3.3 TSP interaction policy (define up front)
Decide and document:
- When a nudge can **override** TSP (rare: safety/queue storage emergencies).
- When TSP can **override** a nudge (common: bus corridor protection).
- Whether TSP requests count against “nudge budgets” (recommended: track separately but report interactions).

### 3.4 Required monitoring KPIs
- Ped delay (avg/max), **denied ped calls**, pedestrian recall coverage ([`FHWA ATSPM Use Cases` (Ped delay)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).
- Bus delay / reliability (OTP, headway variance), priority request interference logs.
- Safety proxies: red-light running risk dashboards (yellow/red actuations) where available ([`FHWA ATSPM Use Cases` (Yellow & Red Actuations)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

## 4) Multi-intersection coordination patterns + recovery rules
Oversaturation guidance stresses working back from the downstream bottleneck and coordinating upstream signals to control flow into a congested intersection to prevent queues from multiplying ([`FHWA “Signal Timing Under Saturated Conditions” (Ch. 2)`](https://ops.fhwa.dot.gov/publications/fhwahop09008/chapter2.htm)). This section turns that into practical “2–3 signal” patterns.

### 4.1 When local nudges are insufficient (expand the control region)
Expand from 1 intersection → 2–3 intersections when:
- Spillback risk persists despite 2–3 local cycles of action.
- Queue is spanning blocks and threatens to back through an upstream intersection.
- The downstream bottleneck is the true constraint (local extra green cannot create storage).

### 4.2 Required visual: bottleneck + upstream metering + boundary constraints diagram

Diagram the control region so it is clear where you meter, where you flush, and where **boundaries** must not be crossed.

```text
        Upstream             Mid-block              Downstream bottleneck

      [A] --------(link1)-------- [B] --------(link2)-------- [C]
       |                           |                           |
       |  Meter here               |  Optional meter           |  Flush here (if storage exists)
       v                           v                           v
   [Protected side street]     [Local street]           [Downstream link]
       ^                           ^
       |                           |
   BOUNDARY: do not           BOUNDARY: do not
   push queues into           block crosswalk /
   residential zone           bus stop zone
```

Key concepts in the diagram:
- **[C]** is the **bottleneck** (e.g., bridge, ramp, lane drop).
- **[B]** and **[A]** are upstream signals where you may meter.
- **BOUNDARY** callouts mark **equity/displacement guardrails** (school, neighborhood, bus corridor).
- Metering must not push queues past these boundaries; if queues approach them, you must scale back metering and/or change strategy.

This visual directly ties jam control to both **coordination** and **equity guardrails**.

### 4.3 Boundary constraints so queues don’t spill into protected areas
For each metering intersection, declare:
- **Max allowable queue** on each approach (by storage or protected-asset threshold).
- **Must-not-block** zones: school crossings, protected side streets, bus stops.

If a queue approaches the boundary → reduce metering (or shift metering upstream/downstream).

### 4.4 Coordination-friendly limits
- **Caps on split changes per cycle** (avoid thrash).
- **Max duration in “nudge mode”** before requiring recovery or operator confirmation.
- **Recovery-to-coordination steps**:
  1. Hold last stable plan for N cycles.
  2. Ramp splits back toward baseline (e.g., 50% of Δ per cycle).
  3. Re-enable normal plan transitions.

Plan transitions can destabilize close intersections; practice examples include programming plan changes to avoid problematic transitions at critical intersections ([`FHWA “Signal Timing Under Saturated Conditions” (Ch. 2)`](https://ops.fhwa.odot.gov/publications/fhwahop09008/chapter2.htm)).

### 4.5 Required visual: jam signature → trigger → nudge → rollback table

Summarize how detected jam signatures map to specific nudges and rollback conditions.

```markdown
| Jam signature | Trigger conditions (simplified) | Primary nudge actions | Key constraints / guardrails | Rollback criteria |
|---|---|---|---|---|
| Oversaturation at critical movement | Split failures (GOR & ROR5 high) for ≥ N cycles; residual queue growing; risk_score ≥ 70 | FLUSH downstream by +3–6s on critical movement; HOLD coordination for 2–3 cycles | Ped hard mins; max added cycle length; equity budget for impacted side streets | risk_score < 40 for 3 cycles **OR** protected-asset queue > Q_protect_warn; then ramp splits back and release hold |
| Downstream bottleneck / storage collapse | Downstream detector occupancy high; low discharge; queue near downstream stopline; risk_score ≥ 75 | METER upstream at B (−3–6s on feeder), optional meter at A; HOLD coordination | Do not exceed A’s queue boundary into protected side street; transit corridor excluded from metering during headway recovery | risk_score < 40 for 3 cycles **OR** A/B boundary queues exceed Q_protect_stop; then restore splits and log boundary breach |
| Blocking-the-box at intersection | Detector/CCTV shows blocking-the-box events ≥ K per 15 min; queues across intersection line | SHORTEN upstream approaches that feed blockage; ADJUST offsets to create clearance; increase all-red if policy allows | Maintain ped crossing safety; don’t starve conflicting approaches beyond delay caps; respect equity budgets | Blocked-box rate < threshold for 30 min; complaints in protected zone; then roll back to baseline and review plan |
| Incident / lane drop upstream | Sudden speed collapse on link; detector volume drop; incident feed confirms event | METER inflow from parallel routes; FLUSH alternate routes; possibly ENABLE special incident plan | Don’t overflow alternate routes into equity-priority neighborhoods; keep bus corridors viable; maintain ped service | Incident cleared; travel times normalize; alternates see growing harm; then revert to normal or pre-incident plan, per playbook |
```

This table is the **operational bridge** between:
- **Jam signatures** (how we detect problems),
- **Triggers** (when we act),
- **Nudges** (what we do), and
- **Rollback** (how we safely stop).

## 5) Implementation artifacts: playbook, action vocabulary, acceptance criteria

### 5.1 Nudge playbook template (jam signature → actions → constraints → monitoring → rollback)

```yaml
playbook_entry:
  signature: "Downstream bottleneck storage collapse"
  objective: "queue-management"
  trigger:
    evidence:
      - "split_failures_high"  # e.g., GOR & ROR5 high
      - "downstream_occupancy_high"
    persistence: ">= 2 cycles"
    confidence_threshold: 75
  control_region:
    intersections: ["B", "C"]
  recommended_actions:
    - action: "METER_UPSTREAM"
      location: "B"
      magnitude: "-4s to feeder split"
      duration: "3 cycles"
      cooldown: "5 cycles"
    - action: "HOLD_COORDINATION"
      location: "B,C"
      duration: "3 cycles"
  constraints:
    ped_hard:
      - "do not reduce below ped minimum split"
      - "max ped wait cap at protected crossings"
    transit_policy:
      - "do not meter bus corridor during headway recovery"
    equity_guardrails:
      - "approach_budget_remaining >= 10 min/day"
  monitoring:
    kpis:
      - "queue_length_vs_storage"
      - "split_failures"
      - "ped_delay_max"
      - "bus_delay"
  rollback:
    exit_criteria:
      - "risk_score < 40 for 3 cycles"
      - "protected_harm_stop triggered"
    steps:
      - "restore baseline splits"
      - "release hold"
      - "log reason_code"
```

### 5.2 Action vocabulary table

```markdown
| Action | Preconditions | Max magnitude | Duration / cooldown | Risks | Rollback |
|---|---|---:|---|---|---|
| Flush downstream (add green) | Downstream can discharge; storage exists to clear | +3–8s split, bounded | 1–5 cycles; cooldown 5–10 cycles | Starves side streets/peds; longer cycle | revert split; exit after N cycles; enforce ped mins |
| Meter upstream inflow | Confirm downstream bottleneck; protect boundaries | −3–8s split (or reduce max) | 2–10 cycles; cooldown 5–20 cycles | Queue displacement into neighborhoods | stop if protected thresholds triggered; ramp back |
| Hold coordination / freeze transitions | Plan transition thrash risk; close intersections | no plan switch; limited split change | 2–10 cycles | Blocks beneficial transitions / TSP | release hold; staged recovery |
| Phase reservice / double-serve major movement (if supported) | Movement imbalance; minor movements can be served less frequently | site-specific | bounded window | Confusing to maintain if esoteric | disable reservice; restore normal phasing |
```

Phase reservice and “serve heavy movements more than once in a cycle” are repeatedly cited as effective in saturated conditions and queue management contexts ([`FHWA “Signal Timing Under Saturated Conditions” (Guidance)`](https://ops.fhwa.dot.gov/publications/fhwahop09008/guidance.htm); [`FHWA “Signal Timing Under Saturated Conditions” (Ch. 2)`](https://ops.fhwa.dot.gov/publications/fhwahop09008/chapter2.htm)).

### 5.3 Shadow-mode validation and go/no-go
Guidance recommends proactive monitoring programs and before/after evaluation to confirm objectives and policies were met ([`FHWA “Traffic Signal Timing Manual” Ch. 7`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter7.htm)).

**Shadow-mode requirements:**
- Run triggers + recommended actions without actuating.
- Log “would-have-acted” timestamps and evidence.
- Compare to observed outcomes (blocked-box proxy, spillback occurrences, split failures).

**Go/no-go acceptance criteria (minimum):**
- False-alarm rate below agreed threshold (by signature, by corridor).
- No sustained increase in pedestrian delay (avg/max) on protected crossings.
- No repeated harm violations: protected approaches stay within caps/budgets.
- No safety regression indicators (e.g., red-light running patterns where measured) ([`FHWA ATSPM Use Cases` (Yellow & Red Actuations)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

**Rollout phases:**
1. Shadow-only.
2. Assisted activation (operator confirm).
3. Limited automation for low-risk signatures.
4. Expanded corridors + periodic retune.

**Operator UX requirements:**
- Option card shows: signature, evidence, confidence, expected benefit, constraints impacted.
- One-click revert.
- Reason codes for confirm/decline.

## Technical Mechanics

### Key Parameters
- Spillback probability / risk score.
- Split shift bounds.
- Dwell times and recovery thresholds.

### Guardrails
- Never violate pedestrian service and clearance behavior.
- Rate-limit interventions.
- Require recovery plan after each nudge.

## MVP Deployment
- 1–2 intersections on a known bottleneck.
- One metering template + one flush template.
- Shadow mode first.

## Evaluation
- Blocked-box frequency reduction.
- Travel time reliability during peaks/incidents.
- Unintended queue displacement.

During oversaturated conditions, relevant performance measures include queue lengths, number of cycle failures, and percent time congested; the objective is to minimize oversaturation duration and manage spillback between intersections ([`FHWA “Traffic Signal Timing Manual” Ch. 7`](https://ops.fhwa.odot.gov/publications/fhwahop08024/chapter7.htm)).

---

## Implementation Checklist
- [ ] Identify priority corridors/intersections where spillback cascades occur.
- [ ] Build **Protected Assets Register** (side streets, schools, equity-priority zones, bus corridors).
- [ ] Select measurable harm metrics + thresholds per protected asset.
- [ ] Define jam signatures and implement signature templates.
- [ ] Implement multi-signal risk scoring with persistence/hysteresis.
- [ ] Define action vocabulary with max magnitudes, durations, cooldowns.
- [ ] Implement governance caps (minutes/hour/day, rolling-window, budgets).
- [ ] Instrument multimodal KPIs (ped delay, denied calls, bus delay/OTP).
- [ ] Build operator UX (evidence, confidence, one-click revert, reason codes).
- [ ] Run shadow-mode for ≥4 weeks and compute precision/recall.
- [ ] Pilot assisted activation with evaluation and public reporting.

## Operations Runbook (SOP)
### A. Normal monitoring
1. Review daily watchdog / detection health so triggers are not driven by bad sensors ([`FHWA ATSPM Use Cases` (Watchdog)](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).
2. Review daily top split-failure locations (GOR/ROR5) to identify oversaturation hotspots ([`FHWA ATSPM Use Cases` (Split Failure)](https://ops.fhwa.odot.gov/publications/fhwahop20002/ch4.htm)).

### B. Assisted nudge activation (operator-confirm)
1. Open the nudge option card.
2. Confirm:
   - signature + confidence
   - protected assets impacted
   - multimodal binders (ped/transit)
   - budget availability
3. Activate recommended action.
4. Monitor for 2–3 cycles:
   - queue boundary thresholds
   - ped delay spikes
   - bus delay spikes
5. If harm threshold exceeded → execute rollback.

### C. Rollback / recovery
1. Revert split deltas to baseline.
2. Release hold(s) after N cycles.
3. Log:
   - what happened
   - what evidence was used
   - why rollback occurred

### D. Post-event review (within 48 hours)
1. Generate the equity/displacement report template.
2. Classify activation outcome: helpful / neutral / harmful.
3. Adjust thresholds if repeated false positives.

## Governance & Equity Guardrails Runbook
### 1. Set policy targets
- Define harm metrics and protected zones (section 1.1–1.2).
- Approve caps/budgets and escalation rules.

### 2. Audit schedule
- Weekly: false positives, budget overruns, top impacted approaches.
- Monthly: neighborhood-level impact report.

### 3. Complaint-driven triggers
- If complaint rate crosses threshold in protected zone:
  - require operator confirmation for that zone
  - reduce max magnitude or disable certain nudges

### 4. Change control
- Any threshold/playbook change requires:
  - reason
  - expected impact
  - rollback plan
  - before/after evaluation plan ([`FHWA “Traffic Signal Timing Manual” Ch. 7`](https://ops.fhwa.odot.gov/publications/fhwahop08024/chapter7.htm))

## Reference Links
- [`FHWA “Signal Timing Under Saturated Conditions” (Guidance)`](https://ops.fhwa.dot.gov/publications/fhwahop09008/guidance.htm)
- [`FHWA “Signal Timing Under Saturated Conditions” (Ch. 2 — State of the Practice)`](https://ops.fhwa.dot.gov/publications/fhwahop09008/chapter2.htm)
- [`FHWA “Traffic Signal Timing Manual” (archived) — Ch. 7`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter7.htm)
- [`FHWA “Automated Traffic Signal Performance Measures” (ATSPM report landing)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/index.htm)
- [`FHWA ATSPM — Sample Use Cases (split failures, ped delay, watchdog, etc.)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)

## Completion Checklist
- ✅ Equity/displacement governance made measurable (see **“1) Equity / displacement governance”**).
- ✅ Spillback-risk inference with multi-signal confirmation + confidence scoring + signature library + false-alarm plan (see **“2) Reliable spillback-risk inference”**).
- ✅ Multimodal ped/transit treated as binders with KPIs and mitigation guidance (see **“3) Multimodal constraints”**).
- ✅ Multi-intersection coordination patterns + **required bottleneck/upstream-metering/boundary diagram** and coordination limits (see **“4.2–4.4”**).
- ✅ Required **jam signature → trigger → nudge → rollback table** plus YAML playbook template and action vocabulary table (see **“4.5”** and **“5) Implementation artifacts”**).
- ✅ Final sections added: **Implementation Checklist**, **Operations Runbook (SOP)**, **Governance & Equity Guardrails Runbook**, **Reference Links**, **Completion Checklist**.

---

Cross-links: Related ideas include what-if button, self-healing intersections, and fast-forward twin.
