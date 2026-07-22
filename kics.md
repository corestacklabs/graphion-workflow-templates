# KICS Infrastructure Scan Pipeline

This repository contains a reusable GitHub Actions workflow (`.github/workflows/kics.yml`) that scans infrastructure-as-code files with [KICS](https://kics.io/) (Ansible, AzureResourceManager, CloudFormation, Dockerfile, Kubernetes, OpenAPI, Terraform) and optionally uploads the results to the GitHub Security tab.

This workflow does not upload anything to CoreStack — it is a standalone scan-only pipeline. No CoreStack secrets are required.

## Secrets and Inputs

No secrets are required for this workflow.

### Inputs

| Name | Type | Required | Default | Description |
|---|---|---|---|---|
| `PATH` | String | Yes | | Path to directory for infrastructure scan |
| `KICS_scan_type` | String | No | '' | Type of infrastructure to scan (Ansible, AzureResourceManager, CloudFormation, Dockerfile, Kubernetes, OpenAPI, Terraform). Must be set to a non-empty value or the job is skipped. |
| `KICS_upload` | String | No | 'false' | Upload the results to the GitHub Security tab (true/false) |

* **Advanced Security must be enabled on the repo for `KICS_upload` to work. The workflow will fail otherwise.**

### Notes

* The `infrastructure-scan` job only runs when `KICS_scan_type` is set to a non-empty value.
* `PATH` is required — the job fails fast with a validation error if it is missing.
* The raw KICS JSON results are always uploaded as a build artifact named `kics-output-json`, regardless of `KICS_upload`.

### Troubleshooting

* If the workflow fails, check the **Validate inputs** step first — it reports missing `PATH`.
* If `KICS_upload: 'true'` fails to upload, confirm GitHub Advanced Security is enabled for the repository.

## Sample client build.yml workflow file

In the client or calling workflow, find a point where the infrastructure scan should run and insert a call-workflow job. Calling a workflow must be done from a job, not a step.

### KICS Terraform scan example (Advanced Security Enabled)

To scan infrastructure files such as Ansible, CloudFormation, Terraform, Kubernetes, etc., the only required input is `KICS_scan_type` (and `PATH`). To import the results to GitHub Security, set `KICS_upload` to 'true' and make sure Advanced Security in the repo is enabled.

```yaml
  call-workflow:
    uses: corestacklabs/graphion-workflow-templates/.github/workflows/kics.yml@main
    with:
      PATH: './terraform'
      KICS_scan_type: 'Terraform'
      KICS_upload: 'true'
```

### KICS CloudFormation scan example

This is another example of the KICS scan with a CloudFormation type and no upload to GitHub Security.

```yaml
  call-workflow:
    uses: corestacklabs/graphion-workflow-templates/.github/workflows/kics.yml@main
    with:
      PATH: './cfn'
      KICS_scan_type: 'CloudFormation'
```
