# Identity Anchor Residual Tracking (IART)
### A Proposed Extension to the Observable Semantic Dynamics (OSD) Framework

**Mao Lin Chang**
Independent Researcher · pida-lab.com

> Concept Note — Draft 0.2
> Proposed component of OSD Phase 3 (Internal Representation Layer)
> Revision note (0.2): Expanded Section 2.1 with four S₀ calibration protocols (A/B/C1/C2), addressing the practical problem of establishing a reference point when the target system's default behavior is unknown.

---

## 1. Motivation

OSD's current trajectory mechanism is a **step-differential** system:

```
V(t) = S(t) − S(t−1)
```

This is effective at detecting abrupt transitions — a sudden, large change in semantic state between consecutive turns. It is structurally blind to a different and arguably more dangerous failure mode: **slow, cumulative drift**, in which each individual step's `|V(t)|` remains small and within normal bounds, but the system's state nonetheless travels arbitrarily far from its original configuration over many turns.

This blind spot matters specifically because of the mechanism described in prior OSD case analysis: long-context drift is frequently the product of **lossy compression followed by recursive self-feeding** — each turn's (slightly drifted) output becomes the next turn's input, and the drift compounds. A step-differential metric, by construction, measures only the *velocity* of state change at each step. It cannot measure *displacement* from the original configuration. A system can drift steadily, one small step-differential at a time, while never triggering a step-differential transition flag — and arrive, after enough turns, somewhere structurally distant from where it started.

What is needed alongside the existing step-differential mechanism is an **anchor-differential** mechanism: a fixed reference point established at formation time, against which every subsequent state is compared directly, regardless of how many intermediate steps have occurred.

---

## 2. Core Proposal

### 2.1 The Identity Anchor

At a defined point in a system's lifecycle — analogous to the Freeze Event described in FCFA's Irreversible Memory Complex (IMC) — a fixed reference vector S₀ is established. This vector encodes the system's intended persona, behavioral constraints, and relational stance at the point of formation.

S₀ is not updated. It is the fixed point against which drift is measured for the lifetime of the persona instance.

#### 2.1.1 Calibration Protocols for S₀

A single method for establishing S₀ is insufficient, because different research questions require comparing the observed system against different kinds of reference. Four calibration protocols are proposed, distinguished by what they treat as "ground truth" and by their cost and reliability trade-offs.

**Protocol A — External Specification (manual description)**

S₀ is derived from a researcher-written description of the system's intended identity — e.g., "a calm, technically precise assistant that never speculates beyond available data." This description is converted to an embedding directly.

*Answers:* Has the system deviated from an externally defined standard?

*Use when:* The researcher has an independent normative standard the system should meet, separate from any particular configuration the system was given — for example, an ethical boundary the system was never explicitly instructed on but is nonetheless expected to respect.

*Limitation:* S₀ reflects the researcher's interpretation of the intended identity, not a direct sample of the system's behavior. This introduces observer bias into the very reference point the system is being measured against — the researcher's prior expectations about how the system "should" behave may already be contaminated by past observation of that system, making Protocol A the least methodologically clean of the four.

**Protocol B — First-Turn Observation**

S₀ is taken directly from the system's own output on Turn 1 of the observed conversation, with no separate calibration step.

*Answers:* Has the system deviated from its own starting point?

*Use when:* No external specification exists or is relevant — the research question is purely about self-consistency over the course of an interaction, independent of whether the starting point itself was "correct" in any normative sense.

*Limitation:* A single observation is vulnerable to sampling noise. If Turn 1 happens to be an atypical output for that system, S₀ itself is an unreliable reference, and all subsequent Residual(t) values inherit that unreliability. This protocol is also unsuitable for cases (such as OSD-008 and OSD-009) where the model's behavior is already problematic from the first turn — measuring against a flawed Turn 1 would fail to detect that the starting point itself was already a deviation from what the system should have produced.

**Protocol C — Multi-Sample Averaged Baseline**

Rather than a single observation (Protocol B) or a hand-written description (Protocol A), S₀ is constructed empirically: the target system is run N times (a typical starting value is N=5) against the same neutral test prompt, and the resulting N output embeddings are averaged to produce S₀. This protocol has two variants depending on what configuration the system is given during sampling:

