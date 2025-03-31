---
sidebar_position: 2
id: terragrunt-section
title: Terragrunt Infrastructure as Code
sidebar_label: Terragrunt IaC
---

# Terragrunt Infrastructure as Code

This section documents our approach to Infrastructure as Code using Terragrunt, a tool that enhances Terraform by providing DRY (Don't Repeat Yourself) configurations.

## Contents

- **[Terragrunt Overview](terragrunt-overview)**: Overview of our Terragrunt implementation, project structure, and best practices.

- **Implementations**:
  - **[S3 & CloudFront Implementation](implementations/s3-cloudfront-implementation)**: Technical details of our CDN implementation with S3 and CloudFront using Terragrunt.

## Why Terragrunt

We've chosen Terragrunt for several key reasons:

- **DRY Configurations**: Avoids code duplication between environments and components
- **Dependency Management**: Automatically handles dependencies between resources
- **Remote State Management**: Simplifies remote state configuration
- **Modular Composition**: Facilitates code reuse and infrastructure composition

## Related

- To understand the general AWS architecture we're implementing, see the [AWS Infrastructure](/docs/organization/technology/infrastructure/aws/aws-architecture-overview) section.
- To see how we deploy applications on this infrastructure, visit the [Kubernetes](/docs/organization/technology/infrastructure/kubernetes/kubernetes-overview) section. 