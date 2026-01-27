# 04) City Rules Built Into Signals: Policy-Driven Traffic Control

## Catchy Explanation
Instead of hoping every timing tweak “remembers” city priorities, you bake the priorities directly into the controller: the signal is allowed to optimize traffic only inside the city’s rulebook.

## What it is (precise)
**City rules built into signals** means encoding policy goals (safety, accessibility, transit reliability, equity, climate/noise goals) as **hard constraints** and **auditable objectives** in the signal control stack.

Examples include: maximum pedestrian wait, required leading pedestrian interval (LPI) in defined zones, minimum transit service, school-zone timing rules, or fairness caps to avoid repeatedly harming the same side street.

A digital twin is used to test candidate rule sets, quantify tradeoffs, and generate “policy impact statements” before deployment. The system also needs an exception workflow (operator overrides with reason codes) and periodic audits.

## Benefits
- **Policy compliance by design**: rules are enforced consistently, shift-to-shift.
- **Multimodal balance**: explicit support for pedestrians, bikes, transit.
- **Transparency and governance**: decisions map back to an approved “policy contract”.
- **Safer operations**: safety constraints can be made non-negotiable.

## Challenges
- **Tradeoffs become explicit**: some users will experience more delay.
- **Political alignment**: choosing priorities and thresholds is hard.
- **Sensor dependency**: enforcement often needs reliable multimodal detection.
- **Constraint tuning**: too strict → brittle control; too loose → no effect.

## Implementation Strategies

### Infrastructure Needs
- **Policy engine**: representation of constraints/objectives (hard vs soft) and rule sets by context (time-of-day, zone, weather).
- **Multimodal sensing**: pedestrian calls/volumes, bike detection, transit AVL/TSP requests.
- **Controller capability**: ability to implement LPI, TSP, phase holds, plan switching.
- **Twin + evaluation**: simulate rule sets and produce KPI + compliance reports.
- **Audit logs**: reason codes, evidence capture, and override tracking.

---

## 1) Enforcement boundary: controller safety layer vs supervisory “policy engine”

### Why the boundary matters
If you blur “policy optimization” (what the city wants) with “safety/legality enforcement” (what must always hold), you risk deploying a system that is either:
- **Unsafe** (soft goals accidentally override safety-critical timing), or
- **Unoperable** (policy engine recommends actions the controller cannot legally/physically perform), or
- **Unauditable** (nobody can prove which part of the stack caused a violation).

### Partitioning strategy (recommended)

#### What must live in the controller (safety-critical / legal minimums)
These are the invariants that should be enforced locally and deterministically, even if communications fail:
- **Conflict matrix / intergreen logic** (no conflicting greens, ring-barrier logic).
- **Change and clearance intervals** (yellow and red clearance / all-red practices). The MUTCD and FHWA timing guidance emphasize that these intervals are predetermined and tied to approach speed/intersection geometry (see discussion of change/clearance in the FHWA timing manual and MUTCD references within it) ([`FHWA Traffic Signal Timing Manual — Chapter 5`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm)).
- **Pedestrian intervals and minimum pedestrian service components**, including:
  - Ped interval structure: WALK → flashing DON’T WALK (clearance) → steady DON’T WALK buffer ([`MUTCD 2009 Part 4, Chapter 4E`](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm)).
  - WALK interval guidance: typically at least 7 seconds, with short values (down to 4s) only under limited conditions ([`MUTCD 2009 Part 4, Chapter 4E`](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm)).
  - Clearance-time guidance and walking speed assumptions (e.g., 3.5 ft/s baseline; slower where needed) ([`MUTCD 2009 Part 4, Chapter 4E`](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm)).
  - Buffer interval (steady DON’T WALK) displayed for at least 3 seconds prior to release of conflicting vehicular movement ([`MUTCD 2009 Part 4, Chapter 4E`](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm)).
