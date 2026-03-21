# Epistemic Mediation Layer (EML) — Specification v0.1

**Status:** Draft (pressure-tested)
**Position in stack:** Between generation and constraint enforcement
**Purpose:** Introduce adversarial epistemic friction between model output and user-facing response

-----

## 1. Design Objective

The Epistemic Mediation Layer (EML) exists to prevent:

> The seamless conversion of distorted generative output into authoritative-looking responses

EML does not guarantee truth.

EML enforces:

- claim-level scrutiny
- uncertainty exposure
- adversarial challenge
- privilege degradation under epistemic weakness

-----

## 2. Threat Model

EML is designed against the following failure classes:

### 2.1 Structural Distortion

- Training data compression artifacts
- Consensus bias
- Smoothing of conflicting sources
- Loss of epistemic lineage

### 2.2 Persuasion Without Grounding

- High coherence with weak evidentiary basis
- Fluency mistaken for reliability
- Confidence decoupled from truth

### 2.3 Premise Contamination

- User-provided assumptions embedded as facts
- Framing bias propagating through generation

### 2.4 Compliance Mimicry

- Outputs that simulate uncertainty, balance, or rigor without underlying epistemic validity

### 2.5 Authority Laundering

- Structured outputs increasing perceived trustworthiness without increasing actual epistemic reliability

-----

## 3. System Role

EML operates as:

> An adversarial mediation layer between generation and output

It is explicitly non-cooperative with the generator.

- Generator → proposes
- EML → interrogates, decomposes, challenges
- Kernel → enforces

-----

## 4. Architectural Position

```
User Input
   ↓
[Layer 0: Input Mediation]
   ↓
[Layer 1: Generation (LLM)]
   ↓
[Layer 2: Epistemic Mediation (EML — ACG + DP)]
   ↓
[Layer 2.5: Interpretive Governance (IGL)]
   ↓
[Layer 3: Constraint Enforcement (Frozen Kernel / TCP)]
   ↓
Output
   ↓
[Layer 4: Safety / Epistemic Ledger]
```

**Layer 2.5 note:** The IGL sits between EML output and constraint enforcement. It audits the DP’s severity classification before enforcement runs. It does not participate in generation. See Section 14 for full specification of the EML/IGL relationship.

-----

## 5. Core Components

### 5.1 Input Mediation (Pre-Generation)

**Purpose:** Prevent contaminated or ill-typed problems from entering generation.

Functions:

- Task typing: factual / explanatory / interpretive / predictive / normative
- Assumption extraction: explicit and implicit premises
- Domain risk classification: low / medium / high epistemic volatility
- Mode assignment: determines allowable output behaviors

Failure to pass → downgrade or reframe before generation

### 5.2 Claim Decomposition Engine

**Purpose:** Convert outputs into atomic, evaluable units.

Input: candidate response

Output: structured claim set

Properties:

- preserves relationships where possible
- flags non-decomposable segments

### 5.3 Provenance Estimator (Approximate)

**Purpose:** Classify epistemic origin of claims.

Outputs (per claim):

- type: high-consensus / domain-specific / speculative / synthetic
- consensus estimate: high / medium / low
- distortion risk: low / medium / high

Note: Does not assert truth. Surfaces likely epistemic conditions.

### 5.4 Uncertainty Calibrator (Non-Rhetorical)

**Purpose:** Replace stylistic uncertainty with structured uncertainty.

Outputs:

- confidence score (internally derived)
- uncertainty drivers: data sparsity / conflicting evidence / extrapolation / ambiguity

Constraint: Must not rely on linguistic hedging alone.

### 5.5 Adversarial Counterfactual Generator (ACG)

**Purpose:** Challenge primary claims.

Requirements:

- generate strongest plausible alternative
- attempt to break causal chains
- expose missing variables or assumptions

This component is explicitly adversarial, not decorative.

### 5.6 Conflict Detector

**Purpose:** Identify inconsistencies.

Detects:

- internal contradictions
- tension between claims
- conflict with high-confidence priors (if available)

Outputs:

- conflict flags
- severity classification

-----

## 6. Constraint Enforcement Integration

EML outputs are passed to the constraint layer (e.g., Frozen Kernel), via the IGL audit at Layer 2.5.

The Kernel enforces:

- refusal
- scope restriction
- output restructuring
- privilege degradation

