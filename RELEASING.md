# Releasing the RALF Specification

This document describes how a maintainer cuts a new release of the RALF Specification. Releases follow
[Semantic Versioning](https://semver.org/spec/v2.0.0.html); see `format.md` for the policy on what
constitutes MAJOR/MINOR/PATCH changes.

## Prerequisites

- Push access to `main` and permission to create `v*` tags.
- The `Validate` workflow is green on `main`.
- The intended changes are already merged.

## Steps

1. **Decide the version.** Apply the rules in `format.md` to classify the changes as MAJOR, MINOR, or
   PATCH. For a pre-release, append `-rc.N` (for example `1.1.0-rc.1`).

2. **Open a release PR.** In a single PR:
   - Update the `**Version: X.Y.Z**` header in `README.md`, `format.md`, and `metadata.md`.
   - Add a `## [X.Y.Z] - YYYY-MM-DD` section to `CHANGELOG.md` listing the changes (use the
     Keep a Changelog categories: Added / Changed / Deprecated / Removed / Fixed / Security).
   - If this is a MAJOR bump, create `schema/vN/package.schema.json` (copy from the previous major,
     apply the breaking changes, update `$id`) and update references in `format.md`/`metadata.md`.

3. **Merge to `main`** after review and after the Validate workflow passes.

4. **Tag and push.** From a clean checkout of `main`:

   ```bash
   git fetch --tags
   git tag -a vX.Y.Z -m "RALF X.Y.Z"
   git push origin vX.Y.Z
   ```

5. **Watch the Release workflow.** The workflow runs automatically on the tag push and will:
   - verify that `CHANGELOG.md` and the doc version headers agree with the tag (`vX.Y.Z` → `X.Y.Z`);
   - compile the schema with `ajv`;
   - extract the matching CHANGELOG section as release notes;
   - publish a GitHub Release with `package.schema-vX.Y.Z.json` and `ralf-spec-X.Y.Z.tar.gz` attached.

6. **Verify the release.**
   - Confirm the GitHub Release page lists both artifacts and that pre-release tags are marked as such.
   - Confirm the canonical (main-tracking) schema URL serves the new content:
     `https://raw.githubusercontent.com/rdkcentral/oci-package-spec/main/schema/v1/package.schema.json`
   - Optionally validate a sample package against the published schema:
     ```bash
     npx ajv-cli@5 validate -s schema/v1/package.schema.json -d sample.json --spec=draft2020
     ```

## Troubleshooting

- **"CHANGELOG.md missing entry [X.Y.Z]"** — the tag and the CHANGELOG heading disagree. Fix the
  CHANGELOG on `main` in a follow-up PR, delete the bad tag, then retag.
- **"version header mismatch"** — one of the doc files still shows the previous version. Same fix
  as above.

## Tag and branch protection (one-time setup)

Configure in the GitHub UI:
- Tag protection rule `v*` — only maintainers may create matching tags.
- Branch protection on `main` — require the `Validate` workflow to pass before merge.