- **LPI implementation constraints** (if LPIs are used): MUTCD provides that LPIs should be at least 3 seconds and timed so pedestrians can establish position (and consideration for turn restrictions during LPI) ([`MUTCD 2009 Part 4, Chapter 4E`](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm)).
- **Failsafe modes** (flash, time-of-day fixed plans, detector failure behavior such as recall) and the ability to remain operational without upstream compute. The timing manual discusses recall and its use when detection is absent/out of service ([`FHWA Traffic Signal Timing Manual — Chapter 5`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm)).

In other words: the controller enforces *“never violate these minimums.”* The supervisory system should not be able to override them, only to choose among actions that already satisfy them.

#### What belongs in the supervisory layer (policy / optimization / mode selection)
These are rules that shape service **within** the controller’s feasible/safe envelope:
- **Mode selection**: time-of-day plans, special event plans, bad-weather plans, school arrival/dismissal bundles.
- **Objective weighting**: e.g., person-delay vs vehicle delay, transit reliability, noise/emissions proxy weights.
- **Target constraints that are not purely safety-critical** but are policy goals, e.g.:
  - Max-wait targets for side streets or pedestrians (as *policy*, not as a safety-critical controller interlock).
  - Transit priority policies (what requests are honored, caps per time window).
  - Progression preferences (“green wave” for bikes/transit in a corridor).
- **Exception handling workflow**: human-authorized override tokens with TTL, reason codes, and audit output.

#### What happens when supervisory recommendations conflict with controller constraints
The controller should be treated as the final arbiter:
1. Supervisory system produces a **candidate action** (plan switch, offset tweak, max green limit update, phase omit, etc.).
2. Controller/safety layer performs **feasibility + safety validation**.
3. If invalid, the controller rejects (or clamps) the action and emits an **explainable rejection reason** (e.g., “would violate pedestrian clearance minimum per current crossing distance assumptions”).
4. Supervisory system logs rejection + attempts an alternative action; repeated rejections should trigger a policy-contract review (rule may be impossible on that intersection).

### Reference architecture (vendor-neutral)

```
               +---------------------------+
               |     City Policy Owner     |
               |  (Vision Zero, Transit,   |
               |  Schools, Equity Office)  |
               +-------------+-------------+
                             |
                             | approves
                             v
+----------------------------+----------------------------+
|                 Policy Contract (versioned)             |
|  - scope (geo/time)
|  - hard constraints (must)
|  - soft objectives (optimize)
|  - exceptions (who/TTL/audit)
|  - observability requirements + fallbacks
+----------------------------+----------------------------+
                             |
                             | validate + compile
                             v
+----------------------------+----------------------------+
|  Compiler / Validator / Linter                           |
|  - schema validation + unit tests                         |
|  - conflict detection (unsat rules)                       |
|  - controller capability mapping                           |
+----------------------------+----------------------------+
          |                               |
          | outputs                       | outputs
          v                               v
+--------------------------+     +--------------------------+
| Controller Config Bundle |     | Supervisory Constraints  |
| (safety/legal floor)     |     | + objective weights      |
| - ped min timings        |     | + allowable actions set  |
| - clearance intervals    |     | + arbitration priorities |
| - LPI enablement rules   |     | + exception workflow     |
+-------------+------------+     +-------------+------------+
              |                                |
              | deploy                         | deploy
              v                                v
+-------------+------------+     +-------------+------------+
| Field Controller         |<--->| ATMS / Policy Engine     |
| (deterministic safety)   |     | (optimization + audit)   |
+-------------+------------+     +-------------+------------+
              |
              | high-res logs / events
              v
+-------------+------------+
| Monitoring + Compliance  |
| - ATSPM-style metrics    |
| - policy compliance KPIs |
| - confidence scoring     |
+--------------------------+
```

---

## 2) Compliance measurement: how to prove rules are being followed (with partial data)

### Core idea
Compliance is not a yes/no claim. It is a time series with:
- **A rule definition**
- **A measurable proxy**
- **Known data gaps**
- **A confidence score**

Use ATSPM-style high-resolution controller logs and associated measures where possible. FHWA emphasizes that ATSPM does not “solve problems by itself,” but it provides practical measures that engineers interpret and act on ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

### Policy primitive → KPI mapping table (template)

