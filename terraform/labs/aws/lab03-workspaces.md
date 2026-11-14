# Lab 03: Two AWS VM Environments with Workspaces

Repeat [Lab 02](./lab02-create-two-vms.md) in two Terraform CLI workspaces. The same configuration creates two EC2 instances in each workspace, with separate state: four instances total after both applies.

> **Cost warning:** This creates twice the number of instances as Lab 02. Review costs and destroy resources in both workspaces when finished. Workspaces separate Terraform state; they are not security boundaries.

## Prerequisites

- Terraform CLI, AWS CLI, and permissions used in [Lab 02](./lab02-create-two-vms.md).
- A working AWS CLI profile.
- A default VPC and subnet in the selected region.

## 1. Create the configuration

Create a new working directory and select the AWS profile:

```bash
mkdir -p ~/terraform-aws-workspaces
cd ~/terraform-aws-workspaces
export AWS_PROFILE=terraform-lab
aws sts get-caller-identity
```

Save this complete configuration as `main.tf`. It is the same setup as [Lab 02](./lab02-create-two-vms.md), with the active workspace added to each VM's name:

```hcl
terraform {
  required_version = ">= 1.5.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

variable "aws_region" {
  description = "AWS region where the instances will be created."
  type        = string
  default     = "us-west-2"
}

variable "instance_type" {
  description = "EC2 instance type used for each lab VM."
  type        = string
  default     = "t3.micro"
}

provider "aws" {
  region = var.aws_region
}

data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"]

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }

  filter {
    name   = "architecture"
    values = ["x86_64"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

resource "aws_instance" "lab" {
  count         = 2
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type

  tags = {
    Name      = "terraform-${terraform.workspace}-vm-${count.index + 1}"
    ManagedBy = "Terraform"
  }
}

output "instance_ids" {
  description = "IDs of the two EC2 instances."
  value       = aws_instance.lab[*].id
}

output "instance_names" {
  description = "Names of the two EC2 instances."
  value       = aws_instance.lab[*].tags.Name
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

Review the plan; it should add two instances. Next, create and apply the second workspace:

```bash
terraform workspace new testing
terraform workspace show
terraform plan
terraform apply
```

The selected workspace should be `testing`, and its plan should also add two instances. Each workspace has its own state, while both use the same Terraform configuration. `terraform workspace list` shows the available workspaces; `*` marks the active one.

## 3. Inspect and clean up

Switch between workspaces to see the outputs tracked in each state's resources:

```bash
terraform workspace select development
terraform output instance_ids

terraform workspace select testing
terraform output instance_ids
```

Destroy the resources from **both** workspaces before finishing:

```bash
terraform destroy
terraform workspace select development
terraform destroy
```

The first destroy applies to `testing`, which is selected at that point. The second applies to `development`. Review each destroy plan and confirm it removes only that workspace's two instances. Do not delete the workspace state before destroying its resources.
