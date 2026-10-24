# AGENTS.md — AsteroidOS Tamagotchi (asteroid/asteroid build superproject)

## What this repository is

This is the **AsteroidOS build superproject**: a Yocto/OpenEmbedded hub that clones and wires
up the real source layers under `src/` and drives image builds.

- `prepare-build.sh <machine>` clones the layers listed in its `layers` array into `src/` and
  sets up `build/conf`. Machines are declared in the `devices` array (e.g. `dory`,
  `emulator`, `lenok`). `bitbake asteroid-image` builds a flashable image.
- `src/` contains OE layers, **each layer is its own separate git repository**:
  `oe-core`, `meta-qt5`, `meta-openembedded`, `meta-smartphone`, `meta-asteroid`,
  `meta-asteroid-community`, `meta-smartwatch`.
- This hub repo's `.gitignore` excludes both `src/` and `build/`. Commits to layer code belong
  in the **layer's own git repo**, not the hub (the hub only tracks `prepare-build.sh`,
  `Dockerfile`, and similar top-level files).
- `build/` is generated build output plus mutable conf (`local.conf`, `bblayers.conf`).
  Machine selections there are session state; never commit them, never "clean up" default.
- The project target: replace the watchface/background with a fully playable Tamagotchi
  (needs system, persistent state, optional LLM communication) implemented via existing
  AsteroidOS mechanisms. Requirements are digested in `PROJECT_BRIEF.md` (full draft
  text provided by the user); commit to that as the binding requirement source.

## Agent roles

- **Local Qwen (`local_qwen` endpoint, quantized):** bounded implementation, focused tests,
  reproducible fixes, small refactors, module analysis. In OpenCode, delegate via the
  `delegate_to_local_qwen` tool (it returns analysis/patches only; the orchestrator applies
  and verifies them).
- **Frontier LLM:** architecture and major decisions, requirements decomposition, ambiguous or
  security-relevant changes, reviews of risky changes, and second opinion after repeated
  failed local repair attempts. It delegates routine implementation.
- **Deterministic tools always win over an LLM** for answerable questions: use file search,
  grep, `devtool`, reading actual `.bb`/`.inc`/QML sources, and bitbake itself.

See `prompts/` for system prompts, task templates, routing rules, and the handoff format.
See `docs/agent-setup-guide.md` for how to use this setup end to end.

## Working practices

### Analyzing a new task
1. Classify the task (route it, see `prompts/routing.md`). Route before reading files.
2. Verify every fact in the actual recipe/source: open the concrete `.bb`, `.inc`, QML, or
   C++ file. Never trust assumptions about APIs, images, package names, or device support.
3. If a fact was already established earlier, reuse it (keep `prompts/context-management.md`
   habit of passing forward only verified facts).

### Context-efficient search and analysis
- Scan `src/meta-asteroid-community/recipes-*` and `src/meta-smartwatch/meta-<device>` first
  for anything about watchfaces, apps, and per-device behavior.
- Grep for symbols (`bitbake` variables like `SRC_URI`, `RDEPENDS`, `PACKAGE_ARCH`, and QML
  types like `AsteroidWatchface`, `Timed`, `Mce`) instead of reading full files.
- Read one recipe/symbol at a time; stop as soon as the change target is clear.
- Pass excerpts (10–30 lines), not whole files, into delegation prompts.

### Implementation and refactoring
- Follow the conventions already present in the target layer (recipe naming, `require`/`SRC_URI`
  patterns, QML import style, plugin bound style).
- Prefer existing AsteroidOS/Qt components and system services over inventing parallel
  systems; study existing `recipes-asteroid` apps and watchfaces and `timed` usage, and
  reuse their QML patterns for dates, animations, and user feedback.
- Keep diffs minimal and one-purpose; no unrelated cleanups mixed in.
- UI must think about power: bounded animation rates, avoid per-frame work, async IO.
- No invented APIs: if an interface can't be confirmed in the layer source, treat it as unknown.

### Asset generation (image API)
- Image assets are generated **once, offline** via `http://192.168.8.2:9000/imagegen`
  (JSON: `prompt`, `width`, `height`) — never at watch build/run time by the agent.
- Verify actual response format before scripting; process/crop into the small on-watch
  formats the target layers actually consume; then commit generated assets into the layer
  repo that ships them. Never autoregenerate during every build.

### Testing and verification
- There is **no repo-wide test suite**. Verification options (verify against actual recipe
  names before using): `bitbake <package>` for a single recipe, `bitbake -e <package>` to
  inspect variables, and `bitbake asteroid-image` for the full image (slow, hours).
- Any logic written in QML/Python/C++ inside a watchface/app repo should come with its own
  unit tests hosted in that source repo and runnable on the host.
- Never report "tests passed" without the actual command and its real output. A distinction
  ladder (changed / run / passed / demonstrated / open risks) lives in `prompts/verification.md`.

### Git and diff review
- Commit inside the affected layer repo with a message describing the change, not the prompt.
- Before handing off or requesting review, show/produce the diff of the affected files.
- Never amend/rebase other people's work; never force push.

### Destructive or external actions
Forbidden without explicit user confirmation:
- Deleting or force-cleaning build trees (`bitbake -c cleanall`, wiping `build/`), mass
  deleting anything, `rm -rf` outside `/tmp`.
- Pushing, overriding remote branches, publishing anywhere.
- Installing packages on the watch or running arbitrary commands on a device over the network.
- Writing secrets (API keys, tokens) into any tracked file. Credentials belong in tool config,
  not code.
- Network access is limited to the approved local endpoints (imagegen, local Qwen) and
  upstream repos clones; no cloud calls.

Use `emulator` (qemux86) as the default verification target when a real watch is not available.

## Escalation rules (local → Frontier)

Escalate when:
- The task tunnels into OS-level behavior that is not verifiable in the local layers
  (a `bitbake` failure unknown and unfixable after one attempt).
- Interfaces between game logic, QML watchface, and persistence don't resolve without a
  design decision.
- Security-relevant change, or unbounded destructive change.
- Two materially different local repair attempts have failed.
- The scope is ambiguous and a wrong assumption is expensive.

Escalation format: use `prompts/templates/frontier-escalation.md`; include what was tried,
the concrete evidence (real command output), and the narrow question that needs answering.

## Result report format (always this, kept short)

```
## Result
Goal: <one line>
Done: <what was changed, files>
Checks: <commands run and their ACTUAL outcomes, verbatim summary>
Not done / open: <gaps, known issues>
Next needed: <optional single next step>
```

Do not report success on anything not actually observed.
