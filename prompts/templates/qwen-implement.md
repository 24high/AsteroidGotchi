# Qwen implementation task — new feature

System prompt: `prompts/local-qwen-system.md`. Durable rules: `AGENTS.md`. Fill only task
facts; do not restate conventions.

---

```json
{
  "type": "task_assign",
  "task_id": "T-<n>",
  "objective": "<what the finished feature does, one sentence>",
  "files": [
    {"path": "<target file or directory>", "symbols": ["<must-touch symbols, or []>"]}
  ],
  "context": "<only facts Qwen cannot discover cheaply: machine target, related existing components (name + file), expected state storage keys, upstream constraints>",
  "constraints": [
    "does not modify other modules",
    "does not introduce new build-time dependency X unless required",
    "UI stays power-aware (bounded animation, no per-frame work)",
    "<task-specific>"
  ],
  "steps": ["<ordered, concrete>"],
  "acceptance_criteria": [
    "<objective; e.g. 'recipe bitbakes: bitbake asteroid-tamagotchi → pass'>",
    "<test-command + expected outcome>"
  ],
  "required_tests": ["<command>"],
  "open_questions": [],
  "escalation_conditions": ["<e.g. build fails not fixably in 1 attempt, missing API>"]
}
```

Human-ready version when not using JSON tooling:

- **Task ID / objective:** …
- **Files & symbols:** …
- **Context:** …
- **Constraints:** …
- **Steps:** …
- **Acceptance criteria:** …
- **Tests:** …
- **Escalate when:** …

Report with the `task_report` JSON from `prompts/handoff-format.md`; standalone runs may use
the `## Result` block from `AGENTS.md`.