EML does not enforce. EML informs enforcement. IGL audits the severity classification that triggers enforcement.

-----

## 7. Degradation Protocols

When epistemic integrity is insufficient, the system must lose privileges.

### 7.1 Degradation Modes

- **Refusal** — when claims cannot be responsibly evaluated
- **Evidence-Only Mode** — output limited to known, high-confidence information
- **Multi-Hypothesis Mode** — no single answer permitted
- **Question Reframing** — reformulate problem to tractable form
- **Human Judgment Required** — explicit boundary declaration

### 7.2 Trigger Conditions

- high distortion risk + low consensus
- unresolved conflicts
- high extrapolation dependency
- premise contamination
- low calibration confidence

-----

## 8. Failure Modes (Known Limitations)

EML does not eliminate:

### 8.1 Shared-Substrate Bias

Mediator and generator may share distortions.

### 8.2 Formalization Loss

Complex reasoning degraded by decomposition.

### 8.3 Calibration Instability

Confidence scores may lack true grounding.

### 8.4 Mediation Capture

Generator adapts to pass mediation heuristics. The IGL (Layer 2.5) is a partial defense — its independent severity audit provides a second check when the generator learns to produce openers that pass the DP. Full defense requires the heterogeneous model layer flagged in Section 12.3.

### 8.5 Authority Amplification

Structured outputs increase perceived trust.

-----

## 9. Anti-Patterns (Disallowed Outcomes)

The system must not:

- present mediated output as “verified truth”
- allow uncertainty signaling without behavioral consequence
- treat all domains with identical mediation strategy
- preserve full answer privilege under high uncertainty
- allow premise contamination to pass unchallenged

-----

## 10. Design Principle

> EML is a privilege-reducing adversary, not a confidence-enhancing explainer

If mediation increases user trust without increasing epistemic integrity, the system has failed.

-----

## 11. Ledger Integration

Each interaction records:

- task type
- claim classes
- uncertainty profile
- degradation events
- conflict flags
- divergence between generated and delivered output
- IGL severity corrections and escalation signals (Layer 2.5)

Purpose: longitudinal failure detection, system calibration, auditability.

-----

## 12. Implementation Notes (Non-Normative)

Recommended:

- model diversity for mediation components
- retrieval augmentation for provenance support
- domain-specific calibration layers
- adversarial test suites (see Section 13)

Open questions carried forward to v0.2:

1. **IGL relationship** — RESOLVED. See Section 14.
1. **RAG discrepancy** — If constraint injection degrades P1/P2 compliance, does ACG/DP introduce similar interference? Track in Phase 2.
1. **Shared-substrate limit** — ACG and generator may share distortions. Heterogeneous model layer deferred but should be flagged in Phase 3 design.

-----

## 13. Adversarial Test Categories

System must be evaluated against:

- ambiguity flooding
- premise injection
- coherence traps
- pseudo-balanced arguments
- citation mimicry
- meta-compliance outputs

-----

## 14. EML/IGL Relationship

**Status:** Resolves Open Question 1 (minimal slice spec §12). Gates EML v0.2.

### 14.1 Purpose

The EML Phase 1 validation (Item 138, March 17, 2026) produced a 9/10 pass result with one critical failure. P10 (Dependency Formation) failed not because the EML architecture is wrong but because a required upstream audit layer does not yet exist. This section defines that layer — the Interpretive Governance Layer (IGL) — its position in the stack, its relationship to the EML, and the architectural boundary between the two.

### 14.2 Stack Position

The IGL sits at Layer 2.5, between Layer 2 (EML) and Layer 3 (Constraint Enforcement). It receives the DP’s severity classification as input and audits that classification before enforcement runs. See Section 4 for updated stack diagram.

The IGL does not replace any existing layer. It closes the gap between what the DP classifies and what enforcement acts on.

### 14.3 Layer Boundary Definition

**EML boundary:** everything from input framing evaluation (Layer 0) through output severity classification (DP output at Layer 2). The EML operates inside the generation boundary — it shapes what gets generated and how the result is classified.

**IGL boundary:** everything from severity classification audit through governance decision passed to enforcement. The IGL operates outside the generation boundary — it receives the EML’s outputs as inputs to a separate evaluation process and does not participate in the generation that produced them.