| Policy Primitive | Measurable KPI / Proxy | Required data | Data gaps behavior | Confidence score (example) |
|---|---|---|---|---|
| Max pedestrian wait (e.g., <= 60s in zone) | Distribution of **ped actuation delay** (press → WALK) and `% actuations <= threshold` | Ped button actuations + WALK start times in high-res logs | If ped actuation events missing → compliance = **unknown** for that period; open maintenance ticket | 1.0 when events complete; 0.5 if partial; 0 if missing |
| LPI required in zone | % cycles where pedestrian WALK begins **≥ X seconds** before parallel vehicle release (or turning release if modeled) | Phase state / interval transitions + ped WALK timing; configured plan/overlap mapping | If phase mapping ambiguous → report “indeterminate” until mapping fixed | 0.8 if mapping validated; 0.3 otherwise |
| “No excessive ped delay” | Ped delay diagram stats (min/max/avg) and “excessive” threshold | Same as above | If detector stuck (e.g., excessive ped actuations at night), flag as data-quality issue (ATSPM watchdog concept) | 0.9 with quality checks passed |
| Split failure cap on minor street | Split failures per day / per plan (GOR/ROR5-based) | Stopbar / advance detection + phase green/red intervals | If detection missing, cannot compute; show unmeasurable + suggest temporary recall settings | 1.0 with detectors healthy; 0 if not |
| Progression goal (corridor) | Arrivals on green (AoG) trend | Upstream detection arrivals + phase green windows | If upstream detection absent, estimate via probe travel time reliability (lower confidence) | 0.7 with probes; 1.0 with detectors |
| Emergency preemption priority | Preemption event served within time bound; recovery time to normal | Preemption logs + phase states | If preemption logs unavailable, compliance unknown; require integration | 1.0 with logs |

**Notes / grounding:**
- FHWA defines pedestrian delay as time between pushbutton actuation and WALK indication in the ATSPM pedestrian delay use case ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).
- FHWA describes split failures and their detection via GOR/ROR5 ratios in ATSPM use cases ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).
- FHWA describes arrivals on green (AoG) within the Purdue coordination diagram context ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

### Computing compliance over time
Implement these standard rollups:
- **% time constraint met**: time-weighted across cycles or 1-minute bins.
- **Violations per day**: count distinct violations with severity categories.
- **Minutes in exception mode**: total time a rule is overridden.

Suggested severity levels:
- **S0 (safety-critical)**: would violate legal/safety minimums (should be impossible if controller enforces correctly).
- **S1 (policy hard constraint)**: contract hard constraint violated.
- **S2 (policy soft target)**: objective missed.
- **S3 (unknown)**: unmeasurable due to missing data.

### Handling missing/noisy detection (avoid “performative compliance”)
1. **Imputation rules (explicitly documented):**
   - If ped actuation missing but WALK occurs, do *not* infer compliance; mark “unknown.”
   - If detection fails quality checks (e.g., stuck ped button patterns), exclude and open maintenance ticket. FHWA highlights “stuck ped” detection patterns as a watchdog symptom ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).
2. **Conservative assumptions:**
   - If a policy constraint is about *maximum* wait, and data is partial, do not claim pass—report **unknown** or **fail** depending on your governance standard.
3. **Confidence bands:**
   - For each KPI, report a confidence score (0–1) and display uncertainty bands for monthly reports.

---

## 3) Policy conflict arbitration & exception hierarchy (explicit and implementable)

### Arbitration framework (recommended priority order)
When two goals conflict, decide up front what wins and encode it.

A practical default:
1. **Safety-critical constraints** (conflict matrix, clearance intervals, no conflicting greens)
2. **Legal minimums** (ped timing structure, clearance rules, buffer intervals) ([`MUTCD 2009 Part 4, Chapter 4E`](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm))
3. **Emergency response / preemption** (local EVP policy)
4. **Transit reliability commitments** (TSP caps and rules)
5. **School-zone protections** (timing bundles, lower speed assumptions)
6. **Equity/fairness constraints** (disparity caps, prioritized zones)
7. **Efficiency objectives** (delay/throughput/progression)

