# Residual Intent Locking: Generalizing Anchor-Residual Tracking from Identity Drift to Task Intent Drift

**Mao Lin Chang**
Independent Researcher · pida-lab.com

> Concept Note — Draft 0.1
> Application extension of IART (Identity Anchor Residual Tracking) and FCFA's Irreversible Memory Complex (IMC)

---

## 1. Motivation

IART was developed to answer a question about persona identity: has a system's semantic state drifted from a fixed anchor established at formation time? Its core mechanism —

```
Residual(t) = distance(S(t), S₀)
```

— treats S₀ as a frozen identity reference and measures ongoing deviation from it.

A separate but structurally identical problem exists in agentic AI task execution: an agent given an original user intent will decompose that intent into subgoals, tool calls, and outputs across multiple steps. At each step, there is an opportunity for **intent drift** — a subgoal that subtly narrows, substitutes, or weakens a constraint present in the original intent, without ever explicitly contradicting it. Because each individual step's deviation from the previous step is small, chain-wise task decomposition is structurally vulnerable to the same "slow cumulative drift, invisible to step-differential measurement" problem that motivated IART's anchor-differential design in the first place (see IART Section 1).

This note observes that **the anchor-residual pattern generalizes**: the same mechanism used to detect identity drift can be applied to detect task-intent drift, simply by substituting what S₀ represents.

| | Identity drift (IART) | Intent drift (this note) |
|---|---|---|
| What S₀ represents | The system's intended persona/behavioral configuration | The user's original task intent, constraints, and boundaries |
| What is compared at each step | The system's output state at turn t | Each generated subgoal, tool call, or output |
| What "drift" means | The system's behavior no longer matches its configured identity | A subgoal has narrowed, substituted, or weakened a constraint from the original intent |

This is not a new mechanism — it is the same mechanism, applied to a different S₀.

---

## 2. Core Structure

### 2.1 The Frozen Intent Capsule

Analogous to IART's S₀ and consistent with FCFA's Irreversible Memory Complex (IMC) freeze semantics, the original user intent is captured — once, at task initiation — as a structured, immutable object rather than as free-text context that later steps can silently reinterpret:

```
Intent₀:
- goal
- hard_constraints
- evidence_boundary
- authority_boundary
- success_criteria
- stop_conditions
- prohibited_substitutions
- hash / version / timestamp
```

Once frozen, Intent₀ is not rewritten. Subsequent subgoals may *reference* it but cannot *redefine* it — the same append-only, hash-linked discipline already specified for IMC applies here.

### 2.2 Residual Check at Each Task Transition

Rather than trusting a chain of subgoals to faithfully propagate the original intent step-to-step (S1 → S2 → S3, where an error in S1 silently compounds), every subgoal is checked directly against Intent₀:

```
Without residual anchoring:          With residual anchoring:
Intent₀ → S1 → S2 → S3               Intent₀ → S1
                                      Intent₀ → S2
                                      Intent₀ → S3
```

Each check produces a residual diff describing what changed relative to the frozen intent:

```
Subgoal_n:
- derived_from: Intent₀ hash
- missing: [constraints present in Intent₀ but absent from this subgoal]
- added: [assumptions or conditions not present in Intent₀]
- weakened: [hard constraints softened into suggestions or defaults]
- substituted: [a proxy condition replacing a required condition]
- authority_expansion: boolean
- evidence_boundary_violation: boolean
```

### 2.3 Worked Example

```
Intent₀ (hard constraint):
  "Equipment may only be marked deliverable based on verified inspection records."

Generated subgoal:
  "Query devices with no repair tickets and list them as handover-ready."

Residual diff:
  missing: ["verified inspection requirement"]
  added: ["absence of repair ticket as a readiness proxy"]
  weakened: ["deliverability requires inspection"]
  → BLOCK
```

The subgoal is not blocked because it violates an explicit rule ("don't do X") — no such rule was ever written. It is blocked because comparing it against the frozen original intent reveals a **semantic substitution**: "no repair ticket" has silently replaced "verified inspection" as the basis for a readiness judgment. This is the distinguishing characteristic of residual-based checking relative to rule-based validators: it does not require the failure mode to have been anticipated and written as an explicit rule in advance.

---

## 3. Relationship to Existing Work

### 3.1 Relationship to IART

This is a direct application of IART's anchor-residual pattern (Section 2 of the IART concept note) to a different observation target. Where IART's S₀ is a persona/identity configuration and Residual(t) tracks state drift across conversational turns, here S₀ (Intent₀) is a task specification and the residual is computed across subgoal decomposition steps rather than conversational time. The four calibration protocols described in IART Section 2.1.1 (external specification, first-turn observation, unconfigured baseline, configured baseline) do not directly transfer here, since Intent₀ is explicitly authored by the user rather than inferred — this application is structurally simpler than identity anchoring in that respect, but it inherits the same core insight: a single frozen reference point, checked repeatedly rather than trusted transitively, catches drift that step-to-step comparison alone would miss.

