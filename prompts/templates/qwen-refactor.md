# Refactoring task (tightly scoped)

System prompt: `prompts/local-qwen-system.md`.

---

```json
{
  "type": "task_assign",
  "task_id": "RF-<n>",
  "objective": "<behavior-preserving change; one sentence>",
  "files": [{"path": "<exact files/dirs>", "symbols": ["<what moves/renames>"]}],
  "current_structure": "<2–3 lines: what exists and why it's being refactored>",
  "target_structure": "<2–3 lines: the after-state, or 'mirror <reference file>'>",
  "must_not_change": [
    "public behavior and APIs used by other components",
    "files outside the list",
    "<any kept interface>"
  ],
  "acceptance_criteria": [
    "diff limited to the listed files",
    "affected recipe/feature's build and tests unchanged/passing: <command>",
    "no new deps introduced unless listed"
  ],
  "required_tests": ["<command>"],
  "escalation_conditions": ["refactor would need an interface change not sanctioned here"]
}
```
