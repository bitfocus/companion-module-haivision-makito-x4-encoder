# Roadmap

Direction for the `haivision-makitox4-encoder` Companion module. This is a living
document — reprioritize freely. Items are grouped by milestone and mapped to the
semver bump they imply (see `RELEASING.md`).

## Where we are (v1.0.0)

Working: device connection + cookie auth, polling, encoder control (start/stop/
toggle/restart for encoders 0–3), encoder config (bitrate/resolution/framerate/
codec), full stream management (UDP/RTP/SRT/RTSP), system presets, preview-service
thumbnails, reboot, and a custom API call escape hatch. Variables, feedbacks, and
ready-made presets ship for all of the above.

## Milestone 1 — Correctness & polish (v1.0.x, patch releases)

Fix the known rough edges flagged in `CLAUDE.md`. These are bugs, not features.

- [x] **Encoder state mapping mismatch.** Feedbacks (`encoder_status`) and
      `encoder_toggle` now use the numeric state mapping (`0/3/5/7/8/128`) from
      `processEncoderStatus()`; resolution/framerate/codec match feedbacks read
      the correct fields (codec compares numeric `codecAlgorithm`).
- [x] **Broken preset variable references.** Presets now reference real variable
      ids (e.g. `encoder0_state`) via the connection's own label (`self.label`)
      instead of the non-existent `$(makitox4:...)` tokens.
- [x] **Write real docs.** `companion/HELP.md` and `README.md` now document setup,
      config fields, and features.
- [x] **`set_stream_destination` no-op removed.** Stream destinations are set via
      the `Create Stream` / `Edit Stream` actions.

## Milestone 2 — Connection robustness (v1.1.0, minor)

- [ ] **Session refresh.** On `401`, re-authenticate and retry transparently
      instead of just clearing cookies and surfacing an error.
- [ ] **Reconnect/backoff.** Detect connection loss during polling and retry with
      backoff, updating `InstanceStatus` accordingly.
- [ ] **Secret handling.** Move the password to Companion's `secret`-type config
      field so it isn't stored/displayed in plaintext.
- [ ] **HTTPS verification option.** Today `rejectUnauthorized` is always `false`.
      Add a config toggle for environments with valid certs.

## Milestone 3 — Feature coverage (v1.x, minor releases)

- [ ] **Audio encoder support** — actions/variables/feedbacks for `/apis/audenc`
      (the device has audio encoders; only video is covered today).
- [ ] **Input source / framing** — select and report video input source and
      framing per encoder.
- [ ] **Richer stream config** — TTL, FEC, SRT passphrase/encryption,
      bandwidth overhead, and more encapsulation options in `create_stream`/`edit_stream`.
- [ ] **Expand SRT stats** — surface more of the SRT statistics block as variables
      and add SRT-health feedbacks (RTT/loss thresholds).
- [ ] **Preset dropdowns** — use the already-built `presetChoices` in the preset
      actions (load/delete/rename/etc.) instead of free-text `.cfg` names.
- [ ] **Metadata / KLV and ancillary data** actions if needed by users.

## Milestone 4 — Engineering quality (ongoing)

- [ ] **Upgrade-script discipline.** Any id/option/config rename ships with an
      entry in `src/upgrades.js`. Establish this before the first breaking change.
- [ ] **Transport refactor.** Extract the HTTP/auth/cookie logic from `index.js`
      into its own module; reduce duplication between `makeRequest` and
      `makeRequestBinary`.
- [ ] **Automated tests** for the pure logic (state/encap/bitrate mappings,
      `.cfg` normalization) so refactors are safe — there is no test suite today.
- [ ] **Configurable encoder/stream counts** instead of the hardcoded 4 encoders /
      10 streams, driven by what the device reports.

## Possible future work

- Companion module for the **Makito X4 Decoder** (separate repo, shares transport patterns).
- Multi-device / device-group control.

## How to propose changes

Open a GitHub issue describing the use case, or add a checklist item here in a PR.
When an item is picked up, implement it on a feature branch off `main`, then cut a
release per `RELEASING.md` with the matching semver bump and a `CHANGELOG.md` entry.
