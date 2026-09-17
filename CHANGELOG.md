# Changelog

All notable changes to the RALF Specification will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.1.0] - 2026-06-01

### Added
- New `urn:rdk:config:runtime` section (REQUIRED for `runtime` packages)
  advertising supported application types via `supportedApplicationTypes`, with
  optional per-type `args`. Lets a config generator select a runtime without
  overloading `packageSpecifier`. Enforced in the schema for runtime packages.
- New `urn:rdk:config:env` section for declaring environment variables the
  runtime manager MUST export into the container (validated against POSIX naming).
- Schema now validates all documented `urn:rdk:config:*` sections instead of
  passing them through unchecked; unknown keys remain accepted (list is extensible).
- `Resource` column added to the "Metadata Available per Package Type" table.
- Test fixtures covering every config section and package type.

### Changed
- Clarified runtime resolution: an explicit runtime `dependencies` entry takes
  precedence over `packageSpecifier`.
- Clarified that a runtime-dependent `application`/`service` MUST provide a runtime
  `dependencies` entry or a `packageSpecifier`; self-contained apps MAY omit both.
  Both fields remain OPTIONAL.
- `urn:rdk:config:overrides` now applies to `application` and `runtime` only
  (`base` removed; schema rejects other scopes).
- `entryPoint` is now OPTIONAL for `resource` packages, REQUIRED otherwise.
- Corrected `maxTimeToSuspendMemoryState` back to an integer type.
- Memory/storage size suffixes are now case-insensitive (`g`/`m`/`b`).
- Fixed a malformed inline `packageSpecifier` JSON example.

## [1.0.3] - 2026-05-11

### Changed
- Schema canonical URL now tracks `main` on raw.githubusercontent.com
  instead of GitHub Pages. The schema `$id`, README, format.md, and
  RELEASING.md were updated accordingly. The GitHub Release asset
  (`package.schema-vX.Y.Z.json`) remains the version-pinned source.

### Removed
- GitHub Pages deploy steps from the release workflow. `deploy-pages@v4`
  requires an `environment:` block, but the `github-pages` environment's
  protection rules in this repository block tag-triggered deploys. Per-
  version schema availability is preserved via Release assets.

## [1.0.2] - 2026-05-11

### Fixed
- Release workflow no longer pinned to the `github-pages` deployment
  environment; tag-triggered Pages deploys are no longer blocked by
  environment protection rules. (1.0.1 tag was published but its
  workflow failed before producing any artifacts.)

## [1.0.0] - 2026-05-11

### Added
- Initial formal release of the RALF Specification.
- `specVersion` metadata field (required) declaring the spec version a package was authored against.
- Versioning policy section in `format.md` covering MAJOR/MINOR/PATCH semantics and schema compatibility.
- Version headers in `README.md`, `format.md`, and `metadata.md`.
- MAJOR-versioned schema paths: schema relocated to `schema/v1/package.schema.json` with `$id`
  pointing at the GitHub Pages canonical URL.
- Tag-driven release workflow (`.github/workflows/release.yml`) publishing a GitHub Release and the
  schema on GitHub Pages.
- PR validation workflow (`.github/workflows/validate.yml`) compiling the schema with ajv.
- Maintainer release runbook (`RELEASING.md`).
