# aws_oidc_terraform.yml

Reusable workflow reference:

`Coveo-IS/.github/.github/workflows/aws_oidc_terraform.yml@main`

## Purpose

- Assume an AWS role through GitHub OIDC.
- Run `terragrunt init` and `terragrunt plan` in a target directory.
- Optionally run `terragrunt apply` against the saved `tgplan` file.
- Upload both `tgplan` and `plan.txt` as a short-lived artifact.

## Trigger

This workflow is reusable only (`on: workflow_call`).
Caller workflows decide when it runs (for example `pull_request`, `push`, or `workflow_dispatch`).

## Quick Start

Plan-only caller example:

```yaml
name: Infra Plan (Dev)

on:
  pull_request:
    paths:
      - infra/dev/**

permissions:
  contents: read
  id-token: write

jobs:
  terragrunt-plan:
    uses: Coveo-IS/.github/.github/workflows/aws_oidc_terraform.yml@main
    with:
      working_directory: infra/dev
      aws_region: us-east-1
      aws_role_arn: arn:aws:iam::123456789012:role/github-oidc-terragrunt-dev
      timeout_minutes: 15
      apply_plan: false
```

Apply caller example:

```yaml
name: Infra Apply (Dev)

on:
  workflow_dispatch:

permissions:
  contents: read
  id-token: write

jobs:
  terragrunt-apply:
    uses: Coveo-IS/.github/.github/workflows/aws_oidc_terraform.yml@main
    with:
      working_directory: infra/dev
      aws_region: us-east-1
      aws_role_arn: arn:aws:iam::123456789012:role/github-oidc-terragrunt-dev
      timeout_minutes: 30
      apply_plan: true
```

## Inputs

| Input | Required | Type | Default | Description |
|---|---|---|---|---|
| `working_directory` | yes | string | none | Directory where Terragrunt commands run. |
| `aws_region` | no | string | `us-east-1` | AWS region for the assumed role session. |
| `aws_role_arn` | yes | string | none | IAM role ARN assumed through OIDC. |
| `timeout_minutes` | no | number | `5` | Job timeout in minutes. |
| `apply_plan` | no | boolean | `false` | When true, runs `terragrunt apply -auto-approve tgplan`. |

## Outputs

| Output | Description |
|---|---|
| `terra_plan_artifact_url` | Artifact URL returned by `actions/upload-artifact` for the uploaded `tgplan` package. |

## Runtime Behavior

- `run-name` is dynamic: `Terragrunt plan|apply * <working_directory> * <branch>`.
- Concurrency group includes workflow name, branch, and working directory.
- `cancel-in-progress` is enabled for plan runs and disabled for apply runs.
- AWS credentials are configured with `aws-actions/configure-aws-credentials` (OIDC).
- Tooling is installed at runtime:
  - OpenTofu `1.12.5` (checksum and signature verified)
  - Terragrunt `1.1.1` (checksum verified)
- The workflow verifies identity with `aws sts get-caller-identity` before Terragrunt commands.
- Plan step writes console output to `plan.txt` and plan binary to `tgplan`.
- Upload step keeps artifacts for `1` day.

## Required Caller Permissions

Set these permissions in caller workflows:

```yaml
permissions:
  contents: read
  id-token: write
```

## Security Notes

- Prefer pinning reusable workflow references to a tag or commit SHA in callers.
- Ensure the IAM role trust policy restricts allowed GitHub OIDC subjects (repo/ref/environment).
- If stronger supply-chain guarantees are required, add signature verification for Terragrunt releases.

## Troubleshooting

- OIDC assume-role fails: confirm caller has `id-token: write` and IAM trust policy matches your repo/ref.
- Terragrunt cannot find files: verify `working_directory` path in the caller.
- Apply does not run: verify caller passes `apply_plan: true`.
- Runs are cancelled unexpectedly: expected for plan runs due to concurrency cancellation.
- Artifact URL is empty: check whether `Upload plan` executed successfully.
