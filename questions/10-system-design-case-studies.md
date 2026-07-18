# 10. System Design / Case Study Questions

Open-ended, senior/staff-level questions. These are typically given as extended discussions (30-60+ minutes) rather than questions with a single "correct" short answer. Interviewers should be evaluating the candidate's reasoning process, how they handle ambiguity, and whether they proactively surface tradeoffs — not whether they arrive at one specific expected design.

---

### 10.1 🔴 Design the end-to-end communication protocol and system architecture for a fully implanted, wireless, bidirectional BCI supporting both motor decoding (for prosthetic control) and somatosensory feedback (stimulation), from electrode to external computer.

**What a strong answer covers:**
- Clarifying questions first: channel count, target bandwidth/latency for motor decode, expected stimulation parameters and update rate, power/battery constraints, expected clinical population and use environment.
- A layered architecture: analog front end and on-chip feature extraction (referencing category 1/2 signal considerations) → implant-side protocol stack → wireless link (referencing MICS band or similar, category 5) → external processing unit → application layer.
- Explicit treatment of the stimulation artifact problem (category 1.18) — how recording and stimulation channels are time-multiplexed or otherwise isolated from each other.
- A clear safety architecture: independent stimulation safety limits enforced in hardware, a defined fail-safe state, and a redundant/out-of-band emergency stop mechanism (category 6, 3.3).
- Explicit latency budget allocation across the pipeline (category 2.3) and justification for closed-loop timing requirements given the somatosensory feedback path.
- Discussion of graceful degradation under link quality loss (category 3.10) and what happens to ongoing stimulation if the link drops.
- Should proactively flag the regulatory/risk implications of this being a Class III, high-risk PMA-pathway device (category 7) and how that shapes validation approach, not just the technical design.
- Bonus: discussion of how firmware updates would be handled safely over the device's lifetime (category 3.7).

---

### 10.2 🔴 A hospital system wants to deploy your company's neural interface to 50 patients across 10 different clinical sites simultaneously, each running slightly different clinic workflows and network environments. How would you design the system's deployment, monitoring, and support architecture to make this manageable?

**What a strong answer covers:**
- Recognizing this is as much an operational/systems design problem as a pure protocol design problem.
- Centralized remote monitoring/telemetry design (with appropriate privacy/security safeguards, category 8) so issues can be detected proactively rather than only when a clinician happens to notice a problem.
- A robust logging and diagnostics architecture (category 9.12) designed specifically to make remote troubleshooting across many sites tractable without physical device access.
- Staged/phased rollout strategy (start with fewer sites/patients, expand as confidence grows) rather than simultaneous full deployment, and reasoning about why that's lower risk.
- Handling site-to-site variability (different network/interference environments) — should reference the graceful degradation and link-quality-monitoring discussion from category 3/5.
- A clear escalation and support process design — when does a local clinician handle an issue vs. when does it require engineering involvement — and how the device/system architecture supports that triage (e.g., self-diagnostic modes from category 9.19).
- Should address version/configuration consistency across sites (all patients running compatible protocol/firmware versions, category 3.18) and how that's managed and monitored at scale.

---

### 10.3 🔴 Your team discovers, post-launch, that under a specific rare combination of conditions (a particular pattern of packet loss during a firmware update), an implanted device can end up in an unresponsive state. How would you handle this as both a technical and organizational problem?