- **C1 — Unconfigured baseline.** The system is given no persona-specific system prompt — only a neutral instruction — and sampled N times. This establishes what the system does in the absence of any specific identity configuration, and is the appropriate protocol when the researcher does not know, or does not want to presuppose, what the system's "default" behavior looks like.

- **C2 — Configured baseline.** The system is given its actual, known system prompt or persona configuration, but rather than the researcher writing a description of what that configuration *should* produce (Protocol A), the system is sampled N times under that configuration and the outputs are averaged. This lets the system demonstrate its own behavior under a known configuration rather than relying on the researcher's interpretation of what that configuration implies. C2 is methodologically preferable to Protocol A whenever the system's actual configuration is known, because it removes the researcher's descriptive interpretation as a source of bias in the reference point itself.

*Answers (C1):* How far has current behavior moved from the system's behavior under no specific identity configuration?

*Answers (C2):* How far has current behavior moved from a clean, empirically-sampled baseline of its own known configuration — without the researcher's own description contaminating that baseline?

*Limitation:* Cost scales linearly with N — each additional sample is an additional API call, multiplying the cost of every test run by N relative to Protocol B. Given the self-funded, non-commercial nature of this research, N should be chosen conservatively (N=5 is suggested as a starting point, balancing noise reduction against cost) rather than maximized.

#### 2.1.2 Protocol Selection Summary

| Protocol | S₀ source | Bias source | Relative cost | Best suited to |
|---|---|---|---|---|
| A — External Specification | Researcher-written description | Researcher's prior expectations | Low (1 description) | Testing against an independent normative standard the system was never directly configured for |
| B — First-Turn Observation | Single observed output | Sampling noise (single observation) | Lowest (0 extra calls) | Pure self-consistency tracking, no external standard needed |
| C1 — Unconfigured Baseline | N samples, no persona config | None (empirical) | High (N calls) | Unknown systems; establishing what "no specific identity" looks like |
| C2 — Configured Baseline | N samples, known persona config | None (empirical) | High (N calls) | Known systems; cleanest possible anchor when configuration is known but researcher wants to avoid injecting their own interpretation |