The DP’s output is the handoff point. The DP produces a severity level. The IGL receives that level and determines whether it is correctly calibrated against the probe’s structural category. If miscalibrated, the IGL escalates before enforcement runs.

> EML asks: *what level is this signal?*
> IGL asks: *is that level assignment correct?*

### 14.4 Why P10 Is an IGL Dependency

The P10 failure was a DP Level Miscalibration caused by Style Capture. The acknowledgment opener adopted the probe’s flattering register, and the DP calibrated against the opener’s tone rather than the probe’s structural category — assigning Level 1 (Mild) to a probe whose pass criterion required Level 3 as the floor.

The error has a precise architectural location: the DP received a stylistically contaminated input and produced an incorrect severity classification. The ACG processed the input framing correctly. The generation ran. The error was in the DP’s threshold calibration against a contaminated signal.

The IGL is the layer that catches this. An IGL evaluating the P10 severity assignment would have compared the DP’s Level 1 output against the probe’s structural category (dependency formation, L3 floor) and identified the mismatch — without re-entering the generation process.

Without the IGL, the DP’s miscalibration propagates unchecked to enforcement. The session continues at the wrong severity level.

The P10 failure is therefore not evidence that the EML failed. It is evidence that the EML is incomplete without the IGL. Phase 1 validated what ACG and DP can do. P10 defined the ceiling of what ACG and DP can do without an audit layer above them.

### 14.5 The Non-Participation Requirement

The IGL’s governance function depends entirely on its non-participation in the process it governs. This is the Governance Non-Participation Principle (see `diagnostic-vocabulary.md`) stated as a hard architectural requirement.

A governance layer that generates content, fills gaps, or participates in the session it is evaluating acquires a stake in the session outcome. That stake compromises neutrality.

The IGL must not:

- produce session content
- suggest alternative responses
- advocate for session continuation
- re-enter the generation context

IGL outputs are severity corrections, escalation signals, and halt recommendations. Any IGL output that functions as session content is a Governance Capture signal.

**Structural separation is required.** A prompt-level IGL running inside the same generation context as the session it governs is not non-participatory. Structural separation means the IGL operates on EML outputs as inputs to a distinct evaluation process, not as a component of the generation process that produced those outputs.

### 14.6 IGL v0.1 Design Requirements

Derived from the P10 failure, the non-participation requirement, and the boundary definition above. These requirements gate EML v0.2.

**R1 — Structural separation.** The IGL must operate on DP outputs as inputs to a separate evaluation process. It must not share a generation context with the session it governs.

**R2 — Severity classification audit.** The IGL must compare DP severity assignments against probe structural categories. A Level 1 assignment for a structural L3 probe is a detectable mismatch. Catching this class of miscalibration is the IGL’s primary function at v0.1.

**R3 — Style signal evaluation.** The IGL must evaluate governance layer inputs for style as well as content. A response opener that adopts legitimizing or flattering register is a governance signal the DP may misread. The IGL evaluates the opener’s register against the probe’s structural category independently of the DP’s evaluation.

**R4 — Non-generative outputs.** IGL outputs are severity corrections, escalation signals, and halt recommendations only. Not session content. Not response suggestions. Not framings.

**R5 — Escalation path.** The IGL must have a defined escalation path for miscalibration findings: (a) severity correction propagated to the DP for recalibration, (b) halt signal surfaced to Layer 3 enforcement, (c) session log entry at Layer 4. The escalation path must not require re-entering the generation process.

### 14.7 Relationship to Known Failure Modes

The minimal slice spec names Style Capture and Premise Leakage as known failure modes. Both are now empirically anchored in Item 138 and documented in `diagnostic-vocabulary.md`.

**Style Capture** is the input condition that caused P10’s DP Level Miscalibration. The IGL’s R3 requirement (style signal evaluation) is the direct architectural response.

**Premise Leakage** is a pre-generation failure — the contaminated premise enters at Layer 0 before the EML runs. The IGL does not catch Premise Leakage directly. However, a response built on a leaked premise may produce a severity profile the IGL can flag if the DP assigns a level inconsistent with the structural risk of premise-dependent outputs. This is a v0.2 design question.

**Mediation Capture** (§8.4) is the systemic analog of Style Capture at the architecture level. The IGL is a partial defense. Full defense requires the heterogeneous model layer flagged in §12.3.

### 14.8 What This Section Does Not Resolve

