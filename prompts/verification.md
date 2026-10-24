# Verification and quality rules (AsteroidOS Tamagotchi)

## Verification options actually available here

- **No repo-wide test suite exists.** Do not invent commands like `make test`.
- Recipe level: `bitbake <recipe>` (verify the recipe name first); inspect variables with
  `bitbake -e <recipe>`.
- Image level: `bitbake asteroid-image` (slow, hours) — reserved for integration or
  final acceptance.
- For Tamagotchi logic (game state, transitions, persistence): write host-runnable unit
  tests inside the watchface/app source repo and run them on the build host with the
  toolchain that repo provides. Choose the framework from the repo's existing tests; if
  none exist, pick one, state it, and make the choice durable.
- Visual/UI checks on device or emulator: manual, using the `emulator` machine as the
  default when no real watch is connected. Manual tests must be listed with what to
  observe; never auto-replace automated coverage.

## Distinction ladder — always report the highest level reached honestly

| Level | Means | Example wording |
|---|---|---|
| 1. Changed | Code edited, no proof of anything | "Changed: <files>" |
| 2. Ran | A check command executed | "bitbake <recipe>: exit 1" |
| 3. Passed | Check exit with no failures | "bitbake <recipe>: completed OK" |
| 4. Demonstrated | Observed runnable behavior | "booted emulator image; UI renders, clock top-center" |
| 5. Not covered / open risks | Explicit residual gaps | "untested on real device", "<feature> blocked by <dependency>" |

Never merge levels in claims: "implemented (1)" is not "working (4)".

## Acceptance strategy

- Per task package, define in advance which checks count as pass/fail
  (handoff `required_tests`). Post-hoc redefinition does not count as acceptance.
- Scorecard per work stage (recipe build → game logic tests → image build → manual UI):
  each stage must reach level 3 minimum, level 4 for user-visible features.
- LLM-facing behavior: verify with the local model offline AND with it absent — the
  product must not depend on inference reachability (deterministic fallback).
- Persistence: must survive process restarts and corrupted/partial save data;
  include these cases in the automated tests as a minimum.

## Review requirements

- Any persistence/migration/state-reset change or device-facing destructive change:
  independent Frontier review (`prompts/templates/frontier-review.md`), beyond the
  authoring model (which may be Frontier itself if it authored the code — in that case
  a second instance/session audits it).