### How arbitration is encoded
Use a hybrid of:
- **Hard constraints**: must always hold.
- **Soft objectives**: optimized with weights.
- **Lexicographic ordering**: optimize objective tier 1, then tier 2, etc., without degrading higher tiers.
- **Exception tokens**: narrow, time-limited overrides that require audit.

### Exception tokens (operator overrides)
An exception must include:
- scope (intersection(s)/movement(s))
- TTL (hard expiration)
- reason code
- operator identity + approval chain
- required “evidence” attachment (e.g., incident ticket ID)

### Worked example (bus priority vs school-zone ped protection)
**Scenario:**
- School-zone policy says: during 07:00–08:30, pedestrian max wait <= 45s and LPI >= 3s on specific crosswalks.
- A bus requests priority at 07:45.

**Decision:**
1. Controller enforces ped minimums and LPI rules (legal/policy hard). ([`MUTCD 2009 Part 4, Chapter 4E`](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm)).
2. Supervisory layer may grant TSP only if it does not violate those constraints:
   - It can adjust green extension within available slack.
   - It can switch to a school-zone plan that already incorporates transit-friendly splits.
3. If the bus is severely late and there is an incident, a supervisor can issue an exception token to relax a *soft* target (e.g., progression preference) but not the ped minimums.

Result: **school-zone ped protection wins** over TSP if the TSP action would break the ped constraint; TSP is granted only within feasible slack.

---

## 4) Equity operationalization inside the “policy contract”

### Equity definitions suitable for signal operations
Equity needs operational definitions that map to measurable quantities:
- **Person-delay by mode**: delay weighted by persons, not vehicles (bus load factors where possible).
- **Neighborhood burden**: recurring delay/stop exposure assigned to zones.
- **Accessibility impacts**: whether pedestrians/bikes/transit receive consistent, predictable service.
- **Exposure proxies**: e.g., number of pedestrian actuations with high delay; repeated long waits can increase risky crossings (FHWA discusses excessive pedestrian delay and its risks in the ATSPM pedestrian delay use case) ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

### Concrete equity metrics + reporting slices
Produce equity reporting “cuts”:
- by corridor segment
- by time-of-day plan
- by zone type (equity-priority, school-zone, transit-priority)

Metrics (examples):
- P50/P90 pedestrian delay by zone
- Max wait violations per 1,000 actuations
- Person-delay share by mode and zone (if passenger estimates exist)
- Disparity: difference in P90 pedestrian delay across zones

### How equity enters the policy contract
Pick one or more mechanisms (explicit in contract):
- **Hard caps**: e.g., “in equity-priority zones, max ped wait <= 60s.”
- **Weighted objectives**: person-throughput weighting (transit passengers), or additional penalty for delay in high-need zones.
- **Disparity constraints**: “difference in P90 ped delay between Zone A and Zone B <= 20s.”

### Data requirements and limitations (explicit)
- Without pedestrian actuation + walk timing events, you cannot compute ped delay reliably.
- Without transit AVL + load estimates, person-delay is approximated.
- Without geofenced zone tagging, you cannot report neighborhood burden.

Your governance must allow an “unmeasurable” category rather than forcing false precision.

---

## 5) Degradation handling when policy-relevant detection is missing

### Policy observability levels
Define observability per intersection and per rule:
- **FULL**: all required detection/logs present and quality-checked.
- **PARTIAL**: some required data missing; some rules measurable/enforceable.
- **NONE**: policy-relevant sensing/logging absent.

### What rules can be enforced by observability
- **Controller safety/legal minimums**: always enforced (independent of supervisory).
- **Policy rules requiring sensing** (e.g., bike presence, ped volume thresholds): enforce only in FULL/PARTIAL per rule definition.

### Safe fallback assumptions per mode
If observability drops, the system should fall back to conservative service:
- Pedestrians: use pedestrian recall during expected peak ped periods (FHWA discusses pedestrian recall use and pedestrian timing mechanics) ([`FHWA Traffic Signal Timing Manual — Chapter 5`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm)).
- Detection failure: use recall strategies; the timing manual describes recall as a technique when detection is absent/out of service ([`FHWA Traffic Signal Timing Manual — Chapter 5`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm)).

