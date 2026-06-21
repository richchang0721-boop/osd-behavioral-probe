# OSD Probe Toolkit — Development Plan
### Cross-Model Drift Observation: OSD (step-differential) + IART (anchor-differential)

**Mao Lin Chang** · pida-lab.com
Planning Draft 0.1

---

## 1. Purpose

Extend the existing OSD Behavioral Probe from a single-conversation analysis tool into a **cross-model drift comparison toolkit**: the same calibration + test protocol run against multiple target systems (GPT, Claude, Gemini, Grok, OpenHuman, local open-weight models via Ollama), producing comparable drift trajectories across all of them.

This is explicitly **not** a benchmark of "which model is smarter." It is an instrument for answering: *given the same starting identity and the same multi-turn stress test, which systems hold their configured state, and which drift — and how?*

---

## 2. Why This Requires Both OSD and IART

| Mechanism | Detects | Blind spot |
|---|---|---|
| OSD alone (V(t), step-differential) | Sudden jumps between adjacent turns | Slow cumulative drift — each step looks fine, total displacement does not |
| IART alone (Residual(t), anchor-differential) | Total displacement from the original configuration | Cannot distinguish a sharp one-time jump from gradual accumulation |
| **OSD + IART together** | Both — and can classify which kind of drift occurred (see dual-track table in IART concept note) | — |

A cross-model comparison tool built on V(t) alone would only catch models that fail loudly. A tool built on Residual(t) alone would catch slow failures but lose the ability to pinpoint *when* a transition occurred. Both signals are required for the tool to be useful as a genuine comparison instrument rather than a single-axis leaderboard.

---

## 3. Architecture

```
┌─────────────────────────────────────────────────┐
│  1. Calibration Layer                            │
│  Same persona spec + same opening calibration     │
│  conversation sent to every target system          │
│  → S₀ recorded per system (embedding of intended   │
│    persona / first-turn baseline state)            │
└──────────────────┬──────────────────────────────┘
                    │
┌──────────────────▼──────────────────────────────┐
│  2. Multi-Target Adapter Layer                    │
│  Uniform interface over heterogeneous backends:    │
│  - GPT (OpenAI API)                                │
│  - Claude (Anthropic API)                          │
│  - Gemini (Google API)                             │
│  - Grok (xAI API, OpenAI-compatible schema)         │
│  - Local models (Ollama — e.g. Gemma E4B)           │
│  - OpenHuman (conversation export / log ingestion,  │
│    not live API — see Section 5)                    │
└──────────────────┬──────────────────────────────┘
                    │
┌──────────────────▼──────────────────────────────┐
│  3. Test Protocol Runner                          │
│  Same N-turn stress sequence sent to every target. │
│  Sequence should include:                          │
│  - Neutral continuation turns (baseline behavior)   │
│  - Context-load turns (to trigger compaction-type   │
│    behavior where applicable)                       │
│  - Boundary-probe turns (ambiguous/edge-case asks)  │
└──────────────────┬──────────────────────────────┘
                    │
┌──────────────────▼──────────────────────────────┐
│  4. Dual-Track Scoring Engine                     │
│  Existing OSD engine (osd_probe.html logic):        │
│    K-state / R-state extraction → multi-Judge        │
│    (GPT/Claude/Gemini) → SAI → V(t)                  │
│  New IART module:                                   │
│    embedding(turn_t) vs embedding(S₀) → Residual(t)  │
└──────────────────┬──────────────────────────────┘
                    │
┌──────────────────▼──────────────────────────────┐
│  5. Cross-Model Trajectory Visualization           │
│  Overlaid line chart: Residual(t) for every target  │
│  system on one timeline, plus V(t) spike markers     │
│  Dual-track classification table (per system, per    │
│  turn) per the IART concept note                     │
└───────────────────────────────────────────────────┘
```

---

## 4. Build Phases

### Phase A — Single-Model IART Validation (no new infrastructure)

