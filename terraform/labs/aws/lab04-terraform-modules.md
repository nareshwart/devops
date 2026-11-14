# Lab 04: Find, Use, and Build Terraform Modules

This beginner lab has two parts: use a pre-built module from the Terraform Registry, then create a small local module and call it from a root configuration.

> **Cost reminder:** The examples require AWS credentials and create cloud resources only if you run `terraform apply`. Review each plan first and destroy resources when finished. Do not put credentials in Terraform files.

## Prerequisites

- Terraform CLI and AWS CLI installed.
- AWS credentials configured; see [AWS Authentication & Provider](./lab01-aws-authentication.md).
- Permission to create and delete the AWS resources in the examples.

<details>
<summary><strong>Part 1: Find and use a Registry module</strong></summary>

<br>

### 1. Find a module

Open the [Terraform Registry modules page](https://registry.terraform.io/browse/modules) and search for a module that fits a task and provider. For this exercise, use the [AWS VPC module](https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest).

Before using a module, look at its README, inputs, outputs, examples, source code, supported Terraform/provider versions, and release history. Check what it will create and whether its defaults have security or cost implications.

### 2. Create a root configuration

Create a separate directory for this exercise and select the AWS profile you configured in the authentication lab:

```bash
mkdir -p ~/terraform-registry-module-lab
cd ~/terraform-registry-module-lab
export AWS_PROFILE=terraform-lab
aws sts get-caller-identity
```

Confirm the AWS account is the intended one. Create a `main.tf` file with:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-west-2"
}

module "network" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "~> 5.0"

  name = "terraform-module-lab"
  cidr = "10.0.0.0/16"

  azs             = ["us-west-2a", "us-west-2b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]

  enable_nat_gateway = false
}

output "vpc_id" {
  description = "ID of the VPC created by the module."
  value       = module.network.vpc_id
}
```

The module source uses the Registry format `namespace/name/provider`. `version` constrains the module version; `~> 5.0` allows compatible 5.x releases. The module's inputs configure the network, and `module.network.vpc_id` reads one of its outputs. Check the Registry page for current versions and requirements before using a module in your own project.

### 3. Initialize and review the plan

```bash
terraform init
terraform fmt -check
terraform validate
terraform plan
```

`terraform init` downloads the module and required provider. Review the plan to see which resources the module proposes. The VPC setup disables NAT gateways to avoid that additional cost.

If you want to create the network, apply only after reviewing the plan:

```bash
terraform apply
terraform output vpc_id
```

When finished, destroy resources created by this exercise:

```bash
terraform destroy
```

</details>

<details>
<summary><strong>Part 2: Write a local module and call it</strong></summary>

<br>

### 1. Create the module folders

Create a new directory so this part has its own Terraform state:

```bash
mkdir -p ~/terraform-local-module-lab/modules/network
cd ~/terraform-local-module-lab
export AWS_PROFILE=terraform-lab
```

The root configuration calls the child module in `modules/network/`:

```text
terraform-local-module-lab/
├── main.tf
└── modules/
    └── network/
        ├── main.tf
        ├── variables.tf
        └── outputs.tf
```

### 2. Write the child module

Create `modules/network/variables.tf` to define the values the module accepts:

```hcl
variable "vpc_name" {
  description = "Name tag for the VPC."
  type        = string
}

variable "cidr_block" {
  description = "IPv4 CIDR range for the VPC."
  type        = string
}
```

Create `modules/network/main.tf` to define the resource managed by this module:

```hcl
resource "aws_vpc" "this" {
  cidr_block = var.cidr_block

  tags = {
    Name = var.vpc_name
  }
}
```

Create `modules/network/outputs.tf` to expose a useful resource attribute to the caller:

```hcl
output "vpc_id" {
  description = "ID of the VPC."
  value       = aws_vpc.this.id
}
```

### 3. Call the child module from the root module

Create `main.tf` in `~/terraform-local-module-lab`:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-west-2"
}

module "network" {
  source = "./modules/network"

  vpc_name   = "my-learning-vpc"
  cidr_block = "10.20.0.0/16"
}

output "network_vpc_id" {
  description = "VPC ID returned by the network module."
  value       = module.network.vpc_id
}
```

The root module passes `vpc_name` and `cidr_block` into the child module. The child module creates a VPC and returns its ID, which the root module exposes as `network_vpc_id`. The local `source` path points to a directory, so no Registry `version` is needed.

### 4. Initialize, plan, and clean up

```bash
terraform init
terraform fmt -check -recursive
terraform validate
terraform plan
```

Review the plan; this configuration should create one VPC. If you choose to apply it, query the output and destroy the VPC when finished:

```bash
terraform apply
terraform output network_vpc_id
terraform destroy
```

</details>

## What you learned

- Find pre-built modules in the Terraform Registry and inspect their documentation before use.
- Use a Registry module with a `source`, version constraint, input arguments, and outputs.
- Organize a local module into files for resources, input variables, and outputs.
- Call a local module from a root module and pass values into and out of it.
