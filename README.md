# .github
Coveo Information Systems (IS) team repositories.

# Workflows File Name Convention

Considering that workflows cannot be organized into subfolders, it's important to use a proper naming
convention to facilitate ease of access. All workflow files must be named accordingly:

`<platform>-<env>-<deployment-method>-<action>.yml`

An example of this would be `aws-dev-terragrunt-plan.yml`. AWS is the platform, it's deploying to the
DEV environment, the terragrunt deployment method is being used, and the plan action is performed.

# Available Workflows

## `aws_oidc_terraform.yml`

Reusable workflow reference:

`Coveo-IS/.github/.github/workflows/aws_oidc_terraform.yml@main`

Documentation:

- Workflow guide: [workflows/aws_oidc_terraform.README.md](workflows/aws_oidc_terraform.README.md)