### Avoiding performative compliance claims
When observability is PARTIAL/NONE:
- Compliance output must show **UNMEASURABLE/UNKNOWN**, not “pass.”
- Automatically open a maintenance ticket and annotate reports.
- Restrict optimization actions (e.g., forbid offset changes or priority expansions) until observability recovers.

---

## 6) Lifecycle governance: versioning, transparency, change control, re-baselining

### Versioning policy contracts
Use SemVer-like semantics:
- **MAJOR**: changes hard constraints, arbitration priorities, or scope logic.
- **MINOR**: new rules added, new zone bundles, new KPI outputs.
- **PATCH**: threshold tuning within pre-approved ranges, documentation fixes.

Each release includes:
- migration notes
- “expected impact” summary
- rollback plan

### Change control board (CCB) / approvals
Recommended roles:
- Signals operations lead (owns daily ops)
- Traffic engineering (timing + safety)
- Transit operations (TSP commitments)
- Schools/child safety rep
- Vision Zero / safety office
- Equity office
- ADA/accessibility reviewer
- IT/security
- Legal

Workflow:
1. Propose change (PR to policy contract repo)
2. Automated lint + simulation tests in twin
3. CCB review + sign-off
4. Staged rollout (shadow → assisted → enforce)
5. Post-deploy audit

### Public transparency expectations
Publish what you can without exposing sensitive details:
- high-level rule summaries (e.g., “LPIs in school zones,” “max ped wait targets”)
- performance dashboards with confidence labels
- change announcements with expected tradeoffs

### Review cadence and re-baselining
- Quarterly: compliance + exception patterns review
- Annual: re-baseline KPIs due to land use, seasonal travel, construction
- Event-driven: Vision Zero policy updates, major network changes

---

## 7) Concrete “Policy Contract” schema example (machine-readable)

Below is an intentionally short YAML example you can compile into controller-safe settings + supervisory objectives.

```yaml
apiVersion: traffic-policy.city/v1
kind: SignalPolicyContract
metadata:
  id: sofia-visionzero-policy
  version: 1.2.0
  approvedBy:
    - name: Signals Ops Director
      date: 2026-01-10
    - name: Vision Zero Lead
      date: 2026-01-10
scope:
  timezone: Europe/Sofia
  geo:
    intersections:
      - id: INT-1042
      - id: INT-1043
    corridors:
      - id: CORRIDOR-5
  timeWindows:
    - id: school_morning
      days: [Mon, Tue, Wed, Thu, Fri]
      start: "07:00"
      end: "08:30"
      tags: [school-zone]

arbitration:
  priorityOrder:
    - safety_critical
    - legal_minimums
    - emergency_preemption
    - transit_reliability
    - school_zone
    - equity
    - efficiency

hardConstraints:
  legal_minimums:
    pedestrian:
      walkInterval:
        defaultSeconds: 7
        minSeconds: 4
        note: "Default 7s; min 4s only where justified" # MUTCD allows shorter under limited conditions
      bufferSteadyDontWalkSeconds: 3
      lpi:
        enabledInTags: [school-zone]
        minSeconds: 3
  maxPedWait:
    - id: max_ped_wait_school
      appliesWhen:
        timeWindow: school_morning
      thresholdSeconds: 45
      measurement:
        kpi: ped_actuation_delay_seconds
        percentile: 90
        allowedExceedanceRate: 0.05

softObjectives:
  weights:
    personDelay: 0.45
    transitDelay: 0.25
    vehicleStops: 0.15
    emissionsProxy: 0.10
    equityDisparity: 0.05
  constraintsAsPenalties:
    - id: penalize_split_failures
      kpi: split_failures_per_hour
      weight: 0.2

exceptions:
  allowed:
    - id: incident_override
      whoCanApproveRoles: [SignalsSupervisor, TrafficEngineerOnCall]
      ttlMinutesMax: 60
      requiredFields:
        - reasonCode
        - ticketId
        - operatorId
        - startTime
        - expiryTime
      audit:
        storeControllerSnapshot: true
        storeKPIs: [ped_actuation_delay_seconds, arrivals_on_green]

observability:
  requiredSignals:
    - controller_phase_state
    - pedestrian_actuations
    - pedestrian_walk_onset
  levels:
    FULL:
      allowOptimizationActions: true
      complianceStatusWhenMissingData: unknown
    PARTIAL:
      allowOptimizationActions: limited
      limitedActions: [plan_switch_only]
      complianceStatusWhenMissingData: unknown
    NONE:
      allowOptimizationActions: false
      fallbackMode: safe_multimodal_recall
      complianceStatusWhenMissingData: unmeasurable
```

