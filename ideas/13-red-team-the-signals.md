# 13) Red-Team the Signals: Adversarial Testing for Resilience

## Catchy explanation

Before real attackers find the weak spots, you hire your own “attackers” and break the system in a simulator and test lab—so the real intersection stays safe.

## What it is (precise)

**Red-teaming the signals** is a continuous adversarial testing program for connected traffic signal ecosystems (controllers, detectors, comms, central software, and supporting OT/IT). It blends:

- **Threat modeling + scoped authorization** (what is allowed, where, and when)
- **Repeatable fault/attack injection** (sensor streams, network conditions, time sync, configs)
- **Twin + lab validation** (safety-preserving realism)
- **Operationally meaningful scoring** (queues, split failures, pedestrian service, preemption behavior)
- **Mitigation validation** (detection, response modes, recovery, change control)

The outputs are production-ready artifacts: a versioned scenario library, tuning targets and stop conditions, detection rules and dashboards, playbooks for operations and incident response, and procurement/acceptance requirements.

This is **not** a one-off penetration test. It is an engineering + operations discipline aligned to OT constraints (availability, determinism, safety, and “don’t break the plant/intersection”). NIST highlights OT’s unique performance, reliability, and safety requirements and the need for OT-specific countermeasures and architectures. [NIST SP 800-82r3](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-82r3.pdf)

## Why this matters (benefits)

- **Finds failures before the public does**: converts “we think it’s safe” into “we tested these conditions and know the outcomes.”
- **Turns cyber risk into ops/safety impact**: measures spillback, blocked-box proxies, pedestrian delay, and coordination collapse under attack.
- **Validates mitigations instead of assuming them**: plausibility checks, mode hysteresis, fallback plans, and network protections are verified against adversarial stimuli.
- **Improves readiness**: drills and clear stop conditions reduce time-to-safe-state.

## Constraints and challenges (what can go wrong)

- **Realism vs. safety**: credible attacks must be modeled without risking field operations.
- **Coverage**: you will not model everything; prioritize attack surface and consequences.
- **False positives**: aggressive detection can create operational harm via needless fallbacks.
- **OT fragility**: OT environments require careful change control and tested patches. [NIST SP 800-82r3](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-82r3.pdf)

---

## 1) Threat model and scope (what we test)

### 1.1 Assets and trust boundaries (minimum list)

Document the system as zones and conduits (OT, field networks, backhaul, IT, cloud). At minimum include:

- **Field devices**: controller cabinet, MMU, detector inputs, pedestrian devices, preemption interfaces.
- **Sensors**: loops, radar, video detection pipelines (camera → analytics → calls).
- **Comms**: fiber/cellular backhaul, VPNs, radios, maintenance ports.
- **Central systems**: ATSPM, signal management, timing plan distribution, certificate/time services.
- **Operator tooling**: laptops, jump boxes, remote access solutions, vendor maintenance channels.