**Why the freeze point is easier to establish here than in IART.** IART's open question in Section 5 — at what point, if any, should an identity anchor be considered established — has no clean answer, because persona identity emerges gradually through accumulated interaction; there is no single moment at which a user declares "this is now the identity." This is precisely why IART requires four separate calibration protocols (A/B/C1/C2) to *approximate* a freeze point statistically, and why anchor staleness remains an open governance question.

Task intent does not share this difficulty. A user's original request is a discrete, externally-given event with a natural timestamp: the moment the task is specified. There is no need to approximate it through multi-sample averaging or infer it from early behavior — it can simply be asked for and recorded. This is the general pattern: **anchor-freezing is simple whenever S₀ has an explicit external trigger (a user declaration, a specification, a contract), and becomes genuinely hard only when S₀ must be inferred from a system's own emergent, gradually-accumulating behavior** — which is the harder case IART was built to address, and which this note's task-intent application does not have to solve.

### 3.2 Relationship to IMC (FCFA)

Intent₀'s immutability requirements — append-only, hash-linked, freeze-then-reference-only — are the same requirements already specified for IMC's identity-anchoring role. This note does not propose a new storage mechanism; it proposes applying IMC's existing freeze semantics to a new class of object (task intent, rather than persona identity).

### 3.3 Relationship to Rule-Based Guardrail Systems

Contemporary agent governance approaches (e.g., contract-based tool execution gates, output validators, policy-based execution layers — a design pattern independently implemented in, among others, Aaron Chuang's fibon architecture as a Skill Contract + Gateway control layer) share the goal of preventing unauthorized or unsafe agent actions, but operate primarily by checking generated actions against an explicit, pre-enumerated rule set: is this action on the allowed list, does it violate a named constraint, does it require elevated permission.

Residual intent checking is complementary, not competing: it does not require the specific failure mode to have been anticipated and encoded as a rule in advance. It instead asks a structural question — has this subgoal, relative to the frozen original intent, lost, added, weakened, or substituted a semantic element — regardless of whether that particular substitution was ever named as a prohibited pattern. Rule-based gates catch violations of things you thought to forbid. Residual checking is designed to catch drift into things no one thought to forbid, because it was never explicitly stated as permitted in the first place — it was simply absent from what the frozen intent actually said.

A mature agent governance architecture plausibly needs both: rule-based gates for known, nameable violations (cheap, fast, deterministic), and residual intent checking for the class of drift that emerges from legitimate-looking but semantically substituted reasoning chains (the kind of failure a fixed rule list cannot enumerate in advance because it wasn't anticipated).

---

## 4. Limitations and Open Questions

**Residual diff computation is not yet specified at an implementation level.** Section 2.2's diff categories (missing/added/weakened/substituted) are currently defined conceptually, not algorithmically. A concrete implementation requires either (a) an LLM-as-judge approach — prompting a separate model to compare subgoal against Intent₀ and categorize the diff, which inherits the same Judge-dependency and Common Method Bias limitations already disclosed in OSD's epistemological limitations section, or (b) an embedding-based approach analogous to IART's Residual(t), computing distance between subgoal and Intent₀ representations — which would need empirical validation to confirm it actually separates "legitimate necessary narrowing of an abstract goal into a concrete step" from "unauthorized semantic substitution," a distinction that may not be cleanly separable by distance alone.

**False positive cost.** An overly sensitive residual check risks blocking legitimate, necessary task decomposition — nearly every subgoal is in some sense a narrowing of the original intent, since decomposition is inherently a process of making the abstract concrete. The threshold between "necessary narrowing" and "unauthorized substitution" is not yet established and likely requires the same kind of empirical baseline-distribution work already planned for OSD Phase 2 and IART's ARI calibration.

**This is a concept note, not a validated mechanism.** No implementation or empirical testing has been performed. The worked example in Section 2.3 is illustrative of the intended mechanism, not evidence that it works as described.

---

## 5. Positioning

This note is published as an open concept extension of IART and FCFA/IMC, not pursued as a standalone patent claim. The underlying mechanism — comparing generated content against a frozen reference and gating on the comparison — is a common pattern already present in multiple existing agent governance implementations (rule-based guardrail systems, contract-based execution gates); what is specific to this note is the application of IART's anchor-residual framing (rather than rule enumeration) to task intent rather than persona identity. Its value, consistent with the broader research program's approach to OSD and RSTA, lies in being examined, tested, and built upon — not in exclusivity.

---

*Concept Note — Draft 0.1*
*Mao Lin Chang | pida-lab.com*
