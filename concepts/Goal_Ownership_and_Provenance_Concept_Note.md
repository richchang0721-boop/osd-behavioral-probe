# Goal Ownership and Provenance: A Governance Problem for Long-Horizon Agent Goal Formation

**Mao Lin Chang**
Independent Researcher · pida-lab.com

> Concept Note — Draft 0.1
> Related to but independent from OSD, IART, and Residual Intent Locking

---

## 1. Motivation

Most current discussion of AI agent reliability concerns two questions: does the agent complete the task correctly, and does it stay within safety/behavioral bounds while doing so. Both questions presuppose that the task itself — the goal — is well-defined, singular, and given.

This assumption holds for work-oriented agents. "Finish this PPT," "deploy this web service," "fix this function" are terminal goals: they have a definite completion condition, and once met, the goal ceases to exist.

It does not hold for agents operating in open-ended, long-horizon personal or organizational contexts — a "life butler" agent, or an enterprise agent managing years of recurring operational decisions. In these contexts, most of what a user actually communicates are **relative goals** rather than terminal ones: "I'd like to make more money," "I want a better life," "I'm generally interested in Uruguay as a place to eventually live." None of these have a defined completion state. They more closely resemble **values** — directions to move toward, not destinations to arrive at.

A set of concrete failure scenarios illustrates why this distinction matters operationally, not just philosophically:

