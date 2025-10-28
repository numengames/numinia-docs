---
sidebar_position: 3
id: terragrunt
title: Terragrunt Infrastructure as Code
sidebar_label: Terragrunt IaC
---

# Terragrunt Infrastructure as Code

Our infrastructure management leverages Terragrunt to provide a DRY (Don't Repeat Yourself) approach to Terraform configurations. This document details our implementation strategy and how we provision AWS resources through code.

## Project Structure

Our infrastructure code resides in the [numinia-terragrunt](https://github.com/numengames/numinia-terragrunt) repository, organized in a clear and logical structure that reflects our deployment hierarchy:

```
infrastructure/
├── production/                 # Production environment
│   ├── account.hcl            # Account-specific configuration
│   └── eu-west-1/             # Regional resources
│       └── numinia/           # Service components
│           ├── vpc/           # Network infrastructure
│           ├── eks/           # Kubernetes cluster
│           ├── s3/            # Object storage
│           └── cloudfront/    # Content delivery
├── .env                       # Environment variables
└── root.hcl                   # Base configuration
```

This structure embodies several key design principles. First, we maintain a clear separation between environments, ensuring production configurations are isolated and protected. Second, our regional organization creates clear boundaries for resources, making it easier to manage multi-region deployments. Finally, each service component is isolated in its own directory, promoting modularity and maintainability.

## Core Configuration Files

### Root Configuration (root.hcl)

The `root.hcl` file serves as the foundation of our Terragrunt configuration, defining common settings and state management. This configuration establishes our state management strategy, utilizing S3 for centralized state storage with encryption enabled.

### Environment Variables (env.hcl)

Our environment-specific configurations are managed through the `env.hcl` file, which centralizes all module sources and infrastructure identifiers. This approach allows us to maintain consistency across all modules while making updates straightforward.

The configuration includes module sources for VPC, EKS, EC2, S3, ACM, and CloudFront. It also contains infrastructure IDs such as VPC ID, Route53 zone IDs, CloudFront distribution ARNs, and other resource identifiers that are referenced across multiple modules.

## AWS Resources We Manage

Through Terragr destroyed, we manage the following AWS resources:

### VPC and Networking

The VPC module implements our network infrastructure using the official AWS VPC module. Configuration includes CIDR blocks, subnet definitions across availability zones, NAT gateway setup, and security group assignments.

### EKS Cluster

The EKS module manages our Kubernetes cluster deployment, including cluster version, node groups, and integration with the VPC. This module handles all EKS-specific configurations including add-ons like CoreDNS, kube-proxy, VPC-CNI, and the EFS CSI driver.

### S3 and CloudFront

S3 buckets are configured through Terragrunt with versioning, encryption, and access controls. CloudFront distributions connect to S3 origins using Origin Access Control for secure access. 

For detailed S3/CloudFront implementation, see [S3 & CloudFront Module](modules/s3-cloudfront.md).

### Route53 and ACM

Route53 manages DNS records for our domains, including alias records for CloudFront distributions. ACM certificates are provisioned for SSL/TLS, with special handling for the us-east-1 requirement when used with CloudFront.

### EC2 Instances

Bastion hosts and VPN instances are managed through the EC2 module, with security groups, IAM roles, and instance type configurations.

> [!NOTE]
> The previous EFS-based storage approach has been deprecated in favor of S3 for better scalability and cost optimization. EFS configurations remain in the repository but are not actively used.

## State Management

Our state management approach ensures consistency and security across all infrastructure components. The S3 backend provides reliable state storage, with state files per component following a clear naming convention that reflects our infrastructure hierarchy.

Access to state files is strictly controlled through IAM roles and policies, with comprehensive audit logging enabled to track all access and modifications. We maintain regular backups of state files and implement version history to enable recovery if needed.

## Deployment Workflow

Infrastructure deployment follows a standardized workflow using Terragrunt commands. We use `terragrunt run-all plan` to preview changes across modules and `terragrunt run-all apply` to implement those changes. This approach ensures that dependencies are respected and resources are created in the correct order.

For specific component deployments and modifications, refer to the respective module documentation in the [numinia-terragrunt](https://github.com/numengames/numinia-terragrunt) repository.

