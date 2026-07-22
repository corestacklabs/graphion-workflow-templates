# Changelog

All notable changes to the reusable SBOM workflow (`.github/workflows/sbom.yml`) are documented here.

## 2026-07-22

### Added

- **Split pipeline workflows.** `create-sbom.yml`, `scan-sbom.yml`, and `upload-sbom.yml` let you compose SBOM generation, vulnerability scanning, and CoreStack upload as independent reusable workflows instead of the monolithic `sbom.yml`. `kics.yml` is now also available standalone (previously only inline in `sbom.yml` via `KICS_scan_type`). See README.md.

### Changed

- **SBOM attachment name in CoreStack.** The uploaded SBOM attachment is now named using the `project_name` input (lowercased, spaces replaced with '-') instead of the GitHub repository name, e.g. `my-project-sbom.json` instead of `my-repo-sbom.json`. This is by design — attachment names should reflect the CoreStack AppSecOps project, not the source repo. Hadolint and Dockle attachment names are unaffected and still use the repository name.

  **Action required:** any downstream automation, saved search, or dashboard filter that matches SBOM attachments by the old `{repo-name}-sbom.json` pattern must be updated to match on `{project-name-slug}-sbom.json`.

- **Reduced image pulls/disk usage for IMAGE scans.** `sbom.yml` and `create-sbom.yml` now pull the image once, save it to a tar shared by Syft and Dockle, and free the daemon's copy as soon as nothing else needs it (skipped when `digest_tool: docker`).
- **Dockle now scans the saved image tar** instead of the live daemon/registry — avoids daemon-read failures under disk pressure and digest re-validation failures on ACR/multi-arch manifest lists.
- **`digest_tool` input added** (`skopeo` default, or `docker`) — controls how the image's SHA-256 RepoDigest is retrieved, with a free fast-path check against Syft's own embedded digest first.
- **Fixed `build_id` auto-increment** for multi-digit or zero-padded trailing numbers (e.g. `2.010` incremented incorrectly before; now increments the full trailing numeric segment and preserves zero-padding).
- Pinned tool versions for reproducibility: Grype `v0.116.0`, Hadolint `v2.14.0`, Dockle `v0.4.15` (previously Dockle resolved dynamically via the GitHub releases API on every run), plus `actions/checkout@v7`, `actions/upload-artifact@v7`, `actions/cache@v6`.
