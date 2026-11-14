# Lab 02: Create Two Azure Virtual Machines

Create two Ubuntu Linux VMs in a new resource group and virtual network. Azure assigns each VM a public IP, and SSH is available from any public IP.

> **Cost and security warning:** VMs, disks, and public IPs can incur charges. This beginner lab opens password-based SSH to the internet for simplicity. Use it only briefly for learning, never for production, and destroy the resources when finished. Terraform state contains the password, so keep the lab directory private and never commit its state files.

## Prerequisites

- Terraform CLI, Azure CLI, and permissions to create resource groups, virtual networks, network interfaces, and VMs.
- A signed-in Azure CLI account and selected subscription; see the [Azure authentication lab](./lab01-azure-authentication.md).

## 1. Choose a region, image, and VM size

- The example uses Ubuntu 22.04. To choose another supported Linux image, see Microsoft's guide to [finding Linux VM images](https://learn.microsoft.com/azure/virtual-machines/linux/cli-ps-findimage) and update `source_image_reference`.
- The default size is `Standard_B1s`. Check [Azure VM sizes](https://learn.microsoft.com/azure/virtual-machines/sizes/overview) for sizes available in your region and subscription.
- The default region is `eastus`. Change `location` if needed and check the [Azure pricing calculator](https://azure.microsoft.com/pricing/calculator/) before applying.

## 2. Create and run the configuration

Use a separate directory so this lab has its own state:

```bash
mkdir -p ~/terraform-azure-two-vms
cd ~/terraform-azure-two-vms
az account show --output table
export TF_VAR_subscription_id="$(az account show --query id --output tsv)"
printf "Enter a strong VM admin password: "
read -s TF_VAR_admin_password
printf '\n'
export TF_VAR_admin_password
```

Enter a strong password that meets Azure's current Linux VM requirements. It is read without echoing and passed to Terraform through the environment. Azure assigns public IPs automatically; no IP address or source range is needed.

Confirm the subscription is correct, then save the following as `main.tf`:

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
  name     = "rg-terraform-two-vms-lab"
  location = var.location
}

resource "azurerm_virtual_network" "lab" {
  name                = "vnet-terraform-two-vms-lab"
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
  name                = "nsg-terraform-two-vms-lab"
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
  name                = "pip-terraform-lab-${count.index + 1}"
  location            = azurerm_resource_group.lab.location
  resource_group_name = azurerm_resource_group.lab.name
  allocation_method   = "Static"
  sku                 = "Standard"
}

resource "azurerm_network_interface" "lab" {
  count               = 2
  name                = "nic-terraform-lab-${count.index + 1}"
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
  name                = "vm-terraform-lab-${count.index + 1}"
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

The network security group permits SSH from any source; other inbound traffic remains blocked. Azure automatically assigns a public IP to each VM.

Initialize, validate, and review the plan:

```bash
terraform init
terraform fmt -check
terraform validate
terraform plan
```

Confirm the plan shows two VMs, two public IPs, and the intended SSH rule, then apply:

```bash
terraform apply
```

Inspect the VM names and public IP addresses:

```bash
terraform output vm_names
terraform output vm_public_ips
```

Connect to a VM using its public IP and the `azureuser` password you entered:

```bash
ssh azureuser@<public-ip>
```

## 3. Clean up

When finished, destroy all resources from the same directory:

```bash
terraform destroy
```

Review the destroy plan and confirm. Keep the local state until cleanup is complete.
