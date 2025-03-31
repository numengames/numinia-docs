---
sidebar_position: 3
id: kubernetes-infrastructure
title: Kubernetes Infrastructure
sidebar_label: Kubernetes
---

# Kubernetes Infrastructure

This section documents our Kubernetes infrastructure, which serves as the foundation for application deployment and management across our organization. Our Kubernetes implementation follows a GitOps approach using Flux CD to manage deployments and configurations.

## Contents

- **[Kubernetes Overview](kubernetes-overview)**: Overview of our Kubernetes architecture, cluster design, and operational strategy.

- **Security**:
  - **[SSL Certificates](security/ssl-certificates)**: How SSL certificates are managed and integrated with our applications.

- **Deployments**:
  - **[Deploying New App](deployments/deploying-new-app)**: Step-by-step guide for deploying a new application to our cluster.
  - **[Creating Organization](deployments/creating-organization)**: Process for creating a new organization in our multi-tenant infrastructure.

## Key Features

- EKS-based managed Kubernetes
- GitOps deployment model with Flux CD
- Multi-tenant architecture with organization isolation
- Shared infrastructure services (monitoring, logging, storage)
- Automated SSL certificate management
- Integration with AWS services

## Related Guides

- For information about the underlying AWS infrastructure, see the [AWS Infrastructure](/docs/organization/technology/infrastructure/aws/aws-architecture-overview) section.
- To understand how the infrastructure is provisioned with IaC, check the [Terragrunt](/docs/organization/technology/infrastructure/terragrunt/terragrunt-overview) section. 