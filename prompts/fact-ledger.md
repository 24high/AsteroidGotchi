# Fact ledger — AsteroidOS Tamagotchi

Compact, verified facts. Every entry was read from source or measured. Reuse
these instead of re-deriving them; do not re-verify a fact without new evidence.
Newest verified facts go at the top of their section.

Audit date: 2026-10-21. Sources marked `(upstream @<sha>)` are read from the
GitHub repo the local recipe fetches, at the pinned `SRCREV`.

---

## Build environment

- **BLOCKER (as of 2026-10-21):** `bitbake` aborts before parsing:
  `ERROR: The following required tools (as specified by HOSTTOOLS) appear to be
  unavailable in PATH: chrpath diffstat gawk lz4`. The repo `Dockerfile` also
  lists `texinfo` and `python3-setuptools`, both missing on this host.
  User is installing: `sudo apt-get install -y diffstat gawk chrpath texinfo
  python3-setuptools liblz4-tool`.
- Working invocation (after the deps are installed), from `build/`:
  `MACHINE=emulator BB_ENV_PASSTHROUGH_ADDITIONS="MACHINE" PATH=$PATH:../src/oe-core/bitbake/bin bitbake -e <recipe>`
  - `BB_ENV_EXTRAWHITE` was renamed → the old name makes bitbake exit with code 1.
  - `build/conf/local.conf` contains only `DISTRO = "asteroid"` and
    `PACKAGE_CLASSES = "package_ipk"` — **no `MACHINE`**, so `MACHINE` must be
    exported.
- Host tools available for host-side verification: `g++ 13.3.0`, `make`, `git`,
  `python3` (3.12, no `pip3`). **No** `qmake`, `qmlscene`, `qmltestrunner`,
  `cmake`, `docker`.
- `build/conf/bblayers.conf` lists 40 layers, including
  `meta-smartwatch/meta-emulator` and `meta-asteroid-community`.
- Emulator machine: `src/meta-smartwatch/meta-emulator/conf/machine/emulator.conf`
  → `MACHINE_HAS_WLAN = "false"`, `MACHINE_HAS_SPEAKER = "true"`,
  `MACHINE_DISPLAY_ROUND = "false"`, display `800x800`
  (`.../asteroid-launcher-configs/kms.json:4`).

## Local endpoints (measured)

- **imagegen**: `POST http://192.168.8.2:9000/imagegen`, JSON
  `{"prompt","width","height"}` → **200 `image/png`** (verified 128×128,
  PNG colortype 2, `tEXt` "parameters" chunk containing
  `Steps / CFG scale / Guidance / Eta / Seed / Size / Model / RNG`).
  While the image model loads it returns **503** `{"detail":"Modell wird noch
  geladen."}` → treat 503 as retryable, never as a permanent failure.
  `GET /openapi.json` and `POST /videogen` also exist.
- **local LLM**: `POST http://192.168.8.2:9000/v1/chat/completions`, model
  `qwen3.8-flash-next-iq3_s`.
  - **It is a reasoning model.** With the default chat template it spends the
    whole completion budget on `reasoning_content` and returns **empty content**
    (measured: `max_tokens=2048` → `completion_tokens=2048`,
    `content_len=0`, `reasoning_len=7266`, `finish_reason="length"`).
  - **Fix:** send `chat_template_kwargs: {"enable_thinking": false}` →
    measured 40.7 s, `content_len=5362`, `reasoning_len=0`, `finish_reason=stop`.
  - The OpenCode `delegate_to_local_qwen` tool does **not** forward that
    parameter; it failed twice with "Local Qwen returned empty output".
    Workaround in use: call the OpenAI-compatible endpoint directly.
  - Latency for a trivial answer with thinking enabled: ~77 s.

## Watchface / background seam

- The launcher loads the watchface from the DConf key
  `/desktop/asteroid/watchface`; the default is the **code-level**
  `defaultValue` `"file:///usr/share/asteroid-launcher/watchfaces/000-default-digital.qml"`
  `(upstream @11e9ea06 src/qml/MainScreen.qml:237-241`).
- Nightstand watchface key `/desktop/asteroid/nightstand/watchface`
  `(MainScreen.qml:250-254)`; wallpaper key
  `/desktop/asteroid/background-filename`, default
  `"file:///usr/share/asteroid-launcher/wallpapers/full/000-flatmesh.qml"`
  `(MainScreen.qml:485-489)`.
- The watchface is a **plain QML file** in a `Loader`
  `(MainScreen.qml:346-361)`; the wallpaper is a **QML scene** too.
- A loaded watchface inherits the launcher's scope: `wallClock`, `use12H`,
  `nightstand`, `compositor`, `displayAmbient`, `burnInProtectionManager` are
  reachable without importing them (ids of the enclosing component).
