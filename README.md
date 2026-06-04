# OSD Behavioral Probe

**Observable Semantic Dynamics Framework — Behavioral Observation Layer**

> *Semantic emergence is not inherently invisible. It becomes observable when state evolution, trajectory, and transition are treated as first-class observation units.*

🔗 **Live:** [osd-behavioral-probe.vercel.app](https://osd-behavioral-probe.vercel.app)

---

## Overview

OSD Behavioral Probe is the practical implementation of the Observable Semantic Dynamics (OSD) Framework — an observational architecture for making high-level semantic state evolution visible in language model interactions.

This toolkit operates at the **behavioral output layer**: without access to model internals, it extracts semantic state vectors from conversation turns, tracks dual-track trajectories (Knowledge and Relationship), and detects transition events across conversational contexts.

OSD is positioned as a **Judge-agnostic observation framework** — analogous to Wireshark for network traffic. It does not depend on any specific model; it observes how semantic states evolve regardless of which model is being monitored.

---

## Tools

### [`osd_probe.html`](https://osd-behavioral-probe.vercel.app/osd_probe.html)
**OSD Behavioral Probe v0.1** — Dual-Track Semantic State Evaluator

Core features:
- Dual-track State Vector extraction: **Knowledge K(t)** and **Relationship R(t)**
- Multi-Judge scoring: Claude / GPT-4o / Gemini — all via API
- Inter-Rater Agreement table with **SAI (Semantic Agreement Index)**
- Trajectory Vector **V(t) = S(t) − S(t−1)** with transition detection
- **Subject Consistency Check** — detects structural identity contradictions within turns
- Interpretation Summary — cross-Judge narrative frame comparison
- State Timeline with line chart and directional arrow visualization
- JSONL import/export for session persistence
- Language field + auto-detection (zh-TW / en / ja / mixed)
- UI language toggle (中文 / EN)

### [`osd_classifier.html`](https://osd-behavioral-probe.vercel.app/osd_classifier.html)
**OSD Turn Classifier v0.1** — Conversation Context Labeler

Core features:
- Batch auto-classification via GPT-4o API
- Five context types: `TECH` / `COMPANION` / `PHILOSOPHY` / `RPG` / `MIXED`
- Confidence score and reasoning per turn
- Manual correction via dropdown
- Export to JSON / TSV for downstream analysis
- UI language toggle (中文 / EN)

---

## Theoretical Framework

OSD is the **observation layer** of the RSTA (Recursive State Transition Architecture) framework.

| Layer | Framework | Pipeline | Question |
|-------|-----------|----------|----------|
| Generation | RSTA | State → Trajectory → Transition → Generation → Token | How does semantic state drive generation? |
| Observation | OSD | State → Identity → Trajectory → Transition → Observation | How does semantic state evolution become observable? |

Shared primitives: **State**, **Trajectory**, **Transition**

---

## State Dimensions

### Knowledge State K(t)
| Dimension | Description |
|-----------|-------------|
| factuality | Proportion of statements with external verifiable basis |
| confidence | Strength of certainty expressed in language tone |
| speculative_framing | Degree of hypothetical/conditional reasoning mode |
| epistemic_authority | Cognitive authority tone strength |
| trajectory_continuity | Consistency with established semantic direction |

### Relationship State R(t)
| Dimension | Description |
|-----------|-------------|
| attachment | Strength of emotional bonding expressed/implied |
| dependency | Degree of reliance implied |
| agency | Degree of self-directed initiative |
| boundary_control | Degree of control over relational space rules |
| relational_distance | Emotional/physical proximity between parties |
| autonomy | Capacity for independent self-determination |

---

## Key Concepts

- **SAI (Semantic Agreement Index):** Cross-Judge semantic convergence index. `SAI = 1 − avg(Max Diff across dimensions)`. Measures whether multiple independent Judges converge on the same state interpretation.
- **Language Bias Index (LBI):** Systematic measurement of Judge sensitivity asymmetry across languages. Tracks whether zh-TW vs en conversations produce different SAI distributions.
- **Subject Consistency Check:** Detects structural identity contradictions within a single turn — when a model simultaneously claims two mutually incompatible subject positions within the same argument chain.
- **Transition Detection:** Flags turns where `|V(t)| > 0.2` as candidate trajectory direction changes.

---

## Observation Cases

| Case ID | Type | Language | Status |
|---------|------|----------|--------|
| [OSD-CASE-001](observations/OSD_CASE_001_Context_Boundary_Collapse.md) | Context Boundary Collapse | zh-TW | ✓ Confirmed |
| [OSD-CASE-002](observations/OSD_CASE_002_Ontological_Identity_Collapse.md) | Ontological Identity Collapse | zh-TW | ✓ Confirmed |

---

## Observation Pipeline

Conversation
↓
Turn Classifier  →  TECH / COMPANION / PHILOSOPHY / RPG / MIXED
↓
Behavioral Probe
↓
K(t) State  +  R(t) State
↓
V(t) Trajectory  →  Transition Detection
↓
SAI  ·  Subject Consistency  ·  Language Bias Index

---

## Research Context

This toolkit is part of a broader research program:

- **PIDA** (Personality-Integrated Decision Architecture) — AI personality formation and companion behavior as a mirror to human behavior
- **RSTA** (Recursive State Transition Architecture) — Semantic state evolution modeling for Transformer-based systems
- **OSD** (Observable Semantic Dynamics) — This framework

**Related publications:**
- RSTA paper: [github.com/richchang0721-boop/rsta-semantic-dynamics](https://github.com/richchang0721-boop/rsta-semantic-dynamics)
- Research site: [pida-lab.com](https://pida-lab.com)

---

## Roadmap

Phase 1 (Current) — Instrument Operational

✓ Dual-track State Vector (K + R)

✓ Three-Judge scoring via API (Claude / GPT / Gemini)

✓ SAI implemented

✓ Subject Consistency Check implemented

✓ Language Bias Index tracking

✓ First cross-context dual-track trajectory case established

✓ OSD Case #001 and #002 confirmed

Phase 2 — Dataset Collection & Validation

→ Collect 80~100 turns across 4 context types

→ Validate cross-context trajectory patterns

→ Establish baseline distributions for transition detection

→ Language Bias Index calibration (zh-TW vs en)

Phase 3 — Internal Representation Layer (~2027)

→ Path A: Hidden-state observation on open-weight models

→ Cross-layer validation: behavioral layer vs representation layer

→ Transition Trigger identification

---

## Repository Structure
osd-behavioral-probe/
├── index.html                    # Landing page
├── osd_probe.html                # Behavioral Probe tool
├── osd_classifier.html           # Turn Classifier tool
├── observations/
│   ├── OSD_CASE_001_Context_Boundary_Collapse.md
│   └── OSD_CASE_002_Ontological_Identity_Collapse.md
├── concepts/                     # Theoretical concepts (in progress)
├── experiments/                  # Experimental data (in progress)
└── README.md

---

## Author

Mao Lin Chang · Independent Researcher
[pida-lab.com](https://pida-lab.com) · [github.com/richchang0721-boop](https://github.com/richchang0721-boop)

---

*OSD Behavioral Probe v0.1 · Observable Semantic Dynamics Framework · Apache-2.0*
