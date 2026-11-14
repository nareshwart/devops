# Lab 03: Two Azure VM Environments with Workspaces

Repeat [Lab 02](./lab02-create-two-vms.md) in two Terraform CLI workspaces. The same configuration creates two Azure VMs in each workspace, with separate state: four VMs total after both applies.

> **Cost and security warning:** This creates twice the number of VMs and public IPs as Lab 02. It also opens password-based SSH to the internet, so use it only briefly for learning, never for production, and destroy resources in both workspaces when finished. Workspaces are not security boundaries. Terraform state contains the administrator password; keep it private and do not commit state files.

## Prerequisites

- Terraform CLI, Azure CLI, and permissions used in [Lab 02](./lab02-create-two-vms.md).
- A signed-in Azure CLI account and selected subscription.

## 1. Create the configuration

Create a new working directory and provide the subscription ID and password as in Lab 02:

```bash
mkdir -p ~/terraform-azure-workspaces
cd ~/terraform-azure-workspaces
az account show --output table
export TF_VAR_subscription_id="$(az account show --query id --output tsv)"
printf "Enter a strong VM admin password: "
read -s TF_VAR_admin_password
printf '\n'
export TF_VAR_admin_password
```

Save this complete configuration as `main.tf`. It uses the same VM setup as [Lab 02](./lab02-create-two-vms.md), with the active workspace included in resource names so each workspace creates separate resources:

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
  description = "Azure subscription ID for this lab."
  type        = string
}

variable "location" {
  description = "Azure region for the lab resources."
  type        = string
  default     = "eastus"
}

variable "vm_size" {
  description = "Azure VM size available in the selected region."
  type        = string
  default     = "Standard_B1s"
}

variable "admin_password" {
  description = "Password for the VM administrator."
  type        = string
  sensitive   = true
}

provider "azurerm" {
  features {}

  subscription_id = var.subscription_id
}

resource "azurerm_resource_group" "lab" {
  name     = "rg-${terraform.workspace}-two-vms-lab"
  location = var.location
}

resource "azurerm_virtual_network" "lab" {
  name                = "vnet-${terraform.workspace}-two-vms-lab"
  address_space       = ["10.42.0.0/16"]
  location            = azurerm_resource_group.lab.location
  resource_group_name = azurerm_resource_group.lab.name
}

resource "azurerm_subnet" "lab" {
  name                 = "internal"
  resource_group_name  = azurerm_resource_group.lab.name
  virtual_network_name = azurerm_virtual_network.lab.name
  address_prefixes     = ["10.42.1.0/24"]
}

resource "azurerm_network_security_group" "lab" {
  name                = "nsg-${terraform.workspace}-two-vms-lab"
  location            = azurerm_resource_group.lab.location
  resource_group_name = azurerm_resource_group.lab.name

  security_rule {
    name                       = "AllowSSHFromAnyAddress"
    priority                   = 100
    direction                  = "Inbound"
    access                     = "Allow"
    protocol                   = "Tcp"
    source_port_range          = "*"
    source_address_prefix      = "*"
    destination_port_range     = "22"
    destination_address_prefix = "*"
  }
}

resource "azurerm_subnet_network_security_group_association" "lab" {
  subnet_id                 = azurerm_subnet.lab.id
  network_security_group_id = azurerm_network_security_group.lab.id
}

resource "azurerm_public_ip" "lab" {
  count               = 2
  name                = "pip-${terraform.workspace}-${count.index + 1}"
  location            = azurerm_resource_group.lab.location
  resource_group_name = azurerm_resource_group.lab.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

resource "azurerm_network_interface" "lab" {
  count               = 2
  name                = "nic-${terraform.workspace}-${count.index + 1}"
  location            = azurerm_resource_group.lab.location
  resource_group_name = azurerm_resource_group.lab.name
  depends_on          = [azurerm_subnet_network_security_group_association.lab]

  ip_configuration {
    name                          = "internal"
    subnet_id                     = azurerm_subnet.lab.id
    private_ip_address_allocation = "Dynamic"
    public_ip_address_id          = azurerm_public_ip.lab[count.index].id
  }
}

resource "azurerm_linux_virtual_machine" "lab" {
  count               = 2
  name                = "vm-${terraform.workspace}-${count.index + 1}"
  computer_name       = "tf-lab-${count.index + 1}"
  resource_group_name = azurerm_resource_group.lab.name
  location            = azurerm_resource_group.lab.location
  size                = var.vm_size
  admin_username      = "azureuser"
  admin_password      = var.admin_password

  network_interface_ids = [
    azurerm_network_interface.lab[count.index].id
  ]

  disable_password_authentication = false

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Standard_LRS"
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-jammy"
    sku       = "22_04-lts-gen2"
    version   = "latest"
  }
}

output "vm_names" {
  description = "Names of the two Azure virtual machines."
  value       = azurerm_linux_virtual_machine.lab[*].name
}

output "vm_public_ips" {
  description = "Public IP addresses used to connect to the virtual machines."
  value       = azurerm_public_ip.lab[*].ip_address
}
```

## 2. Create and apply two workspaces

Initialize Terraform, then create and apply the first workspace:

```bash
terraform init
terraform workspace new development
terraform plan
terraform apply
```

Review the plan; it should add two VMs and their supporting resources. Next, create and apply the second workspace:

```bash
terraform workspace new testing
terraform workspace show
terraform plan
terraform apply
```

The selected workspace should be `testing`, and its plan should also add two VMs and their supporting resources. Each workspace has its own state, while both use the same Terraform configuration. `terraform workspace list` shows the available workspaces; `*` marks the active one.

## 3. Inspect and clean up

Switch between workspaces to see the outputs tracked in each state's resources:

```bash
terraform workspace select development
terraform output vm_names
terraform output vm_public_ips

terraform workspace select testing
terraform output vm_names
terraform output vm_public_ips
```

Destroy the resources from **both** workspaces before finishing:

```bash
terraform destroy
terraform workspace select development
terraform destroy
```

The first destroy applies to `testing`, which is selected at that point. The second applies to `development`. Review each destroy plan and confirm it removes only that workspace's two VMs and their supporting resources. Do not delete the workspace state before destroying its resources.
