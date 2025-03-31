---
sidebar_position: 1
id: aws-architecture-overview
title: AWS Architecture Overview
sidebar_label: Architecture Overview
---

# Base Architecture

Our base infrastructure architecture is built on AWS, providing a secure and scalable foundation for all our services. This document details the technical implementation of our core infrastructure components and the reasoning behind our architectural decisions.

## Network Architecture

### VPC Design
Our Virtual Private Cloud (VPC) implementation leverages the `terraform-aws-vpc` module, carefully configured to support our distributed architecture:

```hcl
vpc_cidr = "10.0.0.0/16"
azs      = ["eu-west-1a", "eu-west-1b", "eu-west-1c"]
```

We've made several key design decisions in our VPC architecture to ensure maximum reliability and security. Our multi-AZ deployment spans three availability zones in eu-west-1, providing a robust foundation that maintains operations even during an AZ failure. This design choice enables us to offer a 99.99% availability SLA to our users.

For internet access from private resources, we've implemented NAT gateways in each public subnet. While this approach carries a higher cost compared to a single NAT gateway, it eliminates potential single points of failure and significantly improves our system's availability. We continuously monitor NAT Gateway usage to optimize costs while maintaining automatic failover capabilities between gateways.

To optimize AWS service communication, we extensively use VPC endpoints. These endpoints keep traffic within the AWS network, reducing latency and eliminating the need for internet access when communicating with AWS services. Our implementation includes S3 Gateway Endpoints for efficient S3 access, Interface Endpoints for EKS API server communication, and CloudWatch Logs endpoints for secure log delivery.

Network monitoring is comprehensive, with VPC flow logs enabled for security auditing, troubleshooting, and compliance requirements. These logs integrate directly with CloudWatch Logs and follow retention policies aligned with our compliance needs.

### Subnet Structure
Our subnet architecture implements AWS best practices with meticulously planned CIDR allocations. The private subnets are configured as follows:

```
eu-west-1a: 10.0.1.0/24   (254 available IPs)
eu-west-1b: 10.0.2.0/24   (254 available IPs)
eu-west-1c: 10.0.3.0/24   (254 available IPs)
```

The choice of /24 subnets for each zone provides 254 usable IPs, striking a balance between address space efficiency and room for growth. We've reserved specific IP ranges for future expansion and maintain consistent sizing across AZs for predictability in our infrastructure planning.

Our workload isolation strategy places EKS worker nodes and internal services in these private subnets, preventing direct internet access while maintaining necessary connectivity through NAT gateways. The subnet design includes dedicated ranges for different workload types and maintains clear separation between application and database tiers.

The public subnets follow a distinct CIDR range pattern:

```
eu-west-1a: 10.0.101.0/24   (254 available IPs)
eu-west-1b: 10.0.102.0/24   (254 available IPs)
eu-west-1c: 10.0.103.0/24   (254 available IPs)
```

We deliberately chose the 10.0.101.0/24 range for public subnets to make them easily distinguishable from private ones, simplifying network troubleshooting and management. These subnets host our load balancers and bastion hosts, with carefully configured security groups controlling access.

## Access Management

### IAM Configuration

Our role-based access control model emphasizes security and auditability through carefully defined roles. The bastion host role (arn:aws:iam::241533135482:role/prod-bastion-host-eks-role) provides limited EKS permissions for cluster management and troubleshooting, while maintaining minimal required permissions for instance operations. Integration with Systems Manager enables secure session management and automated patching.

For storage management, we've implemented the EFS CSI Driver role (arn:aws:iam::241533135482:role/efs-csi-driver-role) with specific permissions for EFS volume lifecycle operations and file system management, strictly limited to required EFS API calls within the cluster's context.

### EC2 Instance Management

Our bastion host configuration reflects careful consideration of security and efficiency. We've selected Amazon Linux 2 as our base AMI for its regular security updates and seamless AWS integration. The instance type (t3.micro) provides cost-effective performance for SSH tunneling while maintaining necessary capabilities.

To ensure high availability, we've enabled auto recovery features and integrated with Systems Manager for secure access without exposing SSH ports. The dedicated security group (sg-094968c8c57296426) implements strict access controls based on our security requirements.

## Storage Architecture

### EFS Implementation

Our EFS configuration through the `terraform-aws-efs` module prioritizes reliability and performance. The implementation spans multiple availability zones to ensure high availability, with at-rest encryption using KMS for data protection. We've selected General Purpose mode for balanced latency and throughput, while the bursting throughput mode provides cost-effective performance for variable workloads.

### EFS Integration with EKS

The integration between EFS and EKS is managed through the EFS CSI Driver, implemented as an EKS add-on for simplified updates and maintenance. A dedicated service account handles access control, while automatic volume provisioning reduces operational overhead. Our storage class configuration emphasizes immediate binding to ensure resource availability when needed, with a "Retain" policy preventing accidental data loss.

## Key Management

### KMS Configuration

Our KMS implementation through `terraform-aws-kms` focuses on maintaining robust security and compliance standards. For EKS secrets encryption, we've implemented annual key rotation and comprehensive audit logging. The EFS encryption configuration ensures automatic encryption of all data at rest, with detailed logging of encryption operations and automated key lifecycle management.

## State Management

### Terraform State Configuration

Our state management strategy prioritizes security and collaboration through a carefully structured S3 backend:

```hcl
remote_state {
  backend = "s3"
  config = {
    encrypt = true
    bucket  = "${local.env}terragrunt-tf-state-${local.account_name}-${local.aws_region}"
    key     = "${path_relative_to_include()}/tf.tfstate"
    region  = local.aws_region
  }
}
```

The S3 backend was chosen for its durability and availability characteristics. We enforce mandatory encryption for all state files and implement environment-specific buckets to prevent state file conflicts. Our path-based key structure maintains clear organization of state files.

State security is implemented through multiple layers, including server-side encryption for data at rest and KMS integration for sensitive state data. Access controls combine IAM policies with bucket policies, while comprehensive logging tracks all access and changes to state files.

## Monitoring and Compliance

### Resource Tagging

Our resource tagging strategy enables effective resource tracking and cost allocation:

```hcl
tags = {
  Environment = "production"
  ManagedBy   = "terragrunt"
  Project     = "numinia"
}
```

These tags serve multiple purposes: identifying resource context for environment-specific policies, tracking infrastructure-as-code managed resources, and enabling project-based cost allocation.

### Logging Configuration

Our logging implementation includes comprehensive VPC Flow Logs integrated with CloudWatch for centralized management. We maintain a 30-day retention period, balancing compliance needs with cost considerations. AWS CloudTrail provides a complete audit trail across all regions, with log validation ensuring integrity and encryption protecting audit data.

## Deployment Guidelines

### Prerequisites

Before deploying infrastructure changes, proper AWS authentication must be configured:

```bash
aws configure
# Configure with appropriate credentials
```

The deployment process requires specific tool versions:
```bash
terraform version # Must be compatible with AWS provider 5.87.0
terragrunt version
```

### Deployment Process

Our standardized deployment process ensures consistency and reliability:

```bash
cd infrastructure/production/eu-west-1/numinia
terragrunt init
terragrunt plan
terragrunt apply
```

This process includes initialization of the backend and provider downloads, preview and validation of infrastructure changes, and controlled implementation of validated changes.

For specific component deployments and modifications, refer to the respective module documentation in the [numinia-terragrunt](https://github.com/numengames/numinia-terragrunt) repository.