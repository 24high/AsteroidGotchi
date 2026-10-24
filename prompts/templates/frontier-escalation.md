# Escalation: problem the local model could not solve

Used when Qwen reports `escalated: true` (after the allowed attempts from
`prompts/routing.md` defaults) or when local work is blocked by something structurally
beyond it. Frontier prompt: `prompts/frontier-system.md`.

---

```json
{
  "task_id": "<original task id>",
  "goal": "<what was being attempted>",
  "attempts": [
    {"n": 1, "hypothesis": "", "change": "<files touched>", "evidence": "<real output>"},
    {"n": 2, "hypothesis": "", "change": "", "evidence": ""}
  ],
  "current_state": "<working tree status; what is broken / uncommitted",
  "constraints_seen": ["<facts the local model confirmed>"],
  "narrow_question": "<the specific decision/answer needed; not 'please analyze'>",
  "options_on_the_table": [{"idea": "", "why_not_yet": ""}],
  "output_contract": "Result block: direct answer to narrow_question, next concrete steps (files+commands), or permission for a destructive step if that is the actual blocker."
}
```

Rules:
- Escalation must contain **real evidence** per attempt, not paraphrased summaries.
- When the blocker is access/permission (target device, missing SDK, network), say that
  explicitly; do not treat it as a knowledge problem.
- Frontier response must answer `narrow_question` and can either proceed itself with a
  deeper analysis or return a fresh, precise `task_assign` to Qwen.
