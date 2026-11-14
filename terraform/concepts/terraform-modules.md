# Terraform Modules

A **Terraform module** is a collection of Terraform configuration files that are managed together. Every Terraform working directory is a **root module**; a `module` block lets that root module call another module, called a **child module**.

Modules help package reusable infrastructure patterns—such as a network, a group of virtual machines, or a database—behind documented inputs and outputs. A module is configuration, not a separate Terraform run or state.

## Find pre-built modules

Start with the [Terraform Registry](https://registry.terraform.io/browse/modules), where you can search public modules by provider, service, and task. For example, the [AWS VPC module](https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest) packages common VPC networking resources.

Organizations may also publish private modules in a private registry, such as the one available in HCP Terraform. Module authors can publish modules from version-controlled repositories as well.

Before using a module, review its:

- Supported provider and Terraform versions.
- Inputs, defaults, outputs, examples, and upgrade notes.
- Source repository, maintainers, release history, and license.
- Resources it creates and the permissions those resources require.
- Cost and security implications of its defaults.

Treat module code as code you are choosing to run: inspect its source and review Terraform's plan before applying it.

## Use a registry module

A registry module source uses the format `namespace/name/provider`. The `version` argument constrains which published module version Terraform can install. This example calls the AWS VPC module:

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

  name = "example-network"
  cidr = "10.0.0.0/16"

  azs             = ["us-west-2a", "us-west-2b"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24"]

  enable_nat_gateway = false
}

output "vpc_id" {
  value = module.network.vpc_id
}
```

The module's documentation defines which input arguments are supported and which outputs are available. For example, `module.network.vpc_id` reads the `vpc_id` output from the child module named `network`. This example is illustrative: creating the VPC resources requires valid AWS credentials and can incur charges.

## Install and update modules

From the root module's working directory:

```bash
terraform init
terraform validate
terraform plan
```

`terraform init` downloads modules into Terraform's local working data. Review the plan before applying. When updating a module, check its changelog and compatibility notes, adjust the version constraint intentionally, and review the resulting plan. `terraform init -upgrade` can update modules within the allowed version constraint.

The dependency lock file `.terraform.lock.hcl` records provider selections, not registry module versions. To make a registry module selection reproducible, pin the module with an exact version (for example, `version = "5.0.0"`) and update it deliberately. Exact version examples should be replaced with a version that the module's documentation lists as available.

For modules hosted in Git, use a Git source address and pin a release tag or commit with `?ref=`. Local child modules use a relative path such as `source = "./modules/network"` and do not have a registry `version` argument:

```hcl
module "network" {
  source = "./modules/network"

  cidr_block = var.network_cidr
}
```

The local module must declare the `cidr_block` input variable. Its output can be referenced from the root module as `module.network.OUTPUT_NAME`.

## What happens to resources and state?

Resources declared by a child module are managed as part of the root module's Terraform operation and state. Their addresses include the module path, for example `module.network.aws_vpc.this`. A module does not isolate state, permissions, credentials, or lifecycle from its caller.

The root module supplies inputs and provider configurations; the child module declares the resources and may expose outputs. A child module can declare provider requirements, but provider configuration and credentials should normally be configured by the root module and passed through as needed.

## Key takeaways

- Search the Terraform Registry or your organization's private registry to find pre-built modules.
- Review a module's documentation, source, versions, defaults, and plan before relying on it.
- Use `source` to locate a module and `version` to constrain a registry module.
- Run `terraform init` to download modules, then validate and review the plan.
- Modules reuse configuration; they do not create a separate state or security boundary.
