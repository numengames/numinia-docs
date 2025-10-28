---
sidebar_position: 1
id: infrastructure-overview
title: Infrastructure Overview
sidebar_label: Overview
---

# Infrastructure Overview

Our infrastructure is implemented on AWS, leveraging Infrastructure as Code (IaC) principles through Terragrunt and Terraform, with Kubernetes orchestrated via Flux CD. This architecture combines the power of declarative infrastructure with GitOps practices, ensuring consistency and reliability across our platform.

## Documentation Structure

Our infrastructure documentation is organized by abstraction level:

• **AWS Services**: Overview of the AWS services we use (VPC, EKS, S3, CloudFront, RDS, Secrets Manager, etc.)

• **Infrastructure Implementation**: How we provision and manage infrastructure through Terragrunt (Infrastructure as Code)

• **Kubernetes Platform**: How we orchestrate applications using EKS, Flux CD, and our multi-tenant architecture

• **Practical Guides**: Step-by-step guides for creating organizations, deploying applications, accessing databases, and managing SSL certificates

## Architecture Overview

Our infrastructure follows a layered approach with three main components:

### Base Infrastructure Layer

The foundation consists of our AWS VPC setup, providing network isolation and security for all our services:

• **Network Design**:
  - Production VPC with carefully planned CIDR ranges (10.0.0.0/16)
  - Multi-AZ deployment across eu-west-1a, eu-west-1b, and eu-west-1c for high availability
  - Private and public subnet segregation ensuring workloads remain secure
  - NAT Gateway configuration for controlled outbound internet access
  - VPN service for secure remote access to the cluster (replacing traditional bastion host approach)

• **Core AWS Services**:
  - **EKS**: Managed Kubernetes cluster hosting all containerized applications
  - **S3**: Object storage for static assets and data
  - **CloudFront**: Global CDN distributing content from S3
  - **RDS**: PostgreSQL databases managed by DevOps team
  - **Route53**: DNS management for all domains
  - **Secrets Manager**: Secure storage for application credentials and keys

### Container Orchestration Layer

Our container platform is built on Amazon EKS, managed through GitOps principles that ensure consistency and traceability:

• **Cluster Structure**:
  ```
  clusters/prod-cluster/
  ├── flux-system/         # Core Flux CD components
  ├── infrastructure/      # Cluster-wide resources (monitoring, networking)
  └── organizations/       # Organization-specific deployments
      ├── active-inference/
      ├── numen-games/
      ├── numinia/
      └── r3s3t/
  ```

• **GitOps Implementation**:
  - Infrastructure as Code through Git repositories
  - Automated synchronization via Flux CD (syncs every 1 minute)
  - Clear separation between infrastructure and application concerns
  - Multi-tenant support with isolated namespaces per organization
  - All changes are version controlled and automatically applied

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
  - S3 for object storage and static asset hosting
  - CloudFront CDN for global content distribution
  - RDS PostgreSQL for relational database needs
  - Secrets Manager for secure credential storage

## Security Implementation

Security is implemented through multiple complementary layers that protect our infrastructure and applications:

• **Network Security**:
  - VPC isolation and segmentation creating logical boundaries
  - Private subnets for all workloads preventing direct internet exposure
  - Controlled internet access through NAT gateways
  - VPN service for secure remote cluster access
  - Security groups controlling traffic at the instance level

• **Access Control**:
  - IAM roles and policies with principle of least privilege
  - RBAC in Kubernetes for fine-grained control
  - AWS Secrets Manager integration for secure credential storage
  - Service accounts with IAM roles for AWS resource access
  - VPN-based access replacing traditional bastion host approach

• **Data Protection**:
  - Encryption at rest for all storage (S3, RDS)
  - Encryption in transit with TLS for all communications
  - Secure state management with encrypted Terraform state
  - Regular security audits and access logging

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

## Multi-Tenant Architecture

Our infrastructure is designed around a multi-tenant model where each customer organization operates in complete isolation while sharing the underlying infrastructure resources. This design allows us to efficiently scale our platform while maintaining security and resource boundaries.

### Organization Structure

Each organization in our system represents a customer who can deploy multiple applications and services within their allocated resources. The architecture supports organizations like:

- **numen-games**: Customer with multiple web3 spaces and AI applications
- **active-inference**: AI-focused organization with ElizaOS deployments
- **art-unchained**: Organization specializing in NFT and digital art spaces
- **r3s3t**: Customer with various web3 applications

### Organization Components

Each organization receives:

- **Dedicated Namespace**: Kubernetes namespace providing logical isolation
- **Custom Domain**: Each organization can have its own domain managed through Route53
- **SSL Certificates**: Automatic TLS certificate provisioning via cert-manager
- **Secrets Access**: Service account with IAM role for AWS Secrets Manager access
- **Resource Quotas**: Defined limits on CPU, memory, and storage
- **Service Selection**: Deploy Hyperfy2 (web3 spaces), ElizaOS (AI agents), or custom applications

### Application Deployment Flow

When a new application is deployed, the workflow follows our GitOps principles:

1. **Configuration in Git**: Application configuration is committed to the [numinia-k8s](https://github.com/numengames/numinia-k8s) repository
2. **Flux Detection**: Flux CD monitors the repository and detects changes within 1 minute
3. **Automatic Sync**: Flux applies the configuration to the Kubernetes cluster
4. **Resource Creation**: Helm charts deploy the application with proper service accounts and networking
5. **DNS Configuration**: Domain mapping is automatically configured through KNative
6. **Certificate Provisioning**: cert-manager requests and validates TLS certificates

This automated flow ensures consistency, traceability, and rapid deployment without manual intervention.

## GitOps Deployment Workflow

Our entire infrastructure follows a GitOps model where Git serves as the single source of truth. This approach provides significant advantages:

- **Version Control**: All infrastructure and application changes are tracked in Git
- **Automatic Synchronization**: Flux CD continuously monitors repositories and applies changes
- **Rollback Capability**: Any change can be reverted through Git history
- **Collaboration**: Multiple team members can review and approve changes
- **Audit Trail**: Complete history of all infrastructure modifications

### Deployment Pipeline

```
Developer → Git Push → GitHub Repository → Flux CD → Kubernetes Cluster
                ↓              ↓                ↓            ↓
          Code Review    State Detection   Auto Sync   Application Running
```

The flow begins when a developer commits changes to either the `numinia-terragrunt` or `numinia-k8s` repository. Once merged to main, the automated synchronization begins, and within minutes, the changes are reflected in the running infrastructure.

For detailed implementation specifics, refer to:
- [AWS Services](aws-services.md) - What AWS services we use
- [Terragrunt IaC](implementation/terragrunt.md) - How we implement infrastructure as code
- [Kubernetes Platform](platform/kubernetes.md) - How our K8s platform works 