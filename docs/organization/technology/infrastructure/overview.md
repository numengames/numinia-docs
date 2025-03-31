---
sidebar_position: 1
id: infrastructure-overview
title: Infrastructure Overview
sidebar_label: Overview
---

# Infrastructure Overview

Our infrastructure is implemented on AWS, leveraging Infrastructure as Code (IaC) principles through Terragrunt and Terraform, with Kubernetes orchestrated via Flux CD. This architecture combines the power of declarative infrastructure with GitOps practices, ensuring consistency and reliability across our platform.

## Documentation Structure

Our infrastructure documentation is organized into three main sections:

• **Base Architecture**: Details our foundational AWS setup, including VPC configuration, networking, and core AWS services implementation.

• **Terragrunt Implementation**: Covers our infrastructure code organization in the [numinia-terragrunt](https://github.com/numengames/numinia-terragrunt) repository, focusing on our modular approach to infrastructure management.

• **Kubernetes Configuration**: Explains our [numinia-k8s](https://github.com/numengames/numinia-k8s) setup, detailing our GitOps implementation with Flux CD and cluster management practices.

## Architecture Overview

Our infrastructure follows a layered approach with three main components:

### Base Infrastructure Layer

The foundation consists of our AWS VPC setup, providing network isolation and security:

• **Network Design**:
  - Production VPC with carefully planned CIDR ranges (10.0.0.0/16)
  - Multi-AZ deployment across eu-west-1a, eu-west-1b, and eu-west-1c
  - Private and public subnet segregation for security
  - NAT Gateway configuration for controlled outbound access

### Container Orchestration Layer

Our container platform is built on Amazon EKS, managed through GitOps principles:

• **Cluster Structure**:
  ```
  clusters/prod-cluster/
  ├── flux-system/         # Core Flux CD components
  ├── flux-kustomization/  # Custom configurations
  ├── infrastructure/      # Cluster-wide resources
  └── organizations/       # Organization-specific deployments
  ```

• **GitOps Implementation**:
  - Infrastructure as Code through Git repositories
  - Automated synchronization via Flux CD
  - Clear separation between infrastructure and application concerns
  - Multi-organization support within single cluster

### Infrastructure Management

Our infrastructure code follows a clear organizational structure:

```
infrastructure/
├── production/                 # Production environment
│   ├── account.hcl            # Account configuration
│   └── eu-west-1/             # Regional resources
│       └── numinia/           # Service components
├── .env                       # Environment variables
└── root.hcl                   # Base configuration
```

## Core Components

Our infrastructure leverages key AWS services and tools:

• **Infrastructure Management**:
  - Terragrunt for infrastructure orchestration
  - Terraform for resource provisioning
  - AWS official modules for reliability

• **Container Platform**:
  - Amazon EKS for Kubernetes
  - Flux CD for GitOps
  - Custom configurations per organization

• **Storage Solutions**:
  - EFS for shared storage
  - EBS for block storage
  - S3 for object storage

## Security Implementation

Security is implemented through multiple complementary layers:

• **Network Security**:
  - VPC isolation and segmentation
  - Private subnets for workloads
  - Controlled internet access

• **Access Control**:
  - IAM roles and policies
  - RBAC in Kubernetes
  - Bastion host for secure access

• **Data Protection**:
  - KMS for key management
  - Encryption at rest
  - Secure state management

## Management Tools

Our platform is managed through a consistent set of tools:

• **Infrastructure Tools**:
  - Terraform (compatible with AWS provider 5.87.0)
  - Terragrunt
  - AWS CLI

• **Kubernetes Tools**:
  - kubectl
  - Flux CLI
  - Kustomize

## Best Practices

When working with our infrastructure:

1. **Version Control**:
   - Use feature branches
   - Follow GitOps workflow
   - Review changes before applying

2. **Security**:
   - Follow least privilege principle
   - Regularly rotate credentials
   - Maintain access logs

3. **Documentation**:
   - Keep documentation updated
   - Document major changes
   - Include examples

4. **Deployment**:
   - Test changes in isolation
   - Use proper planning
   - Follow change management

For detailed implementation specifics, refer to:
- [AWS Architecture Overview](./aws/aws-architecture-overview)
- [Terragrunt Overview](./terragrunt/terragrunt-overview)
- [Kubernetes Overview](./kubernetes/kubernetes-overview) 