**What a strong answer covers:**
- Immediate triage: understanding actual patient risk/impact right now — are any patients currently in this state, is it detectable remotely, does it require emergency intervention.
- Root cause analysis approach — reproducing the specific condition (ideally via the simulation environment discussed in category 9.9) rather than guessing at a fix.
- Should reference the safety-critical update design discussed in category 3.7 and 6 — if the design already had proper A/B firmware banking and rollback, why did this scenario still cause a problem, and what does that reveal about a gap in the original design (a strong candidate should be able to reason about this critically rather than assuming their own hypothetical design is flawless).
- Fix development following the full rigor discussed in category 9 (not a rushed patch) — but balanced against urgency given real patient impact.
- Organizational/communication dimension: notifying regulatory bodies as required (category 7.13's cybersecurity/postmarket surveillance expectations apply analogously to safety issues), communicating with affected clinicians/patients transparently.
- Process improvement: what should change in the FMEA, test coverage, or validation process (category 6.3) so this class of issue is caught before deployment next time — good answers treat this as a systemic learning opportunity, not just a one-off bug fix.

---

### 10.4 🟡 Design a protocol-level approach for handling the scenario where a patient with an implanted device travels internationally, potentially into regions with different RF regulatory allocations (e.g., MICS band availability or restrictions varying by country).

**What a strong answer covers:**
- Recognizing this as a real, known challenge for implantable medical devices, not a purely hypothetical edge case.
- Understanding that a device may need to detect its regulatory/geographic context (or be pre-configured by a clinician before travel) and adapt its RF behavior accordingly.
- Design options: multi-band capable hardware supporting the different regional allocations, with the protocol/firmware selecting the appropriate mode; or a more conservative approach of ensuring the device's core safety functions don't depend on functioning RF telemetry at all, so a temporarily degraded or region-mismatched link doesn't compromise essential safety.
- Should discuss the tradeoff between hardware complexity/cost (supporting multiple bands) versus a software/configuration-only approach.
- A strong answer flags that this ultimately needs to be a collaboration with regulatory/clinical teams — the technical team can build the capability, but policy on how it's actually used (e.g., patient travel guidance) is a broader team decision.

---

### 10.5 🔴 How would you design the system to support collecting data for future decoder improvement (a legitimate and valuable goal) while remaining maximally protective of patient privacy and data minimization principles?

**What a strong answer covers:**
- Should immediately recognize the tension this question is probing (category 7.4, 7.15, 8.11) and not just default to "collect everything, we'll figure out privacy later."
- Discussion of on-device processing to extract only necessary derived features rather than transmitting/storing raw neural data by default (referencing category 2.9, 4.14).
- A tiered consent model: baseline data collection covered by standard clinical-use consent vs. explicit, separate, clearly-described consent for broader research data retention.
- Technical enforcement of consent boundaries — the system architecture should be able to actually enforce different data handling based on what a specific patient has consented to, not just have this be a policy on paper.
- Discussion of the specific challenge that neural data may not be reliably de-identifiable (category 7.6), and how that should inform a more conservative default posture.
- Should mention security/access-control requirements around any research data store (category 8.11) and appropriate governance around who can access it and for what purposes.

---

### 10.6 🟡 Design a testing and validation strategy for a new wireless protocol version before it's ever used with a real patient, given that you can't easily "test in production" the way you might for typical consumer software.

**What a strong answer covers:**
- A layered testing pyramid: unit tests, protocol simulator with injected fault conditions (category 3.20), hardware-in-the-loop bench testing, tissue-phantom RF testing (category 5.7), and finally supervised clinical/research testing under appropriate protocol.
- Should specifically address how the strategy tests not just "does it work" but "does it fail safely" — deliberately testing degraded/adverse conditions, not just the happy path.
- Discussion of how this testing evidence maps to the regulatory verification/validation and traceability requirements discussed in category 6 and 7.
- A strong answer distinguishes between what can be validated in simulation/bench testing versus what genuinely requires real-world/clinical testing, and reasons carefully about that boundary rather than assuming everything can be fully validated in simulation.

---

### 10.7 🔴 Your company is deciding between two architectural approaches for a next-generation device: (A) a higher channel-count, higher-power system with more on-implant processing capability, or (B) a lower channel-count, lower-power system with most processing done externally. Walk through how you'd evaluate this decision.

**What a strong answer covers:**
- Should resist immediately picking a side and instead lay out the actual tradeoff dimensions systematically: decoding performance/capability ceiling, battery life and recharge burden on the patient, thermal/safety margin, implant physical size and surgical considerations, wireless bandwidth requirements and associated power cost, and regulatory/validation complexity (more on-implant processing likely means more complex, harder-to-verify implant firmware).
- Should push for grounding the decision in the actual clinical use case and patient population rather than treating it as a purely technical optimization — e.g., what specific clinical capability does the higher channel count actually unlock, and is that capability difference clinically meaningful enough to justify the added complexity/risk/burden.
- A nuanced answer might propose that this isn't strictly binary — e.g., discussing a hybrid approach with configurable operating modes (category 3.22's polling vs. streaming discussion is a relevant analog) rather than assuming a single fixed architecture must be chosen for all situations.
- Should mention involving other stakeholders (clinical, regulatory, patient advocacy/user research) in this decision, not treating it as an engineering-only choice, and reference how the decision would affect the regulatory pathway (category 7).

---

### 10.8 🟡 A patient reports that their device "feels different" or seems to be behaving inconsistently since a recent firmware update, but all automated diagnostics show the device is functioning within normal parameters. How would you investigate this?

**What a strong answer covers:**
- Taking the patient's subjective report seriously as real signal, not dismissing it just because automated diagnostics look clean — automated diagnostics only check what they were designed to check, and a genuine issue could exist outside that coverage.
- Systematic comparison of pre- and post-update behavior using available logged data (category 9.12) — did any subtle timing, decoding accuracy, or signal characteristic actually shift, even if within nominally "acceptable" bounds.
- Considering non-technical explanations too (without dismissing the patient) — e.g., could the update have changed a default configuration parameter in a way that's technically "working as intended" but produces a different subjective experience, which would be a design/communication issue rather than a bug.
- Should discuss the value of being able to (with patient consent) run the previous firmware version side-by-side or roll back temporarily for comparison, referencing the update rollback capability discussed in category 3.7.
- A strong answer treats this as valuable information regardless of root cause — if current diagnostics can't explain a real, patient-reported change, that's itself a gap in the diagnostic/monitoring architecture worth addressing (category 9.12) for the future, even if this specific instance turns out to have a benign explanation.

---

### 10.9 🔴 Design a strategy for how a decoding algorithm should adapt over a patient's first six months of chronic implant use, given that neural signal characteristics, patient learning, and electrode properties all change substantially over that period.

