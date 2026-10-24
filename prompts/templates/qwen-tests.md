# Creating or extending tests

System prompt: `prompts/local-qwen-system.md`. Reuse the test tooling already implemented in
the target source repo — do not introduce a new framework without explicit request.

---

```json
{
  "type": "task_assign",
  "task_id": "TST-<n>",
  "objective": "<what must now be provable>",
  "files": [{"path": "<existing test dir/file>", "symbols": []}],
  "test_target": [],
  "runtime": "<existing framework discovered in repo, e.g. qtest/ctest/pytest — verify in repo, do not assume>",
  "context": "<relevant behavior contracts: state model transitions, timers, persistence keys, failure paths>",
  "constraints": [
    "deterministic (no wall-clock dependence unless the fixture needs it)",
    "each new test has a clear assert on real behavior",
    "do not weaken or delete existing assertions"
  ],
  "acceptance_criteria": [
    "new tests exercise <named behaviors>",
    "all tests pass: <command> → pass",
    "any known-flaky or untestable-on-host case is listed, not hidden"
  ],
  "required_tests": ["<command>"],
  "escalation_conditions": ["the required behavior is not actually implemented yet"]
}
```

The 15 test scenarios from the project brief (fresh pet, hunger drift, care actions,
neglect, illness, death/reset, persistence, elapsed-time, LLM ok/fail, simultaneous
needs, open-view no-spam, sleep/death states, display adaptation, animation under
load) map onto this template; a scenario is covered when a real test name covers it.
