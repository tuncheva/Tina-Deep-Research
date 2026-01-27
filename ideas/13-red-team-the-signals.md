# 13) Red-Team the Signals: Adversarial Testing for Resilience

## Catchy Explanation
Before real attackers find the weak spots, you hire your own “attackers” and break the system in a simulator—so the real intersection stays safe.

## What it is (precise)
**Red-teaming the signals** is a structured adversarial testing program that uses a digital twin and lab setups to simulate failures and attacks against signal inputs, controllers, and communications. The goal is to identify vulnerabilities, quantify impacts (queues, safety proxies, corridor breakdown), and validate mitigations (sensor fusion, plausibility checks, graded degradation, secure operations). Typical scenarios include detector spoofing, video glare/occlusion, comms delay/jamming, and connected-vehicle data spoofing. Outputs are: an attack library, detection rules, response playbooks, and audited recovery modes.

## Benefits
- **Proactive defense**: finds weaknesses before deployment or exploitation.
- **Impact quantification**: turns “security risk” into measurable operations/safety impacts.
- **Mitigation validation**: tests fallbacks and detection thresholds safely.
- **Operator readiness**: drills and playbooks reduce incident response time.

## Challenges
- **Realism**: simulating credible attacks takes expertise.
- **Coverage limits**: cannot model everything.
- **False alarms**: aggressive detection harms operations.
- **Ongoing cost**: red-teaming is not a one-off exercise.

## Implementation Strategies

### Infrastructure Needs
- **Threat model + asset inventory**: controllers, comms, detectors, CV interfaces.
- **Attack library**: repeatable scenarios and scripts.
- **High-fidelity twin**: ability to inject manipulated sensor streams.
- **Monitoring**: anomaly detection, health checks, ATSPM metrics.
- **Response modes**: graded degradation and safe fallback plans.

### Detailed Implementation Plan
#### Phase 1: Threat Modeling and Test Scope Definition (Weeks 1–3)
The agency should begin by inventorying the assets that can be attacked or that can fail in adversarial ways, including detector channels, video analytics pipelines, controller firmware, comms links, and any connected-vehicle interfaces. The team should define unacceptable outcomes in operational and safety terms, such as sustained spillback, blocked-box events, pedestrian service violations, or loss of preemption behavior, because these outcomes define what “impact” means. The security team should then prioritize threats by likelihood and impact and should define a bounded test scope that is realistic for the agency’s maturity level.

- **Roles**: security lead (threat model), traffic engineering (impact definitions), IT/network (comms architecture), operations (real-world failure history), vendor/ITS engineer (controller/CV specifics).
- **Deliverables**: threat model, prioritized test plan, and an “impact metric” definition document.
- **Risks**: threat model is too theoretical; scope is too broad to execute.
- **Acceptance checks**: top threats are prioritized and mapped to measurable impact metrics.

#### Phase 2: Build a Repeatable Attack and Fault Injection Library (Weeks 4–12)
The engineering team should build a versioned library of test scenarios that includes both malicious attacks and realistic “adversarial-like” faults, because many operational vulnerabilities look like attacks (spoofed detectors) but can also occur due to failures (stuck detectors). The library should include scenarios such as detector spoofing/stuck-on, video occlusion/glare, delayed or dropped comms, time-sync drift, and (where applicable) CV message spoofing or corrupted SPaT/MAP alignment. Each scenario should specify how it is injected, how long it lasts, what evidence should be observed in telemetry, and what “success” looks like for detection and response.

- **Roles**: security engineer (attack design), software engineer (injection tooling), modeler (twin integration), QA engineer (scenario harness).
- **Deliverables**: versioned attack library, injection tooling, and scenario-level evaluation metrics.
- **Risks**: unrealistic injection profiles; insufficient instrumentation to detect injected anomalies.
- **Acceptance checks**: scenarios are reproducible, and telemetry captures expected symptoms.

#### Phase 3: Mitigation Design and Twin Validation (Weeks 13–22)
The team should design mitigations that match real signal operations, such as plausibility checks across sensors, health scoring, dwell times and hysteresis for mode changes, secure communications hardening, and conservative fallback plans. The team should validate each mitigation in the twin against the attack library and should measure both detection performance (time-to-detect, false positives) and operational impact (queues, split failures, safety proxies), because a mitigation that stops attacks but causes constant fallbacks is not viable. The team should document which mitigations are mandatory for production deployment and which are optional enhancements.

- **Roles**: traffic engineering (fallback plans), security (controls), software engineer (implementation), modeler (validation), operations (workflows).
- **Deliverables**: mitigation package, tuned thresholds, twin validation report, and safety/ops sign-off.
- **Risks**: mitigations are too sensitive and increase false alarms; mitigations conflict with performance goals.
- **Acceptance checks**: mitigations reduce impact under attack scenarios while keeping false positives within acceptable bounds.

#### Phase 4: Operational Drills and “Game Day” Exercises (Weeks 23–32)
The agency should run tabletop drills that walk operators through detection, escalation, fallback activation, and recovery steps, because response playbooks are as important as technical defenses. The team should then run controlled “game day” exercises in shadow mode or in a lab/testbed environment, where injected anomalies are applied and the organization practices response without affecting the public. After each drill, the team should conduct an after-action review and update the playbooks, thresholds, and training materials.

- **Roles**: operations (drill participants), security (facilitation), traffic engineering (plan changes), IT/network (incident response), analyst (metrics).
- **Deliverables**: response playbooks, drill reports, training materials, and updated mitigations.
- **Risks**: drills may be skipped due to operational pressure; playbooks may not be maintained.
- **Acceptance checks**: operators can follow playbooks quickly, and drill outcomes result in concrete system updates.

#### Phase 5: Continuous Improvement and Regression Testing (Ongoing)
The agency should treat red-teaming as a continuous program by running a scheduled regression suite whenever controller firmware, detection systems, or optimization logic changes. The team should add new scenarios based on real incidents and on changes in the threat landscape, and it should maintain trend reporting on detection performance and operational impacts. Over time, the program should integrate into procurement and acceptance testing so new deployments must pass the red-team suite before becoming operational.

- **Roles**: program owner (governance), security (regression suite), engineering (maintenance), procurement/vendor manager (acceptance requirements), operations (feedback).
- **Deliverables**: recurring regression reports, updated scenario library, and procurement acceptance criteria.
- **Risks**: program loses funding; testbed diverges from production reality.
- **Acceptance checks**: regression tests run on schedule and findings are remediated with tracked tickets.

### Choices
- **Basic**: focus on data faults and simple spoofing.
- **Advanced**: include CV spoofing and coordinated multi-intersection attacks.
- **Automated**: scheduled red-team regression suite.

## Technical Mechanics

### Key Parameters
- Detection thresholds and dwell times
- Attack injection profiles
- Recovery criteria

### Guardrails
- Always enforce safe fallbacks.
- Keep red-team activities isolated from production unless explicitly planned.
- Log all simulated incidents and outcomes.

## MVP Deployment
- Top-10 attack scenarios.
- Monthly drills in a twin environment.
- One production corridor in shadow monitoring mode.

## Evaluation
- Detection time and false-positive rate.
- KPI degradation under attack (queues, split failures, safety proxies).
- Recovery time and operator response adherence.

## References / Standards / Useful Sources
- NIST AI RMF 1.0 (governance, monitoring, risk management): https://doi.org/10.6028/NIST.AI.100-1

---

Cross-links: Related ideas include delay-tolerant signals, self-healing intersections, and explainable signals.
