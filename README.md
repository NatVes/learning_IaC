# Infrastructure as Code

- [Infrastructure as Code](#infrastructure-as-code)
  - [DevOps mindset](#devops-mindset)
  - [What is IaC?](#what-is-iac)
  - [Types of Infrastructure as Code (IaC)](#types-of-infrastructure-as-code-iac)
  - [Orchestration vs Configuration Management](#orchestration-vs-configuration-management)
    - [Configuration Management (CM)](#configuration-management-cm)
    - [Orchestration](#orchestration)
    - [Summary Comparison](#summary-comparison)
  - [Why Terraform is so popular](#why-terraform-is-so-popular)
  - [How Terraform works](#how-terraform-works)
  - [What is tf.state (terraform.tfstate)?](#what-is-tfstate-terraformtfstate)
    - [Why it is important:](#why-it-is-important)
    - [Why tf.state is sensitive](#why-tfstate-is-sensitive)
    - [Best practices](#best-practices)
  - [How to use Terraform](#how-to-use-terraform)
    - [Setting Up AWS Credentials Using Environment Variables](#setting-up-aws-credentials-using-environment-variables)
    - [1. AWS Credentials Needed](#1-aws-credentials-needed)
    - [2. Set Environment Variables on Windows](#2-set-environment-variables-on-windows)
      - [2.1 Using PowerShell](#21-using-powershell)
      - [2.2 Using Windows → Settings](#22-using-windows--settings)
  - [Terraform Script for Creating an AWS EC2 Instance](#terraform-script-for-creating-an-aws-ec2-instance)
  - [Terraform Core Commands and Workflow](#terraform-core-commands-and-workflow)
    - [1. `terraform init`](#1-terraform-init)
    - [2. `terraform fmt`](#2-terraform-fmt)
    - [3. `terraform plan`](#3-terraform-plan)
    - [4. `terraform apply`](#4-terraform-apply)
    - [5. `terraform destroy`](#5-terraform-destroy)
  - [`.gitidnore` for Terraform](#gitidnore-for-terraform)
    - [Example Template `.gitignore`:](#example-template-gitignore)


## DevOps mindset

A DevOps mindset is an approach focused on delivering software quickly, reliably, and collaboratively through automation and continuous improvement.

- Fast(efficient) — deliver changes quickly and often
-  Stable — systems work without issues
- Reliable — not easy to break
- Collaborative — teams work together
- Improve constantly — learn and get better

🔄 Iteration:  
**build → test → deploy → monitor → improve**

## What is IaC?

**IaC (Infrastructure as Code)** means managing infrastructure (servers, networks, databases) using code instead of manual setup.

**Key points**
- Fast — infrastructure is created quickly
- Standardised — same setup every time
- Reliable — fewer human mistakes
- Collaborative — teams can work together using code
- Automation — create, update, delete automatically
- Consistency — environments stay the same
- Version control — track changes using Git

**Tools**
- Terraform
- Ansible
- AWS CloudFormation
- Pulumi

## Types of Infrastructure as Code (IaC)

**1. Declarative (or Functional) IaC**
- Approach: You describe what the infrastructure should look like, not how to create it. The tool determines the steps to achieve the desired state.
- Characteristics:
  - Idempotent (running the code multiple times has the same effect).
  - Focused on desired state, not commands.
- Tools:
  - Terraform – cloud-agnostic, widely used.
  - AWS CloudFormation – specific to AWS.
  - Kubernetes YAML manifests – for container orchestration.

Example: Define a server with 2 CPUs and 4 GB RAM; Terraform figures out provisioning steps.

**2. Imperative (or Procedural) IaC**
- Approach: You write explicit commands detailing how to create and configure infrastructure step by step.
- Characteristics:
  - Procedural, less abstract.
  - Can be more flexible for complex tasks.
  - Not always idempotent unless carefully managed.
- Tools:
  - Ansible (can be declarative too, but procedural playbooks exist).
  - Chef – uses Ruby scripts to define steps.
  - Puppet – can use procedural logic in manifests.

Example: Write a script that first creates a server, then installs Apache, then sets permissions, in that exact order.

## Orchestration vs Configuration Management

Orchestration and Configuration Management are two approaches to managing infrastructure. They differ in **scope, purpose, and how they relate to IaC types**.

### Configuration Management (CM)

- **Purpose:** Ensure individual machines or services are configured correctly and consistently.  
- **Focus:** Machine-level settings, software installation, users, permissions.  
- **IaC Type:** **Imperative** – you define **step-by-step instructions**.  
- **Example Tools:** Ansible, Chef, Puppet, SaltStack  

**Key idea:** CM is about maintaining the desired configuration on each server or service.

---

### Orchestration

- **Purpose:** Automate coordination of multiple systems, services, or processes.  
- **Focus:** Multi-system workflows, application deployment, scaling, updates.  
- **IaC Type:** **Declarative** – you define **what the final state should be**, tool decides the steps.  
- **Example Tools:** Kubernetes, Docker Swarm, Terraform, Ansible Tower  

**Key idea:** Orchestration is about managing **complex workflows and dependencies** across multiple machines or services.

---

### Summary Comparison

| Feature | Configuration Management | Orchestration |
|---------|-------------------------|---------------|
| Scope | Individual machines | Multiple systems/services |
| Goal | Correct configuration | Workflow automation & coordination |
| IaC Type | Imperative | Declarative |
| Tools | Ansible, Chef, Puppet | Kubernetes, Terraform, Docker Swarm |

## Why Terraform is so popular

Terraform is widely used because it solves several major problems:

**1. Multi-cloud support**  
- Works with AWS, Azure, GCP, etc.
- Avoids vendor lock-in
**2. Declarative approach**
- You define desired state, Terraform handles execution
**3. Strong ecosystem**
- Thousands of providers and modules
**4. Version control friendly**
- Infrastructure stored as code (Git)
**5. Plan before apply**
- Preview changes before execution (`terraform plan`)
**6. Automation**
- Easy integration with CI/CD pipelines

## How Terraform works

**Terraform workflow:**

**Step 1:** Write  
Define infrastructure in `.tf` files (HCL language)  
**Step 2:** Plan  
- `terraform plan` compares:
  - desired state (code)
  - current state (state file)
- Shows what will change  

**Step 3:** Apply
`terraform apply` executes changes

**Internal architecture**

Terraform has two main components:

1. Core
- Calculates changes
- Manages state
- Builds dependency graph
1. Providers
- Plugins that interact with APIs (AWS, Azure, etc.)

**Key concept**

Terraform:

- reads your config
- compares with state
- calls provider APIs
- updates infrastructure

## What is tf.state (terraform.tfstate)?

The **`terraform.tfstate`** file is Terraform’s record of the infrastructure it manages. It **keeps a connection between the real resources running in your environment** (servers, databases, networks) and **what is defined in your Terraform code** (`.tf` files).

- **Real infrastructure** – the actual resources deployed in the cloud or on your servers.  
- **Configuration** – the desired state defined in your Terraform code.  
- **Mapping** – Terraform tracks which real resource corresponds to which part of your code.

### Why it is important:

This **mapping** is stored in the `terraform.tfstate` file. It allows Terraform to:

- Know what exists already, so it doesn’t recreate resources unnecessarily.
- Decide what to create, update, or delete to match the code.
- Track dependencies between resources.

### Why tf.state is sensitive
1. Contains sensitive data
- passwords
- API keys
- infrastructure details
2. Single source of truth
- If corrupted → Terraform loses track
3. Concurrency risk
- Multiple users editing → conflicts
- 
### Best practices
- Use remote state (e.g. S3, Terraform Cloud)
- Enable state locking
- Restrict access
- Never commit to public Git

## How to use Terraform

Terraform requires credentials and variables to manage infrastructure. There are several safe ways to provide them:

1. Environment Variables (env var) *
2. Terraform Variables (tf var)
3. AWS Credentials File

### Setting Up AWS Credentials Using Environment Variables

> **Important:** Never put your AWS Access Key ID or Secret Access Key directly in code or screenshots.

### 1. AWS Credentials Needed

- **AWS Access Key ID**  
- **AWS Secret Access Key**  
- Optionally, **AWS Region** (e.g., `eu-west-1`)

These are provided by AWS IAM.

---

### 2. Set Environment Variables on Windows

#### 2.1 Using PowerShell
```powershell
$Env:AWS_ACCESS_KEY_ID="your_access_key_id_here"
$Env:AWS_SECRET_ACCESS_KEY="your_secret_access_key_here"
```

#### 2.2 Using Windows → Settings
To provide Terraform with credentials or configuration from any folder:

- Press `Win + R`, type `sysdm.cpl` → Enter.
- Go to **Advanced** → **Environment Variables**.
- In the User variables section, click **New**:
```powershell
Variable name: AWS_ACCESS_KEY_ID
Variable value: the value you want to assign
```
- Click OK and close all windows.
- Terraform will automatically read these variables when running commands, so you don’t need to pass them manually.

**To check if set up properly, restart terminal:**

```bash
# Git Bash: 
printenv AWS_ACCESS_KEY
```

## Terraform Script for Creating an AWS EC2 Instance

```bash
# main.tf

# Provider Block
# --------------------------------------------------
# Specifies the AWS cloud provider for Terraform.
# region = "eu-west-1" tells Terraform to create resources in the Europe (Ireland) AWS region.
# Before creating resources, you must run <terraform init> to download provider plugins and dependencies.
provider "aws" {
  # Which region to create it in
  region = "eu-west-1"

  # terraform init - download required dependencies for that cloud service provider  
}

# Resource Block
# -------------------------------------------------
# You state what you want and its desired settings using resource blocks. 
# declares a new EC2 instance named first_app_instance
resource "aws_instance" "first_app_instance" {
  # Which AMI
  # specifies the Amazon Machine Image (AMI) to use; the AMI ID comes from a Terraform variable.
  ami = var.app_ami_id

  # What instance type
  # defines the size/type of the instance.
  instance_type = "t3.micro"

  # Do we want a public IP address
  # ensures the EC2 instance receives a public IP (by default it doesn't have)
  associate_public_ip_address = true

  # Name of recourse
  # assigns a human-readable name to the instance
  tags = {
    Name = "tech601-natalia-tf-instance"
  }
}
```

```bash
# variable.tf

# This file declares a Terraform variable named app_ami_id.
# The variable stores the AMI ID (Amazon Machine Image) used when creating an EC2 instance.

# By using a variable instead of hardcoding the AMI in the resource block, the configuration becomes:

# Reusable – you can change the AMI without editing multiple places in the code.
# Flexible – different environments can use different AMIs.
# Clearer – separates configuration from the resource definition.

variable "app_ami_id" {
  default = "ami-041f717d2124df2b1"
}
```

---

## Terraform Core Commands and Workflow

Terraform has a standard workflow for managing infrastructure. The main commands are described below:

### 1. `terraform init`
- **Purpose:** Initialise a Terraform working directory.  
- **What it does:**  
  - Downloads provider plugins (e.g., AWS, Azure)  
  - Prepares the directory for Terraform operations  

### 2. `terraform fmt`
- **Purpose:** Format Terraform configuration files consistently.
- **What it does:**
  - Automatically fixes indentation, spacing, and syntax style
  - Ensures .tf files are readable and standardised

### 3. `terraform plan`
- **Purpose:** Preview changes Terraform will make to match your code.
- **What it does:**
  - Compares desired state (configuration files) with current state (tf.state)
  - Shows a detailed execution plan of resources to create, update, or delete

### 4. `terraform apply`
- **Purpose:** Apply the planned changes to your infrastructure.
- **What it does:**
  - Creates, updates, or deletes resources to match the declared desired state
  - Requires confirmation (can use `-auto-approve` to skip)

### 5. `terraform destroy`
- **Purpose:** Remove all resources managed by Terraform.
- **What it does:**
  - Deletes infrastructure defined in your configuration
  - Useful for cleaning up test environments or decommissioning resources
  - Requires confirmation 

## `.gitidnore` for Terraform

**Files to Ignore**
- `terraform.tfstate` – tracks real infrastructure state. Contains resource IDs and metadata.
- `terraform.tfstate.backup` – backup of the state file.
- `variables.tf` – may contain sensitive defaults (like AMI IDs, credentials, or secrets).

You can also ignore: `.terraform/` folder, `*.tfvars`, crash logs, and override files.

### Example Template `.gitignore`:

```bash
# Local .terraform directories
.terraform/

# .tfstate files
*.tfstate
*.tfstate.*

# Crash log files
crash.log
crash.*.log

# Exclude all .tfvars files, which are likely to contain sensitive data, such as
# password, private keys, and other secrets. These should not be part of version
# control as they are data points which are potentially sensitive and subject
# to change depending on the environment.
*.tfvars
*.tfvars.json

# Ignore override files as they are usually used to override resources locally and so
# are not checked in
override.tf
override.tf.json
*_override.tf
*_override.tf.json

# Ignore transient lock info files created by terraform apply
.terraform.tfstate.lock.info

# Include override files you do wish to add to version control using negated pattern
# !example_override.tf

# Include tfplan files to ignore the plan output of command: terraform plan -out=tfplan
# example: *tfplan*

# Ignore CLI configuration files
.terraformrc
terraform.rc

# Optional: ignore graph output files generated by `terraform graph`
# *.dot

# Optional: ignore plan files saved before destroying Terraform configuration
# Uncomment the line below if you want to ignore planout files.
# planout
```