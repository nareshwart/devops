# Terraform Scripts: Blocks, Arguments, Variables & Outputs

Terraform configurations are written in HashiCorp Configuration Language (HCL) and are commonly called Terraform scripts. The main focus of this guide is how scripts are structured with **blocks** and **arguments**, followed by how variables and outputs make them reusable and useful.

<details>
<summary><strong><big>1. What does a Terraform script look like?</big></strong></summary>

<br>

Terraform scripts are saved in files ending with `.tf`. Terraform loads all `.tf` files in the current working directory together as one configuration.

```hcl
resource "aws_s3_bucket" "logs" {
  bucket = var.bucket_name
}
```

In this example:

- `resource` is the **block type**.
- `"aws_s3_bucket"` and `"logs"` are the block's labels.
- `bucket = var.bucket_name` is an **argument** inside the block.

The general structure is:

```hcl
<BLOCK_TYPE> "<LABEL>" "<OPTIONAL_SECOND_LABEL>" {
  <ARGUMENT> = <EXPRESSION>
}
```

The exact number and meaning of labels depend on the block type.

</details>

---

<details>
<summary><strong><big>2. Terraform blocks</big></strong></summary>

<br>

### What is a Terraform block?

A **Terraform block** is a structural section in an HCL configuration. It starts with a block type, may include labels, and has a body enclosed in `{ }`. The block type tells Terraform how to interpret the contents; the body can contain arguments and, where allowed, nested blocks.

For example:

```hcl
resource "aws_s3_bucket" "logs" {
  bucket = var.bucket_name
}
```

Here, `resource` is the block type, `"aws_s3_bucket"` and `"logs"` are labels, and `bucket = var.bucket_name` is an argument in the block body. This general meaning of a Terraform block is different from the specific **`terraform` block**, which configures Terraform itself.

### Terraform blocks

Each row summarizes a block, its purpose, and a complete example.

