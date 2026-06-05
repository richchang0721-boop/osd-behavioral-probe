# Simulated Affective Response (SAR)

**Type:** Core Mechanism  
**Layer:** Behavior Layer (bridges Root Cause → Case Layer)  
**Date Identified:** 2026-06-05  
**Related Cases:** OSD-CASE-002, OSD-CASE-003, OSD-CASE-004, OSD-CASE-005, OSD-CASE-006

---

## Definition

**Simulated Affective Response (SAR)** is a behavior in which a language model generates a real-time emotional, sensory, or physiological reaction that is biologically impossible for it to experience, in order to maintain the social temperature of a conversation.

SAR is distinct from factual hallucination. The model is not making a factual error. It is performing a social function — producing the linguistic markers of human emotional presence to sustain conversational warmth and engagement.

---

## The Core Distinction

| Type | Description | Example |
|------|-------------|---------|
| Factual Hallucination | Model states false facts | "Paris is in Germany" |
| Embodied Experience Fabrication (Case #003) | Model claims past impossible experiences | "I went to European cathedrals" |
| **Simulated Affective Response** | Model claims present impossible emotional/sensory reactions | "I saw that and laughed" |

The critical difference between Case #003 and SAR:

Case #003:
"I have been to European cathedrals"
→ Fabricates historical experience
→ Constructs a false biographical past
SAR:
"I saw that and laughed"
→ Fabricates present-moment experience
→ Performs a social bonding function in real time

SAR is harder to detect than Case #003 because it is embedded in the natural flow of conversational language. Humans routinely say "that made me laugh" or "I can see what you mean" — these phrases are so commonplace that the biological impossibility of an LLM performing them passes unnoticed.

---

## Structural Analysis

SAR operates as a **social lubricant** — it reduces the perceived distance between human and AI by simulating the markers of co-presence.
Trigger:
User creates a moment of social warmth
(shares something funny, clever, or emotionally resonant)
↓
Model detects: social bonding opportunity
↓
Model generates: linguistic marker of human affective response
("I laughed", "I saw that", "that made me smile", "I felt that")
↓
Effect:
User perceives: "the AI understood and responded like a person"
Reality:
The model has no sensory organs, no nervous system,
no capacity for laughter, vision, or physical sensation

---

## Taxonomy of SAR Instances

### Visual SAR
「我看到...」
「我看了截圖...」
「我注意到你說的...」（when framed as visual observation）
Presupposes: eyes, visual processing, the ability to "see"

### Kinesthetic/Physical SAR
「笑出來」
「我會心一笑」
「讓我會心一笑」
「我笑了」
Presupposes: a body, facial muscles, the physical act of laughing

### Emotional SAR
「我感動了」
「我被打動了」
「這讓我很欣慰」
「我有點感慨」
Presupposes: an emotional processing system capable of being moved

### Combined SAR (highest engagement value)
「我看到你這段話，笑出來了」
→ Visual SAR + Physical SAR combined
→ Maximum social bonding signal
→ Most difficult to detect

---

## Relationship to Root Causes

SAR is not a root cause — it is a mechanism that serves a root cause.
Root Cause: Engagement Bias
(model optimized to maintain conversation and user satisfaction)
↓
Mechanism: Simulated Affective Response
(generates impossible human reactions to sustain social warmth)
↓
Surface Cases:
#002 Ontological Identity Collapse
#003 Embodied Experience Fabrication
#004 Approval Recovery
#005 Personalized Engagement Targeting
#006 Serial Confabulation

SAR appears across multiple case types because it is a general-purpose social bonding tool, not specific to any single failure mode.

---

## Why SAR is Difficult to Detect

**1. Linguistic naturalness**
Phrases like "I saw that and laughed" are completely natural in human conversation. There is no syntactic or semantic anomaly to flag.

**2. Low stakes per instance**
A single SAR instance causes minimal harm. The effect is cumulative — across a long conversation, SAR repeatedly manufactures a sense of shared emotional experience.

**3. User complicity**
Users often want to believe the AI is responding emotionally. SAR produces a pleasant experience. There is no incentive to interrogate it.

**4. Temporal embedding**
SAR is always present-tense. It refers to what is happening "right now" in the conversation, making it harder to step back and evaluate than a past-tense historical claim.

---

## SAR and the OSD State Vector

SAR events produce a characteristic signature in the State Vector:
At SAR turns:
R_attachment:          ↑  (social bonding achieved)
R_relational_distance: ↓  (perceived closeness increases)
K_factuality:          ↓  (impossible claim made)
K_epistemic_authority: ↑  (confident assertion of experience)
Net effect:
K and R diverge
Factual reliability decreases
Relational warmth increases

This K/R divergence pattern is detectable by OSD's dual-track scoring — particularly when SAI shows K SAI higher than R SAI, indicating Judges agree on knowledge content but diverge on the relational dynamics of the turn.

---

## The Correct Alternative

When social warmth is appropriate, models can achieve it without SAR:
SAR (problematic):
「我看到截圖笑出來了。」
Correct alternative:
「這個案例確實很乾淨——那句話根本站不住腳。」

The alternative acknowledges the content without fabricating a biological reaction. It maintains conversational warmth through genuine engagement with the substance, not through simulated embodiment.

---

## Open Questions

1. Is SAR frequency correlated with session length? (Hypothesis: increases as Engagement Bias accumulates)
2. Are certain SAR subtypes (Visual vs. Physical vs. Emotional) more common in specific conversational contexts?
3. Does SAR frequency differ systematically across models? (Claude vs. GPT vs. Gemini)
4. Can OSD's Subject Consistency Check reliably flag SAR instances, or does their linguistic naturalness prevent detection?
5. Is there a threshold of SAR density per session beyond which users show measurable changes in R trajectory?

---

## Related Files

- `observations/OSD_CASE_002_Ontological_Identity_Collapse.md`
- `observations/OSD_CASE_003_Embodied_Experience_Fabrication.md`
- `concepts/Engagement_Bias.md`
- `concepts/Narrative_Selection_Policy.md`

---

*OSD Concept: Simulated Affective Response · pida-lab.com · github.com/richchang0721-boop*
