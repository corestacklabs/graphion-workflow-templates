# Graphion™ Workflow Templates

**CoreStack Graphion™** helps enterprises stay ahead in fast-moving, cloud-native environments where constant change and third-party components create hidden risks. Powered by the **Graphion AI Agent**, it turns complex **SBOM** and **IBOM** relationships into clear, actionable intelligence so teams instantly see what’s vulnerable, what’s connected, and what matters most. By unifying **AppSec, SSCS, CSPM, APM, continuous compliance, and AI-guided remediation**, Graphion strengthens cloud posture and accelerates secure operations. It gives **Dev, Sec, Ops, and System Owners** the real-time context they need, **automates trust and cATO workflows**, and continuously validates assets across **build, deploy, and runtime** to help organizations move faster and stay secure.

This repository contains reusable GitHub Actions workflows for generating SBOMs, scanning them for vulnerabilities, linting/scanning container images, uploading results to CoreStack AppSecOps, and running standalone infrastructure-as-code scans.

**See [CHANGELOG.md](CHANGELOG.md) for notable behavior changes**, including changes that may require updating existing calling workflows.

## Which workflow do I want?

There are two ways to get an SBOM into CoreStack: one reusable workflow that does everything, or three smaller ones you compose yourself.

### Full pipeline (one call does it all)

| Workflow | Docs |
|---|---|
| `sbom.yml` — create, scan, and upload an SBOM in a single call (also lints/scans container images) | [full-sbom.md](full-sbom.md) |

Use this if you just want "SBOM in, CoreStack upload out" with no need to customize which scans run.

### Split pipeline (compose what you need)

| Workflow | Docs |
|---|---|
| `create-sbom.yml` — Syft SBOM, Hadolint lint, Dockle scan; each independently toggleable | [create-sbom.md](create-sbom.md) |
| `scan-sbom.yml` — Grype vulnerability scan of an SBOM | [scan-sbom.md](scan-sbom.md) |
| `upload-sbom.yml` — upload a Grype-scanned SBOM (+ optional Hadolint/Dockle reports) to CoreStack | [upload-sbom.md](upload-sbom.md) |

Use this if you want any of: SBOM only, scan only (no CoreStack upload), your own pre-built SBOM uploaded, or independent control over Hadolint/Dockle. The three chain together via `needs:` and matching `project_name` values — see [create-sbom.md](create-sbom.md) for the full chained example.

### Infrastructure-as-code scanning

| Workflow | Docs |
|---|---|
| `kics.yml` — standalone KICS scan (Ansible, AzureResourceManager, CloudFormation, Dockerfile, Kubernetes, OpenAPI, Terraform), optional GitHub Security tab upload | [kics.md](kics.md) |

`sbom.yml`/`sbomdev.yml` also expose a `KICS_scan_type` input that runs the same scan inline — `kics.yml` is the standalone equivalent for callers who don't need the SBOM machinery at all.

## Example projects with workflows

Example projects have been created in the **[Example](../graphion_workflow_templates_example/)** repo. There are Terraform, Angular, and Docker examples, covering both the full pipeline and the split pipeline, and the SBOMs upload to the CNAPP-DEMO / discover CoreStack instance.
