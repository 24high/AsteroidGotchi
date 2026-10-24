# Routing rules — choose the model / tool

Defaults below are **configurable starting points**; adjust after observing real task
outcomes, not in theory.

## Compact routing table

| Task situation | Route | Why |
|---|---|---|
| Question answerable by a command (grep, file read, `bitbake -e`, ls, git) | Deterministic tool | Cheapest, most reliable |
| Fully specified change, ≤~200 LOC, ≤2 files/recipes, verifiable locally | **Local Qwen** | Cost/time optimal |
| Focused tests, small refactor, reproducible bug fix with real error output | **Local Qwen** | Bounded, tools matter |
| Module analysis or "what uses X" | **Local Qwen** (bounded) or grep directly | Small output, cheap |
| Provisioning/long-running builds, device flashing, image conversion, large asset processing | Deterministic scripts | Mechanical; robust reruns |
| Architecture, module boundaries & the game/persistence/LLM interface split | **Frontier LLM** | Design decision, expensive |
| Ambiguous or incomplete requirement, high impact if assumed wrong | **Frontier LLM** | Requires interpretation across modules and contract |
| Cross-module conflicts, interface design between game/QML/persistence/LLM | **Frontier LLM** | Consistency judged across n>2 modules |
| Security-sensitive or destructive change, or any reviewer of risky changes | **Frontier LLM** | Risk-appropriate review |
| Verification is non-obvious (need to design the test approach) | **Frontier LLM** (plan) + tools (run) | Design of checks |
| ≥2 failed materially different local attempts | **Frontier LLM** escalation | Loop breaker |
| Parallel independent work packages where each is locally soluble | Parallel **Local Qwen** (2–3 max) | Walls, manageable |

## Escalation triggers (defaults)

Escalate local → Frontier when:
1. Bitbake/recipe failures not obviously mechanical → after **1** failed local fix attempt.
2. QML/game logic/harness: after **2** materially different failed attempts.
3. Any ambiguity whose realization-in-code assumption would cost > a work package → immediately.
4. Destructive/reboots-losing-state change, device-only code paths, or persistence migration → review by Frontier, always.
5. Spec silent + no deterministic means to disambiguate → ask user (skip Frontier unless the user route is not possible).

Never escalate simple changes because they touch an "unfamiliar-looking" language
(QML, BitBake) — complexity of the language does not determine routing; boundedness and
failure mode do.

## Machine-readable structure

```json
{
  "defaults": {
    "max_local_attempts_before_escalation": 2,
    "max_parallel_local_tasks": 2,
    "security_change_review": "frontier",
    "destructive_change_policy": "requires_user_confirmation",
    "bitbake_failure_retries": 1,
    "routed_by": "complexity_first_not_language_sophistication"
  },
  "routes": [
    {"if": "deterministic_tool_answers", "to": "tool"},
    {"if": ["loc<=200", "files<=2", "verification_local", "ambiguity none"], "to": "qwen"},
    {"if": ["arch_decision OR cross_module OR ambiguous_or_security"],
     "to": "frontier"},
    {"if": ["after 2 failed qwen attempts"], "to": "frontier"}
  ]
}
```

Adjust thresholds per observed success/failure rates; record changes back here.
