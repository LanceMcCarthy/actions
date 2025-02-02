# Gomboc.AI Scan Terraform Action

## When to use this action

Use this action:
- In a deployment workflow to get remediations to your Terraform plan. Optionally have remediations in a new PR or on your current branch
- Locally to get a set of observations about your Terraform plan.

## Setting up your workflow

Gomboc's Terraform Action requires a few settings to run on Github Actions.

---

Start by creating or expanding an existing workflow that you have, like so:

```
name: Gomboc.AI Terraform Scan

on:
  pull_request:
  push:
    branches: [ main ]

permissions:
  id-token: write
  contents: read

jobs:
  gomboc:
    runs-on: ubuntu-latest
    name: Gomboc Terraform Action
    steps:
      ...
```

---

Note the minimal set of permissions required:

| Permission | Description |
| --- | --- |
| `id-token: write` | Required to authenticate you in our service |
| `contents: read` | Required to read your Terraform plan and HCL templates |

---

Finally, add one or more Gomboc.AI actions:

```
- name: Gomboc.AI - Terraform Scan
  uses: Gomboc-AI/actions/terraform/scan@main
```

## Variables

| variable | Required | Default | Description | Additional permissions |
| --- | --- | --- | --- | --- |
| `gomboc-config` | No |  `gomboc.yaml` | Path to Gomboc.AI's configuration file | N/A |
| `tf-directory` | No | `.` | The root directory for the Terraform configuration | N/A |
| `tf-plan` | No | `tfplan.json` | A filepath to a local JSON file describing your Terraform plan (relative to tf-directory) | N/A |
| `access-token` | No |   | Access token needed to perform API side effects (`secrets.GITHUB_TOKEN`) | N/A |
| `create-pr` | No |  `false` | Create a new PR with remediations | `contents: write` |
| `commit-on-current-branch` | No |  `false` | Commit remediations into the current branch | `contents: write` |

## Bring it all together

Your completed Gomboc.AI Terraform Workflow should look something like this:

```
name: Gomboc.AI Terraform

on:
  pull_request:
  push:
    branches: [ main ]

permissions:
  id-token: write
  contents: write

jobs:
  gomboc:
    runs-on: ubuntu-latest
    steps:
      - name: Gomboc.AI - Terraform Scan
        uses: Gomboc-AI/actions/terraform/scan@main
        with:
          access-token: ${{ secrets.GITHUB_TOKEN }} 
          commit-on-current-branch: true
```

> **Note**
> `secrets.GITHUB_TOKEN` is provided by GitHub and can be used to authenticate on behalf of GitHub Actions. Read more about it [here](https://docs.github.com/en/actions/security-guides/automatic-token-authentication).

## About Gomboc.AI's configuration file

It is a YAML file that specifies different parameters for the scan action.

Here's an example:

```
policies: 
  must-implement:
    - Deletion Protection
    - Request Tracing
    - Client Authentication via IAM SigV4
    - Skip Terraform Destroy
```

| Element | Required | Description |
| --- | --- | --- |
| <kbd>policies</kbd> | Yes | An object describing your policies |
| <kbd>policies.must-implement</kbd> | Yes | A list of capabilities that will be enforced |