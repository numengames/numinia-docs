---
sidebar_position: 2
id: aws-services
title: AWS Services
sidebar_label: AWS Services
---

# AWS Services

This document provides a comprehensive overview of all AWS services we use in our infrastructure. While these services are provisioned and managed through Terragrunt (for those we control), this document explains what each service does and how it fits into our overall architecture.

## Core Services

Our infrastructure relies on several key AWS services to provide a secure, scalable, and reliable platform for hosting web3 spaces and AI applications.

### Networking (VPC)

Our Virtual Private Cloud provides the foundational network layer that isolates and protects all our resources.

The VPC is configured with a CIDR block of `10.0.0.0/16`, providing 65,536 possible IP addresses. We deploy across three availability zones (eu-west-1a, eu-west-1b, eu-west-1c) to ensure high availability and fault tolerance. Within this VPC, we maintain strict separation between public and private subnets.

Private subnets (10.0.1.0/24, 10.0.2.0/24, 10.0.3.0/24) host all our workloads, including the EKS worker nodes and application pods. These subnets have no direct internet access, requiring traffic to pass through NAT gateways for outbound connectivity. This design protects our applications from direct internet exposure while still allowing necessary external communications.

Public subnets (10.0.101.0/24, 10.0.102.0/24, 10.0.103.0/24) host resources that require direct internet access, such as load balancers and our VPN instance. We've chosen the 10.0.101.0/24 range to make these easily distinguishable during network troubleshooting.

To optimize communication with AWS services, we extensively use VPC endpoints. These keep traffic within the AWS network, reducing latency and eliminating the need for internet access when communicating with services like S3, EKS API server, and CloudWatch Logs.

Network monitoring is comprehensive, with VPC flow logs enabled for security auditing and troubleshooting. These logs integrate with CloudWatch Logs and follow retention policies aligned with compliance requirements.

### Compute

Our compute infrastructure is built on Amazon EKS, which provides managed Kubernetes hosting for all our containerized applications. The EKS cluster runs Kubernetes version 1.32 and operates in private subnets for security.

The cluster uses EKS-managed node groups with t3.medium instances running Amazon Linux 2023. We configure auto-scaling from 4 to 10 nodes, with a desired state of 5 nodes. This provides sufficient capacity for normal operations while allowing dynamic scaling during peak loads.

Access to the cluster has evolved from a bastion host approach to a VPN service, which provides more flexible and secure remote access. The VPN runs on a small EC2 instance and integrates with our IAM roles for authentication.

For troubleshooting and legacy access, we maintain a t3.micro bastion host instance. This instance uses Amazon Linux 2 and integrates with Systems Manager for secure access without exposing SSH ports. The dedicated security group implements strict access controls based on our security requirements.

### Storage (S3)

We leverage Amazon S3 for object storage, particularly for static assets and application data. Our primary S3 bucket serves as the origin for our content delivery network.

The S3 bucket is configured with strict security measures. Public access is completely blocked at the bucket level, ensuring that content is only accessible through CloudFront. We enforce this through bucket policies that specifically allow access from our CloudFront distribution using Origin Access Control (OAC).

Server-side encryption using AES-256 protects all data at rest. Versioning is enabled to maintain file history and enable recovery from accidental deletions or modifications. We configure lifecycle policies to manage data retention and cost optimization.

For cross-origin access, we've configured CORS rules that allow GET and HEAD requests from any origin, exposing the ETag header for cache validation. These settings ensure that web applications can properly access static assets while maintaining security boundaries.

The bucket integrates seamlessly with CloudFront for global content distribution. All files uploaded to S3 become accessible through the CDN, providing low-latency access for users worldwide.

### Content Delivery (CloudFront)

CloudFront serves as our global content delivery network, distributing content from S3 to users with minimal latency. Our CloudFront distribution is configured with several advanced features.

Origin Access Control (OAC) replaces the legacy Origin Access Identity, providing a modern and secure method for CloudFront to access S3. The OAC is configured with sigv4 signing behavior, ensuring that all requests from CloudFront to S3 are properly authenticated.

Our configuration includes automatic HTTP to HTTPS redirects, enforcing secure connections for all users. We use a custom SSL certificate from ACM configured with SNI-only support and a minimum TLS version of 1.2.

Cache optimization is handled through AWS managed cache policies and origin request policies. The CachingOptimized policy reduces origin requests while the CORS-S3Origin policy handles cross-origin requests appropriately.

