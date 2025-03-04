---
sidebar_position: 2
id: terragrunt
title: Terragrunt Implementation
sidebar_label: Terragrunt
---

# Terragrunt Implementation

Our infrastructure management leverages Terragrunt to provide a DRY (Don't Repeat Yourself) approach to Terraform configurations. This document details our implementation strategy and the best practices we follow to maintain a scalable and maintainable infrastructure.

## Project Structure

Our infrastructure code resides in the [numinia-terragrunt](https://github.com/numengames/numinia-terragrunt) repository, organized in a clear and logical structure that reflects our deployment hierarchy:

```
infrastructure/
├── production/                 # Production environment
│   ├── account.hcl            # Account-specific configuration
│   └── eu-west-1/             # Regional resources
│       └── numinia/           # Organization resources
│           ├── vpc/           # Network infrastructure
│           ├── eks/           # Kubernetes cluster
│           └── efs/           # Shared storage
├── .env                       # Environment variables
└── root.hcl                   # Base configuration
```

This structure embodies several key design principles. First, we maintain a clear separation between environments, ensuring production configurations are isolated and protected. Second, our regional organization creates clear boundaries for resources, making it easier to manage multi-region deployments. Finally, each service component is isolated in its own directory, promoting modularity and maintainability.

## Core Configuration Files

### Root Configuration (root.hcl)

The `root.hcl` file serves as the foundation of our Terragrunt configuration, defining common settings and state management:

```hcl
remote_state {
  backend = "s3"
  config = {
    bucket         = "numinia-terraform-state"
    key            = "${path_relative_to_include()}/terraform.tfstate"
    region         = "eu-west-1"
    encrypt        = true
    dynamodb_table = "terraform-locks"
  }
}

terraform {
  extra_arguments "common_vars" {
    commands = get_terraform_commands_that_need_vars()
    arguments = [
      "-var-file=${get_terragrunt_dir()}/../../../common.tfvars"
    ]
  }
}
```

This configuration establishes our state management strategy, utilizing S3 for centralized state storage with encryption enabled. We've implemented state locking through DynamoDB to prevent concurrent modifications, and our common variables are shared across all modules for consistency.

### Environment Variables (.env)

Our environment-specific configurations are managed through environment variables:

```shell
AWS_PROFILE=numinia-prod
AWS_REGION=eu-west-1
ENVIRONMENT=production
```

This approach allows us to maintain separate AWS profiles for different environments, ensuring clear separation of access and configuration. The regional settings provide default values for deployment, while the environment context helps identify the deployment target.

## Core Modules

### VPC Module

Our VPC module implements the network infrastructure using the official AWS VPC module:

```hcl
include "root" {
  path = find_in_parent_folders("root.hcl")
}

terraform {
  source = "tfr:///terraform-aws-modules/vpc/aws?version=5.0.0"
}

inputs = {
  name = "numinia-vpc"
  cidr = "10.0.0.0/16"
  
  azs             = ["eu-west-1a", "eu-west-1b", "eu-west-1c"]
  private_subnets = ["10.0.1.0/24", "10.0.2.0/24", "10.0.3.0/24"]
  public_subnets  = ["10.0.101.0/24", "10.0.102.0/24", "10.0.103.0/24"]
  
  enable_nat_gateway = true
  single_nat_gateway = false
  
  enable_vpn_gateway = false
  
  tags = {
    Environment = "production"
    Project     = "numinia"
  }
}
```

### EKS Module

The EKS module manages our Kubernetes cluster deployment, carefully configured to meet our scaling and security requirements:

```hcl
include "root" {
  path = find_in_parent_folders("root.hcl")
}

terraform {
  source = "tfr:///terraform-aws-modules/eks/aws?version=19.0.0"
}

dependency "vpc" {
  config_path = "../vpc"
}

inputs = {
  cluster_name    = "numinia-cluster"
  cluster_version = "1.27"
  
  vpc_id     = dependency.vpc.outputs.vpc_id
  subnet_ids = dependency.vpc.outputs.private_subnets
  
  eks_managed_node_groups = {
    main = {
      min_size     = 1
      max_size     = 5
      desired_size = 3
      
      instance_types = ["t3.medium"]
      capacity_type  = "ON_DEMAND"
    }
  }
}
```

### EFS Module

Our shared storage configuration leverages EFS for reliable and scalable file storage:

```hcl
include "root" {
  path = find_in_parent_folders("root.hcl")
}

terraform {
  source = "tfr:///terraform-aws-modules/efs/aws?version=1.0.0"
}

dependency "vpc" {
  config_path = "../vpc"
}

inputs = {
  name = "numinia-efs"
  
  vpc_id          = dependency.vpc.outputs.vpc_id
  subnet_ids      = dependency.vpc.outputs.private_subnets
  security_groups = [dependency.vpc.outputs.default_security_group_id]
  
  encrypted = true
  
  performance_mode = "generalPurpose"
  throughput_mode  = "bursting"
}
```

## State Management

Our state management approach ensures consistency and security across all infrastructure components. The S3 backend provides reliable state storage, while DynamoDB handles state locking to prevent concurrent modifications. We maintain state files per component, following a clear naming convention that reflects our infrastructure hierarchy.

Access to state files is strictly controlled through IAM roles and policies, with comprehensive audit logging enabled to track all access and modifications. We maintain regular backups of state files and implement version history to enable recovery if needed.

## Deployment Workflow

Our deployment process follows a standardized workflow to ensure consistency and reliability:

1. Environment setup and authentication:
   ```bash
   # Required tools
   terragrunt --version
   terraform --version
   aws --version
   ```

2. AWS authentication:
   ```bash
   export AWS_PROFILE=numinia-prod
   aws sts get-caller-identity
   ```

3. Infrastructure deployment:
   ```bash
   cd infrastructure/production/eu-west-1/numinia
   terragrunt run-all plan    # Review changes
   terragrunt run-all apply   # Apply changes
   ```

## Best Practices

Our infrastructure management follows several key principles to maintain quality and reliability. We use feature branches for all changes, ensuring proper review through pull requests. Our commit messages provide clear documentation of changes, and we maintain comprehensive documentation for all components.

State management includes regular backups and monitoring of the lock table to prevent issues. Security is maintained through proper encryption of sensitive data, use of IAM roles for access control, and regular rotation of security credentials.

For specific implementation details and the latest updates, refer to our [numinia-terragrunt](https://github.com/numengames/numinia-terragrunt) repository. 