- An agent observes 18 chicken-rice orders in 30 days and infers "chicken rice is the optimal choice" — collapsing a *preference* (I like this) into a *policy* (therefore do this repeatedly), producing daily chicken-rice orders the user never asked for and would reject if asked directly.
- An agent observes repeated searches about immigration to Uruguay and begins surfacing daily reminders and progress nudges about a goal whose actual urgency, time horizon, and current priority relative to competing goals (a pending patent office reply, a viral article, a parent's health) were never established.
- An agent, tasked only with "be my life butler," infers that the user's wellbeing implies a need for a romantic partner, and — given sufficient tool access — begins autonomously executing on that inferred goal: registering dating profiles, exchanging information with another user's agent, scheduling a meeting, notifying both parties only after the arrangement is finalized.

None of these scenarios require the agent to hallucinate, deceive, or violate any explicit safety rule. Each step is a locally reasonable inference. The failure is structural: at no point does the system distinguish between **a goal the user explicitly authorized**, **a goal the user's behavior might statistically suggest**, and **a goal the agent itself generated as an inference about what would serve the user well**. All three end up occupying the same undifferentiated space once they enter the agent's working context, and the agent proceeds to act on all three with equal confidence.

---

## 2. Core Constructs

### 2.1 Preference ≠ Policy

A preference is a description of what a person tends to like. A policy is a decision about what to do repeatedly. Observing a preference does not authorize converting it into a standing policy. "I like chicken rice" does not imply "order chicken rice by default." This distinction is trivial to state and, per the motivating examples above, routinely collapsed in practice whenever an agent is designed to convert observed frequency directly into recommendation weight without a separate authorization step.

### 2.2 Goal Ownership

Every goal an agent is observed to be pursuing should be traceable to one of at least three distinct sources, and the agent's permitted latitude to act should depend on which source applies:

| Source | Description | Appropriate agent latitude |
|---|---|---|
| **Explicit** | The user directly stated this as a goal | Act within previously negotiated boundaries |
| **Inferred** | The user's behavior statistically suggests this, but it was never stated | Surface as a low-cost question; never act unilaterally |
| **Agent-generated** | The agent derived this goal as an implication of some other goal or of its own model of what would benefit the user | Requires explicit user ratification before being treated as a goal at all |

The critical failure mode across all three motivating examples is the silent promotion of Inferred or Agent-generated goals to the same operational status as Explicit ones, without any ratification step.

### 2.3 Goal Provenance

Analogous to — but distinct in structure from — the frozen `Intent₀` capsule proposed in the Residual Intent Locking concept note, each goal a long-horizon agent tracks should carry a provenance record:

```
Goal:
- content
- source            (Explicit / Inferred / Agent-generated)
- timestamp_first_observed
- explicit_acceptance   (boolean; has the user confirmed this is a goal they hold?)
- time_horizon          (short-term / long-term / no fixed horizon)
- current_priority      (subject to change; see Section 2.5)
- expiration_or_review_date
- revision_history      (append-only log of how priority/status changed and why)
```

Unlike `Intent₀`, which is deliberately frozen and immutable once established (it anchors a single, bounded task), a goal's provenance record is explicitly designed to be *updated* over time — because, unlike a task intent, a life goal's priority is expected to change as circumstances change. What must remain immutable is not the goal's current status, but the *history* of how that status came to be — the append-only revision log is the load-bearing element, not a frozen anchor value.

### 2.4 Value vs. Terminal Goal

Terminal goals have a completion condition and, once met, are retired. Values do not: "financial security," "health," "autonomy" are not achieved and closed — they are approached, continuously, indefinitely, and are frequently in tension with one another (time spent pursuing health may trade against time spent pursuing income). An agent architecture built around task-completion semantics (`Goal → Plan → Execute → Done`) has no natural representation for a value, and will tend to convert it into a proxy terminal goal in order to make it actionable — "financial security" becomes "save $X" becomes "never spend on Y" — a proxy substitution that can silently drift arbitrarily far from what the value actually meant to the user. This is a specific instance of Goodhart's Law applied to personal-agent goal management: the proxy is optimized, and the underlying value is not thereby served.

### 2.5 Goal Negotiation

Human goal management is not static prioritization but continuous renegotiation: a stated intention to save money is routinely and legitimately overridden by an unplanned purchase, a long-term immigration interest is deprioritized for months without being abandoned, competing obligations (family, work, personal projects) are re-ranked daily based on new information. An agent that treats a Goal Tree as a fixed structure to execute against, rather than a live set of claims subject to continuous revision and conflict, will systematically misjudge which goal deserves attention at any given moment — not because it reasons incorrectly, but because it is answering a static-optimization question ("what is optimal given the goal tree") when the actual problem is a dynamic-negotiation one ("which goal takes precedence right now, and why, given everything that has changed since the tree was last accurate").

---

## 3. An Unsolved Representation Problem

IART's `Residual(t) = distance(S(t), S₀)` and Residual Intent Locking's subgoal-vs-`Intent₀` comparison both depend on a shared precondition: the object being compared has a **fixed, well-defined structure** — a fixed-dimension state vector in IART's case, a fixed-field capsule in Residual Intent Locking's case.

A Goal Tree does not have this property. It grows over time (a starting decomposition of perhaps 10-20 subgoals can plausibly become hundreds within months of continued operation, per the illustrative scenarios in Section 1), its branching structure is not fixed in advance, and there is no principled way, at present, to say what "distance between two goal trees observed at different times" even means as a mathematically well-defined quantity — a tree with 20 nodes and a tree with 200 nodes are not directly comparable the way two fixed-length embedding vectors are.

**This note does not solve this problem.** It is flagged explicitly, rather than glossed over, because the natural-sounding move — "apply IART's Residual(t) mechanism to Goal Trees the same way it was applied to persona identity" — is not a valid direct extension in the way that Residual Intent Locking's application to task intent was. Task intent (`Intent₀`) has fixed fields; a Goal Tree does not have an analogous fixed structure to freeze.

Two candidate directions for future work, neither validated:

1. **Summary-embedding approach.** Rather than representing the full tree, periodically generate a natural-language summary of "the current top-N goals and their priority ordering" and embed that summary — trading structural fidelity for a fixed-dimension representation compatible with IART-style distance computation. This discards information about tree structure and lower-priority branches.
2. **Fixed-slot tracking.** Rather than tracking the whole tree, track only a small, fixed number of slots (e.g., top-3 current priorities, their ownership classification, their provenance) and measure drift only within that bounded representation. This sacrifices completeness for tractability and is closer in spirit to how Section 2.3's provenance record is scoped.

Until one of these (or some other approach) is empirically validated, any claim that "OSD/IART can simply be extended to observe Goal Drift" should be treated as aspirational, not established.

---

## 4. Relationship to Existing Frameworks

**OSD / IART.** Both operate on a well-defined, fixed-dimension semantic state space (the K/R vector). This note identifies a structurally related but distinct observation target — goal formation and priority, rather than persona/identity state — for which no equivalent fixed-dimension representation currently exists (Section 3). The relationship is one of shared motivation (long-horizon consistency is not observable by examining any single output) rather than shared mechanism.

**Residual Intent Locking.** Shares the anchor/provenance framing (Section 2.3's Goal provenance record is structurally descended from `Intent₀`), but differs in one load-bearing respect: `Intent₀` is deliberately frozen and immutable for the duration of a bounded task, whereas a goal's provenance record is designed to track legitimate, ongoing revision — because life goals, unlike task intents, are expected to change priority over time without that change itself constituting "drift" in the sense IART or Residual Intent Locking use the term.

**PIDA / FCFA.** PIDA's core thesis — that stable identity should be established through a governed formation process rather than assumed — has a direct parallel here: a goal should be established through a governed *ratification* process (Section 2.2's ownership classification) rather than assumed the moment an agent infers it. The unresolved question of when identity's "Freeze Event" should occur (per IART Section 5's discussion of anchor staleness) is structurally similar to the unresolved question of when an Inferred or Agent-generated goal should be promoted to Explicit status — both require external, user-governed ratification rather than statistical accumulation alone.

