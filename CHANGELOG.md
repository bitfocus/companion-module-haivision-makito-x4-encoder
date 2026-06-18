# Changelog

All notable changes to this module are documented here.

The format is based on [Keep a Changelog](https://keepachangelog.com/), and this
module adheres to [Semantic Versioning](https://semver.org/). Add a new section
at the top for each release; see `RELEASING.md` for the full release process.

## [Unreleased]

### Added
- `CLAUDE.md`, `ROADMAP.md`, `RELEASING.md`, and this changelog.

## [1.0.0]

Initial release.

### Added
- Connection to Haivision Makito X4 Encoder via REST API with cookie-based
  (`SessionID`) authentication and optional polling.
- Encoder control actions: start, stop, toggle, restart for encoders 0–3.
- Encoder configuration actions: set bitrate, resolution, framerate/GOP, codec.
- Stream management: create, edit, delete, start, stop, toggle, restart
  (UDP / RTP / SRT / RTSP).
- System preset actions: save, load, delete, rename, duplicate, set startup,
  set autosave.
- Preview service enable/disable and per-encoder thumbnails (rendered via Jimp).
- Reboot device and a generic custom API call action.
- Variables for device info, per-encoder status (0–3), and up to 10 streams.
- Feedbacks for encoder status, bitrate comparison, resolution/framerate/codec
  match, connection status, and encoder thumbnail.
- Ready-made presets for encoder control, settings, thumbnails, and status.
