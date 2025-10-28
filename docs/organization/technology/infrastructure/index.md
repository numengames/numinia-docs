---
sidebar_position: 1
id: infrastructure
title: Infrastructure
sidebar_label: Infrastructure
---

# Infrastructure

This section documents our infrastructure architecture, implementation, and management practices. Our infrastructure is built on AWS using Infrastructure as Code (IaC) principles with Terragrunt and Terraform, orchestrated through Kubernetes with GitOps practices via Flux CD.

## Quick Start

New developers should start by understanding:

1. **Infrastructure Overview**: Complete architecture and how components interact
2. **AWS Services**: What AWS services we use and why
3. **Terragrunt Implementation**: How we provision infrastructure through code
4. **Kubernetes Platform**: How applications run and scale
5. **Practical Guides**: Step-by-step instructions for common tasks

## Documentation Structure

### Core Architecture

- **[Infrastructure Overview](overview.md)**: Complete narrative of our infrastructure architecture, components, and how they work together to support our multi-tenant web3 platform.

### AWS Services

- **[AWS Services](aws-services.md)**: Comprehensive overview of all AWS services we use including VPC networking, EKS compute, S3 storage, CloudFront CDN, RDS databases, Secrets Manager, Route53 DNS, and ACM certificates.

### Infrastructure Implementation

- **[Terragrunt IaC](implementation/terragrunt.md)**: How we manage infrastructure as code using Terragrunt, including project structure and AWS resources we provision.
- **[S3 & CloudFront Module](implementation/modules/s3-cloudfront.md)**: Detailed implementation of our static assets infrastructure using S3 and CloudFront.

### Kubernetes Platform

- **[Kubernetes Platform](platform/kubernetes.md)**: Architecture of our EKS cluster, GitOps workflow with Flux CD, and multi-tenant organization management.

### Practical Guides

- **[Creating Organization](guides/creating-organization.md)**: Step-by-step process for setting up a new organization in our multi-tenant infrastructure.
- **[Deploying Application](guides/deploying-application.md)**: Complete guide for deploying applications like Hyperfy2 or ElizaOS to our Kubernetes cluster.
- **[Database Access](guides/database-access.md)**: How applications connect to RDS PostgreSQL databases through AWS Secrets Manager.
- **[SSL Certificates](guides/ssl-certificates.md)**: Automated SSL certificate management for organizations.

## Architecture Overview

Our infrastructure follows a layered approach that integrates AWS infrastructure with Kubernetes orchestration to create a scalable multi-tenant platform for web3 spaces and AI applications.

### Infrastructure Layer (AWS)

The foundation consists of our AWS VPC setup, providing network isolation and security for all services. We deploy across multiple availability zones in eu-west-1 for high availability, with private and public subnet segregation ensuring workloads remain secure. A VPN service provides secure remote access to the cluster.

Core AWS services include:
- EKS for managed Kubernetes hosting
- S3 for object storage and static assets
- CloudFront for global content delivery
- RDS PostgreSQL for relational databases
- Secrets Manager for secure credential storage
- Route53 for DNS management
- ACM for SSL/TLS certificates

### Platform Layer (Kubernetes)

Our container platform is built on Amazon EKS, managed through GitOps principles with Flux CD. The cluster runs in private subnets with automated deployment and configuration management. Each customer organization operates in complete isolation within shared infrastructure, with dedicated namespaces, custom domains, SSL certificates, and secrets access.

GitOps ensures that all infrastructure and application changes flow through Git repositories, providing version control, automatic synchronization, rollback capabilities, and a complete audit trail.

### Multi-Tenant Architecture

Organizations like numen-games, active-inference, numinia, and r3s3t operate as isolated tenants within our shared cluster. Each organization can deploy multiple applications and services within their allocated resources, including Hyperfy2 (web3 spaces), ElizaOS (AI agents), and custom applications.

The architecture supports: logical namespace isolation, custom domain management through Route53, automatic TLS certificate provisioning via cert-manager, service account IAM roles for AWS Secrets Manager access, and defined resource quotas for CPU, memory, and storage.

## Deployment Workflow

Our deployment follows a GitOps model where Git serves as the single source of truth:

```
Developer → Git Push → GitHub Repository → Flux CD → Kubernetes Cluster
                ↓              ↓                ↓            ↓
          Code Review    State Detection   Auto Sync   Application Running
```

Changes committed to either the `numinia-terragrunt` or `numinia-k8s` repository are automatically detected by Flux CD (within 1 minute), synchronized to the cluster, and applied as running infrastructure.

## Technology Stack

- **Infrastructure as Code**: Terragrunt + Terraform
- **Container Orchestration**: Amazon EKS (Kubernetes 1.32)
- **GitOps**: Flux CD with 1-minute sync interval
- **Storage**: S3 + CloudFront for static assets, RDS PostgreSQL for databases
- **Configuration Management**: Kustomize
- **Certificate Management**: cert-manager with ACM
- **Secret Management**: AWS Secrets Manager

## Key Principles

- **Infrastructure as Code**: All infrastructure managed through version control
- **GitOps**: Git as single source of truth for all changes
- **Multi-Tenant**: Organizations operate in isolation while sharing resources
- **Scalability**: Horizontal scaling of both compute and storage
- **Security**: Defense-in-depth with multiple security layers
- **Observability**: Comprehensive logging and monitoring

For detailed information, start with the [Infrastructure Overview](overview.md) to understand how everything works together.
