# Clean up GHCR Images

This action deletes old untagged GHCR container images outside of a defined retention window.

## Inputs

Below are the action's inputs that need to be defined in the Action's `with` block.

| Required | Input | Summary |
|----------|--------|---------|---------|
| ✔ | `github-token` | GitHub token with package delete permissions. |
| ✔ | `ghcr-package-name` | The GHCR package/image name to clean. |
| ✔ | `retention-days` | Number of days to keep untagged images, e.g. "7". |


## Workflow Permissions

You need to have the following permissions set at the top of the workflow.

```yaml
permissions:
  contents: read
  packages: write
```

## Example

```yaml
name: Publish Image to GHCR

on:
  workflow_dispatch:

permissions:
  contents: read
  packages: write

env:
  REGISTRY: "ghcr.io"
  REPO_OWNER: "lancemccarthy"
  IMG_NAME: "awesome-app"

jobs:
  dockerfile_to_dockerhub:
    name: "Dockerfile Build and Publish"
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v6

    - name: Get package metadata
      id: meta
      uses: docker/metadata-action@v5
      with:
        images: ${{env.REGISTRY}}/${{env.REPO_OWNER}}/${{env.IMG_NAME}}
        tags: |
          type=raw,value=latest
          type=sha,format=short

    - name: Build and Publish Images (arm64, amd64)
      uses: docker/build-push-action@v5
      with:
        platforms: linux/arm64,linux/amd64
        push: true
        tags: ${{steps.meta.outputs.tags}}
        labels: ${{steps.meta.outputs.labels}}

    - name: Delete untagged GHCR versions older than retention
      uses: LanceMcCarthy/actions/cleanup-ghcr-images@v1
      with:
        github-token: ${{secrets.GITHUB_TOKEN}}
        ghcr-package-name: ${{env.IMG_NAME}}
        retention-days: "7"
```