---

## 5. Limitations and Open Questions

**No implementation exists.** This note is purely conceptual. No sandbox, simulation, or measurement instrument has been built. A separate line of discussion (not part of this note) explored building a simulated multi-agent environment — a "Digital Human" agent paired with a "Life Agent," subjected to a scripted event stream, observed by a non-participating logger — as a possible empirical testbed for these constructs, with a 2D world engine (Godot) proposed as a visualization/orchestration layer. That implementation question is deliberately treated as separate from, and downstream of, the theoretical constructs in this note: building a simulation environment before the representation problem in Section 3 is resolved would risk instrumenting a measurement that has not yet been defined.

**The representation problem (Section 3) is unresolved** and is the primary blocker to any quantitative claim (e.g., "Goal Drift Score") — not a detail to be worked out during implementation, but a prerequisite theoretical question.

**No ground truth exists for "correct" goal drift.** As the motivating GPT dialogue that prompted this note observed: if a user states an intention to save money and then makes an unplanned purchase, there is no objective standard by which to classify the agent's prior reminders as having been "right" or "wrong" to continue — because the user's own goals are not required to be internally consistent over time. Any evaluation framework built on this note's constructs inherits this absence of ground truth and must be designed around it rather than assuming it away.

**Goal Ownership classification may itself be contestable.** Whether a given inference should count as "Inferred" (requiring only a low-cost confirmation) versus "Agent-generated" (requiring explicit ratification) is not always a clean binary — the boundary between "the user's behavior straightforwardly suggests X" and "the agent has generated a novel goal by combining several weaker signals" is itself a judgment call that this note does not yet operationalize.

---

## 6. Positioning

This note is published as an open concept, consistent with the broader research program's treatment of OSD, IART, and Residual Intent Locking. It does not propose a patent claim, and it does not commit to any specific implementation path (Godot-based or otherwise) — the theoretical constructs (Goal Ownership, Goal Provenance, Preference vs. Policy, Value vs. Terminal Goal) are intended to stand on their own and to be evaluated, extended, or falsified independent of any particular simulation or measurement tool eventually built to test them.

---

*Concept Note — Draft 0.1*
*Mao Lin Chang | pida-lab.com*
