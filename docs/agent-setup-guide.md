# Agent setup guide — AsteroidOS Tamagotchi repository

Platform-independent quick guide for the generated agent infrastructure; optional
OpenCode integration at the end.

## File map

| File | Who uses it | Purpose |
|---|---|---|
| `AGENTS.md` (root) | Both models | Durable project rules, working practices, report format |
| `PROJECT_BRIEF.md` | Frontier LLM (once per project) | Requirements digest: goal, constraints, known facts, open questions |
| `src/meta-asteroid-community/AGENTS.md` | Both models | Layer-specific recipe/QML conventions |
| `build/AGENTS.md` | Both models | Build-dir rules |
| `prompts/local-qwen-system.md`       | **System prompt for local Qwen** | Role + hard rules for the implementation model |
| `prompts/frontier-system.md` | **System prompt for Frontier LLM** | Coordinator role, when to self-implement |
| `prompts/templates/*.md` (8 files) | Task packages | Ready-to-use per-task templates |
| `prompts/routing.md` | Orchestrator/user | Model/tool decision table + thresholds |
| `prompts/handoff-format.md` | Frontier → local | Delegation / result-report JSON shapes |
| `prompts/context-management.md` | Orchestrator | Context and token hygiene |
| `prompts/verification.md` | Orchestrator + reviewers | Distinction ladder, acceptance strategy |
| `opencode.json` | Harness | local_qwen endpoint/model, early compaction |

## Per model

- **Local Qwen:** use `prompts/local-qwen-system.md` as the system prompt on every call.
  Task content is a filled copy of the matching template from `prompts/templates/`
  (JSON shapes from `prompts/handoff-format.md`). Never send conversation
  history; send facts + excerpts only.
- **Frontier LLM:** use `prompts/frontier-system.md` as the system prompt its session
  starts with; keep one long-lived session per workstream when possible (its context
  prompt carries the fact ledger forward). Consult the specific templates when the
  task kind matches.
- **Human user:** read `prompts/routing.md` when deciding who gets a task; use
  `prompts/verification.md` to audit claims.

## Starting a task (standard flow)

1. Write the goal as one sentence. Route it per `prompts/routing.md`.
2. Collect context deterministically (grep/read the relevant recipe/QML lines) and fill
   the matching template in `prompts/templates/` — verifying every file path and command
   against the repo.
3. Local-routed: dispatch to Qwen per `prompts/handoff-format.md` (`task_assign`). Frontier-routed: use the matching
   `frontier-*` template.
4. Apply the returned patch if in delegated mode; run the `required_tests` yourself;
   record actual output (see distinction ladder in `prompts/verification.md`).
5. Report via the result format from root `AGENTS.md`; update the fact ledger with
   what was established (new commands that work, files/names verified, decisions made).

## Escalation flow

First failure: analyze real output, retry with a materially different approach (do not
repeat the same fix). Second failure across distinct approaches: fill
`prompts/templates/frontier-escalation.md` and hand to the Frontier session. If the
Frontier session cannot resolve it (missing facts only the user knows), ask the user —
with the narrow question from the escalation package, never with the full history.

## Accepting results

A change is accepted when (a) all acceptance criteria in its task package are met, and
(b) the required checks reached at least level 3 ("passed") in the distinction ladder —
level 4 ("demonstrated", run on emulator/real device) for user-facing features. High-risk
changes additionally require an independent review via `prompts/templates/frontier-review.md`.

## Optional OpenCode integration

- `opencode.json` already registers the local model as `local_qwen` and enables
  early compaction (with automatic compaction kept as the final safety net).
- In OpenCode, dispatch local tasks via the `delegate_to_local_qwen` tool: it takes the
  task, and only bounded excerpts/files (never history); the orchestrating model reviews,
  applies, and verifies each proposal with its own tools before reporting results.
  Note: the delegated endpoint returns analyses/patches only — it cannot run tests or
  touch files; the orchestrator does that.
- Keep the API key/secrets out of `opencode.json` — the endpoint here assumes no
  authentication on the trusted local LAN; if that changes, add credentials through
  OpenCode's credential service, never into tracked files.

## Improving the system incrementally

- Tune only what observed evidence justifies: change escalation thresholds in
  `prompts/routing.md`, add recurring context patterns to `AGENTS.md` (not to every
  task), and refine template fields that keep being left empty.
- Do not add new agents/files without a concrete recurring need; duplication across
  prompt layers is a bug in this setup — fix it at the source layer.