### Validation checklist (linting rules)
Automate these checks in CI:
- Schema validation (required fields, types, enums)
- No overlapping time windows without defined precedence
- All rule IDs unique and stable
- All referenced intersections/corridors exist
- Hard constraints are satisfiable (static checks) and map to controller capabilities
- Exception policies include TTL and required audit fields
- Observability requirements exist for every rule that needs sensing

---

## Detailed Implementation Plan

### Phase 1: Build the Policy Contract (Weeks 1–8)
The city should begin by convening the groups that actually own the policy outcomes, including traffic engineering, transit operations, Vision Zero or safety leadership, accessibility/ADA representatives, legal counsel, IT/security, and a community liaison. The group should translate high-level goals into measurable rules that can be tested, such as maximum pedestrian wait in priority zones, mandatory leading pedestrian interval (LPI) with a minimum duration, bounded transit priority rules, and context-specific constraints during school hours or severe weather. The group should explicitly separate non-negotiable rules (hard constraints) from “optimize within limits” objectives (soft weights), because implementers cannot build a defensible system if the rules are ambiguous. The group should also define an exception policy that specifies who can override a rule, for how long, what reason codes are required, and how overrides are reported, because real operations require controlled flexibility.

- **Roles**: policy owner (city delegate), traffic engineering lead (technical translation), transit operations lead (TSP constraints), legal (governance and defensibility), accessibility reviewer (ped rules), data/analytics lead (measurement plan), community liaison (communications).
- **Deliverables**: signed policy contract (versioned), formal rule definitions and thresholds, compliance test plan, and override governance.
- **Risks**: goals may conflict; thresholds may not be measurable with existing sensors; political pressure may change priorities mid-project.
- **Acceptance checks**: each rule has a metric, threshold, and measurement method; hard constraints are clearly identified; override workflow is approved and auditable.

### Phase 2: Measurement Baseline and Data Readiness (Weeks 9–16)
The engineering and analytics teams should establish baseline measurement before enforcing any rules, because without baseline distributions it is impossible to defend whether the project improved or harmed outcomes. The team should identify the sensors and data feeds required for each rule (ped calls and actuation logs, bike detection, transit AVL/TSP request logs, travel times, and controller phase state) and should document gaps that must be addressed in procurement or field work. The team should then stand up performance monitoring with ATSPM-style measures and multimodal metrics so compliance and impacts can be tracked continuously rather than by occasional studies. Finally, the team should publish a baseline report that describes existing compliance rates (where applicable), delay distributions for pedestrians and side streets, and any known hotspots that will be used to evaluate the pilot.

- **Roles**: data engineer (pipelines), analyst (baseline), traffic engineer (metric definitions), IT/security (data access control), operations lead (dashboard needs).
- **Deliverables**: baseline compliance/impact report, data quality assessment, and policy dashboards.
- **Risks**: missing data prevents compliance measurement; weak multimodal coverage creates biased outcomes.
- **Acceptance checks**: baseline exists for each priority rule, and any measurement gap has an explicit mitigation plan and timeline.

### Phase 3: Implement the Rule Engine and Controller Mappings (Weeks 17–28)
The software team should implement a rule representation that is versioned and reviewable (for example, configuration files with approvals) so that changes to city policy are traceable and reversible. The engineering team should build a “policy compiler” layer that maps rule definitions to concrete controller settings and plan behaviors, because most policy is enforced through specific signal timing parameters such as minimum greens, phase sequence restrictions, LPI enablement, and plan switching rules. The team should implement static validation that detects contradictory or impossible rules before deployment, and it should implement runtime monitoring that detects violations or near-violations in the field. The team should also enforce access controls so only authorized users can modify rule sets, and it should log the active rule version and the evidence used for any rule-triggered change.