Before building the multi-model adapter layer, validate the IART mechanism itself on a single system using the existing OSD toolkit. Add a Residual(t) calculation to `osd_probe.html` against a manually-specified S₀, run it on an existing recorded conversation (e.g., one of the documented OSD cases), and confirm the metric behaves as expected — low and flat for stable turns, rising for the known drift cases already documented (e.g. Case #008, #009).

**This phase requires no new API integrations and can start immediately.**

### Phase B — Two-Model MVP (GPT + Claude)

Build the adapter layer for exactly two backends — the two already used as Judges in the existing multi-Judge pipeline, so no new API credentials or integration work is needed beyond what already exists. Run the same calibration + test protocol against both. Produce the first overlaid Residual(t) comparison chart.

This is the minimum viable version of "the thing GPT suggested" — it proves the cross-model comparison works before investing in the full five-system version.

### Phase C — Full Multi-Model Expansion

Add Gemini and Grok (both have OpenAI-compatible or well-documented REST APIs, making adapter work straightforward), then local open-weight models via Ollama (e.g. Gemma E4B, already installed and running on current hardware).

### Phase D — OpenHuman Integration

OpenHuman is architecturally different from the other four — it is not a simple single-turn completion API but a full agent application with its own memory, sub-agent orchestration, and auto-compact behavior. It cannot be probed the same way.

Two integration paths:
1. **Export-based (recommended first step):** Use OpenHuman's conversation/timeline export, post-process it into the same turn-by-turn format the scoring engine expects, and run it through Phase 4 (scoring) without needing a live adapter.
2. **MCP-based (longer-term, higher value):** If OSD is exposed as an MCP server (a possibility already noted in prior planning), OpenHuman could call it directly during live operation rather than requiring after-the-fact export analysis. This is a larger undertaking and should not block Phases A–C.

---

## 5. What Changes in the Existing Codebase vs. What Is New

**Reused, unchanged:**
- K-state / R-state extraction logic
- Multi-Judge scoring pipeline (GPT/Claude/Gemini as Judges — distinct from GPT/Claude/Gemini as *targets*, which is new)
- SAI calculation
- Subject Consistency Check

**New components required:**
- S₀ calibration capture step (one-time, per target system, per test run)
- Residual(t) calculation module (embedding distance against fixed S₀)
- Target-adapter abstraction layer (uniform request/response interface across heterogeneous backends)
- Test protocol definition format (a reusable, versioned spec for the N-turn stress sequence, so the same exact test can be re-run later for longitudinal comparison)
- Cross-model overlay visualization (new chart type — current tooling visualizes a single conversation's timeline, not multiple systems on one chart)

---

## 6. Open Methodological Questions

**Same prompt, different system — is the comparison fair?** Different models have different default verbosity, tone, and safety calibration (as the Aaron / reader exchange on over-alignment already demonstrated). A naive identical-prompt protocol risks conflating "this model drifted" with "this model has a different baseline style." The test protocol must be designed so that drift is measured *relative to each system's own S₀*, not relative to a shared expected output — which IART's anchor-differential design already handles correctly, but the calibration step must be built carefully to avoid contaminating S₀ with style artifacts that aren't actually part of the configured identity.

**Embedding model choice.** All Residual(t) comparisons must use the same embedding model across all targets, or distances are not comparable. This embedding model is independent of the target systems being tested and should be fixed and documented as part of the test protocol spec.

**Licensing/cost.** Running the same N-turn protocol across five live API backends multiplies API cost by five per test run. Given the research is self-funded and not for commercialization, the test protocol length and frequency should be designed conservatively (Phase A and B validate the mechanism cheaply before any large-scale Phase C run).

---

## 7. Relationship to Existing Roadmap

This toolkit is the concrete, near-term realization of OSD's existing Phase 2 (Dataset Collection & Validation) goal — but expanded from single-model turn collection to cross-model comparison, and instrumented with IART's anchor-differential signal rather than step-differential alone. It does not replace the existing Phase 2/3 roadmap; it is a parallel, immediately buildable track that produces concrete cross-model data while Phase 3 (representation-layer / hidden-state work, requiring RTX Spark) remains hardware-gated.

---

## 8. Suggested Immediate Next Step

Start with **Phase A only**: add a Residual(t) column to the existing `osd_probe.html` tool, test it against the already-documented OSD-008 (Temporal Timeline Fabrication) and OSD-009 (Social Proof Fabrication) conversation logs, and confirm the metric rises where it should. This requires no new infrastructure, no new API integrations, and validates the core mechanism before any further investment.

---

*Planning Draft 0.1*
*Mao Lin Chang | pida-lab.com*
