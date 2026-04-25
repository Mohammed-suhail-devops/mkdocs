# GitHub Actions

Repo path:

```text
.github/workflows/terragrunt-cd.yaml
```

## Purpose

GitHub Actions is the controlled entry point for infrastructure operations. The workflow lets an operator choose the target environment, runner, resource folder, command, and whether the run should be plan-only.

## What It Does

- Authenticates to Azure.
- Exports Terraform Azure authentication variables.
- Checks out the repository.
- Installs pinned Terraform and Terragrunt versions.
- Runs `terragrunt init`.
- Runs `terragrunt plan`, `apply`, `refresh`, or `destroy`.

## Sample Workflow Shape

```yaml
on:
  workflow_dispatch:
    inputs:
      tgResource:
        type: choice
        options:
          - resources/aks-resource-group
          - resources/public-ip
          - resources/aks-cluster
      tgEnv:
        type: choice
        options:
          - dev
          - qa
          - prod
      planOnly:
        type: boolean
      tgCommand:
        type: choice
        options:
          - CreateOrUpdate
          - Destroy
          - refresh
```

## Sample Execution Logic

```bash
cd "${TG_RESOURCE}"

terragrunt init --terragrunt-non-interactive

if [[ "${PLAN_ONLY}" == "true" ]]; then
  terragrunt plan -lock=false
elif [[ "${TG_COMMAND}" == "CreateOrUpdate" ]]; then
  terragrunt apply -lock=false -auto-approve -parallelism=1
elif [[ "${TG_COMMAND}" == "Destroy" ]]; then
  terragrunt destroy -lock=false -auto-approve
fi
```

## Portfolio Notes

- The real workflow uses repository and environment secrets.
- Secrets are not documented here.
- The important pattern is scoped execution: one selected `resources/<component>` folder at a time.
