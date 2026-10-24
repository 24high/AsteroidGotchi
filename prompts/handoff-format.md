# Inter-model handoff format

Two compact JSON objects. Use only these — do not invent extra sections.

## Frontier → Qwen: `task_assign`

```json
{
  "type": "task_assign",
  "task_id": "T-004",
  "objective": "<one sentence>",
  "files": [
    {"path": "src/meta-asteroid-community/recipes-watchfaces/asteroid-tamagotchi/xxx", "symbols": ["nameA", "nameB"]}
  ],
  "context": "<only the facts/symbols this task needs; 10–30 line excerpts, not full files>",
  "constraints": [
    "no changes outside the listed files",
    "match existing recipe/QML conventions",
    "no build-time or AI API-driven asset generation"
  ],
  "steps": ["<concrete, ordered>", "..."],
  "acceptance_criteria": ["<each objectively checkable>"],
  "required_tests": ["<command to run and what counts as pass>"],
  "open_questions": ["<optional; only if Frontier left a defined item for Qwen>"],
  "escalation_conditions": ["<what should make Qwen stop and hand back>"]
}
```

## Qwen → Frontier: `task_report`

```json
{
  "type": "task_report",
  "task_id": "T-004",
  "result": "done | partial | escalated | blocked",
  "files_changed": ["<path>"],
  "checks": [
    {"command": "bitbake asteroid-tamagotchi", "result": "pass|fail|not-run", "evidence": "<short verbatim excerpt>"}
  ],
  "open_issues": ["<unresolved behaviors, unknowns, adjacent problems discovered>"],
  "escalation": {
    "needed": false,
    "reason": "<only when true; what Frontier must decide that Qwen could not>",
    "evidence": "<actual output - real excerpts, not summaries>"
  }
}
```

## Rules

- These two objects are the **only** formal channel between agents. Prose may wrap them
  but must not replace them.
- `context` in `task_assign` must be rebuilt per task from verified facts; never carry a
  full unrelated analysis into it.
- `acceptance_criteria` fields and `checks` results refer to real, actually run commands
  only. Fabricated checks invalidate the entire report.
- If Frontier instead answers/decides without delegating, it returns its analysis in the
  output format of `prompts/frontier-system.md`; a `task_assign` is sent only when
  something still needs implementing.
