# Lab 04: Find, Use, and Build Terraform Modules in Azure

This beginner lab has two parts: call a pre-built Azure module from the Terraform Registry, then build a small local module and call it from a root configuration.

> **Before applying:** Review every plan and make sure it targets the intended Azure subscription. These examples create a resource group and virtual network. Do not put credentials in Terraform files.

## Prerequisites

- Terraform CLI and Azure CLI installed.
- A signed-in Azure CLI account and selected subscription; see [Azure Authentication & Provider](./lab01-azure-authentication.md).
- Permission to create and delete resource groups and virtual networks.

<details>
<summary><strong>Part 1: Find and use a Registry module</strong></summary>

<br>

### 1. Find a module

Search the [Terraform Registry modules page](https://registry.terraform.io/browse/modules) by cloud, task, or provider. For this exercise, use Microsoft's [Azure Verified Module for resource groups](https://registry.terraform.io/modules/Azure/avm-res-resources-resourcegroup/azurerm/latest).

Before using a module, review its README, inputs, outputs, examples, source, supported Terraform/provider versions, and release history. Check what it creates, what permissions it needs, and whether its defaults have security or cost implications.

### 2. Create the root configuration

Create a separate directory for this exercise and confirm the selected subscription:

```bash
mkdir -p ~/terraform-registry-module-azure
cd ~/terraform-registry-module-azure
az account show --output table
```

Create `main.tf`:

```hcl
terraform {
  required_version = ">= 1.9.0, < 2.0.0"

  required_providers {
    azapi = {
      source  = "Azure/azapi"
      version = "~> 2.12"
    }
  }
}

provider "azapi" {}

module "resource_group" {
  source  = "Azure/avm-res-resources-resourcegroup/azurerm"
  version = "0.5.0"

  name             = "rg-terraform-registry-module-lab"
  location         = "eastus"
  enable_telemetry = false
}

output "resource_group_id" {
  description = "ID of the resource group created by the module."
  value       = module.resource_group.resource_id
}
```

The module source uses the Registry format `namespace/name/provider`. This pinned module version uses the AzAPI provider and requires Terraform 1.9 or later. The module's `name` and `location` inputs configure the resource group; `module.resource_group.resource_id` reads its output. `enable_telemetry = false` opts out of module telemetry.

### 3. Initialize and review the plan

```bash
terraform init
terraform fmt -check
terraform validate
terraform plan
```

`terraform init` downloads the module and its required provider. Review the plan before applying. If you choose to create the resource group:

```bash
terraform apply
terraform output resource_group_id
```

Clean up the resource group when finished:

```bash
terraform destroy
```

</details>

<details>
<summary><strong>Part 2: Write a local module and call it</strong></summary>

<br>

### 1. Create the module folders

Use a different directory so this part has its own state:

```bash
mkdir -p ~/terraform-local-module-azure/modules/network
cd ~/terraform-local-module-azure
az account show --output table
export TF_VAR_subscription_id="$(az account show --query id --output tsv)"
```

The root configuration calls the child module in `modules/network/`:

```text
terraform-local-module-azure/
├── main.tf
└── modules/
    └── network/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

### 2. Write the child module

Create `modules/network/variables.tf` to declare the values the module accepts:

```hcl
variable "resource_group_name" {
  description = "Name of the resource group for the network."
  type        = string
}

variable "location" {
  description = "Azure region for the virtual network."
  type        = string
}

variable "vnet_name" {
  description = "Name of the virtual network."
  type        = string
}

variable "address_space" {
  description = "Address range for the virtual network."
  type        = list(string)
}
```

Create `modules/network/main.tf` to define the resource managed by the module:

```hcl
resource "azurerm_virtual_network" "this" {
  name                = var.vnet_name
  location            = var.location
  resource_group_name = var.resource_group_name
  address_space       = var.address_space
}
```

Create `modules/network/outputs.tf` to expose the virtual network ID:

```hcl
output "vnet_id" {
  description = "ID of the virtual network."
  value       = azurerm_virtual_network.this.id
}
```

### 3. Call the child module from the root module

Create `main.tf` in `~/terraform-local-module-azure`:

```hcl
terraform {
  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
}

variable "subscription_id" {
  description = "Azure subscription ID for this lab."
  type        = string
}

provider "azurerm" {
  features {}

  subscription_id = var.subscription_id
}

resource "azurerm_resource_group" "lab" {
  name     = "rg-terraform-local-module-lab"
  location = "eastus"
}

module "network" {
  source = "./modules/network"

  resource_group_name = azurerm_resource_group.lab.name
  location            = azurerm_resource_group.lab.location
  vnet_name           = "vnet-terraform-local-module-lab"
  address_space       = ["10.30.0.0/16"]
}

output "network_vnet_id" {
  description = "VNet ID returned by the network module."
  value       = module.network.vnet_id
}
```

The root module creates the resource group and passes its name and location, plus the VNet settings, into the child module. The child module creates the VNet and returns its ID. Its local `source` path points to a directory, so no Registry `version` is needed.

### 4. Initialize, plan, and clean up

```bash
terraform init
terraform fmt -check -recursive
terraform validate
terraform plan
```

Review the plan; this configuration should create a resource group and one VNet. If you choose to apply it, query the output and destroy the resources when finished:

```bash
terraform apply
terraform output network_vnet_id
terraform destroy
```

</details>

## What you learned

- Search the Terraform Registry and assess a module before using it.
- Call a Registry module with a source, version, input arguments, and outputs.
- Organize a local module into resource, input-variable, and output files.
- Call a local module from a root configuration and pass values in and out.
