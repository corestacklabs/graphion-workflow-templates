# Changelog

All notable changes to the reusable SBOM workflow (`.github/workflows/sbom.yml`) are documented here.

## Unreleased

### Changed

- **SBOM attachment name in CoreStack.** The uploaded SBOM attachment is now named using the `project_name` input (lowercased, spaces replaced with `-`) instead of the GitHub repository name, e.g. `my-project-sbom.json` instead of `my-repo-sbom.json`. This is by design — attachment names should reflect the CoreStack AppSecOps project, not the source repo. Hadolint and Dockle attachment names are unaffected and still use the repository name.

  **Action required:** any downstream automation, saved search, or dashboard filter that matches SBOM attachments by the old `{repo-name}-sbom.json` pattern must be updated to match on `{project-name-slug}-sbom.json`.
