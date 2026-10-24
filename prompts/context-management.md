# Context and token management rules

Universal rules first; optimizations requiring runtime support are separated at the end.

## Universal (works with manual invocation and any harness)

1. **Progressive acquisition, never upfront dumping.** Route a task first
   (`prompts/routing.md`), then fetch only the facts the routed step needs. The default
   order: grep for symbols → read the 10–30 relevant lines → only then a full file, and
   only if the symbol view is insufficient.
2. **Search before asking.** Symbol search (`grep -rn "SRC_URI\|RDEPENDS\|"<TypeName>" src/`)
   answers most repo questions. Use `git -C src/<layer> grep` to search only the affected
   layer when possible — layers are independent repos.
3. **Fact ledger.** Maintain one compact list of established facts
   (verified recipe/paths, available commands, chosen architecture, open questions).
   Create it per work-package it: pass verbatim into each new task package, and prune
   facts that age out. Do not re-verify a fact for every task.
4. **Excerpts, not files.** Task packages carry `symbol: excerpts` (10–30 lines).
   Never attach full conversations; the handoff format contains everything needed.
5. **One-pass rule for similar errors.** If two consecutive model calls try to answer
   the same question, one of them should have been a tool. Stop and switch to tools.
6. **Concise result reports.** The formats in `AGENTS.md` and `prompts/handoff-format.md`
   are caps: no narration, no restating of the task, no success language without evidence.
7. **Repair loops.** Max 2 materially different attempts locally, then escalate
   (per `prompts/routing.md`). An "identical retry with more tokens" does not count as
   a different attempt and is prohibited.
8. **Compact handoff summaries.** When moving between models, carry the fact ledger +
   the relevant diff hunk (or a ≤20-line behavioral description) — never the audit trail
   of all intermediate outputs.
9. **Stable prompt prefixes.** Keep the system prompts (`prompts/*-system.md`) and
   project docs (`AGENTS.md` and layer AGENTS.md) byte-identical across tasks so a
   prefix cache, when supported, applies. Put volatile content at the end of prompts.
10. **Controlled parallelization.** Only locally soluble work packages with no shared
    files may run in parallel (max 2–3). Merge and verify serially afterwards.
    Never parallelize tasks depending on the same file or architecture decision
    still under Frontier review.

## Requiring runtime support (optional; enable what your harness has)

- **Prompt caching** where the model provider supports it: reuse the same system-prompt
  prefix sequence in API calls. Detect savings via measured input token counts; do not
  assume a specific rate.
- **Compaction** where available (e.g. OpenCode) keeps long sessions within the window;
  in this project config, "early compaction" is enabled and automatic compaction remains
  a final safety net. Trigger facts branching are in `opencode.json`, not here.
- **Context window guarding** — for the local Qwen: respect the configured limits from
  `opencode.json` (`local_qwen.max_tokens`, `timeout_ms`); split tasks that would push
  output above budget instead of truncating.

## Honesty requirement

Never assert a token or cost saving without measuring both sides. Report measurements
in percentages or counts, whatever exists, not aspirational claims.
