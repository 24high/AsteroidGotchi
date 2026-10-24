# PROJECT_BRIEF — AsteroidGotchi (AsteroidOS Tamagotchi)

Concise, authoritative digest of the project assignment (source: the user's project
description, supplied in German; treat the original wording as binding where this
digest paraphrases). Code-tree facts live in root `AGENTS.md`; layer conventions in
`src/meta-asteroid-community/AGENTS.md`. Read this once per project; do not re-analyze
the full description in every session.

## Goal (1 sentence)

Replace AsteroidOS's default watchface and background with a fully playable, visually
high-quality virtual pet (Tamagotchi style) that feels natively integrated into the OS
and communicates via an LLM in the style of Local_AI_esp32 — a working product, not a
concept, static prototype, or UI demo.

## Binding requirements

1. **Watchface replacement** — pet + themed environment replace the old
   watchface/background completely; no remnants of the old display may remain visible.
   Clock time stays top-center, highly readable, visually subordinate; no overlap with
   pet, speech bubbles, or status info. Keep other OS functions intact.
2. **Display adaptation** — use actual/dynamic display information of the target
   device; no hardcoded screen dimensions.
3. **Graphics quality** — high-quality hand-drawn-style 2D game art (no emoji/placeholder
   look, no photorealism, no heavy 3D): original cute character with recognizable
   silhouette, consistent visual identity, small detailed environment, expressive eyes,
   smooth resource-light animations, distinct visuals for joy/hunger/thirst/tiredness/
   boredom/illness/affection/grief, idle animations, smooth state transitions, visible
   state change over time, dedicated states for sleep/illness/death.
4. **Mandatory image generation API** — all suitable graphics MUST be generated via
   `POST http://192.168.8.2:9000/imagegen` with JSON `{prompt, width, height}` (exact
   request format as supplied; do not invent extra fields/auth/endpoints). Probe the
   real endpoint and response format before scripting; do not assume response keys.
   Modular asset architecture (character, environment, effects, objects as separate
   layers); character consistency via a fixed visual description reused in every prompt;
   pre/post-processing may be needed for animation coherency; reproducible script;
   caching; generation happens OFF-line (never at watch build/runtime); verify images;
   document which calls succeeded / what was post-processed / what shipped. Merely
   prepared prompts do not satisfy the acceptance criterion.
5. **Playable mechanics** — needs: hunger, thirst, energy/sleep, cleanliness/hygiene,
   health, mood/happiness, affection, boredom. Values/rates/thresholds centrally
   configurable. Real care actions (feed, drink, put to bed, clean, play, medicine,
   petting) each affect values and produce visible reactions; overfeeding/neglect/
   sleep deprivation/illness have plausible consequences; optional mini-games.
   Continuous time progression even when unused (compute elapsed time from persistent
   timestamps, not timers assumed to run).
6. **Neglect, death, reset** — progressive state model (healthy → need → unwell →
   ill → critical → dead); single short unmet need must not kill; neglect severity/
   duration/combination decide. Death shows an appropriate presentation, disables normal
   care interactions, requires an EXPLICIT, confirmed reset; ordinary screen changes or
   app restarts must NOT resurrect the pet. Reset starts a fresh pet.
7. **Persistence** — all needs/health, life/dev state, last calculation and care
   action timestamps, sleep state, affection, death/reset flag, optional personality/
   stage/events. Use existing AsteroidOS storage facilities; robust defaults/migration/
   corrupted-save handling; avoid per-frame writes.
8. **LLM integration (Local_AI_esp32 source)** — analyze the project first; port only
   what is technically reasonable; if on-watch inference is infeasible, use a minimal
   adaptation or an external inference endpoint over an existing connection — never an
   invented interface. Game functions must fully work when the LLM is unreachable.
   LLM provides ONLY communication on top of the deterministic game state — never state
   ownership, actions, or critical transition control.
9. **Pet communication** — short, cute, natural, occasionally humorous, state-dependent
   personality-driven speech bubbles. When needs exist: report the most urgent one first,
   with urgency reflected; avoid repetition; critical states must not be trivialized by a
   joke. When well-fed: charming/funny incidental remarks, varied, never manipulative.
   Message triggers: need appears; need stays unmet; need fulfilled; watch view
   opened; idle wellbeing; sleeping/dead states. Cooldowns + caching + no per-render-
   frame requests; immediate visual reaction while inference is slow; local fallback
   message on failure.
10. **Speech presentation** — compact bubble, bounded length (word-boundary-safe
    truncation), never occludes pet or clock, matching expression/animation; optional
    audio speech synthesis only if hardware supports and it is actually implemented and
    tested — no unproven claims.
11. **Technical rules** — follow AsteroidOS code/style conventions; reuse existing
    libs/services; keep UI / game mechanics / persistence / LLM-link modular; async /
    non-blocking for LLM, network, storage; treat CPU/RAM/flash/energy budgets as scarce;
    minimal background processes; robust handling of disconnects, invalid model output,
    timeouts; bound and validate all LLM output; keep builds compatible with the
    project's build tooling; tests + comprehensible logs; document deviations.

## Required test coverage (minimum scenarios)

Fresh healthy pet; hunger/energy progression; each care action; long-term neglect
progression; illness and critical state; death + explicit reset; persistence across app
AND system restarts; elapsed-time correctness with display off; LLM reachable (success)
and unreachable/failed; simultaneous needs; repeated watch-view openings without message
spam; sleeping and dead pet; display adaptation/readability across supported resolutions;
animation/UI responsiveness under load. Automated for game logic + state transitions;
manual for visuals/UX on a real device or available dev environment.

## Build/delivery requirements

Work within the existing repository; run every available build/test command actually
(never claim unexecuted results); if a full build is impossible (missing hardware/SDK),
continue all independent work and document precisely what is missing. Deliverables:
code changes, graphics/animation assets, playable game with death/reset, persistence,
LLM integration with fallback logic, automated tests for core mechanics, docs
(architecture/build/install/config), changed-file overview, actual build & test results,
open items list. Prioritize an integrated working solution over a broad, partly working
demo.

## Honest unknowns (verify in the repo, do not assume)

- Which exact services/QML APIs are available for time, touch, notifications, audio,
  persistence on the supported watch layers (verify per device).
- Real display resolution(s) and the usable graphics pipeline.
- The actual imagegen response format and endpoint reachability from this network.
- Local_AI_esp32's model, inference path, and what is portable.
- Target device choice(s); the `emulator` machine is the default fallback verification
  target when no real watch is available.