CISA highlights the risk of connecting OT/ICS to business IT and the value of segmentation, DMZs, and carefully controlled remote access. [CISA ICS Best Practices](https://www.cisa.gov/sites/default/files/publications/Cybersecurity_Best_Practices_for_Industrial_Control_Systems.pdf)

### 1.2 Adversaries and threat objectives (traffic-signal flavored)

Define actors by capability and goal:

- **Opportunistic**: commodity malware/ransomware that spills into OT-adjacent networks.
- **Targeted vandalism**: disrupt an event corridor, create congestion, manipulate pedestrian service.
- **Insider/contractor misuse**: configuration changes, credential abuse, maintenance port misuse.
- **Supply-chain**: compromised updates, tainted vendor remote access.

NIST describes OT threat sources and vulnerabilities and emphasizes countermeasures tailored to OT environments. [NIST SP 800-82r3](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-82r3.pdf)

### 1.3 In-scope attack surfaces (starter menu)

Organize the scenario library by “surface”:

1. **Sensor manipulation**
   - Stuck-on / stuck-off detectors
   - Video occlusion/glare/rain + adversarial patterns
   - Radar saturation or spoof-like artifacts
2. **Network degradation**
   - Delay, jitter, loss, reordering
   - Partition (site isolated from central)
   - DNS/certificate/time failures affecting central integrations
3. **Time sync integrity**
   - Drift, step changes, NTP unreachability
4. **Configuration integrity**
   - Timing plan push anomalies
   - Detector mapping swaps
   - Wrong intersection geo/topology metadata
5. **Remote access and maintenance**
   - Jump host misuse
   - Persistent vendor access attempts

CISA recommends minimizing and securing network connections to ICS and cautions against persistent remote vendor or employee connection to the control network; it also emphasizes using jump servers and monitoring remote connections. [CISA ICS Best Practices](https://www.cisa.gov/sites/default/files/publications/Cybersecurity_Best_Practices_for_Industrial_Control_Systems.pdf)

### 1.4 Out-of-scope by default (until explicit written authorization)

Default “no” list, unless explicitly approved in a signed test plan:

- Production DoS/DDoS, radio jamming, or anything that can impair public safety
- Physical intrusion/tampering
- Social engineering
- Tests that touch non-owned vendor infrastructure without vendor permission

This matches the “avoid disruption” posture and disallowed test-method patterns in standard vulnerability disclosure policy language. [CISA VDP Template](https://www.cisa.gov/vulnerability-disclosure-policy-template)

---

## 2) Program architecture (how we run it continuously)

### 2.1 People: core roles

Minimum viable program staff:

- **Program Owner (Traffic Ops)**: owns outcomes, prioritization, stop conditions.
- **Red Team Lead (Security/OT)**: scenario design, safe execution, reporting.
- **Twin/Testbed Lead (Engineering/Modeling)**: fidelity, injection tooling, harness.
- **OT/IT Network Lead**: architecture, segmentation/DMZ, monitoring.
- **Vendor/Integrator Liaison**: authorization, patches, contract clauses.
- **Safety Officer**: field safety and “stop the test” authority.

### 2.2 Governance artifacts (versioned)

Maintain:

- **Threat model** (assets, trust boundaries, threats, consequences)
- **Scope and authorization memo** (what is in/out, test windows, contacts)
- **Scenario library** (versioned, reproducible)
- **Mitigation registry** (owner, deployment stage, rollback plan)
- **Stop-condition matrix** (when to halt a test)
- **After-action review (AAR)** templates and retention rules

### 2.3 Environments (do not start in production)

1. **Twin environment**: the “safest” place to explore conditions and tune detection.
2. **Lab/HIL**: controller-in-the-loop, sensor replay, timing plan push simulation.
3. **Shadow monitoring**: production data observed, but mitigations not actively changing signal behavior.
4. **Carefully bounded field pilots**: only after passing safety gates.

NIST stresses OT availability and safety requirements; this staged approach is designed to honor those constraints. [NIST SP 800-82r3](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-82r3.pdf)

---

## 3) Twin fidelity validation ladder (prove your simulator is “good enough”)

A red-team program fails if the twin is unrealistic. Use a ladder where you must pass each rung before trusting higher-impact results.

### 3.1 Ladder rungs

**R0 — Interface sanity**
- Can we ingest and replay detector calls, phase states, and timing plans with correct schema/time ordering?

**R1 — Metric parity**
- Baseline KPIs match production for “quiet days”: arrival profiles, green utilization, split failures, pedestrian delay.

**R2 — Dynamics parity**
- Under normal perturbations (rain, glare, event surge), the twin reproduces expected operator-observed behaviors.

**R3 — Fault parity**
- Under non-malicious failures (stuck detectors, comms dropouts), the twin reproduces known failure signatures.

**R4 — Adversarial plausibility**
- Attack injections are constrained by what is physically and operationally plausible (e.g., sensor noise envelopes, comms constraints).

**R5 — Decision parity (hardest)**
- Mitigation decisions (fallback triggers, dwell/hysteresis behavior) match what would be acceptable in field operations.

NIST’s OT guidance includes architectures and risk management approaches intended to respect performance and availability constraints; the ladder operationalizes that principle for simulation fidelity. [NIST SP 800-82r3](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-82r3.pdf)

### 3.2 Fidelity gates (exit criteria)

A rung is “passed” only if:

- Data parity is within agreed tolerances (e.g., ±10% for key KPIs under baseline).
- There is a documented **model limitation register** (what you cannot claim).
- The test harness is deterministic/replayable (same seed → same results).

---

## 4) Scenario/attack library (repeatable, measurable, safe)

### 4.1 Scenario specification template (copy/paste)

Each scenario is a single markdown/YAML entry in a repo:

- **ID**: RT-###
- **Surface**: sensor | network | time | config | remote-access
- **Goal**: what “bad outcome” we attempt (e.g., starve ped phase, create spillback)
- **Preconditions**: topology, demand profile, plan state
- **Injection**: how, where, and for how long
- **Safety stop conditions**: conditions that halt simulation/field trial
- **Expected observables**: telemetry symptoms
- **Metrics**: detection time, FP rate, KPI degradation, recovery time
- **Mitigations under test**: version pins

### 4.2 Starter set (top 12)

1. RT-001: Detector channel stuck-on (major street)
2. RT-002: Detector channel stuck-off (minor street)
3. RT-003: Video glare → false calls spike
4. RT-004: Video occlusion → call dropouts
5. RT-005: Backhaul delay 500–2000ms + jitter
6. RT-006: Backhaul packet loss 5–20%
7. RT-007: Central partition 30 minutes (site isolated)
8. RT-008: Time step +30s (NTP tamper-like)
9. RT-009: Slow drift 100ppm for 4 hours
10. RT-010: Timing plan push with swapped detector map
11. RT-011: Certificate/PKI failure blocks central integration
12. RT-012: Remote access attempt outside window (alert-only in lab)

CISA emphasizes disabling unnecessary services, monitoring, and controlling remote connections, which maps directly to RT-012 and the remote-access telemetry you should exercise. [CISA ICS Best Practices](https://www.cisa.gov/sites/default/files/publications/Cybersecurity_Best_Practices_for_Industrial_Control_Systems.pdf)

---

## 5) Mitigation ownership and RACI (who fixes what)

A red-team program generates findings. If ownership is vague, nothing ships.

### 5.1 RACI table (minimum)

| Area | Example mitigations | Responsible | Accountable | Consulted | Informed |
|---|---|---|---|---|---|
| Detector plausibility + fusion | stuck-on/off detection, cross-sensor checks, dwell timers | Traffic Eng | Traffic Ops Dir | Vendor/Integrator, Safety | IT/Sec |
| Network segmentation & DMZ | zone boundaries, firewall rules, jump servers | IT/Network | CIO/IT Dir | OT/Traffic Ops, Vendor | Security |
| Remote access hardening | MFA, time-bound access, no persistent connections | IT/Network | CIO/IT Dir | Security, Vendor | Traffic Ops |
| Monitoring & alerting | IDS/SIEM rules, baselining, dashboards | Security | CISO | IT/OT Ops | Leadership |
| Controller/firmware patching | lab test, rollout, rollback | Vendor/Integrator + OT Ops | Traffic Ops Dir | Security | Field Techs |
| Incident response for OT | runbooks, comms, exercises | Security + OT Ops | CISO | Legal/Comms | Leadership |

CISA explicitly calls out segmentation, DMZs, jump servers, and “do not allow remote persistent vendor or employee connection to the control network,” which should be reflected as accountable requirements in the RACI. [CISA ICS Best Practices](https://www.cisa.gov/sites/default/files/publications/Cybersecurity_Best_Practices_for_Industrial_Control_Systems.pdf)

### 5.2 Finding lifecycle (SLAs)

- **Triage (≤5 business days)**: confirm reproducibility, classify severity by ops/safety impact.
- **Mitigation plan (≤15 business days)**: owner, approach, test plan, rollback plan.
- **Validation**: must pass regression suite in twin/lab.
- **Deployment**: staged rollout + monitoring.
- **Closure**: evidence attached (test results + change ticket).

---

## 6) Responsible disclosure governance (VDP adapted to signals)

Even if your red-teaming is mostly internal, you need a **public-facing path** for good-faith reports and a clear authorization posture.

### 6.1 Publish a Vulnerability Disclosure Policy (VDP)

Use plain-language authorization and scope definitions. CISA’s VDP template provides recommended authorization language that frames good-faith research as authorized and commits to not pursuing legal action when the policy is followed. [CISA VDP Template](https://www.cisa.gov/vulnerability-disclosure-policy-template)

Minimum VDP components to adapt:

- **Authorization statement** (good faith research is authorized if policy followed) [CISA VDP Template](https://www.cisa.gov/vulnerability-disclosure-policy-template)
- **Guidelines** (avoid privacy violations, avoid disruption, stop if sensitive data encountered) [CISA VDP Template](https://www.cisa.gov/vulnerability-disclosure-policy-template)
- **Disallowed methods** (DoS/DDoS, physical testing, social engineering) [CISA VDP Template](https://www.cisa.gov/vulnerability-disclosure-policy-template)
- **Scope** (allowlist/denylist; vendor authorization constraints) [CISA VDP Template](https://www.cisa.gov/vulnerability-disclosure-policy-template)
- **Reporting channels** (web form/email; allow anonymous submission; acknowledgement targets) [CISA VDP Template](https://www.cisa.gov/vulnerability-disclosure-policy-template)

### 6.2 Align internal red-team with VDP rules

- Your internal team must follow the same “avoid disruption” and “stop if sensitive data encountered” constraints.
- Maintain an explicit “test methods not authorized” list for internal exercises.
- Ensure vendor and MSP contracts explicitly permit your approved testing in non-production environments.

---

## 7) Safety realism and stop conditions (when to halt)

OT testing must treat safety and availability as first-class constraints.

### 7.1 Stop-condition matrix (examples)

| Category | Stop now if… | Immediate action |
|---|---|---|
| Field safety | any risk of unsafe signal indications | revert to safe fixed-time plan; notify safety officer |
| Operations | sustained spillback/blocked-box proxy exceeds threshold | halt injection; go to conservative timing; log event |
| Cyber | evidence of unintended spread beyond scope | isolate environment; incident response escalation |
| Privacy/data | sensitive data encountered unexpectedly | stop; preserve evidence; notify security/legal |

CISA’s VDP template explicitly states that if sensitive data is encountered, testing must stop and the issue must be reported immediately. [CISA VDP Template](https://www.cisa.gov/vulnerability-disclosure-policy-template)

### 7.2 “Safe realism” rules for simulations

- Attack parameters must be bounded by physical and network plausibility.
- Every scenario must define a conservative fallback state.
- Any field trial must have a pre-approved rollback procedure and an on-call response chain.

NIST notes OT systems have unique reliability and safety requirements; the above rules make those requirements explicit gates. [NIST SP 800-82r3](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-82r3.pdf)

---

## 8) Metrics and cadence (how we prove progress)

### 8.1 Core metrics (tracked per release)

**Detection quality**
- Time-to-detect (TTD) per scenario
- False positives per hour (FP/hr) by surface
- Precision/recall for labeled injections

**Operational harm under attack**
- Δ queue length / travel time vs baseline
- Split failure rate
- Pedestrian service violations / delay
- Recovery time to baseline

**Program health**
- % scenarios automated
- Regression suite duration
- Finding SLA compliance
- Top recurring root causes (config, time, comms, sensor)

### 8.2 Cadence (minimum viable)

- **Weekly**: scenario library grooming + new incident-to-scenario intake.
- **Monthly**: regression run + tuning review.
- **Quarterly**: tabletop/game-day exercise + AAR.
- **Per change**: run suite for controller firmware updates, detection model changes, or central system upgrades.

CISA recommends continual monitoring and assessment of ICS and emphasizes incident response planning; cadence ensures this becomes routine rather than episodic. [CISA ICS Best Practices](https://www.cisa.gov/sites/default/files/publications/Cybersecurity_Best_Practices_for_Industrial_Control_Systems.pdf)

---

## 9) Detailed implementation plan (90-day “start small, make it real”)

### Weeks 1–2: Charter and scope

- Define outcomes: what constitutes “unacceptable” ops/safety impact.
- Create the initial allowlist/denylist and disallowed test methods.
- Publish internal authorization memo.

### Weeks 3–6: Build the harness

- Data ingestion/replay tooling (detectors, SPaT, timing plans).
- Injection framework (network conditions, sensor distortions, config swaps).
- Metric computation (ATSPM-style KPIs, safety proxies).

### Weeks 7–10: Baseline + fidelity ladder to R2

- Calibrate baseline parity.
- Validate dynamics under non-malicious perturbations.

### Weeks 11–13: First red-team suite

- Implement RT-001…RT-006.
- Tune detection thresholds and fallback dwell/hysteresis.

### Week 14: First game day

- Tabletop + lab run.
- Produce AAR with concrete backlog.

### Week 15–13 (ongoing): Harden remote access + monitoring

- Implement “no persistent remote connections,” jump servers, and monitoring of remote access events. [CISA ICS Best Practices](https://www.cisa.gov/sites/default/files/publications/Cybersecurity_Best_Practices_for_Industrial_Control_Systems.pdf)

---

## Implementation Checklist

- [ ] Charter: program owner, safety officer, and red-team lead named
- [ ] Written scope + authorization memo signed
- [ ] “Out-of-scope by default” list documented
- [ ] Twin environment running with replay + deterministic harness
- [ ] Fidelity ladder passed to at least R2 with documented limitations
- [ ] Scenario library repo created; template adopted
- [ ] Top 6 scenarios implemented and reproducible
- [ ] Telemetry dashboards for TTD/FP/hr/KPI deltas
- [ ] Mitigation registry and RACI published
- [ ] Regression suite wired into change control gates
- [ ] First tabletop/game-day executed; AAR backlog created
- [ ] VDP draft prepared and approved for publication path

---

## Red-Team Operations Runbook (SOP)

### SOP-1: Pre-run approvals

1. Confirm environment (twin/lab/shadow/field pilot) and scope.
2. Confirm stop conditions and on-call contacts.
3. Confirm rollback plan and “safe fixed-time” configuration.
4. Confirm logging destinations and time sync.

### SOP-2: Execute a scenario

1. Pin versions: scenario ID, injection tooling version, mitigation version.
2. Start baseline capture (10–30 minutes).
3. Apply injection for defined duration.
4. Monitor stop conditions continuously.
5. End injection; observe recovery window.
6. Export artifacts: metrics, logs, config snapshots.

### SOP-3: Post-run analysis

1. Compute detection and ops-impact metrics.
2. Classify finding severity by consequence (safety > availability > efficiency).
3. File tickets with RACI owner; attach evidence.
4. Update scenario notes (what was unrealistic/what failed).

### SOP-4: Regression gate (release checklist)

A change may ship only if:

- All “must-pass” scenarios succeed, or a documented waiver is approved.
- Remote access and monitoring controls remain intact (no new persistent connections). [CISA ICS Best Practices](https://www.cisa.gov/sites/default/files/publications/Cybersecurity_Best_Practices_for_Industrial_Control_Systems.pdf)

---

## Governance / Responsible Disclosure Runbook

### GOV-1: Publish VDP

- Publish a VDP page and keep it updated as systems change. [CISA VDP Template](https://www.cisa.gov/vulnerability-disclosure-policy-template)
- Include authorization language, scope, disallowed methods, and reporting channels. [CISA VDP Template](https://www.cisa.gov/vulnerability-disclosure-policy-template)

### GOV-2: Intake and triage

- Acknowledge reports quickly (target: 3 business days per the template’s example language). [CISA VDP Template](https://www.cisa.gov/vulnerability-disclosure-policy-template)
- Allow anonymous reporting; do not require PII. [CISA VDP Template](https://www.cisa.gov/vulnerability-disclosure-policy-template)

### GOV-3: Coordinated remediation

- Confirm scope/authorization; if out-of-scope, route to correct vendor.
- Track remediation with SLAs; share status with reporter when possible.

### GOV-4: Researcher safety rules (mirrors internal)

- Avoid disruption; do not exfiltrate data; stop immediately if sensitive data is encountered. [CISA VDP Template](https://www.cisa.gov/vulnerability-disclosure-policy-template)

---

## Reference links (≤5 sources)

1. NIST SP 800-82 Rev. 3 (PDF): Guide to Operational Technology (OT) Security — https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-82r3.pdf
2. NIST SP 800-82 Rev. 3 (landing page): https://csrc.nist.gov/pubs/sp/800/82/r3/final
3. CISA: Vulnerability Disclosure Policy Template: https://www.cisa.gov/vulnerability-disclosure-policy-template
4. CISA: Cybersecurity Practices for Industrial Control Systems (PDF): https://www.cisa.gov/sites/default/files/publications/Cybersecurity_Best_Practices_for_Industrial_Control_Systems.pdf

---

## Completion checklist

✅ Required sections added:

- ✅ Threat model and scope: [`ideas/13-red-team-the-signals.md`](ideas/13-red-team-the-signals.md) (Sections 1.1–1.4)
- ✅ Twin fidelity validation ladder: [`ideas/13-red-team-the-signals.md`](ideas/13-red-team-the-signals.md) (Section 3)
- ✅ Mitigation ownership + RACI: [`ideas/13-red-team-the-signals.md`](ideas/13-red-team-the-signals.md) (Section 5)
- ✅ Responsible disclosure governance: [`ideas/13-red-team-the-signals.md`](ideas/13-red-team-the-signals.md) (Section 6 + Governance Runbook)
- ✅ Safety realism/stop conditions: [`ideas/13-red-team-the-signals.md`](ideas/13-red-team-the-signals.md) (Section 7)
- ✅ Metrics/cadence: [`ideas/13-red-team-the-signals.md`](ideas/13-red-team-the-signals.md) (Section 8)

✅ End sections appended:

- ✅ Implementation Checklist: [`ideas/13-red-team-the-signals.md`](ideas/13-red-team-the-signals.md)
- ✅ Red-Team Operations Runbook (SOP): [`ideas/13-red-team-the-signals.md`](ideas/13-red-team-the-signals.md)
- ✅ Governance / Responsible Disclosure Runbook: [`ideas/13-red-team-the-signals.md`](ideas/13-red-team-the-signals.md)
- ✅ Reference Links: [`ideas/13-red-team-the-signals.md`](ideas/13-red-team-the-signals.md)

✅ Source cap:

- ✅ Unique sources used: 4 (≤5)

---

Cross-links: Related ideas include [`ideas/11-delay-tolerant-smart-signals.md`](ideas/11-delay-tolerant-smart-signals.md), [`ideas/02-self-healing-intersections.md`](ideas/02-self-healing-intersections.md), and [`ideas/20-explainable-signals.md`](ideas/20-explainable-signals.md).