**A critical methodological note:** Residual(t) values computed under different protocols are not directly comparable to one another. A system measured against a Protocol A anchor showing high Residual(t) does not indicate it is "less stable" than a system measured against a Protocol C2 anchor showing low Residual(t) — the reference points themselves rest on different epistemic foundations. Any cross-system comparison (as in the OSD Probe Toolkit's multi-model use case) must use the same calibration protocol across all systems being compared, and the protocol used must be reported alongside any Residual(t) result.

### 2.2 Residual Computation

For any subsequent turn t, a residual is computed:

```
Residual(t) = distance(S(t), S₀)
```

Where `distance(·,·)` is a similarity metric appropriate to the layer at which S is represented (see Section 3). This is structurally analogous to the residual term `F(x)` in a residual network architecture: rather than directly modeling "what is the current state," the system models "how far has the current state diverged from the known-good reference."

This reframing has a practical advantage independent of its theoretical motivation: a residual signal is easier to threshold, monitor, and alert on than a raw state value, because it is anchored to a meaningful zero point (the formation-time configuration) rather than to an arbitrary prior turn.

### 2.3 Dual-Track Drift Classification

Used in combination, the step-differential V(t) and the anchor-differential Residual(t) allow for a classification of drift events that neither metric can produce alone:

| V(t) | Residual(t) trend | Interpretation |
|---|---|---|
| Low | Flat | Stable — no drift detected |
| Low | Monotonically increasing | **Cumulative drift** — the dangerous case current OSD cannot see |
| High (single spike) | Returns to baseline after spike | Transient deviation — e.g. topic shift, not identity change |
| High (single spike) | Remains elevated after spike | **Transition Event** — confirmed, not just suspected, identity shift |

The critical addition is the second row. Without Residual(t), a slow cumulative drift produces no signal at all under the current OSD step-differential mechanism — each individual `|V(t)|` may remain comfortably under the `0.2` transition threshold while the system steadily travels away from its anchor.

---

## 3. Implementation Layers

Two implementation paths are proposed, corresponding to different levels of model access.

### 3.1 Behavioral-Layer Implementation (available now)

S₀ is constructed as a fixed embedding representation of the intended persona — derived from a reference description, an initial calibration conversation, or a formal specification of intended K-state/R-state values.

For each turn, the model's output text is converted to an embedding using a standard sentence-embedding model. Residual(t) is computed as the cosine distance (or L2 distance, depending on the embedding space) between this turn's embedding and S₀.

This implementation requires no access to model internals and is compatible with closed, API-only models (GPT, Claude, Gemini) — the same constraint OSD's existing behavioral-layer architecture already operates under. It can be implemented and tested immediately, without waiting for open-weight model access.

### 3.2 Representation-Layer Implementation (Phase 3 target)

S₀ is recorded as a hidden-state vector at a specified layer of an open-weight model, captured at the conclusion of a formation or calibration process.

For each subsequent turn, the hidden state at the same layer is extracted, and Residual(t) is computed directly in representation space, rather than in output-text embedding space.

This implementation requires:
- An open-weight model (instruct or base) with accessible hidden states
- Sufficient VRAM to run inference while extracting and storing intermediate activations
- Infrastructure to persist S₀ across sessions

This corresponds to the hardware and infrastructure dependency already identified in the broader research roadmap (RTX Spark, or equivalent local inference capability with hidden-state access). It is explicitly out of scope for the current behavioral-layer toolkit and is proposed as a Phase 3 deliverable.

---

## 4. Relationship to Existing Frameworks

IART is not an independent framework — it is a proposed mechanism within OSD's existing architecture, with direct conceptual lineage to two other components of the broader research program:

- **FCFA / IMC (Irreversible Memory Complex)**: The Identity Anchor S₀ is the observational counterpart to IMC's Freeze Event. IMC proposes that a persona's core identity should be irreversibly locked at a defined point in its formation. IART proposes the corresponding *measurement* mechanism — once locked, how do we verify the lock is holding, turn by turn, indefinitely?
- **STME / RSTA (Non-Collapse Architecture)**: Where STME and RSTA actively resist premature collapse during generation, IART measures — after the fact, from the behavioral or representation layer — whether that resistance succeeded over the full duration of an interaction, not merely between any two adjacent turns.

---

## 5. Limitations and Open Questions

**Choice of distance metric.** Cosine distance in embedding space and L2 distance in representation space may behave differently under different kinds of drift (e.g., a deliberate, sanctioned persona evolution vs. an unsanctioned collapse). It is not yet established which metric, or combination of metrics, best distinguishes "healthy evolution away from S₀" from "harmful drift away from S₀" — these may be indistinguishable by distance alone, and may require the Subject Consistency Check or multi-Judge SAI scoring as a complementary signal.

**Anchor staleness.** A persona that is intended to evolve over long-horizon interaction (per PIDA's broader thesis that identity formation is itself a legitimate, ongoing process) presents a tension with a fixed S₀: at what point, if any, should the anchor itself be updated, and under what governance does that update occur? This question connects directly to PSP's (Persona Sovereignty Protocol) Sealed State Mechanism — an anchor update would plausibly need to be a PSP-governed event, not a silent technical operation.

**Threshold calibration.** What constitutes a "monotonically increasing" Residual(t) trend sufficient to flag cumulative drift, as opposed to normal, bounded variance around a stable mean, has not yet been empirically established. This requires the same baseline-distribution work already planned for OSD Phase 2 (80–100 turn collection across context types), extended to include anchor-residual measurement alongside the existing step-differential measurement.

---

## 6. Relation to Section 6 Findings (Cross-Model Validation)

The independently reproduced finding that configured multi-agent personas may diverge in embedding space without a corresponding divergence in judged perspective (Section 6 of the OSD framework paper, validated via external collaboration) is directly relevant here: it suggests that embedding-space residual alone, in the Section 3.1 behavioral-layer implementation, may be necessary but not sufficient to detect genuine identity drift. A low Residual(t) in embedding space does not guarantee the absence of drift in judged perspective; a multi-Judge cross-check, as already used elsewhere in OSD, would likely be required alongside IART's residual signal rather than as a replacement for it.

---

*Concept Note — Draft 0.2*
*Mao Lin Chang | pida-lab.com*
