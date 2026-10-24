# Frontier architectural decision

System prompt: `prompts/frontier-system.md`.

---

```json
{
  "decision_needed": "<e.g. where the persistent game state lives (app vs. service), how LLM inference is accessed on-watch>",
  "options": [
    {"id": "A", "outline": "", "risks": "", "cost": "", "closest_existing_component": "<most similar existing AsteroidOS component>"}
  ],
  "constraints": ["power/energy", "RAM/flash", "given devices listed in prepare-build.sh", "must degrade gracefully without LLM"],
  "evaluation_criteria": ["maintainability", "verifiability", "resource footprint", "risk containment"],
  "output_contract": "Decision: one option (or a hybrid), justified in ≤5 bullets; files/symbols to be touched; risks with mitigation; what to build first/second (serial vs parallel)."
}
```

Rules:
- Ground everything in actual layer contents cited by path; no invented APIs.
- Name the files Qwen will need to touch and the acceptance criteria already established
  by any sibling recipe.
- "Cost" refers to concrete engineering effort, not abstract narrative time.
