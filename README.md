# companion-module-haivision-makito-x4-encoder

A [Bitfocus Companion](https://bitfocus.io/companion) module to control and
monitor **Haivision Makito X4 Encoder** devices over their REST API.

It provides actions, feedbacks, variables, and presets for encoder control
(start/stop/settings), stream management (UDP/RTP/SRT/RTSP), system `.cfg`
presets, encoder thumbnails, and device control.

## Usage

Add the connection in Companion and enter the device IP, port, and credentials.
See [companion/HELP.md](./companion/HELP.md) for configuration and feature details.

## Development

This is a plain JavaScript (CommonJS) module built on `@companion-module/base`.

```bash
yarn install     # install dependencies (Yarn 4)
yarn format      # run Prettier
yarn package     # build a distributable package (companion-module-build)
```

- [`CLAUDE.md`](./CLAUDE.md) — architecture and conventions overview.
- [`ROADMAP.md`](./ROADMAP.md) — planned work and known issues.
- [`RELEASING.md`](./RELEASING.md) — how to cut and submit a release.
- [`CHANGELOG.md`](./CHANGELOG.md) — version history.

## License

[MIT](./LICENSE)
