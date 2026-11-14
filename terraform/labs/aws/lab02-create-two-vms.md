# Lab 02: Create Two AWS Virtual Machines

Create two Ubuntu EC2 instances with Terraform using your AWS CLI profile and the region's default VPC. No inbound access rules are added.

> **Cost warning:** EC2 instances and attached storage can incur charges. Check your account's pricing and limits, review the plan, and destroy the lab resources when finished.

## Prerequisites

- Terraform CLI, AWS CLI, and permissions to describe images and create/manage EC2 instances.
- A working AWS CLI profile; see the [AWS authentication lab](./lab01-aws-authentication.md).
- A default VPC and at least one default subnet in the selected region.

## 1. Choose a region, image, and instance type

- The example uses Ubuntu 22.04 and discovers the latest matching Canonical AMI in the configured region. To select another image, see [finding an AMI](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/finding-an-ami.html) and update the AMI filters in the configuration.
- The default size is `t3.micro`. Check [EC2 instance types](https://aws.amazon.com/ec2/instance-types/) for available sizes and [EC2 pricing](https://aws.amazon.com/ec2/pricing/on-demand/) before changing it.
- The default region is `us-west-2`. Change `aws_region` if needed; the AMI must be available in that region.

## 2. Create and run the configuration

```bash
mkdir -p ~/terraform-aws-two-vms
cd ~/terraform-aws-two-vms
export AWS_PROFILE=terraform-lab
aws sts get-caller-identity
```

Confirm the account is correct, then save the following as `main.tf`:

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
    Name      = "terraform-lab-vm-${count.index + 1}"
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

Initialize, validate, and review the plan:

```bash
terraform init
terraform fmt -check
terraform validate
terraform plan
```

Confirm the plan shows two instances in the intended account and region, then apply:

```bash
terraform apply
```

Inspect the instance IDs and names:

```bash
terraform output instance_ids
terraform output instance_names
```

The instances use default VPC settings. A default subnet may assign public IP addresses, but the configuration adds no inbound rules or remote-access setup.

## 3. Clean up

When finished, destroy the resources from the same directory:

```bash
terraform destroy
```

Review the destroy plan and confirm. Keep the local state until cleanup is complete.