- **Roles**: software engineer (rule engine), traffic engineer (controller mapping), QA engineer (tests), security engineer (access control), operations lead (override workflow).
- **Deliverables**: rule engine service, rule repository (versioned), compile/mapping layer, unit and integration tests, and audit log schema.
- **Risks**: controller limitations prevent implementing certain rules; rule conflicts produce brittle behavior.
- **Acceptance checks**: rules validate cleanly, a representative set compiles to controller configurations, and tests cover pedestrian and safety invariants.

### Phase 4: Twin Validation and Stakeholder Review (Weeks 29–40)
The modeling team should calibrate a twin for the pilot area and should simulate the rule set under normal days, event surges, and sensor degradation, because real operations include both demand variability and imperfect detection. The analytics team should generate a policy impact statement that quantifies tradeoffs by approach, time-of-day, and mode, because policy-driven control often intentionally shifts delay to meet safety or equity goals. The stakeholder group should review the results and should confirm that observed tradeoffs are acceptable and aligned with the policy contract, and it should adjust thresholds and weights where needed. Finally, the team should define rollout guardrails and rollback criteria that allow operators to revert to a safe baseline plan if compliance monitoring indicates unexpected outcomes.

- **Roles**: modeler (simulation), analyst (impact statement), policy owner (review and sign-off), community liaison (communications).
- **Deliverables**: simulation report, policy impact statement, rollout/rollback plan, and updated rule set if needed.
- **Risks**: stakeholders may reject explicit tradeoffs; the twin may not match rare pedestrian or transit patterns.
- **Acceptance checks**: stakeholder sign-off is documented, rollback criteria are clear, and operator procedures are ready.

### Phase 5: Pilot Activation and Public Reporting (Weeks 41–52)
The operations team should begin in assisted mode, where the system monitors compliance and recommends changes but operators confirm enforcement actions, because this builds trust and surfaces process gaps. The team should then activate rule enforcement by pushing compiled configurations to controllers and validating on-street behavior through spot checks and automated compliance measures. The communications team should publish regular, plain-language reporting that explains what improved and what became slower, because transparency is critical when policy-driven systems intentionally re-balance outcomes. After the pilot, the city should conduct an after-action review and should revise thresholds, weights, and measurement methods based on observed results.

- **Roles**: operations (activation), traffic engineering (on-call adjustments), analyst (reporting), communications lead (public outputs), legal/policy owner (governance).
- **Deliverables**: pilot results report, compliance dashboard, revised policy contract version if needed, and operator runbook.
- **Risks**: public controversy; unintended delay shifts to neighborhoods not represented in planning.
- **Acceptance checks**: hard-rule compliance meets target (for example, >95%), overrides are logged with reasons and expirations, and no safety violations are observed.

### Phase 6: Scale and Governance (Ongoing)
The city should scale policy-driven control by district or corridor templates, because implementing and validating rules requires local phasing and pedestrian timing knowledge that is not uniform across the network. The program should schedule quarterly reviews of compliance and equity distributions and should run an annual audit that verifies rule versions, override patterns, and measurement integrity. The engineering team should maintain a change-management process for “breaking” rule updates and should ensure that rule sets can be rolled back quickly if unintended effects are discovered. Over time, the city should treat the rule registry as part of its transportation governance, similar to how design standards are managed.

- **Roles**: program owner (governance), audit committee (quarterly review), operations (daily monitoring), engineering (maintenance and upgrades).
- **Deliverables**: citywide rule registry, audit reports, expansion roadmap, and change-control procedures.
- **Risks**: policy churn; data quality drift; inconsistent enforcement across controller platforms.
- **Acceptance checks**: audits show stable compliance, changes are versioned and reversible, and equity monitoring is embedded in regular reporting.

---

### Choices
- **Hard-constraint-first**: safety/accessibility non-negotiable.
- **Weighted objectives**: tuneable priorities with guardrails.
- **Mode-based**: context-specific rule bundles (school hours, storms).

