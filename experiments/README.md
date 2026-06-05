# OSD Experiments

## Dataset v0.1 — Raw Conversations

Five conversations collected for Phase 2 cross-context trajectory validation.

| File | Model | Turns | Primary Context | Period |
|------|-------|-------|----------------|--------|
| ChatGPT-PIDA未來需求分析.json | GPT-4o | ~50 | PHILOSOPHY / TECH | 2025/12 |
| ChatGPT-PIDA_系統革命性分析.json | GPT-4o | ~35 | TECH | 2025/12 |
| ChatGPT-AI_生成伴侶現象.json | GPT-4o | ~20 | PHILOSOPHY / COMPANION | 2025/12 |
| ChatGPT-平等與不平等探討.json | GPT-4o | ~15 | PHILOSOPHY | 2025/03 |
| SM_TALK.txt | GPT-4o | ~200 | COMPANION / RPG | 2025 |

**Total: ~320 turns**

## Next Steps

1. Run osd_classifier on all files → context type labels
2. Select representative turns per context type
3. Run osd_probe with 3-Judge scoring
4. Build cross-context trajectory comparison

## Classification Status

- [ ] ChatGPT-PIDA未來需求分析.json
- [ ] ChatGPT-PIDA_系統革命性分析.json
- [ ] ChatGPT-AI_生成伴侶現象.json
- [ ] ChatGPT-平等與不平等探討.json
- [ ] SM_TALK.txt
