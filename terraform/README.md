# Terraform & Infrastructure as Code

Welcome to the **Terraform** module. Learn how to describe infrastructure with HCL, manage it with Terraform, and practice provisioning resources in AWS and Azure.

This module is divided into two focused sections:

- **`concepts/`**: Core Terraform ideas, architecture, and reference guides (Theory)
- **`labs/`**: Step-by-step exercises for authenticating and provisioning cloud resources (Practice)

> **Cloud cost reminder:** Several labs create billable resources. Review each plan before applying and destroy lab resources when finished.

---

## 📚 Concepts (Architecture & Core Ideas)

| # | Concept Topic | Description |
|---|---|---|
| 01 | [Terraform Introduction, Architecture & Workflow](./concepts/terraform-intro-architecture-workflow.md) | Terraform, providers, state, and the init-plan-apply workflow |
| 02 | [Terraform Scripts: Blocks, Arguments, Variables & Outputs](./concepts/terraform-scripts.md) | HCL structure, common blocks, inputs, outputs, and expressions |
| 03 | [Terraform Providers](./concepts/terraform-providers.md) | Provider requirements, configuration, authentication, and versioning |
| 04 | [Terraform Modules](./concepts/terraform-modules.md) | Reusable configuration, the Terraform Registry, and using pre-built modules |
| 05 | [Terraform State](./concepts/terraform-state.md) | How state maps Terraform configuration to managed resources |
| 06 | [Terraform Backends & State Management](./concepts/terraform-backends-state-management.md) | State storage, backend configuration and migration, locking, and safe state operations |
| 07 | [Terraform Workspaces](./concepts/terraform-workspaces.md) | Separate state instances for one configuration |
| 08 | [Terraform Cloud (HCP Terraform)](./concepts/terraform-cloud.md) | Hosted Terraform runs, collaboration, and HCP Terraform workspaces |

---

## 🛠️ Labs (Hands-On Practice)

### AWS Labs

| # | Lab | Focus Area |
|---|---|---|
| 01 | [AWS Authentication & Provider](./labs/aws/lab01-aws-authentication.md) | Configure AWS CLI credentials and use them with the Terraform AWS provider |
| 02 | [Create Two AWS Virtual Machines](./labs/aws/lab02-create-two-vms.md) | Create and destroy two EC2 instances |
| 03 | [AWS Workspaces](./labs/aws/lab03-workspaces.md) | Create two EC2 environments using separate Terraform CLI workspaces |
| 04 | [Terraform Modules](./labs/aws/lab04-terraform-modules.md) | Find and use a Registry module, then build and call a local module |
| 05 | [Deploy a Highly Available Web Application](./labs/aws/lab05-web-application.md) | Build a VPC, load balancer, and auto-scaling web tier |

### Azure Labs

| # | Lab | Focus Area |
|---|---|---|
| 01 | [Azure Authentication & Provider](./labs/azure/lab01-azure-authentication.md) | Sign in with Azure CLI and configure the Terraform AzureRM provider |
| 02 | [Create Two Azure Virtual Machines](./labs/azure/lab02-create-two-vms.md) | Create and destroy two Azure Linux VMs |
| 03 | [Azure Workspaces](./labs/azure/lab03-workspaces.md) | Create two Azure VM environments using separate Terraform CLI workspaces |
| 04 | [Terraform Modules](./labs/azure/lab04-terraform-modules.md) | Find and use a Registry module, then build and call a local module |
| 05 | [Deploy a Highly Available Web Application](./labs/azure/lab05-web-application.md) | Build an Azure Load Balancer and auto-scaling web tier |

---

## Suggested Learning Path

1. Read the introduction, scripts, providers, and state concepts.
2. Complete the AWS or Azure authentication lab.
3. Create and destroy the two-VM lab in that cloud.
4. Continue to the workspace lab to compare separate state instances.
5. Read the modules concept to learn how to find and reuse pre-built Terraform code.
