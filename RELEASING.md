# Releasing this module

This module follows the standard **Bitfocus Companion** module release flow.
As of Companion v4.0 the module release cycle is decoupled from Companion's own
releases — we cut versions on our own schedule, and the Bitfocus build system
picks them up.

References (read these if anything below is unclear):

- Releasing a module: https://companion.free/for-developers/module-development/module-lifecycle/releasing-your-module/
- Versioning rules: https://companion.free/for-developers/git-workflows/versioning/
- Developer Portal: https://developer.bitfocus.io/

## Versioning

Use semantic versioning `MAJOR.MINOR.PATCH` (e.g. `1.2.3`). Only plain
`major.minor.patch` releases are bundled into stable Companion builds.

- **PATCH** — bug fixes, no new actions/feedbacks/variables, no config changes.
- **MINOR** — new actions/feedbacks/variables/presets that don't break existing configs.
- **MAJOR** — breaking changes to action/feedback option ids, variable ids, or
  config fields. A MAJOR bump usually needs an **upgrade script** (see below).

The version lives in **two files that must stay in sync**:

- `package.json` → `"version"`
- `companion/manifest.json` → `"version"`

## Pre-release checklist

1. All changes merged to `main` and tested against a real Makito X4 (or with the
   `custom_api_call` action against a device).
2. `yarn format` run clean (CI fails otherwise).
3. `yarn package` succeeds locally — this runs `companion-module-build` and is the
   same packaging the CI `module-checks` workflow performs.
4. If any action option id, feedback id, variable id, or config field changed,
   add an **upgrade script** to the array in `src/upgrades.js` so existing user
   configurations migrate cleanly. Never silently rename/remove an id without one.
5. `CHANGELOG.md` updated with a new section for the version (see that file).
6. `version` bumped identically in `package.json` and `companion/manifest.json`.

## Cutting the release

```bash
# 1. Bump versions (edit package.json + companion/manifest.json), update CHANGELOG.md
# 2. Commit the bump
git add package.json companion/manifest.json CHANGELOG.md
git commit -m "Release v1.2.3"

# 3. Tag with a leading "v" and push tag + main
git tag v1.2.3
git push origin main
git push origin v1.2.3
```

A GitHub Release (Releases → Draft a new release → create tag `v1.2.3`) is an
acceptable alternative to the local `git tag` — it creates and pushes the tag for you.

Pushing triggers `.github/workflows/companion-module-checks.yaml`
(`bitfocus/actions` → `module-checks`). The tag itself does **not** auto-publish
to Companion — submission is a manual step in the portal.

## Submitting to the Bitfocus Module Store

1. Go to the **Developer Portal**: https://developer.bitfocus.io/ (log in with GitHub).
2. **My Connections** in the sidebar → select this module
   (`haivision-makito-x4-encoder`).
3. **Submit Version** at the bottom → choose the tag you just pushed.
4. Tick **Is Prerelease** if it's a beta (goes to Companion beta builds, not stable).
5. Submit. Once the module workflow passes, a prerelease lands in Companion beta
   builds within ~6 hours; stable `major.minor.patch` versions are bundled with
   Companion releases.

## First-time listing (one-time, already done for this module)

This module is already registered with Bitfocus (id `haivision-makito-x4-encoder`,
manufacturer `Haivision`). For reference, a brand-new module is claimed by posting
in the Bitfocus Slack `#module-development` channel with your GitHub username and
the desired name in `manufacturer-product` format before the first portal submission.

## After releasing

- Verify the new version appears in the Developer Portal as accepted.
- Start the next development cycle on a feature branch off `main`; do not commit
  further version bumps until the next release.
