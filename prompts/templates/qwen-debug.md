# Qwen debugging task (focused)

System prompt: `prompts/local-qwen-system.md`. One hypothesis per retry, max 2 different
attempts before escalation.

---

```json
{
  "type": "task_assign",
  "task_id": "DBG-<n>",
  "objective": "Reproduce, diagnose, fix.",
  "symptom": "<actual observed behavior>",
  "evidence": "<real error output, test failure, stack trace, or log excerpt — verbatim>",
  "files": [{"path": "<suspect files, ideally 1–3>", "symbols": []}],
  "repro_command": "<exact command to trigger>",
  "constraints": [
    "smallest correct fix; no unrelated refactors",
    "no retry of an already-failed fix without new evidence",
    "do not change tests to make them pass unless the test itself is provably wrong"
  ],
  "acceptance_criteria": [
    "repro_command then demonstrates the fixed behavior",
    "existing tests / affected recipe build still pass"
  ],
  "required_tests": ["<repro_command>"],
  "max_attempts": 2,
  "escalation_conditions": [
    "2 materially different attempts failed",
    "cause lies outside the listed files",
    "failed output suggests missing runtime/SDK"
  ]
}
```

Escalation output must follow `prompts/templates/frontier-escalation.md`, containing:
what was tried (with evidence), narrow remaining question.
