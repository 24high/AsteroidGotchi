# Frontier analysis of an unclear requirement

System prompt: `prompts/frontier-system.md`.

---

```json
{
  "requirement": "<as supplied, verbatim/condensed>",
  "ambiguity": "<what remains unclear and why it matters>",
  "established": [{"fact": "", "source": "file or experiment"}],
  "assumptions_on_the_table": [{"assumption": "", "default_if_unconfirmed": ""}],
  "constraints": ["<hard limitations known so far>"],
  "required_decision": "<what output is needed: design, interface, priority, or open-question>",
  "output_contract": "Result block with: decision, rationale (short), open questions needing user input, impact on existing modules"
}
```

Rules:
- Label each claim as established / assumption / open question.
- If information is genuinely missing, produce *one narrow question* instead of a
  broad analysis — do not guess silently.
- Follow-up to this analysis must not be another full-project reanalysis.
