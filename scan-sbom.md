# Scan SBOM Pipeline

This repository contains a reusable GitHub Actions workflow (`.github/workflows/scan-sbom.yml`) that runs [Grype](https://github.com/anchore/grype) against a CycloneDX SBOM and produces a vulnerability-annotated SBOM.

This is the second stage of the split SBOM pipeline: **[create-sbom.yml](create-sbom.md) → scan-sbom.yml → [upload-sbom.yml](upload-sbom.md)**. It never touches the container image or source code — it only downloads an SBOM artifact, scans it, and re-uploads the result. **CoreStack requires the SBOM to be Grype-scanned before upload** — `upload-sbom.yml` will not accept an unscanned SBOM, so this stage is required, not optional, if you're uploading to CoreStack.

## Secrets and Inputs

No secrets are required for this workflow.

### Inputs

| Name | Type | Required | Description |
|---|---|---|---|
| `project_name` | String | Yes | Tag used to find the input SBOM artifact (`sbom-cdx-<project_name>`, produced by `create-sbom.yml`) and to name the output artifact (`sbom-scanned-<project_name>`). Must match the value passed to `create-sbom.yml`/`upload-sbom.yml`. |

### Notes

* The input artifact must be named `sbom-cdx-<project_name>` and contain exactly one file — any filename works, it's read regardless of what it's called. If you're bringing your own SBOM instead of using `create-sbom.yml`, upload it yourself as a single-file artifact with that name before calling this workflow.
* Output is `sbom-scanned-<project_name>`, containing `<repo-name>-sbom.json` (CycloneDX with vulnerability data attached).

### Troubleshooting

* "SBOM artifact is empty" or "must contain exactly one file" means either `create-sbom.yml` didn't run first with the same `project_name`, or a custom-uploaded SBOM artifact was named wrong or bundled more than one file.
* Grype failures print stderr directly in the step log — check for a malformed or empty SBOM file as the usual cause.

## Sample client build.yml workflow file

`scan-sbom.yml` sits between `create-sbom.yml` and `upload-sbom.yml`:

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
```

### Scan without uploading to CoreStack

Just omit the `upload-sbom` job — the Grype-scanned SBOM and its critical/high/medium counts are still available as a workflow artifact and in the job summary:

```yaml
  create-sbom:
    needs: build
    uses: corestacklabs/graphion-workflow-templates/.github/workflows/create-sbom.yml@main
    secrets:
      REGISTRY_PASSWORD: ${{ secrets.REGISTRY_PASSWORD }}
    with:
      IMAGE: ${{ needs.build.outputs.tags }}
      REGISTRY_USERNAME: ${{ needs.build.outputs.organization }}
      project_name: 'Docker Container Scan Only'

  scan-sbom:
    needs: create-sbom
    uses: corestacklabs/graphion-workflow-templates/.github/workflows/scan-sbom.yml@main
    with:
      project_name: 'Docker Container Scan Only'
```
