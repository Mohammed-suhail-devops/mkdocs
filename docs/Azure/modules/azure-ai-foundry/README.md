# Azure AI Foundry Module

Repo paths:

```text
modules/azure-ai-foundry/
resources/ai-foundry/terragrunt.hcl
```

## Purpose

Creates AI project-style resources using Azure providers. This module demonstrates how platform infrastructure can include AI workspace or project provisioning alongside AKS.

## Provisioned Resources

- AI project metadata.
- AI service subdomain or project resource.
- AzureRM and AzAPI provider usage pattern.

## Sample Terragrunt Wrapper

```hcl
terraform {
  source = "../../modules/azure-ai-foundry"
}

include "root" {
  path = find_in_parent_folders()
}

locals {
  environment = lower(get_env("ENV", "dev"))
  global      = yamldecode(file("${get_terragrunt_dir()}/../../config/${local.environment}.yaml"))
}

inputs = {
  ai_project_name         = local.global.ai_project_name
  ai_project_display_name = local.global.ai_project_display_name
  ai_project_description  = local.global.ai_project_description
  ai_foundry_subdomain    = local.global.ai_foundry_subdomain
  ai_resource_group_name  = local.global.ai_resource_group_name
  ai_location             = local.global.ai_location
}
```

## Sample Terraform Logic

```hcl
resource "azapi_resource" "ai_project" {
  type      = "Microsoft.MachineLearningServices/workspaces@2024-04-01"
  name      = var.ai_project_name
  location  = var.ai_location
  parent_id = data.azurerm_resource_group.ai.id

  body = jsonencode({
    properties = {
      friendlyName = var.ai_project_display_name
      description  = var.ai_project_description
    }
  })
}
```

## Notes

The exact Azure AI resource type can change by use case. Keep provider versions pinned and validate against the target Azure API version.
