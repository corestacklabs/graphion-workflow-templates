# Upload SBOM Pipeline

This repository contains a reusable GitHub Actions workflow (`.github/workflows/upload-sbom.yml`) that uploads a Grype-scanned SBOM (and, optionally, Hadolint/Dockle reports) to CoreStack AppSecOps.

This is the third and final stage of the split SBOM pipeline: **[create-sbom.yml](create-sbom.md) → [scan-sbom.yml](scan-sbom.md) → upload-sbom.yml**. It performs the same CoreStack API calls (auth, project lookup, SBOM definition lookup/create, version upload, attachment upload, container findings ingest) as the `upload-sbom` job in `sbom.yml`/`sbomdev.yml` — only the artifact-passing mechanism is different, since this now runs as its own reusable workflow rather than a second job in the same workflow file.

## Secrets and Inputs

The following secrets must be available in the client's organizational secrets. The API key must come from a user with appropriate access to the client's tenant in CoreStack.

### Secrets

| Name | Type | Required | Description |
|---|---|---|---|
| `CORESTACK_ACCESS_KEY` | Secret | Yes | CoreStack API access key |
| `CORESTACK_SECRET_KEY` | Secret | Yes | CoreStack API secret key |

* **Note:** To obtain the `CORESTACK_ACCESS_KEY` and `CORESTACK_SECRET_KEY` for your CoreStack Tenant, find the instructions in the following document. The steps needed are listed in the section titled: How to get the Access Key and Secret Key

* **[CoreStack External APIs](https://docs.corestack.io/docs/corestack-api-modules)**

### Inputs

| Name | Type | Required | Default | Description |
|---|---|---|---|---|
| `project_name` | String | Yes | | AppSecOps project name (e.g. Payment Service) - used to resolve project_id, and as the tag to find artifacts produced by `scan-sbom.yml`/`create-sbom.yml` (`sbom-scanned-<project_name>`, `hadolint-json-report-<project_name>`, `dockle-report-<project_name>`) |
| `project_tags` | String | No | | Optional tag filter to disambiguate project lookup when multiple projects share the same name (e.g. `service=governance`). Parsed as `key=value`. |
| `run_hadolint` | String | No | 'true' | Look for and attach a Hadolint report artifact (true/false) - must match what was passed to `create-sbom.yml` |
| `run_dockle` | String | No | 'true' | Look for and attach a Dockle report artifact (true/false) - must match what was passed to `create-sbom.yml` |
| `sbom_definition_name` | String | No | repo name | Name for the SBOM definition (defaults to filename without extension) |
| `sbom_definition_tags` | String | No | | Optional tag filter to disambiguate SBOM definition lookup when names collide across projects (e.g. `service=governance`). Parsed as `key=value`. |
| `build_id` | String | No | 1.0.1 | Build/version identifier (defaults to github.run_id-github.run_attempts) - must be unique |
| `api_base_url` | String | No | api.corestack.io | CoreStack API base URL (defaults to production CoreStack API) |
| `force_upload` | String | No | false | Allow upload if same content exists elsewhere (true/false) |
| `sanitize_licenses` | String | No | false | Strip invalid SPDX license entries from the SBOM before upload (true/false) |

### Notes

* **This workflow only accepts a Grype-scanned SBOM.** It requires an `sbom-scanned-<project_name>` artifact (produced by `scan-sbom.yml`) — it does not fall back to an unscanned SBOM, because CoreStack requires vulnerability-scanned SBOMs. If you have your own SBOM, run it through `scan-sbom.yml` first (see [scan-sbom.md](scan-sbom.md)).
* `sbom_definition_name` must stay the same across every upload for a given project — CoreStack only allows one SBOM definition per project. Reuse the same name across test runs and real runs; new uploads just create new versions under it.
* `run_hadolint`/`run_dockle` gate whether this workflow *looks for and attaches* those artifacts at all, independent of whether a matching artifact happens to exist — set them to `false` if `create-sbom.yml` was called with those disabled, so this workflow doesn't try to look for artifacts that were never produced.
* `sanitize_licenses` runs here (not in `create-sbom.yml`/`scan-sbom.yml`) so it applies uniformly regardless of which upstream stages actually ran.
* If for some reason you need to scan multiple directories in the same repository, make sure to use the `sbom_definition_name` variable, and change it for each client workflow. If it is not set, it defaults to the repository name, so all SBOMs will upload to the same definition, regardless of the `project_name`.
* GitHub will not pass secrets in an input. They must be passed in the secrets section of call-workflow.

### Troubleshooting

* "No scanned SBOM found" means `scan-sbom.yml` didn't run first with the same `project_name`, or its artifact wasn't named `sbom-scanned-<project_name>`.
* If the workflow fails, check the **Validate inputs** step first — it reports missing `project_name` or missing CoreStack secrets.

## Sample client build.yml workflow file

```yaml
  create-sbom:
    needs: build
    uses: corestacklabs/graphion-workflow-templates/.github/workflows/create-sbom.yml@main
    secrets:
      REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
    with:
      IMAGE: ${{ needs.build.outputs.tags }}
      REGISTRY_USERNAME: ${{ needs.build.outputs.organization }}
      project_name: 'Docker Container'
      dockerfile_path: "${{ needs.build.outputs.path }}/dockerfile"

  scan-sbom:
    needs: create-sbom
    uses: corestacklabs/graphion-workflow-templates/.github/workflows/scan-sbom.yml@main
    with:
      project_name: 'Docker Container'

  upload-sbom:
    needs: [create-sbom, scan-sbom]
    uses: corestacklabs/graphion-workflow-templates/.github/workflows/upload-sbom.yml@main
    secrets:
      CORESTACK_ACCESS_KEY: ${{ secrets.CORESTACK_ACCESS_KEY }}
      CORESTACK_SECRET_KEY: ${{ secrets.CORESTACK_SECRET_KEY }}
    with:
      project_name: 'Docker Container'
      sbom_definition_name: 'docker-example-sbom'
```