| Block type | Example syntax | Explanation | Full syntax example |
| :--- | :--- | :--- | :--- |
| `terraform` | `terraform { ... }` | Configures Terraform itself, not a provider instance or infrastructure resource.<br><br>`required_version` constrains the CLI version; `required_providers` declares provider sources and versions; backend settings configure state storage.<br><br>Choose tested version constraints and commit `.terraform.lock.hcl`. Backend settings cannot use ordinary input variables; changing a backend may require running `terraform init` again. | <pre><code>terraform {&#10;  required_version = "&gt;= 1.5.0, &lt; 2.0.0"&#10;&#10;  required_providers {&#10;    aws = {&#10;      source  = "hashicorp/aws"&#10;      version = "~&gt; 5.0"&#10;    }&#10;  }&#10;}</code></pre> |
| `provider` | `provider "aws" { ... }` | Configures a provider plugin so Terraform can connect to a platform or service; it does not create resources by itself.<br><br>Common settings select a region, project, or subscription. Use an assigned cloud identity, credential profile, or another secure mechanism; never hardcode secrets.<br><br>Use `alias` for another provider configuration. Resources use the default provider unless the `provider` meta-argument selects another. | <pre><code>provider "aws" {&#10;  region = var.aws_region&#10;}&#10;&#10;provider "aws" {&#10;  alias  = "secondary"&#10;  region = "us-east-1"&#10;}</code></pre> |
| `resource` | `resource "TYPE" "NAME" { ... }` | Declares an object Terraform should create and manage, such as a VM, network, database, or DNS record.<br><br>The first label is the resource type; the second is its local name. Together, `aws_s3_bucket.logs` form its address. Terraform tracks managed resources in state, and references to their attributes can establish dependencies.<br><br>Some changes replace a resource. Review the plan before applying; use `depends_on` only when a dependency cannot be inferred. | <pre><code>resource "aws_s3_bucket" "logs" {&#10;  bucket = var.bucket_name&#10;&#10;  tags = {&#10;    Purpose = "application logs"&#10;  }&#10;}</code></pre> |
| `data` | `data "TYPE" "NAME" { ... }` | Reads information about an existing object or supported source without managing that object's lifecycle.<br><br>The first label selects the provider data source; the second gives it a local name. Reference attributes as `data.TYPE.NAME.ATTRIBUTE`, for example `data.aws_vpc.selected.cidr_block`.<br><br>A data block reads information; it does not create or take ownership of the object. | <pre><code>data "aws_caller_identity" "current" {}&#10;&#10;output "account_id" {&#10;  value = data.aws_caller_identity.current.account_id&#10;}</code></pre> |
| `variable` | `variable "NAME" { ... }` | Declares an input so callers can customize a module for different environments without changing its implementation.<br><br>`type` constrains the value; `description` documents it; `default` makes it optional; `validation` can enforce rules. Reference it with `var.NAME`. Without a default, the caller must provide a value.<br><br>Values can come from variable files, CLI options, environment variables, or a parent module. `sensitive = true` redacts some CLI output but does not encrypt values in state. Avoid committing secrets. | <pre><code>variable "bucket_name" {&#10;  description = "Globally unique name for the bucket."&#10;  type        = string&#10;&#10;  validation {&#10;    condition     = length(var.bucket_name) &gt; 2&#10;    error_message = "The bucket name must be longer than 2 characters."&#10;  }&#10;}</code></pre> |
| `output` | `output "NAME" { ... }` | Exposes a value calculated by a module, such as a resource ID, endpoint, or address.<br><br>`value` is the expression to expose; `description` documents it. Root-module outputs are displayed after apply and queried with `terraform output`. A parent module accesses a child output with `module.NAME.OUTPUT_NAME`.<br><br>Outputs are recorded in state. Marking one sensitive redacts some CLI output but does not remove its value from state. | <pre><code>output "bucket_arn" {&#10;  description = "ARN of the managed bucket."&#10;  value       = aws_s3_bucket.logs.arn&#10;}</code></pre> |
| `locals` | `locals { ... }` | Names expressions for reuse within one module and helps avoid repeated derived values or settings.<br><br>Locals are calculated by the configuration; callers cannot set them directly. Reference a local with `local.NAME`. | <pre><code>locals {&#10;  common_tags = {&#10;    Environment = var.environment&#10;    ManagedBy   = "Terraform"&#10;  }&#10;}&#10;&#10;resource "aws_s3_bucket" "logs" {&#10;  bucket = var.bucket_name&#10;  tags   = local.common_tags&#10;}</code></pre> |
| `module` | `module "NAME" { ... }` | Calls another module to reuse a package of related Terraform configuration. The root module is the working directory where Terraform runs; a child module is called from another module.<br><br>`source` locates the module. Other arguments pass values to its input variables; outputs are available as `module.NAME.OUTPUT_NAME`. The child module must declare the inputs and outputs used by the caller. | <pre><code>module "network" {&#10;  source     = "./modules/network"&#10;  cidr_block = var.network_cidr&#10;}&#10;&#10;output "network_id" {&#10;  value = module.network.network_id&#10;}</code></pre> |

</details>

---

<details>
<summary><strong><big>3. Arguments: values inside blocks</big></strong></summary>

<br>

An **argument** assigns a value to a named setting inside a block using `=`. Arguments are how you configure a resource, provider, variable, or module. The expected argument names and value types depend on the block type and provider schema.

```hcl
resource "aws_s3_bucket" "logs" {
  bucket = var.bucket_name
}
```

Here, `bucket` is an argument and `var.bucket_name` is its value expression. The value can be a literal, a reference, or a more complex expression.

A **nested block** groups related settings inside its parent. Unlike an argument assignment, it has a block body enclosed in braces:

```hcl
resource "aws_s3_bucket" "logs" {
  bucket = var.bucket_name

  tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}
```

In this example, `tags` is an argument whose value is an object. By contrast, some provider schemas define nested blocks such as `lifecycle_rule { ... }`. Whether a setting is an argument or nested block depends on the block type's schema.

### Common argument value forms

| Value form | Example | Use |
| :--- | :--- | :--- |
| Literal | `instance_type = "t3.micro"` | Set a fixed string, number, or boolean |
| Variable reference | `region = var.aws_region` | Accept a value from the caller |
| Resource reference | `subnet_id = aws_subnet.app.id` | Connect resources and let Terraform infer dependencies |
| Local reference | `tags = local.common_tags` | Reuse a computed value within the module |
| Expression | `name = "${var.project}-${var.environment}"` | Derive a value from other expressions |
| Collection or object | `tags = { Environment = var.environment }` | Supply grouped values such as maps, lists, and objects |

### Meta-arguments

Terraform defines special arguments that can be used with many resource and module blocks:

| Meta-argument | Purpose |
| :--- | :--- |
| `depends_on` | Declares an explicit dependency when Terraform cannot infer it from references |
| `count` | Creates a numbered set of instances |
| `for_each` | Creates instances from a map or set, identified by keys |
| `provider` | Selects a provider configuration |
| `lifecycle` | Controls selected lifecycle behaviors, such as replacement rules |

Meta-arguments control how Terraform handles a block rather than setting a provider-specific property. Use `count` or `for_each` to manage multiple instances from one block. Prefer references between resources to express dependencies; Terraform can infer those automatically in many cases.

</details>

---

<details>
<summary><strong><big>4. Input variables</big></strong></summary>

<br>

Input variables let a module accept values from its caller rather than hardcoding every setting.

```hcl
variable "bucket_name" {
  description = "Globally unique name for the S3 bucket."
  type        = string
}

variable "environment" {
  description = "Environment label applied to the bucket."
  type        = string
  default     = "dev"
}

variable "aws_region" {
  description = "AWS region for the provider."
  type        = string
  default     = "us-west-2"
}
```

Use an input variable in an expression with the `var.` namespace:

```hcl
bucket = var.bucket_name
```

## Variable properties

- **`type`** constrains the accepted value, such as `string`, `number`, `bool`, `list(string)`, `set(string)`, `map(string)`, or an `object({...})`.
- **`description`** documents the variable's purpose.
- **`default`** makes the variable optional. Without a default, Terraform will ask for a value if none is supplied.
- **`sensitive = true`** redacts the value in some CLI output. It does not encrypt the value in state or make it safe to commit.
- **`validation`** can enforce rules for input values.

Example validation:

```hcl
variable "environment" {
  description = "Environment label applied to the bucket."
  type        = string
  default     = "dev"

  validation {
    condition     = contains(["dev", "stage", "prod"], var.environment)
    error_message = "Choose dev, stage, or prod."
  }
}
```

Values can be provided through a `terraform.tfvars` file, a named `.tfvars` file passed with `-var-file`, the `-var` option, or supported environment variables. Keep credentials and other secrets out of source control and use a secure secrets workflow.

</details>

---

<details>
<summary><strong><big>5. Outputs</big></strong></summary>

<br>

An output publishes a value calculated by a module. Root-module outputs are shown after an apply and can be queried with `terraform output`; child-module outputs can be referenced by the calling module.

```hcl
output "bucket_name" {
  description = "Name of the managed S3 bucket."
  value       = aws_s3_bucket.logs.bucket
}

output "bucket_arn" {
  description = "ARN of the managed S3 bucket."
  value       = aws_s3_bucket.logs.arn
}
```

Common output arguments include:

- **`value`**: the expression to expose.
- **`description`**: explains the output.
- **`sensitive = true`**: redacts the value in some CLI output. The value may still be stored in state.

Outputs are useful for sharing resource attributes with people, scripts, or other modules. Avoid outputting secrets unless necessary, and protect state because it can contain sensitive values.

</details>

---

<details>
<summary><strong><big>6. Putting blocks and arguments together</big></strong></summary>

<br>

This small example brings together provider configuration, a resource, input variables, and outputs:

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

provider "aws" {
  region = var.aws_region
}

variable "aws_region" {
  description = "AWS region where the bucket will be created."
  type        = string
  default     = "us-west-2"
}

variable "bucket_name" {
  description = "Globally unique name for the S3 bucket."
  type        = string
}

variable "environment" {
  description = "Environment label applied to the bucket."
  type        = string
  default     = "dev"
}

resource "aws_s3_bucket" "logs" {
  bucket = var.bucket_name

  tags = {
    Environment = var.environment
    ManagedBy   = "Terraform"
  }
}

output "bucket_name" {
  description = "Name of the managed S3 bucket."
  value       = aws_s3_bucket.logs.bucket
}

output "bucket_arn" {
  description = "ARN of the managed S3 bucket."
  value       = aws_s3_bucket.logs.arn
}
```

The dependency is expressed by the reference `var.bucket_name` from the resource and the references to `aws_s3_bucket.logs` from the outputs. Terraform uses these references to determine the order of operations.

| Configuration item | Role in the example |
| :--- | :--- |
| `terraform` | Declares Terraform and AWS provider requirements |
| `provider` | Configures the AWS region |
| `variable` | Accepts region, bucket name, and environment values |
| `resource` | Declares the S3 bucket Terraform manages |
| `output` | Exposes the created bucket's name and ARN |

The example requires valid AWS credentials and a globally unique bucket name. Review the plan before applying it, and remove resources only when their deletion is intended.

</details>

---

<details>
<summary><strong><big>Key takeaways</big></strong></summary>

<br>

- Terraform configuration is made up of block types, labels, arguments, and expressions.
- `resource` blocks manage objects; `data` blocks read existing information.
- Input variables make module configuration reusable.
- Outputs expose useful values from a module.
- Keep credentials secure; `sensitive` hides some CLI display but does not protect values in state.
- Terraform infers many dependencies from references between configuration elements.

</details>
