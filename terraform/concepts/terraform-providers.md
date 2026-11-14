# Terraform Providers

A **Terraform provider** is a plugin that lets Terraform interact with a platform or service through its API. Providers supply resource types and data sources—for example, a cloud provider can create virtual machines and networks, and read information about existing resources.

Terraform's core CLI evaluates configuration and plans changes; the provider translates those operations into API requests. A provider does not create infrastructure merely by being declared or configured. Terraform acts on resources and data sources that the configuration uses.

## Declare and install a provider

Declare a provider's source and acceptable versions in the `terraform` block's `required_providers` section:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

In this example, `aws` is the provider's local name, `hashicorp/aws` is its source address in the Terraform Registry, and `~> 5.0` is its version constraint. The constraint allows compatible releases in the 5.x series; choose a constraint that fits the project's tested compatibility policy.

Run `terraform init` to install the providers required by the configuration. Terraform records the selected provider versions and package checksums in `.terraform.lock.hcl`. Commit this lock file so collaborators and automation use consistent selections. When intentionally updating providers, review the result of `terraform init -upgrade` and commit the reviewed lock-file changes.

Provider source and version requirements are different from provider configuration:

- **`required_providers`** identifies the plugin and constrains its version.
- **`provider` blocks** configure that plugin for a particular use, such as selecting a region or project.

## Configure a provider

A provider configuration typically selects a target region, project, or subscription. Authentication should use securely managed credentials or an assigned workload identity, not secrets written directly in configuration.

```hcl
variable "aws_region" {
  description = "AWS region to manage."
  type        = string
  default     = "us-west-2"
}

provider "aws" {
  region = var.aws_region
}

resource "aws_s3_bucket" "logs" {
  bucket = "example-app-logs"
}
```

The `provider` block configures the AWS provider; the `resource` block declares an object for Terraform to manage. The example assumes valid AWS credentials are available through a secure mechanism and that the bucket name is globally available.

Provider arguments vary by provider. Consult that provider's documentation for its supported settings and authentication methods. Prefer environment-based credentials, cloud identity roles, or a credential manager. Do not commit access keys or passwords to source control. Values marked `sensitive` can still be stored in state.

## Multiple configurations with aliases

An alias lets one configuration use multiple instances of the same provider, such as separate regions. Resources select a non-default configuration with the `provider` meta-argument:

```hcl
provider "aws" {
  region = "us-west-2"
}

provider "aws" {
  alias  = "east"
  region = "us-east-1"
}

resource "aws_s3_bucket" "west_logs" {
  bucket = "example-app-west-logs"
}

resource "aws_s3_bucket" "east_logs" {
  provider = aws.east
  bucket   = "example-app-east-logs"
}
```

Resources use the default provider configuration unless they explicitly select an alias. Provider aliases configure connections; they do not by themselves establish separate state or security boundaries.

## Provider lifecycle and version management

1. Declare provider sources and version constraints in `required_providers`.
2. Configure provider settings and credentials using secure mechanisms.
3. Run `terraform init` to install providers and initialize the working directory.
4. Review `terraform plan` to see the proposed resource changes.
5. Apply only after reviewing and approving the changes.

The version constraint describes which versions are acceptable; `.terraform.lock.hcl` records the selected versions for the working directory. Update providers deliberately, inspect plans after upgrades, and test changes before applying them to important environments. A provider upgrade can change behavior or expose configuration differences.

## Providers, resources, and data sources

| Configuration element | Purpose |
| :--- | :--- |
| **Provider requirement** | Names a provider source and constrains acceptable versions. |
| **Provider configuration** | Supplies settings for a provider connection, such as a region. |
| **Resource** | Declares an object Terraform should create and manage through a provider. |
| **Data source** | Reads information from a provider without managing the source object's lifecycle. |

For example, `aws_s3_bucket` is a resource type supplied by the AWS provider, while `aws_caller_identity` is a data source type. Provider types are referenced in configuration, while Terraform state tracks managed resource instances.

## Key takeaways

- Providers are plugins that let Terraform work with platform and service APIs.
- Declare provider sources and version constraints in `required_providers`; configure provider settings in `provider` blocks.
- `terraform init` installs providers and maintains the selected versions in `.terraform.lock.hcl`.
- Use secure authentication methods and protect state, which may contain sensitive values.
- Aliases let resources select among multiple configurations of the same provider.