- **Touch inside a watchface:** `PanelsGrid` is a `GestureFilterArea`
  `(upstream @11e9ea06 src/qml/misc/PanelsGrid.qml:33)`;
  `GestureFilterArea::childMouseEventFilter` → `filterMouseEvent` → returns
  `false`, i.e. it does **not** consume child mouse events, and it only grabs
  the mouse after a swipe threshold is crossed
  `(src/gesturefilterarea.cpp:78-96, 100-137, 168-188)`.
  ⇒ A `MouseArea` inside a watchface is expected to receive taps while panel
  swipes keep working. **Not yet demonstrated on the emulator.**

## AsteroidOS APIs confirmed on-watch

- **Time**: `Nemo.Time` `WallClock` (the `timed` daemon). `MainScreen.qml:170-174`.
- **Layout / no hardcoded sizes**: `Dims.w()/h()/l()` are percentages of
  `Screen.desktopAvailableWidth/Height`; `Dims.l()` uses the smaller side
  `(upstream qml-asteroid @bbd14a66 src/controls/qml/Dims.qml)`.
- **QML plugin pattern** to copy for our own C++ plugin:
  `org.asteroid.utils` registers singletons in
  `qml-asteroid @bbd14a66 src/utils/src/utils_plugin.cpp:33-39`
  (`qmlRegisterSingletonType<DeviceSpecs>(uri, 1, 0, "DeviceSpecs", ...)`).
- **Persistence** (proven pattern): `QSettings top("asteroid-weatherfetch","asteroid-weatherfetch")`
  → INI under `~/.config/<org>/<app>.conf`
  `(upstream asteroid-weatherfetch @8c479fa2 src/WeatherSettings.cpp:55)`.
- **Async HTTP** (proven pattern): `QNetworkAccessManager` + `finished` signal
  `(upstream asteroid-weatherfetch @8c479fa2 src/UrlFetcher.cpp:11-19)`.
- **TTS**: `espeak` is on the watch — `asteroid-skedaddle` has
  `DEPENDS += "... espeak"` and `RDEPENDS:${PN} += "espeak ..."`
  `src/meta-asteroid-community/recipes-asteroid/asteroid-skedaddle/asteroid-skedaddle_git.bb:11-12`.
- **Recipe conventions**
  - in-tree sources: `SRC_URI = "file://..."`, `S = "${WORKDIR}/sources"`,
    `UNPACKDIR = "${S}"` — example
    `src/meta-asteroid/recipes-asteroid/asteroid-launcher/asteroid-launcher-configs_git.bb`.
  - watchface recipe: `FILES:${PN} = "/usr/share/asteroid-launcher/ /usr/share/fonts/"`,
    `do_install` copies `${S}/${PN}/usr/share/` → `${D}/usr/share/`
    — example `src/meta-asteroid-community/recipes-watchfaces/unofficial-watchfaces/unofficial-watchface.inc`.
  - image inclusion: `src/meta-asteroid/recipes-core/image/asteroid-image.bb`
    inherits `src/meta-asteroid/classes/asteroid-image.bbclass`, whose
    `IMAGE_INSTALL` lists the packages that ship on the watch.
    `PACKAGE_FEED` in `src/meta-asteroid-community/conf/layer.conf` only feeds
    the package feed; it does **not** install into the image.

## Local_AI_esp32 (PocketGPT) — reuse triage

- Model: decoder-only transformer, 12 layers, hidden 448, 8 heads / 2 KV heads,
  vocab 6144, context 64, output capped at 24 tokens, 30,460,864 parameters
  (`docs/MODEL_CARD.md:3,5`). Runs on ESP32-S3 only → **not portable to the
  watch**; the watch will use an external endpoint.
- **REUSE — the "device calls a PC-hosted LLM" pattern and its guards**
  (`xiao_ai/keyboard_interfaces/xiao_network/xiao_network.ino`):
  - `:85` POST `{base}/api/generate`; `:91-93` body
    `{ "model": MODEL, "prompt": question, "system": "Reply in plain ASCII with
    one short sentence under 100 characters." }`
  - `:36-37` 15 s connect timeout / 30 s total timeout
  - `:158` base URL must start with `http://` and must **not** contain
    `localhost`, `127.0.0.1` or `@`
  - `:169` model name charset is restricted
  - `:104` the JSON response is read through a field allowlist filter
  Our backend is OpenAI-compatible (`/v1/chat/completions`), **not** Ollama's
  `/api/generate` — reuse the *rules*, not the endpoint.
- **DO NOT REUSE**: the 30 M model weights and all ESP32 firmware.

## Open questions

- Licensing of imagegen-produced assets is **not established** by any source
  read so far. Until confirmed, the assets stay local and the recipe is not
  published to a feed.
- The watch's LLM endpoint is site specific (`192.168.8.2` is a LAN address):
  it must be user-configurable on the watch.
- `dconf` is only pulled in transitively by `mlite`; the system-db path
  (`/etc/dconf/db/local.d`) is **not** established in the layers, so changing
  the default watchface via a dconf override is unverified. The launcher patch
  route is the deterministic alternative.
