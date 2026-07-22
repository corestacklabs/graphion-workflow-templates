# Create SBOM Pipeline

This repository contains a reusable GitHub Actions workflow (`.github/workflows/create-sbom.yml`) that generates a CycloneDX SBOM with [Syft](https://github.com/anchore/syft), lints a Dockerfile with [Hadolint](https://github.com/hadolint/hadolint), and scans a container image with [Dockle](https://github.com/goodwithtech/dockle). Each of the three can be turned on or off independently.

This is the first stage of the split SBOM pipeline: **create-sbom.yml → [scan-sbom.yml](scan-sbom.md) → [upload-sbom.yml](upload-sbom.md)**. It does not talk to CoreStack and does not require CoreStack credentials — it only produces artifacts for the next stage to pick up. If you want the older all-in-one behavior, see `sbom.yml` / [full-sbom.md](full-sbom.md) instead.

## Secrets and Inputs

### Secrets

| Name | Type | Required | Description |
|---|---|---|---|
| `REGISTRY_PASSWORD` | Secret | No | Password for container registry if needed (only used when `IMAGE` is set) |

### Inputs

| Name | Type | Required | Default | Description |
|---|---|---|---|---|
| `project_name` | String | Yes | | Tag used to name the output artifacts (`sbom-cdx-<project_name>`, `hadolint-json-report-<project_name>`, `dockle-report-<project_name>`). Must match the value passed to `scan-sbom.yml`/`upload-sbom.yml` so they can find these artifacts. |
| `FILE` | String | FILE or PATH or IMAGE | | Path to input file for SBOM generation (e.g. Dockerfile or binary) - mutually exclusive with IMAGE and PATH |
| `PATH` | String | FILE or PATH or IMAGE | | Path to directory for SBOM generation |
| `IMAGE` | String | FILE or PATH or IMAGE | | Container image reference for SBOM generation (e.g. myregistry/myapp:1.0) - mutually exclusive with PATH and FILE |
| `REGISTRY_USERNAME` | String | No | | Username for container registry (only used when `IMAGE` is set) |
| `REGISTRY_REGION` | String | No | | Region for container registry - **only for AWS ECR** |
| `digest_tool` | String | No | 'skopeo' | Tool used to retrieve the image RepoDigest SHA-256. Use `skopeo` (default, no credential storage) or `docker` (docker login + pull + inspect). |
| `dockerfile_path` | String | No | | Path to Dockerfile for hadolint analysis. Leave empty to auto-extract from image or history when IMAGE is provided. |
| `generate_sbom` | String | No | 'true' | Generate a CycloneDX SBOM with Syft (true/false) |
| `run_hadolint` | String | No | 'true' | Lint the Dockerfile with Hadolint (true/false) - only applies when IMAGE is provided |
| `run_dockle` | String | No | 'true' | Scan the image with Dockle (true/false) - only applies when IMAGE is provided |

### Notes

* `run_hadolint` and `run_dockle` are no-ops when `IMAGE` isn't set — there's nothing for either tool to scan with just `FILE`/`PATH`. Leave them at their defaults for non-image scans.
* The workflow pulls the image at most once and only saves it to a tar when Syft or Dockle actually need it, regardless of which combination of flags is enabled — that consolidation is the whole point of this workflow existing as its own step.
* `digest_tool` only matters when `generate_sbom` is true and `IMAGE` is set — it controls how the image's SHA-256 RepoDigest gets embedded into the SBOM.
* Output artifacts only appear for the tools you enabled: `sbom-cdx-<project_name>` (if `generate_sbom`), `hadolint-json-report-<project_name>` (if `run_hadolint` and `IMAGE`), `dockle-report-<project_name>` (if `run_dockle` and `IMAGE`).

### Troubleshooting

* If the workflow fails, check the **Validate inputs** step first — it reports missing FILE/PATH/IMAGE or missing registry credentials.
* If `scan-sbom.yml`/`upload-sbom.yml` can't find an expected artifact, confirm `project_name` is passed identically to every stage of the pipeline.

## Sample client build.yml workflow file

`create-sbom.yml` is one stage of a chain — it's typically followed by `scan-sbom.yml` and then `upload-sbom.yml`, each its own job with `needs:` pointing back through the chain.

### Container image example (Syft + Hadolint + Dockle, then hand off to scan-sbom.yml)

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
```

### SBOM-only example (no Hadolint/Dockle, e.g. an Angular build via PATH)

```yaml
  create-sbom:
    needs: angular-build
    uses: corestacklabs/graphion-workflow-templates/.github/workflows/create-sbom.yml@main
    with:
      PATH: ${{ needs.angular-build.outputs.path }}
      project_name: 'Angular App'
```

### Container linting only, no SBOM (Hadolint + Dockle, skip Syft)

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
      generate_sbom: 'false'
```
