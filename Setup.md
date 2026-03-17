# Sample GitHub Action Based SBOM Template

This project contains a reusable GitHub action pipeline to create, scan, and publish an SBOM to CoreStack.

## Required Secrets and Variables

The following secrets must be available in the client's organizational secrets.  The API key must come from a user with appropriate access to the client's tenant in CoreStack. Variables are populated as needed or required in the project's build.yml. 

### Secrets

| Name | Type | Required | Description |
|---|---|---|---|
| `CORESTACK_ACCESS_KEY` | Secret | Yes | CoreStack API access key |
| `CORESTACK_SECRET_KEY` | Secret | Yes | CoreStack API secret key |
| `REGISTRY_PASSWORD` | Secret | No | Password for container registry if needed |

### Inputs

| Name | Type | Required | Unique | Description |
|---|---|---|---|---|
| `FILE` | String | FILE or PATH or IMAGE | No | Path to input file for SBOM generation (e.g. Dockerfile or binary) - mutually exclusive with IMAGE and PATH |
| `PATH` | String | FILE or PATH or IMAGE | No | Path to directory for SBOM generation (e.g. folder holding package.json and /src directory) - mutually exclusive with IMAGE and FILE |
| `IMAGE` | String | FILE or PATH or IMAGE | No | Container image reference for SBOM generation  (e.g. myregistry/myapp:1.0) - mutually exclusive with PATH and FILE |
| `REGISTRY_USERNAME` | String | No | No | Username for container registry if needed |
| `project_name` | String | Yes | Yes | AppSecOps project name (e.g. Payment Service) - used to resolve project_id |
| `sbom_definition_name` | String | No | Yes | Name for the SBOM definition (defaults to filename without extension) |
| `build_id` | String | No | Yes | Build/version identifier (defaults to github.run_id-github.run_attempts) - must be unique |
| `api_base_url` | String | No | No | CoreStack API base URL (defaults to production CoreStack API) |
| `force_upload` | String | No | No | Allow upload if same content exists elsewhere (true/false) |

### Notes

* If for some reason you need to scan multiple directories in the same repository, make sure to use the `sbom_definition_name` variable, and change it for each client workflow.  If it is not set, it defaults to the repository name, so all SBOMs will upload to the same definition, regardless of the `project_name`

* If scanning a container image, the `REGISTRY_PASSWORD` secret and `REGISTRY_USERNAME` input are available so that Syft can pull and scan the image.

* `build_id` must be unique.  The reuseable workflow will try to make any entry unique.  If set to 2.0.2, it will set the next id to 2.0.3, 2.0.4, etc.  If set to 2.0.1.1-beta, it will set the next id to 2.0.1.2-beta, 2.0.1.3-beta, etc.  If left blank, it will set to 1.0.0 and increment from there.

* `IMAGE` is passed in the following format:  registry/user/image:tag (e.g. docker.io/username/demo:latest)

* GitHub will not pass secrets in an input.  They must be passed in the secrets section of call-workflow.

### Troubleshooting

* Most steps in the workflow will echo information at the completion of the step or job.

* If the workflow fails, be sure to check the inputs section of Set Up Jobs.  Check to see if any input values failed to pass from the calling workflow.

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

## Example projects with workflows

Example projects have been created in the `graphion_workflow_templates_example` repo.  There are 3 examples including Terraform, Angular, and Docker projects.  A workflow has been created and tested for each, and the SBOMs upload to the CNAPP-DEMO CoreStack instance.