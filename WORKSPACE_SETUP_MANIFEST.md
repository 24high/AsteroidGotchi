# WORKSPACE_SETUP_MANIFEST — files created for the agent infrastructure

Workspace: `/home/dh/Dokumente/GitHub/AsteroidGotchi` (AsteroidOS build superproject, Yocto).

## Created/modified files

```
AGENTS.md — project-wide agent rules: working practices, verification honesty, escalation, result format
src/meta-asteroid-community/AGENTS.md — layer-specific conventions for app/watchface/game recipes and QML work
build/AGENTS.md — rules for the generated Yocto build directory (conf discipline, destructive limits)
PROJECT_BRIEF.md — authoritative requirements digest: goal, requirement summary, repo facts, assumptions, open questions
opencode.json — local_qwen endpoint/model config and early compaction setting
prompts/local-qwen-system.md — system prompt for the local implementation model (delegated + interactive modes)
prompts/qwen-system.md — pointer file so "Qwen system prompt" references resolve to local-qwen-system.md
prompts/frontier-system.md — system prompt for the Frontier LLM as coordinator/reviewer, incl. when to implement itself
prompts/routing.md — routing table and machine-readable escalation thresholds (configurable defaults)
prompts/handoff-format.md — JSON task_assign / task_report formats, the only formal inter-agent channel
prompts/context-management.md — progressive context acquisition, fact ledger, repair loop limits, caching rules
prompts/verification.md — verification options for this repo, distinction ladder, acceptance strategy
prompts/templates/qwen-implement.md — template: new local implementation task
prompts/templates/qwen-debug.md — template: focused debugging with real evidence
prompts/templates/qwen-refactor.md — template: tightly scoped refactoring
prompts/templates/qwen-tests.md — template: creating/extending tests
prompts/templates/frontier-requirement-analysis.md — template: analysis of an unclear requirement
prompts/templates/frontier-architecture.md — template: architectural decision
prompts/templates/frontier-escalation.md — template: escalation after failed local attempts
prompts/templates/frontier-review.md — template: independent review of a completed change
docs/agent-setup-guide.md — introduction guide: file map, flows, OpenCode integration, improvement loop
WORKSPACE_SETUP_MANIFEST.md — this manifest
```

## Not done (by instruction)

- The Tamagotchi product itself (watchface, game mechanics, persistence, LLM client,
  image assets) was NOT implemented — this workspace pass set up agent infrastructure only.
- No packages were installed, no build was run, no destructive commands executed.
- Secrets were not written to any file; `opencode.json` contains only endpoint/model
  parameters for the additionally configured local runtime.
