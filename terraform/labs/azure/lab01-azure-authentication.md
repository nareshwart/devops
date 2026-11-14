# Lab 01: Azure Authentication and Terraform Provider

This lab signs in to Azure with the Azure CLI, then configures the Terraform AzureRM provider to use that authenticated CLI identity and a selected subscription. It verifies access by reading the current client configuration; it does not create Azure resources.

## Objectives

- Authenticate to Azure without putting credentials in Terraform files.
- Configure the AzureRM provider with the subscription to manage.
- Initialize Terraform and verify the identity and subscription available to the provider.

## Prerequisites

- Terraform CLI installed.
- An Azure account with access to a subscription.
- Permission to read the current client configuration in that subscription.

## Step 0: Install the Azure CLI if needed

If `az version` already works, skip this step. Otherwise, install the Azure CLI using Microsoft's official [Azure CLI installation guide](https://learn.microsoft.com/cli/azure/install-azure-cli). For example:

| Operating system | Install command |
| :--- | :--- |
| macOS with Homebrew | `brew install azure-cli` |
| Windows with WinGet | `winget install --exact --id Microsoft.AzureCLI` |
| Linux | Follow the installation guide for your distribution. |

Open a new terminal if required, then verify the installation:

```bash
az version
```

## Step 1: Sign in and select a subscription

Sign in using the Microsoft Entra ID account that has access to your Azure subscription—typically the same work, school, or learner account you use for the Azure Portal. On a computer with a browser, the CLI typically opens a browser and may require MFA.

On a headless Linux VM or an SSH session, use device-code sign-in instead. The CLI displays a URL and code; open the URL in a browser on another device and complete the sign-in:

```bash
az login --use-device-code
```

If you are signing in from an Azure VM with a managed identity enabled, you can authenticate with that identity instead:

```bash
az login --identity
```

The managed identity must have the required Azure role assignments for the resources Terraform will manage. The subscription ID is used to select the target subscription; it is not a login credential.

```bash
az login
az account list --output table
# Replace the placeholder with your subscription ID.
az account set --subscription "<subscription-id>"
az account show --output table
```

Set the Terraform input from the selected CLI subscription:

```bash
export TF_VAR_subscription_id="$(az account show --query id --output tsv)"
```

This exports the subscription ID as a Terraform input; it is not a credential. Do not put access tokens, client secrets, or passwords in Terraform files.

## Step 2: Create the Terraform configuration

Create a file named `main.tf` in your lab directory:

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    azurerm = {
      source  = "hashicorp/azurerm"
      version = "~> 4.0"
    }
  }
}

variable "subscription_id" {
  description = "Azure subscription ID selected for this configuration."
  type        = string
}

provider "azurerm" {
  features {}

  subscription_id = var.subscription_id
}

data "azurerm_client_config" "current" {}

output "azure_subscription_id" {
  description = "Azure subscription used by the provider."
  value       = data.azurerm_client_config.current.subscription_id
}

output "azure_tenant_id" {
  description = "Azure tenant associated with the authenticated identity."
  value       = data.azurerm_client_config.current.tenant_id
}
```

The provider uses the Azure CLI sign-in for local authentication. The subscription ID tells it which subscription to target; authentication still depends on the identity obtained through `az login`.

## Step 3: Initialize and verify

Run these commands from this lab's directory:

```bash
terraform init
terraform fmt -check
terraform validate
terraform plan
terraform apply
```

Review the plan before applying. This example only reads the Azure client configuration, so it should not propose creating or changing resources. Confirm the apply when prompted, then inspect the output:

```bash
terraform output
```

The output includes the subscription and tenant IDs. Treat local Terraform state and command output as data that may be sensitive.

## Authentication notes

- Azure CLI authentication is convenient for local learning and interactive use.
- For automation, prefer workload identity federation or a managed identity when supported by the environment.
- Keep client secrets, access tokens, and passwords out of Terraform configuration and source control.
- The AzureRM provider requires a subscription to be selected or supplied; this example explicitly supplies it as an input.

## Summary checklist

- [ ] Signed in to Azure CLI and selected the intended subscription.
- [ ] Set `TF_VAR_subscription_id` from the selected CLI account.
- [ ] Initialized and validated the Terraform configuration.
- [ ] Applied the read-only identity check and reviewed its outputs.
