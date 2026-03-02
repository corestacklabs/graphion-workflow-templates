# Sample GitHub Action Based Container Build & Deploy Template

This project contains a reusable GitHub action pipeline to build, scan, and publish a container image.

## Required Secrets and Variables

The following need to be defined in the target project's repo. Secrets should only be stored in GitHub action secrets. Variable can be stored in GitHub or defined in the project's build.yml. 

| Name | Type | Purpose |
|---|---|---|
| `REGISTRY_TOKEN` | Secret | Token for logging into remote container registry |
| `SONAR_TOKEN` | Secret | Token for interacting with SonarQube server |
| `SONAR_HOST_URL` | Variable | URL of the SonarQube server |
| `REGISTRY_URL` | Variable | URL of the remote container registry |
| `REGISTRY_USER` | Variable | Remote container registry user |
| `REPO` | Variable | Repo to push final image to on remote registry |
| `STAGE_REPO` | Variable | Repo where temporary image should be stored |
| `IMAGE_NAME` | Variable | Name of the image to publish to the remote registry |
| `IMAGE_TAG` | Variable | Tag for the image |
| `BUILD_LANGUAGE` | Variable (Optional)| Specifies language (node, python, etc) used so correct tools run |
| `SRC_DIR` | Variable (Optional)| Location o source code used to autobuild docs | 
| `TEST_DIR` | Variable (Optional)| Location of test code used to unit, integration, e2e, etc. test app | 

## Sample build.yml workflow file 

The `tbd` provides a good example of how to use these workflows in a project to build it's container image and deploy to EKS. You can find that build file [here](https://github.com/corestacklabs/graphion-workflow-templates/tbd/blob/main/.github/workflows/build.yml).