**What a strong answer covers:**
- Should draw directly on the non-stationarity, drift, and calibration discussion from category 2 (2.6, 1.4, 1.17) and synthesize it into a coherent phased strategy rather than treating it as unrelated facts.
- A reasonable phased approach: initial conservative, heavily-supervised calibration in the first days/weeks (higher clinical involvement, more frequent recalibration), transitioning to more autonomous/adaptive online recalibration as the system and patient stabilize, informed by increasing confidence from accumulated data.
- Should address the co-adaptation dynamic explicitly — the patient is also learning/adapting to the device, not just the device adapting to a fixed patient signal, and design should account for this bidirectional learning process rather than assuming a one-directional model-fitting problem.
- Discussion of how to detect when adaptation is and isn't working well (signal-quality/decode-confidence monitoring from category 2.6) and what the fallback/escalation path is if the decoder isn't converging well for a particular patient.
- Should mention that any adaptive algorithm running with reduced clinical supervision still needs the safety guardrails discussed in category 6 (bounded, validated operating ranges) — "adaptive" shouldn't mean "unconstrained."

---

### 10.10 🟡 How would you design a minimal viable version of this entire system for an early feasibility clinical study (a handful of patients, heavily clinically supervised), knowing you'll need to scale it up significantly for later, larger trials?

**What a strong answer covers:**
- Should explicitly reason about what can legitimately be simplified for a small, heavily-supervised early study (e.g., relying more on real-time clinician oversight rather than fully autonomous safety monitoring, since a clinician is physically present) versus what safety-critical elements should never be simplified even at small scale (hard stimulation limits, fail-safe behavior).
- Discussion of designing the early system's architecture to not create technical debt that blocks later scaling — e.g., even if an early version doesn't need multi-site deployment support, avoiding architectural choices that would make that support very difficult to add later.
- Should mention that this feasibility-study version still needs full risk analysis, traceability, and regulatory engagement appropriate to a first-in-human or early feasibility study (referencing category 7), not skip that rigor just because the initial patient count is small — "minimal" refers to feature scope, not to safety/validation rigor.
- A thoughtful answer might discuss what data/insights specifically need to be captured during this early phase to inform the larger-scale design decisions later (e.g., real-world link reliability data that will inform whether the RF design needs revision before scaling).

---

### 10.11 🔴 Two engineers on your team disagree about whether a proposed safety feature (an additional independent hardware interlock on the stimulation output) is worth the added cost, complexity, and implant size it would require. How would you help resolve this disagreement?

**What a strong answer covers:**
- Rather than just picking a side, a strong answer describes a process: grounding the discussion in the actual risk analysis (category 6.3, 6.10) — what specific failure mode does this interlock address, what's its estimated severity and likelihood without the interlock, and how much does the interlock actually reduce that risk versus the other safety layers already in place (defense in depth, category 6.19) — rather than debating it on general engineering-elegance grounds.
- Should consider whether the added complexity of the interlock itself could introduce new failure modes that partially offset its safety benefit (echoing the "more capability isn't always safer" discussion in category 6.13) — genuinely evaluating both sides rather than assuming more redundancy is automatically better.
- Discussion of how this decision should ultimately be made through the formal risk-management process (referencing ISO 14971-style methodology from category 6.10) with documented reasoning, rather than resolved purely through informal engineering debate, since this kind of decision needs to be auditable and defensible to regulators and, more importantly, genuinely well-reasoned given the patient safety stakes.
- A mature answer recognizes that reasonable engineers can disagree on judgment calls like this, and the goal isn't necessarily to prove one engineer "right," but to ensure the decision-making process itself is rigorous, well-documented, and appropriately involves the necessary risk/safety expertise rather than being resolved by whoever argues more persuasively in a meeting.

---

### 10.12 🟡 If you had to remove one of the following from an early-stage neural interface prototype to hit an aggressive timeline — full wireless telemetry (use a wired bench connection instead), on-device adaptive recalibration (use fixed, manually-updated calibration instead), or multi-band RF support (support only one region's regulatory band) — which would you cut, and how would you reason through that decision?

**What a strong answer covers:**
- This is intentionally a judgment/prioritization question with no single correct answer — the interviewer should be evaluating the reasoning process, not the specific choice.
- A strong answer explicitly separates "what's needed for the current stage's actual goals" (e.g., an early prototype's purpose might be validating the core signal processing/decoding approach, for which a wired bench connection is a perfectly reasonable simplification that doesn't compromise the phase's actual goals) from "what would be an unacceptable simplification given what this stage needs to demonstrate or learn."
- Should reason about reversibility and future cost — is this a decision that can be easily revisited/added later without major rework (suggesting lower risk to defer), or does deferring it create significant technical debt or lock in decisions that will be expensive to change later.
- A good answer would likely favor cutting wireless telemetry for an early bench-stage prototype (a very standard, low-risk simplification at that stage) over cutting multi-band RF support or adaptive recalibration, but the specific choice matters less than whether the candidate reasons about it in terms of the actual goals of the current development phase, rather than just picking based on surface-level perceived complexity.

---
