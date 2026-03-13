# Sample GitHub Action Based SBOM Template

This project contains a reusable GitHub action pipeline to create, scan, and publish an SBOM to CoreStack.

## Required Secrets and Variables

The following secrets must be available in the client's organizational secrets.  The API key must come from a user with appropriate access to the client's tenant in CoreStack. Variables are populated as needed or required in the project's build.yml. 

### Required Secrets

| Name | Type | Purpose |
|---|---|---|
| `CORESTACK_ACCESS_KEY` | Secret | CoreStack API access key |
| `CORESTACK_SECRET_KEY` | Secret | CoreStack API secret key |

### Variables

| Name | Type | Required | Purpose |
|---|---|---|---|
| `FILE` | String | FILE or PATH or IMAGE | Path to input file for SBOM generation (e.g. Dockerfile or binary) - mutually exclusive with IMAGE and PATH |
| `PATH` | String | FILE or PATH or IMAGE | Path to directory for SBOM generation (e.g. folder holding package.json and /src directory) - mutually exclusive with IMAGE and FILE |
| `IMAGE` | String | FILE or PATH or IMAGE | Container image reference for SBOM generation  (e.g. myregistry/myapp:1.0) - mutually exclusive with PATH and FILE |
| `project_name` | String | Yes | AppSecOps project name (e.g. Payment Service) - used to resolve project_id |
| `sbom_definition_name` | String | No | Name for the SBOM definition (defaults to filename without extension) |
| `build_id` | String | No | Build/version identifier (defaults to github.run_id-github.run_attempts) - must be unique |
| `api_base_url` | String | No | CoreStack API base URL (defaults to production CoreStack API) |
| `force_upload` | String | No | Allow upload if same content exists elsewhere (true/false) |

### Notes

If for some reason you need to scan multiple directories in the same repository, make sure to use the `sbom_definition_name` variable, and change it for each client workflow.  If it is not set, it defaults to the repository name, so all SBOMs will upload to the same definition, regardless of the `project_name`

## Sample client build.yml workflow file 

In the client workflow, find a point where the SBOM can be created and insert a call-workflow job.

```yaml
name: Angular Build Test

on: workflow_dispatch

env:
  WORKING_DIRECTORY: './Web'
  NODE_VERSION: 10.9.0

jobs:
  angular-build:
    name: Build Angular App
    runs-on: ubuntu-latest
    [...]

  call-workflow:
    needs: angular-build
    uses: corestacklabs/graphion-workflow-templates/.github/workflows/sbom.yml@main
    secrets:
      CORESTACK_ACCESS_KEY: ${{ secrets.CORESTACK_ACCESS_KEY }} 
      CORESTACK_SECRET_KEY: ${{ secrets.CORESTACK_SECRET_KEY }} 
    with:
      PATH: ${{ needs.angular-build.outputs.path }}
      project_name: ClientAppName
```