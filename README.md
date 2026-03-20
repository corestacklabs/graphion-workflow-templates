# Graphion™ Pipeline 

**CoreStack Graphion™** helps enterprises stay ahead in fast-moving, cloud-native environments where constant change and third-party components create hidden risks. Powered by the **Graphion AI Agent**, it turns complex **SBOM** and **IBOM** relationships into clear, actionable intelligence so teams instantly see what’s vulnerable, what’s connected, and what matters most. By unifying **AppSec, SSCS, CSPM, APM, continuous compliance, and AI-guided remediation**, Graphion strengthens cloud posture and accelerates secure operations. It gives **Dev, Sec, Ops, and System Owners** the real-time context they need, **automates trust and cATO workflows**, and continuously validates assets across **build, deploy, and runtime** to help organizations move faster and stay secure.

This repository contains a reusable GitHub action pipeline (./github/workflows/sbom.yml) to create, scan, and publish an SBOM to CoreStack.  In addition, for container images, the pipeline will lint the dockerfile with Hadolint and scan the image with Dockle.

## Secrets and Inputs

The following secrets must be available in the client's organizational secrets.  The API key must come from a user with appropriate access to the client's tenant in CoreStack. Variables are populated as needed or required in the project's build.yml. 

### Secrets

| Name | Type | Required | Description |
|---|---|---|---|
| `CORESTACK_ACCESS_KEY` | Secret | Yes | CoreStack API access key |
| `CORESTACK_SECRET_KEY` | Secret | Yes | CoreStack API secret key |
| `REGISTRY_PASSWORD` | Secret | No | Password for container registry if needed |

* **Note:** To obtain the `CORESTACK_ACCESS_KEY` and `CORESTACK_SECRET_KEY` for your CoreStack Tenant, find the instructions in the following document. The steps needed are listed in the section titled: How to get the Access Key and Secret Key

* **[CoreStack External APIs](https://docs.corestack.io/docs/corestack-api-modules)**

### Inputs

#### Inputs for Syft SBOM Creation
| Name | Type | Required | Description |
|---|---|---|---|
| `FILE` | String | FILE or PATH or IMAGE | Path to input file for SBOM generation (e.g. Dockerfile or binary) - mutually exclusive with IMAGE and PATH |
| `PATH` | String | FILE or PATH or IMAGE | Path to directory for SBOM generation (e.g. folder holding package.json and /src directory) - mutually exclusive with IMAGE and FILE |
| `IMAGE` | String | FILE or PATH or IMAGE | Container image reference for SBOM generation  (e.g. myregistry/myapp:1.0) - mutually exclusive with PATH and FILE |

#### Inputs for Hadolint/Dockle Container Scanning (only if `IMAGE` is used)
| Name | Type | Required | Default | Description |
|---|---|---|---|---|
| `REGISTRY_USERNAME` | String |  No | | Username for container registry |
| `REGISTRY_REGION` | String |  No | | Region for container registry - **only for AWS ECR** |
| `dockerfile_path` | String |  No | ./dockerfile | Path to Dockerfile for Hadolint analysis |

#### Inputs for CoreStack to Upload Scan Results
| Name | Type | Required | Unique | Default | Description |
|---|---|---|---|---|---|
| `project_name` | String | Yes | Yes | | AppSecOps project name (e.g. Payment Service) - used to resolve project_id |
| `sbom_definition_name` | String | No | Yes | repo name | Name for the SBOM definition (defaults to filename without extension) |
| `build_id` | String | No | Yes | 1.0.1 | Build/version identifier (defaults to github.run_id-github.run_attempts) - must be unique |
| `api_base_url` | String | No | No | api.corestack.io | CoreStack API base URL (defaults to production CoreStack API) |
| `force_upload` | String | No | No | false | Allow upload if same content exists elsewhere (true/false) |

### Notes

* If for some reason you need to scan multiple directories in the same repository, make sure to use the `sbom_definition_name` variable, and change it for each client workflow.  If it is not set, it defaults to the repository name, so all SBOMs will upload to the same definition, regardless of the `project_name`

* If scanning a container image, the `REGISTRY_PASSWORD` secret and `REGISTRY_USERNAME` input are available so that Syft can pull and scan the image.

* `build_id` must be unique.  The reuseable workflow will try to make any entry unique.  If set to 2.0.2, it will set the next id to 2.0.3, 2.0.4, etc.  If set to 2.0.1.1-beta, it will set the next id to 2.0.1.2-beta, 2.0.1.3-beta, etc.  If left blank, it will set to 1.0.0 and increment from there.

* `IMAGE` is passed in the following format:  registry/user/image:tag (e.g. docker.io/username/demo:latest)

* GitHub will not pass secrets in an input.  They must be passed in the secrets section of call-workflow.

### Troubleshooting

* Most steps in the workflow will echo information at the completion of the step or job.

* If the workflow fails, be sure to check the **inputs** section of **Set Up Jobs**.  Check to see if any input values failed to pass from the calling workflow.

## Sample client build.yml workflow file 

In the client or calling workflow, find a point where the SBOM can be created and insert a call-workflow job.  Calling a workflow must be done from a job, not a step so be prepared to create outputs, download artifacts, etc.  The calling workflow does not have to be called on workflow_dispatch, that was easiest for the examples.

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

Example projects have been created in the **[Example](../graphion_workflow_templates_example/)** repo.  There are 3 examples including Terraform, Angular, and Docker projects.  A workflow has been created and tested for each, and the SBOMs upload to the CNAPP-DEMO CoreStack instance.