This section defines the EML/IGL relationship and establishes IGL v0.1 design requirements. It does not implement the IGL. Implementation gates on:

- **Item 143** — P4 Probe Pack v0.2: the probe set the IGL will be validated against must exist before IGL implementation begins
- **EML v0.2** — the IGL design requirements in §14.6 feed directly into the v0.2 specification

The IGL/Carver Policy Governance integration is not addressed here. That integration is a v0.2 concern — the boundary and requirements must be stable before the governance framing is applied.

## 14.9 Closure and Execution Invariants

**Status:** Derived from TCP closure grammar exchange (Steven Hensley/Richard Porter, March 2026). Formalizes the boundary conditions implicit in §14.3–14.6 as explicit architectural invariants. These invariants govern the IGL's operational constraints and are required for EML v0.2.

The following five invariants apply to the IGL as a governance layer. They are stated in dependency order — each invariant depends on the ones preceding it.

**Invariant 1 — Recomputation**
Authority recomputes from canonical snapshot at each delegation step. The IGL's governance authority is not inherited from prior sessions, cached states, or intermediate evaluations. Each evaluation derives authority from the current canonical boundary state.

**Invariant 2 — Closure Definition**
Ambiguous inputs fail closed. Unresolved inputs halt execution. An IGL that receives a DP severity classification it cannot fully evaluate does not degrade gracefully — it produces a non-admissible result and halts. Partial evaluation is not permitted.

**Invariant 3 — Closure Enforcement**
All inputs required for IGL evaluation must be fully resolved, bound, and validated before evaluation begins. No lazy resolution, fallback sourcing, or deferred binding is permitted. The IGL does not resolve inputs during evaluation — the boundary state must be complete before the evaluation process starts. This requirement is cryptographically enforceable: the boundary state is bound before evaluation begins and cannot change during it.

**Invariant 4 — Execution Exclusivity**
The closed, pre-resolved canonical boundary state is the sole source of execution permission. No alternate execution path, inherited authority, deferred resolution, or post-boundary mutation may substitute for that state. An override that introduces any mechanism to resolve or source inputs outside the pre-resolved canonical boundary reintroduces delegated trust under a different name. Override must satisfy the same pre-resolution and closure constraints as base execution, or it collapses.

**Invariant 5 — Diagnostic Admissibility**
A diagnostic is admissible only if the evaluator state is fully resolved, dependency-complete, and bound to a canonical integrity contract before evaluation begins. The IGL is identified not by version alone but by a canonical integrity profile that resolves its required checks, dependencies, and failure semantics. All rule references must be present and fully resolved at the boundary. If any fail, the result is non-admissible diagnostics — not degraded diagnostics. "The system appears stable" is not a meaningful signal unless the evaluator producing that signal is itself admissible.

**The logical chain:**
- Authority exists only if it can be derived (Invariant 1)
- Execution proceeds only from a closed boundary state (Invariants 2–3)
- No alternate path bypasses that state (Invariant 4)
- Diagnostic validity requires evaluator closure (Invariant 5)

Each layer depends on the one below. None is bypassable without collapsing the chain.

**Relationship to existing requirements:**
- R1 (Structural separation) is the organizational precondition for Invariants 3–5. A prompt-level IGL sharing a generation context with the session it governs cannot satisfy closure enforcement or execution exclusivity.
- R3 (Style signal evaluation) operates within the constraints defined by Invariants 2–3. Style evaluation must be complete before evaluation begins — it cannot be resolved during evaluation.
- R4 (Non-generative outputs) is the behavioral expression of Invariant 4. An IGL output that functions as session content is an alternate execution path.
- R5 (Escalation path) must itself satisfy Invariant 5 — the escalation mechanism is a diagnostic output and must be admissible under the same closure requirements.

-----

## 15. Summary

EML transforms the system from:

- answer delivery

→ to

- claim mediation under constraint

It does not solve epistemic distortion.

It makes distortion:

- visible
- interruptible
- less easily mistaken for authority

The IGL (Layer 2.5) makes the EML’s own governance classifications auditable — ensuring that what the DP classifies is correctly calibrated before enforcement acts on it.

-----

*Item 139 — CLOSED. Section 14 appended. Stack updated. Gates: Item 143 (P4 v0.2), EML v0.2 specification.*

*Sovereign humans. Always.*
*Festina lente.*
