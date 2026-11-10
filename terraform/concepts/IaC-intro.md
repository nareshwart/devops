# Infrastructure as Code (IaC): Introduction

<details>
<summary><strong>What is Infrastructure?</strong></summary>

<br>

Infrastructure is the foundation that supports applications and services. It includes the compute, storage, networking, and security resources needed to run workloads.

Examples of infrastructure:

- Virtual machines or bare-metal servers
- Networking components such as VPCs, subnets, routers, firewalls, and load balancers
- Storage services such as disks, object storage, and databases
- Security policies and identity controls

In simple terms, infrastructure is the environment where applications run.

</details>

---

<details>
<summary><strong>How is Infrastructure Created?</strong></summary>

<br>

Infrastructure can be created in several ways:

- Manually from a cloud provider portal or UI
- Using shell scripts and command-line tools
- Using automation frameworks that define resources as code

For example, a developer may create a VM by clicking through a cloud console. The same VM can also be created by writing configuration code that defines the server, network, and security rules.

</details>

---

<details>
<summary><strong>What is Infrastructure as Code (IaC)?</strong></summary>

<br>

Infrastructure as Code is the practice of defining and managing infrastructure using code instead of manual clicks or one-off scripts.

If we write infrastructure in a file and treat it like software, we can:

- Create environments consistently
- Repeat deployments in a predictable way
- Store configuration in version control
- Review changes before applying them
- Destroy and recreate resources safely

In short, IaC means: "Define infrastructure in code, then let automation create and manage it."

</details>

---

<details>
<summary><strong>Why is IaC Useful?</strong></summary>

<br>

IaC brings the benefits of software engineering to cloud operations:

- **Reproducibility:** the same configuration produces the same result
- **Version control:** changes are tracked in Git
- **Reviewability:** teams can inspect infrastructure changes before deployment
- **Automation:** provisioning becomes faster and less error-prone
- **Scalability:** large environments can be managed consistently
- **Disaster recovery:** infrastructure can be rebuilt from code

</details>

---

<details>
<summary><strong>What are IaaS, PaaS, and SaaS?</strong></summary>

<br>

Cloud providers offer services in different layers:

| Model | Meaning | Example |
| :--- | :--- | :--- |
| **IaaS** | Infrastructure as a Service | Virtual servers, networking, storage |
| **PaaS** | Platform as a Service | Managed databases, app hosting platforms |
| **SaaS** | Software as a Service | Gmail, Office 365, CRM tools |

### IaaS

The cloud provider manages the physical hardware and virtualization layer, while the user manages operating systems, applications, and configuration.

Examples:

- EC2 instances on AWS
- Azure virtual machines
- Compute Engine on GCP

### PaaS

The provider manages more of the platform, so developers focus on application code rather than infrastructure maintenance.

Examples:

- AWS Elastic Beanstalk
- Azure App Service
- Google App Engine

### SaaS

The software is fully managed and delivered over the internet.

Examples:

- Gmail
- Office 365
- Dropbox

</details>

---

<details>
<summary><strong>What tools implement IaC?</strong></summary>

<br>

Many tools can implement infrastructure as code. Some are tied to a particular cloud provider, while others support multiple providers or focus on configuring systems.

### Cloud-specific tools

- AWS CloudFormation
- Azure Resource Manager (ARM)
- Google Cloud Deployment Manager

### Multi-cloud and configuration management tools

- Terraform
- Ansible
- Chef
- Puppet

Terraform is a widely used IaC tool because it is declarative and supports many cloud providers.

</details>

---

<details>
<summary><strong>How does Terraform fit into IaC?</strong></summary>

<br>

Terraform uses a declarative approach. You define the desired end state of your infrastructure, and Terraform determines the changes needed to reach that state.

Example:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"

  tags = {
    Name = "web-server"
  }
}
```

This configuration describes an EC2 instance. Terraform can provision that resource in AWS based on the configuration.

</details>

---

<details>
<summary><strong>What is a typical IaC workflow?</strong></summary>

<br>

1. Write infrastructure configuration in code
2. Initialize the provider and backend
3. Plan the desired changes
4. Apply the configuration
5. Validate and monitor the environment
6. Update or destroy resources when needed

</details>

---

<details>
<summary><strong>Key Takeaway and Summary</strong></summary>

<br>

Infrastructure as Code is the process of managing infrastructure with version-controlled configuration files instead of manual operations. It helps teams build consistent environments, reduce human error, improve speed, and manage infrastructure at scale.

- Infrastructure is the compute, network, and storage foundation for applications.
- Infrastructure can be created manually or through automation.
- IaC means writing infrastructure definitions as code.
- Cloud services fall into IaaS, PaaS, and SaaS models.
- Tools such as Terraform, CloudFormation, Ansible, Chef, and Puppet help implement IaC.
- IaC makes infrastructure repeatable, scalable, and easier to manage.

</details>
