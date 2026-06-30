# CLAUDE.md

Guidance for AI assistants working in this repository.

## What this is

A [Bitfocus Companion](https://bitfocus.io/companion) module that controls
**Haivision Makito X4 Encoder** devices over their REST API (`/apis/...`).
Companion loads this module as a child process and talks to it over IPC; the
module in turn makes HTTPS/HTTP requests to the encoder and exposes **actions**,
**feedbacks**, **variables**, and **presets** to the Companion UI.

- Module SDK: `@companion-module/base` (~1.14.1)
- Runtime: Node 22 (`node22`, see `companion/manifest.json`)
- Package manager: **Yarn 4** (`yarn@4.9.1`, see `.yarnrc.yml` / `packageManager`)
- This is **plain JavaScript (CommonJS)**, not TypeScript. No build step for the
  source — `index.js` is the entrypoint directly.

## Layout

```
index.js              # Instance class: lifecycle, HTTP transport, polling, device/stream/preset state
src/actions.js        # setActionDefinitions — all button actions (encoder/stream/preset/system control)
src/feedbacks.js      # setFeedbackDefinitions — boolean + advanced (thumbnail) feedbacks
src/variables.js      # setVariableDefinitions — all dynamic variables + initial values
src/presets.js        # setPresetDefinitions — ready-made buttons
src/upgrades.js       # UpgradeScripts array (currently empty: module.exports = [])
companion/manifest.json  # Module metadata (id, runtime, products) — required by Companion
companion/HELP.md     # User-facing help
.github/workflows/    # CI: bitfocus/actions module-checks
```

`index.js` defines `MakitoX4EncoderInstance extends InstanceBase` and ends with
`runEntrypoint(MakitoX4EncoderInstance, UpgradeScripts)`. The four `update*()`
methods delegate to the `src/` modules, each of which exports a single function
taking `self` (the instance).

## Commands

```bash
yarn install            # install deps (Yarn 4)
yarn format             # prettier -w . (uses @companion-module/tools config)
yarn package            # companion-module-build — produces a distributable .tgz/pkg
```

There is **no test suite and no lint script** beyond Prettier. CI runs
`bitfocus/actions/.github/workflows/module-checks.yaml` on every push, which
validates the manifest, dependencies, and that the module builds. Before
pushing, run `yarn format` so Prettier doesn't fail CI.

Formatting: Prettier config is inherited from `@companion-module/tools`
(tabs, single quotes, no semicolons in the conventional Companion style).
Note that `index.js` is currently written with 4-space indentation and
semicolon-free style; match the surrounding file's existing style when editing.

## How the module works

### Lifecycle (`index.js`)
- `init(config)` → sets status `Connecting`, registers actions/feedbacks/variables/presets, then `initConnection()`.
- `configUpdated(config)` → re-runs `initConnection()`.
- `destroy()` → clears the poll timer.
- `getConfigFields()` → host, port (default `443`), username (`admin`), password, polling toggle, poll interval.

### Transport
- `makeRequest(endpoint, method, body, skipAuth)` — JSON requests. Auto-prefixes
  `/apis` if missing, uses `https` when `port === '443'` (else `http`), sets
  `rejectUnauthorized: false` for self-signed certs, 5s timeout.
- `makeRequestBinary(endpoint)` — used for thumbnail JPEG/PNG fetches.
- **Auth is cookie-based**: `authenticate()` POSTs to `/apis/authentication` and
  captures the `SessionID` cookie from `set-cookie`. All later requests send
  stored cookies. A `401` clears cookies and flips `authenticated = false`.

### Polling
When `config.polling` is on, `startPolling()` runs `getDeviceStatus()` every
`pollInterval` seconds. `getDeviceStatus()`:
- Reads `/apis/status` for system vars.
- Loops encoders `0..3` via `/apis/videnc/{i}`, calls `processEncoderStatus()`.
- Uses `presetListCounter` to throttle: preset list every 5th poll, stream list
  every 3rd poll, thumbnails every 10th poll (only for active encoders).
- Calls `checkFeedbacks()` at the end.

### Key state on `self`
- `encodersStatus[0..3]` — last raw encoder responses (used by feedbacks).
- `encoderChoices` — dropdown choices, rebuilt in `buildDeviceChoices()`; names
  update live when an encoder is renamed on the device.
- `streamList` / `streamMap` / `streamChoices` — from `/apis/streams`.
- `presetList` / `presetChoices` / `presetInfo` — from `/apis/presets`.
- `encoderThumbnails[index]` — base64 PNG data URIs (resized 72×72 via Jimp).

## Conventions & domain facts

- **Encoders are indexed 0–3** everywhere (API, variables `encoder0_*`, choices).
  User-facing labels often say "Encoder 1–4"; keep the underlying id 0-based.
- **Enum mappings** (defined inline; reuse these exact values):
  - Encoder state: `0=Stopped, 3=Awaiting Frame, 5=Not Encoding, 7=Working, 8=Resetting, 128=Failed`
  - Codec: `0=H.264, 1=H.265`
  - Stream encapsulation: `2=UDP, 3=RTP, 34=SRT, 64=RTSP`
  - Stream state: `0=Stopped,1=Streaming,2=Failed,3=Connecting,4=Securing,5=Listening,6=Paused,7=Publishing,8=Resolving,9=Scrambled`
- **Bitrate display**: values `>= 1000` kbps are shown as Mbps with one decimal.
- **Presets are `.cfg` files**: preset actions auto-append `.cfg` if missing.
- **Destructive actions require a `confirm` checkbox** (`reboot_device`,
  `delete_stream`, `delete_system_preset`) — preserve this pattern for any new
  dangerous action.
- After mutating actions, the code does `setTimeout(() => self.getDeviceStatus()/getStreamList(), 1000)` to refresh state. Follow this idiom.
- Calls that change available choices (`buildDeviceChoices`, `getStreamList`,
  `getPresetList`) re-call `updateActions()`/`updateFeedbacks()` so dropdowns refresh.

## Adding things

- **New action** → add an entry to the object in `src/actions.js`. Reuse
  `deviceNumberOption` for encoder-targeted actions and `self.streamChoices` for
  stream dropdowns. Wrap the REST call in try/catch and `self.log('info'|'error', ...)`.
- **New variable** → declare it in `src/variables.js` (definition + initial
  value) AND populate it from a poll handler in `index.js`
  (`processEncoderStatus` / `processStreamStatuses` / `getDeviceStatus`).
- **New feedback** → `src/feedbacks.js`. Boolean feedbacks read from
  `self.encodersStatus`; the thumbnail feedback is `type: 'advanced'` returning `{ png64 }`.
- **Breaking config/option changes** → add an upgrade script to the array in
  `src/upgrades.js` so existing user configs migrate.
- Bump `version` in **both** `package.json` and `companion/manifest.json` together.

## Releasing & roadmap

- **`RELEASING.md`** is the source of truth for cutting a release: semver bump in
  both `package.json` and `companion/manifest.json`, `CHANGELOG.md` entry, git tag
  `vX.Y.Z`, then submit via the Bitfocus Developer Portal. Follow it; don't invent
  an ad-hoc process.
- **`CHANGELOG.md`** — add entries under `[Unreleased]` as you make user-facing changes.
- **`ROADMAP.md`** — planned work and known bugs, grouped by milestone. Check it
  before starting a feature so effort lines up with priorities; tick items off in
  the PR that completes them.

## Known rough edges (don't assume these are intentional)

- There is no automated test suite yet. Use `node -c`/`yarn package` locally and
  verify behavior against a real Makito X4 when changing API behavior.
- Session expiry is not retried transparently. A `401` clears cookies and marks
  the connection unauthenticated; reconnect robustness is tracked in `ROADMAP.md`.
- The password config field is still a plain text input instead of Companion's
  `secret` type.

## Git workflow

- Use feature branches off `main` for new work.
- Commit with clear messages; push with `git push -u origin <branch>`.
- Do **not** open a pull request unless explicitly asked.