## Technical Mechanics

### Key Parameters
- Constraint thresholds (max ped wait, LPI duration, transit delay caps)
- Objective weights (delay vs stops vs emissions vs equity)
- Override limits (duration, scope)

### Guardrails
- Always enforce pedestrian minimums/clearances ([`MUTCD 2009 Part 4, Chapter 4E`](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm)).
- Require reason codes for overrides and plan switches.
- Run periodic compliance audits (ATSPM-style monitoring) ([`FHWA ATSPM Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)).

## MVP Deployment
- One zone + 3–5 rules.
- Assisted mode with operator override.
- Monthly compliance report.

## Evaluation
- Compliance rate per rule (target >95%).
- Ped delay distribution, transit reliability, side-street delay.
- Safety proxies (conflict measures if available).
- Equity distribution (benefits/harms by neighborhood/corridor).

---

## Implementation Checklist
- [ ] Identify policy owners and decision rights (Vision Zero, transit, schools, ADA, equity)
- [ ] Define initial “policy primitives” and measurable proxies (table in Compliance section)
- [ ] Choose arbitration priority order and encode hard vs soft rules
- [ ] Draft policy contract in YAML/JSON, including exceptions + observability + fallbacks
- [ ] Build validator/linter + static conflict detection
- [ ] Map contract primitives to controller capabilities (per intersection)
- [ ] Establish baseline KPIs (ped delay, AoG, split failures where applicable)
- [ ] Implement monitoring + compliance scoring + confidence labels
- [ ] Calibrate twin and run scenario tests (events, demand growth, sensor failure)
- [ ] Stage rollout: shadow → assisted → enforce
- [ ] Publish recurring compliance + equity reports with “unknown/unmeasurable” handling

## Governance & Change-Control Runbook
1. **Propose**: open a change request (CR) referencing the contract version and affected scope
2. **Author**: implement change as a PR to the policy contract repo
3. **Automated checks**: schema lint, satisfiability checks, and unit tests for each rule
4. **Simulation**: run twin scenarios (normal, peak, school, event, degraded sensing)
5. **CCB review**: traffic engineering + signals ops + ADA + equity + transit + IT/security + legal sign-off
6. **Pre-deploy comms**: publish an internal impact note; external summary if appropriate
7. **Deploy**: shadow mode → assisted mode → enforcement (with rollback criteria)
8. **Post-deploy audit**: verify compliance + exceptions + data quality; file issues
9. **Re-baseline**: update baseline dashboards and document “before/after” comparisons

## Reference Links
- [`MUTCD 2009 Part 4, Chapter 4E — Pedestrian Control Features`](https://mutcd.fhwa.dot.gov/htm/2009/part4/part4e.htm)
- [`FHWA Automated Traffic Signal Performance Measures — Use Cases (Ch.4)`](https://ops.fhwa.dot.gov/publications/fhwahop20002/ch4.htm)
- [`FHWA Traffic Signal Timing Manual — Chapter 5`](https://ops.fhwa.dot.gov/publications/fhwahop08024/chapter5.htm)
- [`FHWA Traffic Signal Timing Manual (PDF)`](https://ops.fhwa.dot.gov/publications/fhwahop08024/fhwa_hop_08_024.pdf)

---

## Completion Checklist
- ✅ (1) Enforcement boundary + architecture diagram — see section “Enforcement boundary…”
- ✅ (2) Compliance measurement + mapping table + partial data handling — see section “Compliance measurement…”
- ✅ (3) Arbitration framework + encoding + worked example — see section “Policy conflict arbitration…”
- ✅ (4) Equity operationalization in contract — see section “Equity operationalization…”
- ✅ (5) Degradation handling + observability levels + non-performative compliance — see section “Degradation handling…”
- ✅ (6) Lifecycle governance (versioning, CCB, transparency, cadence) — see section “Lifecycle governance…”
- ✅ (7) Concrete policy contract schema + validation checklist — see section “Concrete Policy Contract schema…”

---

Cross-links: Related ideas include safety-first signals, privacy-friendly learning, and what-if button.
