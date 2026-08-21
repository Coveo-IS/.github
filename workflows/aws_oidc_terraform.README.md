# aws_oidc_terraform.yml

Reusable workflow reference:

`Coveo-IS/.github/.github/workflows/aws_oidc_terraform.yml@main`

## Purpose

- Run Terragrunt with AWS OIDC credentials.
- Create and upload a plan artifact.
- Optionally apply the generated plan.

## Quick Start

Plan only caller example:

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

Apply caller example (manual):

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
| `timeout_minutes` | no | number | `5` | Workflow job timeout in minutes. |
| `apply_plan` | no | boolean | `false` | When true, runs `terragrunt apply` on the saved plan. |

## Outputs

| Output | Description |
|---|---|
| `terra_plan_artifact_url` | URL of the uploaded plan artifact from the reusable workflow run. |

## Behavior Notes

- `apply_plan: false` runs plan only.
- `apply_plan: true` runs plan then apply using the generated plan file.
- Plan output is streamed to the Actions log and also saved in `plan.txt`.
- Plan artifact retention is currently set to `1` day.
- Concurrency group is based on workflow, branch, and working directory.

## Approval Gates

Approval gates are configured in caller workflows via GitHub Environments.

High-level flow:

1. Configure required reviewers in repository settings under Environments.
2. Put apply in a dedicated caller job.
3. Set `environment: <name>` on that apply job.

Note: native environment approvals are reviewer-based allow-lists, not strict multi-approval counts.
If you need true N-of-M approvals, use custom protection rules or an approval action that supports
minimum approval counts.

## Security Notes

- Keep caller workflow permissions minimal (`id-token: write`, `contents: read`).
- Prefer pinning reusable workflow references to a tag or commit SHA for change control.
- OpenTofu checksum/signature verification is present.
- Terragrunt is checksum-verified; consider full signature verification in future hardening.

## Troubleshooting

- OIDC assume-role fails. Confirm caller has `id-token: write` and IAM trust policy allows the GitHub OIDC subject for your repo/ref.
- Terragrunt cannot find files. Verify `working_directory` points to the correct stack path.
- Apply is not running. Verify `apply_plan: true` is passed by the caller.
- Run waits unexpectedly. Check concurrency group collisions with another in-progress run.
- Missing artifact. Check whether the plan step failed before artifact upload.
