# Orchestration

## Infrastructure as Code

Infrastructure as Code (IaC) allows you to define and manage your cloud infrastructure using code rather than manual processes. This approach brings version control, repeatability, and automation to your infrastructure management.

## Why use Orchestration?

Orchestration tools help you automate the provisioning and management of your Leafcloud resources. Instead of manually creating instances, networks, and storage through the web interface, you can define your infrastructure in code and deploy it automatically.

**Benefits:**
- **Reproducibility**: Create identical environments every time
- **Version Control**: Track changes to your infrastructure over time
- **Automation**: Deploy complex infrastructures with a single command
- **Documentation**: Your code serves as documentation for your infrastructure
- **Collaboration**: Team members can review and collaborate on infrastructure changes

## Available Tools

Leafcloud supports multiple orchestration tools, each with its own strengths:

### Terraform

Terraform is a popular open-source infrastructure as code tool that uses a declarative configuration language. It's cloud-agnostic and works seamlessly with OpenStack.

**Best for:** Multi-cloud deployments, complex infrastructure, and teams that prefer declarative syntax.

**[Get started with Terraform →](../terraform/creating-instances/)**

### Ansible

Ansible is an automation tool that excels at configuration management and application deployment. It uses a simple, human-readable language (YAML) and doesn't require agents on target machines.

**Best for:** Configuration management, application deployment, and teams that prefer imperative workflows.

**[Get started with Ansible →](../ansible/creating-instances/)**

## Getting Started

Choose the tool that best fits your team's workflow and requirements:

1. **[Terraform](../terraform/creating-instances/)** - If you need a declarative approach and plan to manage complex infrastructure
2. **[Ansible](../ansible/creating-instances/)** - If you prefer imperative workflows and need configuration management capabilities

Both tools integrate seamlessly with Leafcloud's OpenStack infrastructure, allowing you to automate your entire cloud workflow.

