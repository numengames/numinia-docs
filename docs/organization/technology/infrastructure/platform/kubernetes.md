---
sidebar_position: 4
id: kubernetes-platform
title: Kubernetes Platform
sidebar_label: Kubernetes Platform
---

# Kubernetes Platform

Our Kubernetes infrastructure operates on Amazon EKS (Elastic Kubernetes Service), which provides managed Kubernetes hosting for all our containerized applications. We've embraced GitOps principles through Flux CD, ensuring our Git repository serves as the single source of truth for our cluster's state.

## Cluster Architecture

### GitOps Structure

We maintain all our Kubernetes configurations in our [numinia-k8s](https://github.com/numengames/numinia-k8s) repository, which tracks changes, enables review processes, and maintains a complete history of how our infrastructure has evolved over time.

The repository follows this structure:

```
clusters/
└── prod-cluster/
    ├── flux-system/         # Flux core components
    ├── flux-kustomization/  # Custom configurations
    ├── infrastructure/      # Cluster-wide resources
    └── organizations/       # Organization-specific deployments
        ├── active-inference/
        ├── numen-games/
        ├── numinia/
        └── r3s3t/
```

This structure reflects our commitment to clear separation of concerns. The `flux-system` directory contains everything needed for GitOps automation. The `infrastructure` directory houses shared cluster resources, while organization-specific configurations maintain clear boundaries between different teams' resources.

We implement Kustomize for configuration management, allowing us to maintain a single source of truth while adapting configurations for different environments or requirements without code duplication.

### Flux CD Configuration

Flux CD continuously monitors our Git repository and automatically applies changes to our cluster. The core configuration uses a one-minute sync interval, ensuring rapid updates when changes are pushed to the repository. This creates a tight feedback loop between code and running infrastructure.

The GitRepository resource is configured to monitor the main branch of our numinia-k8s repository, enabling automatic synchronization of all configurations.

## Kubernetes and AWS Integration

Our EKS cluster integrates deeply with AWS services through several mechanisms. The EFS CSI Driver add-on provides access to shared storage, although we've migrated to external storage (S3 and RDS). Service accounts with IRSA (IAM Roles for Service Accounts) allow pods to access AWS resources like Secrets Manager and S3 without storing credentials.

The cluster operates in private subnets of our VPC, ensuring that workloads have no direct internet access. All ingress traffic flows through load balancers in public subnets, and egress traffic passes through NAT gateways.

Access to the cluster has evolved from a bastion host to a VPN service, providing more flexible and secure remote access. This VPN allows authorized users to connect and manage the cluster through kubectl commands.

## Multi-Tenant Architecture

Our cluster architecture supports multiple organizations while maintaining strict isolation. We achieve secure multi-tenancy through a combination of namespace isolation, resource quotas, and network policies.

Each organization operates within its own namespace with carefully defined resource limits and network boundaries. This isolation ensures that organizations can't interfere with each other's workloads while sharing the same cluster infrastructure.

Organizations follow a consistent structure where base configurations, applications, and organization-specific infrastructure are clearly separated. This organization supports our customers' diverse needs while maintaining operational simplicity for our team.

For detailed information on the multi-tenant design and how organizations are created, see the [Creating Organization](../guides/creating-organization.md) guide.

## Serverless Architecture with KNative

Our platform leverages KNative to provide serverless capabilities for all deployed applications. KNative allows us to scale applications from zero to many instances automatically based on demand, and scale back down to zero when not in use. This serverless approach is critical for our multi-tenant architecture where hundreds of applications may be deployed simultaneously.

### Cost Optimization Through Scale-to-Zero

Traditional Kubernetes deployments keep pods running continuously, consuming resources and incurring costs even when idle. With KNative, applications scale to zero pods when they receive no traffic, consuming zero compute resources. When traffic arrives, KNative automatically creates pods within seconds to handle the request. This enables us to host hundreds of applications at the cost of only the applications currently serving traffic.

### KNative and Kourier Implementation

We use KNative Serving for automatic scaling and Kourier as our lightweight ingress gateway. Kourier is specifically designed for KNative and provides efficient routing, load balancing, and SSL/TLS termination without the overhead of a full ingress controller. This combination delivers:

- **Automatic scaling**: Pods scale from 0 to configured limits based on request volume
- **Fast cold starts**: Applications become available within seconds when scaled up from zero
- **Request-based lifecycle**: Applications only consume resources when actively serving traffic
- **Multi-tenant isolation**: Each organization's applications scale independently

This architecture is particularly valuable for web3 spaces and AI applications that may experience intermittent traffic patterns. It allows us to efficiently support multiple customers without provisioning dedicated infrastructure for each application.

## External Services Integration

Applications running in our Kubernetes cluster access AWS services through well-defined patterns. S3 is accessed via connection credentials stored in AWS Secrets Manager, with service account IAM roles providing appropriate permissions. RDS databases are connected using connection strings from Secrets Manager, ensuring each organization has isolated access to their designated database resources.

This externalized approach provides several benefits: simplified cluster management, reduced Kubernetes complexity, independent storage scaling, and robust backup and disaster recovery capabilities managed by AWS.

For detailed information on how applications access databases, see the [Database Access](../guides/database-access.md) guide.

## Security and Operations

Security is implemented through multiple complementary layers. RBAC configuration ensures proper access management through service accounts, each with limited scope. We integrate closely with AWS IAM to maintain consistent access controls across our entire infrastructure.

Network policies start from a default deny-all stance, with explicit allow rules for necessary communication paths. Secret management uses AWS Secrets Manager for secure storage and automated rotation, accessed through service accounts configured with appropriate IAM roles.

## Application Deployment

Application deployment follows our GitOps workflow. Changes committed to the numinia-k8s repository are automatically detected by Flux CD and applied to the cluster. This ensures consistency, traceability, and rapid deployment without manual intervention.

For practical deployment guides, see:
- [Creating a New Organization](../guides/creating-organization.md)
- [Deploying a New Application](../guides/deploying-application.md)
- [Database Access Patterns](../guides/database-access.md)

## Serverless Benefits for Multi-Tenant Platform

The combination of KNative's scale-to-zero capabilities with our multi-tenant architecture creates a highly cost-effective platform. Organizations can deploy multiple applications (Hyperfy2 spaces, ElizaOS agents, or custom services) without worrying about infrastructure costs when those applications are idle. This model enables true pay-per-use pricing where organizations only pay for the resources consumed during active traffic periods, making it economically viable to support hundreds of applications across dozens of organizations on shared cluster infrastructure.

## Related Documentation

- [AWS Services](../aws-services.md) - AWS infrastructure that hosts the cluster
- [Terragrunt IaC](../implementation/terragrunt.md) - How the EKS cluster is provisioned
- [Infrastructure Overview](../overview.md) - Complete infrastructure architecture overview
- [Creating Organization](../guides/creating-organization.md) - Setting up new organizations
- [Deploying Application](../guides/deploying-application.md) - Application deployment process