To support single-page applications, we've configured custom error responses that redirect 403 and 404 errors to index.html with a 200 status code. This ensures that client-side routing works correctly regardless of the requested path.

Price class optimization targets European traffic (PriceClass_100), balancing cost and performance for our primary user base.

### Database (RDS PostgreSQL)

Relational database needs are served by Amazon RDS with PostgreSQL. The RDS instances are managed by our DevOps team outside of the Terragrunt repository, but they integrate seamlessly with our overall infrastructure.

The database configuration emphasizes high availability through multi-AZ deployment, ensuring automatic failover if a primary database instance fails. Automated backups are configured with configurable retention policies, providing point-in-time recovery capabilities.

Security is implemented through VPC isolation and SSL/TLS encryption for all connections. The database resides in private subnets, accessible only from within the VPC. Applications connect using connection strings stored securely in AWS Secrets Manager.

Performance tuning is handled at the instance level, with appropriate sizing for production workloads. Vertical scaling capabilities allow us to adjust resources as workloads grow without service interruption.

Each organization within our multi-tenant platform has access to specific database resources through connection strings retrieved from Secrets Manager. This approach ensures proper isolation and security between organizations while providing seamless access within the Kubernetes cluster.

### Secrets Management (Secrets Manager)

AWS Secrets Manager serves as our centralized secret storage solution, replacing the previous KMS-based approach. This service provides secure storage, automatic rotation, and comprehensive audit logging.

Each organization has a service account configured with an IAM role that grants access to organization-specific secrets. The service account pattern allows us to maintain strict segregation between organizations while providing seamless access to required secrets within the Kubernetes cluster.

Secret access is logged for auditing purposes, providing a complete trail of who accessed which secrets and when. We configure automatic rotation for database passwords and API keys, reducing security risks from long-lived credentials.

Integration with Kubernetes is seamless through the External Secrets Operator. Service accounts in the cluster can retrieve secrets from Secrets Manager using IAM roles, with secrets mounted as files in pods or injected as environment variables.

The fine-grained access control ensures that each organization can only access its designated secrets, maintaining the multi-tenant isolation that's central to our architecture.

## DNS Management (Route53)

Route53 manages all DNS records for our domains, providing reliable DNS resolution with health checks and traffic routing capabilities.

Our Route53 hosted zone serves as the central point for DNS management. We configure alias records for CloudFront distributions, ensuring that DNS resolution points to the correct endpoints with proper health checking.

DNS configuration integrates with ACM for certificate validation, with automatic creation of validation records when certificates are provisioned. This automation reduces operational overhead and ensures certificates stay valid.

## Certificate Management (ACM)

AWS Certificate Manager handles all SSL/TLS certificates for our infrastructure. Certificates are issued by Let's Encrypt and managed automatically through certificate validation and renewal.

For CloudFront distributions, certificates must be in the us-east-1 region. We handle this through our Terragrunt configuration, which manages the cross-region resource deployment.

DNS validation is used for all certificates, with Route53 automatically creating the necessary validation records. This approach is more reliable than email validation and ensures certificates can be renewed automatically.

## Access Control (IAM)

Identity and Access Management controls all access to AWS resources. We implement role-based access control with carefully defined roles that follow the principle of least privilege.

Service accounts within our Kubernetes cluster are configured with IAM roles that provide fine-grained access to AWS resources. These roles are specific to each organization's needs, ensuring that workloads can access only the resources they require.

The bastion host maintains an IAM role with limited EKS permissions for cluster management and troubleshooting. This role has minimal required permissions for instance operations while integrating with Systems Manager for secure session management.

Security audits are comprehensive, with CloudTrail logging all API calls and access attempts. This audit trail is essential for compliance and security investigations.

## How Services Work Together

These AWS services form an integrated platform that supports our applications:

- **Network and Security**: VPC provides the foundation, with security groups and IAM controlling access
- **Compute**: EKS hosts our applications, with worker nodes in private subnets for security
- **Storage**: S3 stores static assets, which CloudFront distributes globally
- **Database**: RDS provides persistent data storage accessed through Secrets Manager credentials
- **Management**: Route53 handles DNS, ACM provides certificates, and Secrets Manager secures credentials

All services are provisioned and managed through Terragrunt in the [numinia-terragrunt](https://github.com/numengames/numinia-terragrunt) repository, with the exception of RDS which is managed directly by the DevOps team. This Infrastructure as Code approach ensures consistency, repeatability, and version control for all our infrastructure changes.

