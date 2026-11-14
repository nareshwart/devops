# Lab 01: AWS Authentication and Terraform Provider

This lab configures local AWS authentication with your own AWS CLI access keys, then uses the authenticated identity in a Terraform AWS provider. It verifies access by reading the caller identity; it does not create AWS infrastructure.

## Objectives

- Authenticate to an AWS account without putting credentials in Terraform files.
- Configure the AWS provider to use the standard AWS credential chain.
- Initialize Terraform and verify the identity available to the provider.

## Prerequisites

- Terraform CLI installed.
- An AWS account and an access key ID and secret access key for your learner identity. An AWS console password alone cannot authenticate the AWS CLI.

## Step 0: Install the AWS CLI if needed

If `aws --version` already works, skip this step. Otherwise, install AWS CLI v2 using the official [AWS CLI installation guide](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html). For example:

| Operating system | Install command |
| :--- | :--- |
| macOS with Homebrew | `brew install awscli` |
| Windows with WinGet | `winget install --exact --id Amazon.AWSCLI` |
| Linux | Follow the installation guide for your distribution and CPU architecture. |

Open a new terminal if required, then verify the installation:

```bash
aws --version
```

## Step 1: Configure your AWS CLI credentials

Create a named profile with your own AWS CLI credentials:

```bash
aws configure --profile terraform-lab
```

When prompted, enter your access key ID, secret access key, default region (for example, `us-west-2`), and output format. AWS CLI stores the profile locally in your home directory (`~/.aws/credentials` and `~/.aws/config`); do not copy credentials into Terraform files or commit those local files.

Select the profile for Terraform and verify the AWS CLI identity:

```bash
export AWS_PROFILE=terraform-lab
aws sts get-caller-identity
```

The output should identify the AWS account and assumed role or user available to the profile. Do not paste credentials or account details into source control.

## Step 2: Create the Terraform configuration

Create a file named `main.tf` in your lab directory. The complete example is also available as [`main.tf`](./main.tf):

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
  description = "AWS region used by the provider."
  type        = string
  default     = "us-west-2"
}

provider "aws" {
  region = var.aws_region
}

data "aws_caller_identity" "current" {}

output "aws_account_id" {
  description = "AWS account ID associated with the active credentials."
  value       = data.aws_caller_identity.current.account_id
}

output "aws_caller_arn" {
  description = "ARN associated with the active AWS credentials."
  value       = data.aws_caller_identity.current.arn
}
```

No access key or secret key is specified in the provider block. The AWS provider obtains credentials through the standard AWS credential chain, which uses the profile selected by `AWS_PROFILE`.

## Step 3: Initialize and verify

Run these commands from this lab's directory:

```bash
terraform init
terraform fmt -check
terraform validate
terraform plan
terraform apply
```

Review the plan before applying. This example only queries the AWS caller identity, so it should not propose creating or changing infrastructure. Confirm the apply when prompted, then inspect the output:

```bash
terraform output
```

The output contains the account ID and ARN returned by AWS. Treat local Terraform state and command output as data that may be sensitive.

## Authentication notes

- `AWS_PROFILE` selects a profile from the AWS shared configuration and credentials files; it does not put credentials in the Terraform configuration.
- Keep the access key pair private, restrict access to the local credential files, and never commit them to source control.
- For real deployments and automation, prefer short-lived role-based credentials over long-lived access keys.

## Summary checklist

- [ ] Configured your own CLI access key pair in the `terraform-lab` profile.
- [ ] Confirmed the AWS CLI can call `sts get-caller-identity`.
- [ ] Initialized and validated the Terraform configuration.
- [ ] Applied the read-only identity check and reviewed its outputs.
