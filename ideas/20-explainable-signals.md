# 20) Explainable Signals (Auditable Reasons for Timing Changes)

## Catchy Explanation
Like a referee explaining a call, an explainable signal doesn’t just change timing—it records a short, auditable “why” with evidence.

## What it is (precise)
**Explainable signals** attach an auditable explanation to each non-trivial timing decision (plan switch, major split/offset change, mode transition, priority action). Explanations are not free-form storytelling; they are structured **reason codes + evidence** generated at decision time:

- reason code taxonomy (controlled vocabulary)
- key inputs (sensor/health signals, thresholds)
- predicted deltas (twin or model outputs)
- constraints that were binding (e.g., ped mins)
- operator overrides and notes

This supports accountability, faster diagnosis, and governance aligned with risk management frameworks (for example, the [NIST AI Risk Management Framework](https://doi.org/10.6028/NIST.AI.100-1:1)).

## Benefits
- **Trust and accountability**: operators and auditors can see why changes happened.
- **Faster incident diagnosis**: root cause analysis becomes searchable.
- **Safer automation**: discourages “mystery optimization” and supports rollback.
- **Compliance support**: easier to document decision processes and control evidence.

## Challenges
- **Information overload**: explanations must be brief by default.
- **Post-hoc risk**: explanations must be generated with immutable evidence at decision time.
- **Standardization**: codes should be consistent across vendors and corridors.
- **Privacy**: evidence must avoid exposing personal data.

---

## Implementation Strategies

### Infrastructure Needs
- **Explainability contract**: which actions require explanations and at what level of detail.
- **Reason code taxonomy**: controlled vocabulary (e.g., 100 safety, 200 efficiency, 300 faults).
- **Append-only decision log**: timestamps, inputs, action, constraints, reason, evidence.
- **Dashboards**: “why did this change at 17:20?” query UI.
- **Twin-backed counterfactuals** (optional): “if we did nothing, spillback risk rises to 60%.”

### Detailed Implementation Plan

#### Phase 1: Define the Explainability Contract and Taxonomy (Weeks 1–4)
The agency should begin by defining which actions require explanations (for example, any plan switch, any split change above a threshold, any mode transition, and any priority action), because explainability must be scoped to be operationally feasible. The team should define a minimum evidence schema that can be generated automatically at decision time, including timestamps, location, current mode, triggering signals, thresholds crossed, and binding constraints, because post-hoc explanations are not defensible. The team should define a controlled reason code taxonomy that is small enough to be usable (for example, 10–50 codes) and should include categories for safety, efficiency, faults, policy compliance, and operator override.

- **Roles**: program owner (scope and governance), operations (usability), traffic engineering (action definitions), data/engineering (schema), privacy/security (evidence constraints).
- **Deliverables**: explainability contract, reason code taxonomy v0, evidence schema, and retention policy.
- **Risks**: taxonomy becomes too large; evidence schema requires data that is not available.
- **Acceptance checks**: contract is signed off and every required evidence field has a defined data source.

#### Phase 2: Implement Append-Only Logging and the Operator “Timeline” UI (Weeks 5–10)
The engineering team should implement an append-only decision log that captures every explained action in a tamper-resistant way (for example, write-once storage or integrity checks), because auditability depends on evidence integrity. The team should build an operator-facing timeline UI that allows searching by intersection, corridor, time range, and reason code, because the primary operational value is being able to answer “what changed and why” quickly. The team should integrate log entries with operator notes and overrides so that human decisions are recorded in the same system.

- **Roles**: software engineer (logging service), data engineer (storage), operations (UI requirements), security (integrity controls), QA (testing).
- **Deliverables**: operational logging pipeline, timeline UI, and access control rules.
- **Risks**: logs become noisy without good filtering; weak access controls expose sensitive operational data.
- **Acceptance checks**: >95% of relevant actions appear in the log with valid schema and the UI supports rapid queries.

#### Phase 3: Evidence Quality, Counterfactuals, and Integration with Decision Systems (Weeks 11–18)
The team should enrich explanations with predicted KPI deltas and binding constraints whenever a decision is driven by a model or a twin, because operators need to see what the system thought would happen. If a digital twin is available, the team can add counterfactual summaries such as “if no change, spillback probability increases,” but it should keep counterfactuals optional and clearly labeled with uncertainty. The team should integrate explainability into each decision-producing subsystem (what-if tools, fast-forward twin, self-healing mode transitions, priority credits) so explanations are consistent rather than implemented as an afterthought.

- **Roles**: modeling team (predicted deltas), software engineer (integration), operations (usability), privacy reviewer (evidence minimization).
- **Deliverables**: evidence-enriched explanation records, counterfactual integration (optional), and updated reason code usage guidance.
- **Risks**: predicted deltas may be misleading if state estimation is weak; counterfactuals can increase compute and complexity.
- **Acceptance checks**: predicted deltas are present for model-driven actions and the system clearly indicates confidence/uncertainty.

#### Phase 4: Governance, Audits, and Continuous Improvement (Ongoing)
The agency should establish a governance process where operators and engineers review samples of decision logs weekly and run completeness audits monthly, because explainability must remain reliable as systems evolve. The team should track override patterns and reason code distributions to identify common failure modes and training needs, and it should refine taxonomies and evidence schema carefully through versioned changes. The agency should also publish internal (and optionally public) summaries that demonstrate accountability without exposing sensitive details.

- **Roles**: audit owner (governance), operations (weekly sampling), engineering (schema maintenance), analyst (trend reporting), privacy/security (review).
- **Deliverables**: audit reports, taxonomy updates (versioned), and periodic governance summaries.
- **Risks**: audits become checkbox exercises; taxonomy drift reduces comparability across time.
- **Acceptance checks**: audit completeness stays above target and operators report reduced time-to-diagnosis for incidents.

### Choices
- **Reason codes only**: minimal overhead.
- **Reason + evidence**: recommended baseline.
- **Narratives + counterfactuals**: highest value but more work.

---

## Canonical Evidence Schema & Portability Strategy

This section turns explainable signals into a **city-defined, vendor-adaptable contract**. The city owns the canonical schema and reason code taxonomy; each vendor maps its internal events into that schema. This mirrors how security and audit standards encourage well-defined log formats with local extensions (e.g., [NIST SP 800-92, Guide to Computer Security Log Management](https://csrc.nist.gov/publications/detail/sp/800-92/final:1)).

### City-Owned Canonical Model

**Principles**
- **City owns the schema**: the canonical evidence schema, reason codes, and retention tiers are defined by the city, not a vendor.
- **Adapters at the edge**: each controller/ATMS/vendor adapter translates native events into the canonical schema.
- **Stable over time**: the canonical schema evolves slowly via versioned changes (e.g., v1, v1.1), with deprecation periods.
- **Extensible**: vendors may add namespaced extension fields, but cannot remove or repurpose required fields.

**Core objects** (conceptual)
- **Decision**: one timing decision (e.g., plan switch, split change, offset adjustment, priority grant).
- **Episode**: a rollup over multiple micro-decisions (see "Logging Strategy" below).
- **EvidenceSnapshot**: measured and derived state at decision time.
- **ConstraintSet**: policy and safety constraints evaluated.
- **UncertaintyBlock**: confidence and known limitations for this decision.

### Example Canonical Evidence Schema (JSON)

Below is an implementation-oriented JSON example for a **single decision event**. Required fields are marked in comments; optional fields are allowed to be null or omitted.

```json
{
  "schema_version": "explainable-signals.v1",           // required
  "decision_id": "uuid-1234-...",                       // required
  "episode_id": "uuid-episode-5678",                    // optional (for grouping)
  "timestamp_utc": "2026-01-28T08:17:32Z",              // required
  "source_system": "atms-x" ,                           // required (ATMS, controller, twin, etc.)

  "location": {                                           // required
    "jurisdiction_id": "city-123",                      // required
    "corridor_id": "corridor-7",                        // optional
    "intersection_id": "int-045",                       // required
    "approach_id": "NB",                                // optional
    "lane_group_id": "NB-through"                       // optional
  },

  "action": {                                             // required
    "action_type": "plan_switch",                       // required (e.g., plan_switch, split_change)
    "previous_state": {
      "plan_id": "plan-3",
      "cycle_length_s": 90,
      "offset_s": 20,
      "splits": {"phase2": 30, "phase6": 30, "ped": 30}
    },
    "new_state": {
      "plan_id": "plan-4",
      "cycle_length_s": 110,
      "offset_s": 35,
      "splits": {"phase2": 40, "phase6": 40, "ped": 30}
    },
    "scope": "intersection",                            // required (intersection/corridor/network)
    "intended_duration_s": 1800,                         // optional (how long we expect to keep this change)
    "rollback_hint": {                                   // optional
      "rollback_possible": true,
      "rollback_to_decision_id": "uuid-previous"        // optional
    }
  },

  "trigger_context": {                                   // required
    "mode": "adaptive",                                // required (coordinated, free, adaptive, manual, etc.)
    "trigger_type": "volume_threshold_crossed",        // required (rule, operator_override, twin_scenario, etc.)
    "reason_code": "210",                              // required (see taxonomy table)
    "reason_code_label": "Through-volume congestion management", // required (human-readable)
    "incident_flags": ["queue_spillback_risk"],        // optional
    "priority_entities": ["bus_4213"],                 // optional
    "operator_id": null,                                // optional (null when fully automatic)
    "operator_note": null                               // optional free-text
  },

  "evidence": {                                          // required (block present; fields can be partial)
    "data_timestamp_utc": "2026-01-28T08:17:28Z",      // required (freshness anchor)
    "detector_health": {                                 // optional but recommended
      "nb_loop_1": {"status": "ok", "staleness_s": 2},
      "nb_loop_2": {"status": "suspect", "staleness_s": 45}
    },
    "measured_metrics": {                               // required high-level metrics
      "queue_length_m": {"p95": 120, "max": 150},
      "approach_speed_kph": {"mean": 15},
      "volume_veh_per_hr": 900,
      "occupancy_pct": 85
    },
    "aog_state": {                                      // optional (Available Green / performance index)
      "aog_score": 0.62,
      "aog_reference": "v2.3"
    },
    "data_quality_flags": ["nb_loop_2_suspect"],
    "model_inputs_reference": "s3://.../inputs/uuid-1234" // optional (pointer, not raw data)
  },

  "constraints_evaluated": {                             // required
    "binding_constraints": [
      {
        "constraint_id": "ped_min_green",
        "constraint_type": "safety",
        "description": "Minimum ped walk + clearance",
        "binding": true
      },
      {
        "constraint_id": "max_cycle_length",
        "constraint_type": "policy",
        "description": "Max cycle length 120s",
        "binding": false
      }
    ],
    "violations": []                                     // required (empty array if none)
  },

  "uncertainty": {                                      // required block
    "data_freshness_s": 4,                              // required
    "sensor_health_overall": "degraded",              // required (ok/degraded/bad/unknown)
    "model_confidence_level": "medium",               // required when model-driven
    "model_error_bounds": {                             // optional, recommended
      "metric": "queue_length_m",
      "lower_p10": 90,
      "upper_p90": 180
    },
    "calibration_status": "calibrated_2025q4",        // optional tag
    "known_limitations": [                              // optional but recommended
      "No probe data for side streets after 22:00",
      "Detector nb_loop_2 marked suspect"
    ]
  },

  "predictions_and_deltas": {                           // optional but recommended
    "kpi": {
      "delay_s_per_veh": {"before": 80, "after": 55},
      "queue_length_m": {"before": 150, "after": 100},
      "bus_travel_time_s": {"before": 600, "after": 540}
    },
    "counterfactual_baseline": "no_change",           // optional
    "counterfactual_deltas": {                          // optional
      "queue_length_m": {"baseline_after": 180},
      "spillback_risk_pct": {"baseline_after": 70, "chosen_after": 40}
    },
    "counterfactual_caveats": [                         // optional
      "Twin calibrated only for typical weekday peak.",
      "Weather impacts not modeled."
    ]
  },

  "security_metadata": {                                // required
    "created_by": "service:signal-decision-engine",    // required
    "created_at_utc": "2026-01-28T08:17:33Z",          // required
    "write_tenant": "city-123",                        // required
    "hash_chain_prev": "base64-prev-hash",             // optional for tamper-evidence
    "storage_class": "WORM-7y"                         // required (see retention tiers)
  }
}
```

### Canonical schema in YAML (compact example)

A compact YAML view is useful in RFPs and technical appendices:

```yaml
schema_version: explainable-signals.v1   # required
id: uuid-1234-...                        # required (alias of decision_id)
timestamp_utc: 2026-01-28T08:17:32Z      # required
source_system: atms-x                    # required

location:                                # Tier 1 minimum
  jurisdiction_id: city-123
  intersection_id: int-045

action:
  action_type: plan_switch
  scope: intersection

trigger_context:
  mode: adaptive
  trigger_type: volume_threshold_crossed
  reason_code: "210"

uncertainty:
  data_freshness_s: 4
  sensor_health_overall: degraded

security_metadata:
  created_by: service:signal-decision-engine
  storage_class: WORM-7y
```

### Minimum vs Optional Fields (Tiered)

Define **tiers** so you can deploy incrementally and so vendors know what is mandatory.

- **Tier 0 – Identifier Baseline (must be 100% complete)**
  - `schema_version`, `decision_id`, `timestamp_utc`, `source_system`
  - `location.intersection_id`, `action.action_type`, `trigger_context.reason_code`
  - `security_metadata.created_by`, `security_metadata.created_at_utc`

- **Tier 1 – Accountability Minimum (recommended for any automated or safety-relevant action)**
  - Tier 0 fields, plus:
  - `location.jurisdiction_id`, `location.scope`
  - `trigger_context.mode`, `trigger_context.trigger_type`
  - `evidence.data_timestamp_utc`, `evidence.measured_metrics` (at least volume + delay/queue proxy)
  - `constraints_evaluated.binding_constraints` (at least IDs + type)
  - `uncertainty.data_freshness_s`, `uncertainty.sensor_health_overall`
  - Storage class tagged for at least **5–7 years** retention for safety-related actions (aligned with typical public-sector audit guidance on incident logs, e.g. [NIST SP 800-92](https://csrc.nist.gov/publications/detail/sp/800-92/final:1)).

- **Tier 2 – Model-Driven & Counterfactual (for optimization / AI use cases)**
  - Tier 1, plus when a model or twin is used:
  - `predictions_and_deltas.kpi` (before/after for key KPIs such as delay, queues, transit travel time)
  - `model_confidence_level`, `model_error_bounds` when available
  - `counterfactual_baseline` and `counterfactual_caveats` when counterfactuals are shown to operators or the public (to avoid over-claiming causality, consistent with risk management guidance in the [NIST AI RMF](https://doi.org/10.6028/NIST.AI.100-1:1)).

- **Tier 3 – Deep Forensics (optional, often retained shorter)**
  - Pointers to richer artifacts (e.g., `model_inputs_reference`, anonymized raw trace data, scenario files) stored in a separate, access-controlled bucket.
  - Typically shorter retention (e.g., 90–365 days) unless used in active investigations.

### Portability and Vendor Adapters

To keep the system vendor-neutral and portable:

- **Canonical reason code list**: maintained by the city with numeric codes and labels. Vendors map their own codes onto this list.
- **Mapping contracts in procurement**: RFPs and contracts must require vendors to:
  - emit all required Tier 1 fields,
  - implement a mapping from native events to the canonical schema,
  - document any vendor-specific extensions under a namespaced key (e.g., `vendor_x_ext`).
- **Schema registry**: publish the canonical schema (JSON/YAML) in a city Git repository or data catalog, with machine-readable versions and changelog.
- **Compatibility testing**: test suites validate that vendor events are:
  - schema valid;
  - complete for Tier 1 fields;
  - correctly mapped for reason codes and constraint IDs.

---

## Logging Strategy for High-Frequency Micro-Adjustments

Modern adaptive systems can generate thousands of sub-second or per-cycle micro-adjustments. Log everything naively and operators drown; log too little and audits fail. Public-sector logging guidance (e.g., [NIST SP 800-92](https://csrc.nist.gov/publications/detail/sp/800-92/final:1) and control families such as AU-* in [NIST SP 800-53](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final:1)) emphasize **risk-based selection, aggregation, and retention** of logs. This section applies those ideas to signal timing.

### Event-Based vs Sampled vs Aggregated Logging

**1. Event-based logging (log every decision)**
- Appropriate for:
  - low-frequency, high-impact actions (plan switches, mode transitions, priority grants, manual overrides);
  - events tied to safety/policy constraints (e.g., pedestrian service, school zone changes).
- All such events must produce **Tier 1** records.

**2. Sampled logging (log every Nth micro-adjustment)**
- For very high-frequency internal updates (e.g., minor offset nudges, internal controller states), log:
  - every *k*th event (e.g., every 10th cycle), or
  - at most once per time window per movement (e.g., 1/30s per phase) when conditions are stable.
- Ensure that sampling is:
  - **documented and deterministic**, 
  - visible in metadata (e.g., `sampling_strategy: every_10th_decision`).

**3. Aggregated logging (episodes and rollups)**
- Group micro-adjustments into **decision episodes**:
  - an episode is a cluster of related decisions over a window (e.g., 5–15 minutes) where the dominant reason code and context are stable.
- Log **per-episode rollups** instead of each micro-step, including:
  - episode start/end times;
  - count of micro-decisions;
  - dominant reason codes;
  - summary metrics (mean/max queue length, delay, priority granted);
  - counts of oscillations, overrides, and constraint bindings.

### Decision Episode Schema (Rollup Example)

```json
{
  "schema_version": "explainable-signals.episode.v1",
  "episode_id": "uuid-episode-5678",
  "location": {
    "intersection_id": "int-045",
    "corridor_id": "corridor-7"
  },
  "time_window": {
    "start_utc": "2026-01-28T08:15:00Z",
    "end_utc": "2026-01-28T08:20:00Z"
  },
  "decision_counts": {
    "total_decisions": 120,
    "plan_switches": 1,
    "split_tweaks": 100,
    "offset_tweaks": 19
  },
  "dominant_reason_codes": ["210", "230"],
  "binding_constraints_counts": {
    "ped_min_green": 5,
    "queue_spillback_protection": 18
  },
  "oscillation_indicators": {
    "is_oscillating": true,
    "phase2_split_range_s": [25, 45],
    "sign_changes_in_offset": 6
  },
  "priority_summary": {
    "transit_priority_requests": 4,
    "transit_priority_granted": 3,
    "emergency_preemptions": 0
  },
  "performance_summary": {
    "queue_length_m_p95": 130,
    "delay_s_per_veh_p95": 90
  },
  "uncertainty": {
    "sensor_health_overall": "ok",
    "data_freshness_p95_s": 5
  }
}
```

### Alerting and Triage Design

To avoid alarm fatigue, alerts should focus on **anomalies and constraint-related issues** rather than every micro-change. This aligns with general security logging guidance that emphasizes high-value events and anomaly detection over volume ([NIST SP 800-92](https://csrc.nist.gov/publications/detail/sp/800-92/final:1)).

Alert categories:
- **Integrity / configuration anomalies**
  - decisions without explanations (missing Tier 1 fields),
  - sudden changes in decision volume or reason code mix, 
  - repeated overrides of the same automated recommendation.

- **Constraint bindings and violations**
  - frequent binding of safety constraints (e.g., ped minimums, red-clearance limits),
  - any recorded violation of a binding constraint (should be rare, with clear operator justification).

- **Oscillation and instability**
  - repeated up/down adjustments of the same parameter within a short episode,
  - toggling between two plans or strategies within a short timeframe.

- **Sensor and data quality issues**
  - corridors with sustained “degraded” or “bad” sensor health,
  - episodes where model confidence is persistently low.

### Recommended Defaults (Initial Policy)

- **Always event-log (Tier 1+)**
  - plan switches, mode transitions, pedestrian timing changes, school zone changes;
  - any preemption/priority for emergency or transit;
  - any manual override affecting more than one phase or plan.

- **Episode-based rollups**
  - micro-adjustments within a stable plan (split/offset tweaks) are rolled into **5-minute episodes** by default.
  - exposure in UI: show episodes, with drill-down to sample micro-decisions.

- **Sampling thresholds for micro-adjustments**
  - if more than **N = 200** decisions per 5 minutes at an intersection, sample micro-decisions at **1 in 10** for detailed logs and rely on episode rollups for the rest.
  - log sampling strategy in the episode metadata.

- **Escalation rules**
  - automatically raise an incident in the operator UI when:
    - constraint violations > 0 in any episode,
    - oscillation flag is true for more than **3 consecutive episodes**,
    - sensor health = `bad` for more than **15 minutes** while automation is active.

---

## Required visual: Decision → Log → Review pipeline

### Diagram: “Decision → reason code → evidence capture → log store → dashboards → review/export” (with integrity controls)

```text
   [ FIELD & ENGINES ]
   (controllers, ATMS,
    twins, priority logic)
           |
           | 1) Decision made
           v
   [ DECISION ENGINE ]
   - chooses action
   - evaluates constraints
           |
           | 2) Attach explanation
           v
   [ REASON CODE +
     EVIDENCE CAPTURE ]
   - apply canonical
     taxonomy
   - assemble
     evidence +
     uncertainty
   - assign severity
           |
           | 3) Serialize record
           v
   [ LOG PIPELINE ]
   - validate schema
   - enrich with hash
     (prev → current)
   - append-only write
           |
           | 4) Store
           v
   [ LOG STORE ]
   - WORM / immutable
     class for Tier A
   - hash-chain index
   - RBAC on access
           |
   +-------+-----------------------------+
   |       |                             |
   |       | 5a) Operator views          | 5b) Audit / export
   v       v                             v
[ DASHBOARDS / UI ]             [ EXPORT & RECORDS ]
- 10s summary                   - controlled export
- triage views                    interface
- drill-down into               - redaction templates
  decision / episode            - export hashes stored

Integrity controls shown:
- schema validation at ingestion.
- hash chaining for batches.
- WORM/immutable storage for critical tiers.
- RBAC + access logging for UI and exports.
```

---

## Reason Code Taxonomy & Required-Evidence Table (required visual)

A **city-owned taxonomy** links each high-level reason to a minimum evidence set, severity, operator action, and retention tier. Codes are grouped (100x safety, 200x efficiency, 300x faults, 400x policy/compliance, 500x operator/manual, etc.).

### Example reason-code slices

| Reason code | Example label | Category |
|------------:|---------------|----------|
| 110 | Pedestrian safety / turning conflict mitigation | Safety |
| 120 | School zone timing change | Safety/Policy |
| 210 | Through-volume congestion management | Efficiency |
| 230 | Anti-jam spillback protection | Efficiency/Safety |
| 310 | Sensor fault / degraded data fallback | Fault |
| 410 | Policy override (e.g., emergency route protection) | Policy |
| 510 | Operator manual override (pre-planned) | Human |

### Required visual: reason code → evidence → severity → action → retention tier

| Reason code (class) | Required evidence fields (minimum) | Severity (default) | Default operator action / SOP hook | Retention tier (suggested) |
|---|---|---|---|---|
| **110 – Pedestrian safety / turning conflict mitigation** | Tier 1 fields **plus**: ped volumes; recent conflict/safety indicators (if available); list of binding safety constraints (e.g., ped_min_green, max_walk_gap); note if school zone or vulnerable-user flag active | High | Verify no constraint violations; if frequent, escalate to engineering for geometry/plan review; include in safety-focused weekly review | Tier A (7–10 years) |
| **120 – School zone timing change** | All Tier 1 fields; school schedule reference or flag; affected approaches; applicable policy ID | High | Confirm timing change aligns with schedule; ensure any manual changes documented; highlight in daily summary during school term | Tier A (7–10 years) |
| **210 – Through-volume congestion management** | Tier 1 + basic queues/volumes; note of any binding efficiency constraints; if model-driven, include delay/queue `predictions_and_deltas` | Medium | Monitor for oscillation or excessive delay to side streets; if recurring issues, tune thresholds or plans; include in corridor KPI review | Tier B (3–5 years) |
| **230 – Anti-jam spillback protection** | Evidence of queue lengths near boundaries; binding constraint `queue_spillback_protection`; any affected ramps/critical links; uncertainty fields populated | High | Follow anti-jam SOP (see [`ideas/15-anti-jam-nudges.md`](ideas/15-anti-jam-nudges.md:1)); monitor adjacent links for secondary impacts; include in safety/network review | Tier A (7–10 years) |
| **310 – Sensor fault / degraded data fallback** | `uncertainty.sensor_health_overall` = degraded/bad; `data_quality_flags`; description of fallback plan; maintenance ticket/link if available | Medium | Execute sensor-fault SOP: downgrade automation, open maintenance ticket, document duration; include in ops weekly review | Tier B (3–5 years) for decision, Tier C (≤1 year) for deep traces |
| **410 – Policy override / emergency routing** | ID of policy or emergency plan invoked; affected network segments; operator ID and note; any deviations from standard plan | High/Critical | Notify supervisor; document reason and duration; ensure logs are complete for incident reconstruction; may trigger dedicated incident report | Tier A (7–10 years) |
| **510 – Manual operator override (pre-planned)** | Operator ID; override scope; original recommendation vs chosen action; brief note; start/end timestamps | Medium | Review override frequency in weekly governance; if recurring at same site/reason, re-evaluate automation logic or policies | Tier B (3–5 years) |

Cities can extend this table with local codes but should keep the **columns and required-evidence discipline**.

---

## Security & Integrity: Tamper-Evident, Access-Controlled Logs

Explainability logs are sensitive: they describe operational behavior, model decisions, and sometimes system weaknesses. Security guidance for audit logs (e.g., [NIST SP 800-92](https://csrc.nist.gov/publications/detail/sp/800-92/final:1) and AU-* controls in [NIST SP 800-53](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final:1)) highlight **integrity, availability, confidentiality, and auditability of access** as key properties.

### Threat Model for Explainability Logs

Consider at least the following threats:
- **Tampering**: unauthorized modification of past log entries (e.g., to hide a problematic decision).
- **Deletion**: partial or wholesale removal of logs (e.g., to erase an incident trail).
- **Unauthorized access**: staff or third parties viewing sensitive operational details.
- **Sensitive information leakage**: logs unintentionally reveal security posture (e.g., fault states, timing thresholds) or personal data.

### Tamper-Evident Storage Patterns (Municipal-Friendly)

You do not need blockchain to achieve strong integrity. Use patterns that are compatible with municipal IT operations and general audit standards:

1. **Append-only storage**
   - Use log stores or object storage with append-only semantics.
   - Disallow in-place updates to log records.
   - Apply strict controls around log rotation and archival.

2. **Hash chaining and integrity records**
   - For each decision log batch, compute a **cryptographic hash** over the ordered entries.
   - Store `hash_chain_prev` and `hash_chain_current` in a separate integrity index.
   - Periodically anchor the latest hash in a separate system (e.g., write to a separate WORM bucket or an external notarization service).

3. **WORM (Write Once, Read Many) storage**
   - Use storage tiers that support immutability / retention locks for compliance logs, conceptually equivalent to WORM storage recommended for regulated audit logs in sectors like finance and justice.
   - Apply this at least to:
     - Tier 1 decision logs related to safety,
     - integrity indices (hash chains, verification results).

4. **Redundant copies and off-site archival**
   - Maintain at least one off-site or cross-region copy of critical logs.
   - Protect against ransomware and local compromise by separating credentials.

### Access Control and Audit of Access

General public-sector guidance on access control (RBAC, least privilege, separation of duties) applies directly here (see AC-* and AU-* controls in [NIST SP 800-53](https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final:1)).

- **Role-based access control (RBAC)**
  - Define roles like: Operator, Supervisor, Engineer, Security Auditor, External Auditor.
  - Associate clear permissions: view-only, annotate, export, administer.

- **Least privilege and separation of duties**
  - Operators who can change timing should not be able to delete or alter logs.
  - Log administrators should not be able to both **write** and **erase** integrity indices.

- **Access logging**
  - Every access to sensitive logs (search, view details, export) should itself be logged with:
    - user ID / role,
    - time,
    - scope (corridor, time range),
    - purpose/category (operations, audit, legal).

- **Export controls**
  - Require an approval workflow (e.g., supervisor or records officer) for exports beyond a defined scope (time range, corridor size).
  - Store a copy of the exact export set plus a hash, so you can later prove what was disclosed.

### Key Management & Operational Considerations

- **Key management**
  - Use centrally managed keys (HSM or KMS where available) for encrypting logs at rest.
  - Restrict key management operations to security/IT, not to vendors.

- **Service accounts and rotation**
  - Use dedicated service accounts for logging pipelines.
  - Rotate credentials and keys on a regular schedule (e.g., 90 days) and after personnel changes.

### Log Integrity Checks and Verification Cadence

Following audit-log best practices, establish a **regular verification cadence**:
- **Daily**: verify hash chains for new log batches; alert on gaps or mismatches.
- **Weekly**: spot-check log entries for completeness (all required Tier 1 fields populated).
- **Quarterly**: end-to-end drill where you:
  - reconstruct a known incident timeline from logs only;
  - verify archival and restoration from off-site copies.

Record verification outcomes as dedicated log entries in an **integrity audit stream**.

---

## Uncertainty & Causal Validity of Counterfactual Explanations

Explainable signals should be honest about what they know and what they do not. The [NIST AI RMF](https://doi.org/10.6028/NIST.AI.100-1:1) and interpretability guidance emphasize avoiding overstatement of model capabilities and being explicit about limitations. Counterfactuals (“if we had done X instead…”) are especially prone to over-claiming causality.

### Expressing Confidence in Explanations

For each decision (or episode), expose uncertainty fields that can be consumed both by the UI and by auditors.

**Key concepts:**
- **Data freshness**: how old are the inputs?
- **Sensor health**: how trustworthy are detectors and feeds?
- **Model confidence**: how stable is the model prediction in the current regime?
- **Error bounds**: how wrong could we be, based on past calibration?

These are already represented in the canonical schema (`uncertainty` block). Operational rules:
- If `sensor_health_overall` is `degraded` or `bad`, UI should clearly warn operators.
- If `data_freshness_s` exceeds a policy threshold (e.g., 30 s for key detectors), mark explainability as **low confidence**.
- When `model_confidence_level` is low/unknown, do not show precise percentage deltas; show ranges (“delay may improve modestly”) or a neutral statement.

### Counterfactual / Twin-Backed Deltas

When a twin or model generates counterfactuals:
- **Label clearly**: “Model estimate – not observed outcome.”
- **Show ranges, not just point estimates**: show p10–p90 or min–max.
- **Attach caveats**: populate `counterfactual_caveats` with calibration limits, assumptions (e.g., typical weekday, fair weather, no incidents).
- **Distinguish prediction vs attribution**:
  - Prediction: “Model expected queue length to drop from 150 m to ~100 m.”
  - Attribution: “We changed plan because the model predicted spillback risk > 60% under no-change.”

### Required Fields for Honest Explanations

For any decision where model/twin output is used to justify a change, require at least:
- `predictions_and_deltas.kpi` for 1–3 core KPIs;
- `uncertainty.model_confidence_level` (e.g., low/medium/high/unknown);
- `uncertainty.known_limitations` listing at least the most relevant limitation (e.g., “no weekend calibration”);
- `predictions_and_deltas.counterfactual_caveats` when counterfactuals are displayed to operators or public.

### UI Patterns for Uncertainty & "Why / Why Not" Explanations

Operator UX should:
- **Show a 10-second summary**
  - One-line “why” + short tag for confidence, e.g.:
    - "Raised cycle length due to recurring queues on NB; medium confidence (degraded detector)."
- **Use confidence badges**
  - Visual badges (e.g., green/amber/red) tied to `sensor_health_overall` and `model_confidence_level`.
- **Provide a “why not” view**
  - Explicitly list critical alternatives that were evaluated and rejected, for example:
    - "Why not keep current plan? → Model predicted spillback risk > 70%."
    - "Why not give bus priority? → No active request in last 90 s."
- **Show “model unreliable” states**
  - When inputs or model confidence fail thresholds, the UI should:
    - label the decision as **rule-based** or **fallback** when applicable;
    - hide or downplay model-twin deltas;
    - prompt operators if automation should be downgraded.

---

## Operator Cognitive Load: UI Patterns and SOP for Triage

Traffic operations centers are already noisy environments. Logging and explanations must **reduce**, not increase, cognitive load. Principles in public-sector alert management (e.g., security operations centers) stress prioritization, grouping, and clear runbooks rather than raw feeds of events.

### Explanation UX Principles

- **10-second summary first, drill-down later**
  - For each intersection/corridor, show a compact “What changed / Why / Confidence” banner.
  - Use plain language tied to reason code labels.

- **Exception-first dashboards**
  - Default views focus on:
    - current incidents,
    - corridors with oscillations or constraint issues,
    - automation in low-confidence state.

- **Severity levels and grouping**
  - Group related events into incidents or episodes.
  - Map severity (e.g., informational / warning / critical) based on:
    - safety impact,
    - network impact,
    - duration.
  - Collapse low-severity recurring events into a single line with counts.

- **Explainable filters**
  - Operators can filter by reason code categories (safety, efficiency, faults), corridors, time ranges.
  - Provide quick filters for “only safety-relevant decisions” or “only human overrides.”

### SOP: Triage Based on Explanations

Use structured **Standard Operating Procedures (SOP)** so operators respond consistently.

**1. Sensor failure or degraded data**
- Trigger: explanations show `sensor_health_overall = degraded/bad` or frequent `data_quality_flags`.
- Actions:
  1. Acknowledge the issue in the UI and assign to on-shift operator.
  2. Verify via alternate data if available (CCTV, probes, neighboring detectors).
  3. If data cannot be trusted, downgrade automation mode for affected movements (e.g., revert to time-of-day plan) and log this as an explicit decision with reason code “sensor fault fallback.”
  4. Create a maintenance ticket referencing the decision/episode IDs.

**2. Policy/safety constraint binding**
- Trigger: `binding_constraints` heavily populated with safety or legal constraints.
- Actions:
  1. Confirm that binding constraints are **not violated** (no entries in `violations`).
  2. If policy is binding so often that efficiency collapses (e.g., permanent school zone), flag to engineering/policy staff for review.
  3. Note in daily summary for potential policy changes.

**3. Oscillation / unstable control**
- Trigger: episodes flagged `is_oscillating = true` for multiple consecutive periods.
- Actions:
  1. Inspect root causes (reason codes, constraints, sensor health).
  2. Temporarily pin the plan or adjust thresholds (if allowed) to break oscillation.
  3. File an engineering follow-up for retuning, referencing affected episode IDs.

**4. High-impact human overrides**
- Trigger: repeated operator-triggered changes with similar reason codes.
- Actions:
  1. Confirm that override explanations include adequate notes.
  2. Group overrides in daily summary; if recurring, review underlying automation logic.

### Training and Shift-Handover Artifacts

- **Daily summary report**
  - Top N corridors by:
    - safety-relevant decisions,
    - oscillation indicators,
    - sensor health issues,
    - operator overrides.
  - Generated from decision/episode logs.

- **Incident timeline views**
  - For major incidents, reconstruct a timeline:
    - key decisions and reason codes,
    - counterfactuals shown (if any),
    - overrides and constraint bindings.

- **Playbooks and micro-trainings**
  - Use real episodes as training examples during onboarding.
  - Provide mini playbooks embedded in the UI: “What to do if you see X pattern.”

---

## Public Transparency Boundary & Records Retention Policy

Public-sector guidance on records (e.g., state/local records retention schedules and open records laws) typically require **retention by record type** and **disclosure on request**, with redaction of sensitive details. Agencies should align explainable signals with these practices.

### Internal vs Public Views

- **Internal detail**
  - Full canonical records, including:
    - detailed evidence metrics,
    - hash-chain metadata,
    - internal IDs and configuration details.

- **Public-facing summaries**
  - Share **high-level explanations and aggregates**, such as:
    - why plans were changed on a corridor over a day,
    - high-level reason code distributions (e.g., % decisions for safety vs efficiency),
    - summaries of automation vs human overrides.
  - Avoid exposing:
    - exact timing thresholds that could be abused (e.g., signal priority triggers);
    - precise detector layouts and failure patterns;
    - personally identifiable data (probe IDs, vehicle plates, etc.).

### Records Retention and Tiers

State and municipal records schedules commonly distinguish **short-term operational logs** from **long-term policy and incident records** (for example, traffic camera recordings vs incident reports). Apply similar tiers:

- **Tier A – Critical decision logs (safety/incident relevant)**
  - Include: Tier 1 decision records for safety-related actions, major incidents, manual overrides.
  - Recommended retention: **7–10 years**, aligning with long-term audit needs and some state schedules for safety-critical operational records.

- **Tier B – Routine operational episodes**
  - Include: episode rollups, non-safety optimization decisions, performance summaries.
  - Recommended retention: **3–5 years**, sufficient for trend analysis and audits.

- **Tier C – Deep forensics / raw traces**
  - Include: raw detector traces, high-frequency model I/O snapshots.
  - Recommended retention: **90 days – 1 year**, unless connected to an incident or legal hold.

Document these tiers in your official records schedule and cross-reference with overarching guidance (e.g., state general records schedules and [NIST SP 800-92](https://csrc.nist.gov/publications/detail/sp/800-92/final:1) for log lifecycle management).

### Public Records Request (FOIA / Access to Information) Process

- **Standard operating model**
  1. Requests are logged in a case-tracking system.
  2. A records officer identifies which explainability logs are in scope.
  3. Exports are prepared via a controlled interface that:
     - applies date/corridor filters,
     - enforces retention limits (no retrieval beyond retention),
     - produces a machine-readable file plus a human-readable summary.
  4. Redactions and aggregations are applied as per policy.
  5. The exact export (hash + metadata) is filed with the case.

- **Redaction templates**
  - Define templates for:
    - **Public summary**: high-level KPIs, reason-code counts, no IDs.
    - **Detailed but redacted**: decision-level logs with anonymized IDs and removed security-sensitive fields.

- **Purpose limitation**
  - When sharing data with researchers or consultants, document:
    - purpose of access,
    - permitted uses and retention limits,
    - prohibitions on re-identification.

---

## Implementation Checklist

Use this as an onboarding and delivery checklist for teams implementing explainable signals.

1. **Governance & Taxonomy**
   - [ ] Explainability contract approved (which actions require explanations).
   - [ ] Reason code taxonomy v1 defined and documented.
   - [ ] Evidence tiers (Tier 0–3) defined with field lists.

2. **Canonical Schema & Adapters**
   - [ ] Canonical JSON/YAML schema published (with versioning).
   - [ ] Vendor adapters implemented for each ATMS/controller/twin.
   - [ ] Automated schema validation in CI and in production ingest.

3. **Logging & Episodes**
   - [ ] Decision event logging in place for all required actions (Tier 1).
   - [ ] Episode rollup jobs implemented (e.g., 5-minute windows).
   - [ ] Sampling rules for high-frequency events documented and configured.
   - [ ] Alert rules for anomalies, constraints, oscillations, and sensor health implemented.

4. **Security & Integrity**
   - [ ] Append-only storage configured for decision logs.
   - [ ] Hash chaining or equivalent integrity mechanism implemented.
   - [ ] WORM/immutable storage class applied for Tier A logs.
   - [ ] RBAC and least-privilege access to logs enforced.
   - [ ] Access logging enabled for searches, views, and exports.

5. **Uncertainty & Counterfactuals**
   - [ ] `uncertainty` fields populated and surfaced in UI.
   - [ ] Counterfactuals labeled as model estimates with caveats.
   - [ ] UI shows confidence badges and “model unreliable” states.

6. **Operator UX & SOPs**
   - [ ] 10-second summary + drill-down UI implemented.
   - [ ] Incident/episode views with severity and grouping.
   - [ ] Triage SOPs for sensor failure, policy binding, oscillation, overrides documented and trained.
   - [ ] Daily summary and incident timeline features live.

7. **Records & Transparency**
   - [ ] Retention tiers (A/B/C) adopted in records schedule.
   - [ ] Export workflows with approval, redaction, and hashing implemented.
   - [ ] Public-facing reporting template (dashboards or reports) defined.

---

## Operations SOP (Runbook)

This runbook is aimed at operators and supervisors.

### Normal Operations

1. **Start of shift**
   - Review overnight daily summary (incidents, oscillations, sensor issues, overrides).
   - Check dashboards for current critical alerts (safety-related, major incidents).

2. **Monitoring during shift**
   - Use exception-first view.
   - Investigate corridors flagged for:
     - constraint violations or high binding frequency,
     - persistent oscillations,
     - degraded sensor health.

3. **Responding to an alert**
   - Open the episode / incident timeline.
   - Read the 10-second explanation summary and check confidence badge.
   - Drill down into:
     - recent decisions (reason codes and overrides),
     - constraint and uncertainty blocks,
     - counterfactual outputs (if present, paying attention to caveats).
   - Follow the relevant SOP (sensor fault, policy binding, oscillation, overrides).

4. **Documenting actions**
   - When applying manual overrides, always:
     - pick the correct reason code category,
     - enter a short operator note,
     - ensure the change is logged as a decision with `operator_id` and `operator_note`.

### End of Shift / Handover

- Review open incidents and notable corridors.
- Create a short shift summary highlighting:
  - unresolved sensor or configuration issues,
  - recurring automation problems,
  - significant overrides.
- Handover log references to specific episode/decision IDs for continuity.

---

## Security & Records Retention Runbook

This runbook is aimed at security, IT, and records management staff.

### Security & Integrity Tasks

- **Daily**
  - Check log ingestion health.
  - Verify hash chain continuity for the last 24 hours.
  - Review security alerts related to failed access attempts or unusual export activity.

- **Weekly**
  - Sample a small number of decisions and confirm:
    - required Tier 1 fields present,
    - schema versions valid,
    - retention class (Tier A/B/C) correctly assigned.
  - Review access logs for anomalous access patterns.

- **Quarterly**
  - Perform a restoration drill from archival storage.
  - Validate that WORM/immutability policies remain active and unchanged.
  - Review RBAC assignments and remove or downgrade stale accounts.

### Records & Transparency Tasks

- **Records retention review (annual)**
  - Confirm retention periods align with city-wide schedules and legal guidance.
  - Adjust tiers and storage policies if the system’s use or risk profile changes.

- **Responding to public records requests**
  - Use the controlled export interface.
  - Apply the correct redaction template.
  - Store export metadata (hash, scope, requester, purpose).

- **Data sharing with external parties**
  - Ensure a data-sharing agreement is in place.
  - Confirm that shared data is the **minimum necessary** and follows agreed retention.

---

## Reference Links

These references informed the patterns above and provide additional detail on risk management, logging, and governance:

- NIST AI Risk Management Framework (AI RMF 1.0): https://doi.org/10.6028/NIST.AI.100-1
- NIST AI RMF Playbook: https://airc.nist.gov/docs/AI_RMF_Playbook.pdf
- NIST SP 800-92, Guide to Computer Security Log Management: https://csrc.nist.gov/publications/detail/sp/800-92/final
- NIST SP 800-53 Rev. 5, Security and Privacy Controls for Information Systems and Organizations (AU-* and AC-* controls): https://csrc.nist.gov/publications/detail/sp/800-53/rev-5/final
- Representative state/local records retention schedules for transportation and operational logs (varies by jurisdiction; check your state archives or municipal records office).

---

## Completion Checklist

Status legend: ✅ satisfied, ⚠️ partially satisfied or needs local tailoring.

- ✅ **Canonical city-owned taxonomy + evidence schema with vendor adapters**
  - See **"Canonical Evidence Schema & Portability Strategy"**, JSON and YAML examples, and the reason-code table.

- ✅ **Logging for high-frequency adjustments (event vs sampled vs aggregated, episodes, thresholds)**
  - See **"Logging Strategy for High-Frequency Micro-Adjustments"** and decision episode schema.

- ✅ **Security/integrity: tamper-evident logs, RBAC, access/exports audit, verification cadence**
  - See **"Security & Integrity: Tamper-Evident, Access-Controlled Logs"** and **"Security & Records Retention Runbook"**.

- ✅ **Uncertainty + causal validity (confidence, ranges, caveats, model-unreliable state)**
  - See **"Uncertainty & Causal Validity of Counterfactual Explanations"**.

- ✅ **Operator cognitive load (10-second summary, triage/severity, grouping, handover, SOPs)**
  - See **"Operator Cognitive Load: UI Patterns and SOP for Triage"** and **"Operations SOP (Runbook)"**.

- ✅ **Public transparency + records retention (public vs internal, tiers, requests, exports)**
  - See **"Public Transparency Boundary & Records Retention Policy"** and **"Security & Records Retention Runbook"**.

- ✅ **Required visuals**
  - Decision → reason code → evidence capture → log store → dashboards → review/export pipeline: see **"Required visual: Decision → Log → Review pipeline"**.
  - Reason-code table (reason code → required evidence fields → severity → operator action → retention tier): see **"Reason Code Taxonomy & Required-Evidence Table"**.

---

Cross-links: Related ideas include [`ideas/03-real-time-what-if-button.md`](ideas/03-real-time-what-if-button.md:1), [`ideas/08-operator-option-menu.md`](ideas/08-operator-option-menu.md:1), [`ideas/16-fast-forward-twin-next-5-minutes.md`](ideas/16-fast-forward-twin-next-5-minutes.md:1), and [`ideas/19-crowd-smart-crossings.md`](ideas/19-crowd-smart-crossings.md:1).
