# .github
Coveo Information Systems (IS) team repositories.

# Workflows File Name Convention

Considering that workflows cannot be organized into subfolders, it's important to use a proper naming
convention to facilitate ease of access. Deployment workflows should be named accordingly:

`<platform>-<env>-<deployment-method>-<action>.yml`

For example: `aws-dev-terragrunt-plan.yml`.
Reusable workflows may use descriptive names (e.g. `aws_oidc_terraform.yml`).

# Available Workflows

## `aws_oidc_terraform.yml`

Reusable workflow reference:

`Coveo-IS/.github/.github/workflows/aws_oidc_terraform.yml@main`

Documentation:

- Workflow guide: [.github/workflows/aws_oidc_terraform.README.md](.github/workflows/aws_oidc_terraform.README.md)
