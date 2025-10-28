---
sidebar_position: 1
id: creating-organization
title: Creating a New Organization
sidebar_label: Creating Organization
---

# Creating a New Organization

This guide details the process of setting up a new organization within our infrastructure. Each organization operates in complete isolation within our Kubernetes cluster while sharing the underlying infrastructure resources. This multi-tenant architecture allows us to efficiently scale the platform while maintaining security and resource boundaries.

## Prerequisites

Before creating a new organization, ensure you have:

- Access to our Kubernetes cluster via VPN
- Access to our [numinia-k8s](https://github.com/numengames/numinia-k8s) repository
- Admin access to our [numinia-terragrunt](https://github.com/numengames/numinia-terragrunt) repository
- The organization's domain information
- Access to AWS Console for IAM and Secrets Manager
- Required database credentials from the DevOps team (for RDS access)

## Understanding the Organization Structure

Each organization in our system represents a customer who can deploy multiple applications and services within their placed resources. Organizations like `numen-games`, `active-inference`, `art-unchained`, `numinia`, and `numinia` serve as isolated tenants within our shared cluster infrastructure.

An organization receives:
- **Dedicated Kubernetes Namespaces**: Separate namespaces for pre-production and production environments
- **Custom Domain**: Each organization manages its own domain through Route53
- **SSL Certificates**: Automatic TLS provisioning via cert-manager
- **IAM Role and Policies**: Service account with IAM role for AWS resource access
- **Database Access**: Connection strings for RDS PostgreSQL
- **Resource Quotas**: Defined limits on CPU, memory, and storage

## Step 1: Create Organization Directory Structure

Start by creating the directory structure for the new organization in the `numinia-k8s` repository. You can copy an existing organization (like `numinia`) and modify it:

```bash
cd numinia-k8s/clusters/prod-cluster/organizations
cp -r numinia <org-name>
cd <org-name>
```

Remove existing space directories but keep the structure:
- Keep `kustomization.yaml`
- Keep `flux-kustomization.yaml`
- Keep `pre-kustomization.yaml` and `pro-kustomization.yaml` directories
- Keep `org-config/` directory
- Remove specific space directories (like `world/`, `vic/`, etc.) - you'll add your spaces later

## Step 2: Create IAM Role and Policies

Each organization needs a dedicated IAM role with policies for accessing AWS resources. This role will be used by the organization's service accounts to access Secrets Manager and S3.

### 2.1. Create the IAM Role

Create a new IAM role in AWS Console named `<org-name>-secret-manager-role`. Use the same trust relationship as an existing organization role:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<account-id>:oidc-provider/oidc.eks.eu-west-1.amazonaws.com/id/<oidc-id>"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "oidc.eks.eu-west-1.amazonaws.com/id/<oidc-id>:aud": "sts.amazonaws.com",
          "oidc.eks.eu-west-1.amazonaws.com/id/<oidc-id>:sub": "system:serviceaccount:<org-name>:<org-name>"
        }
      }
    }
  ]
}
```

### 2.2. Create IAM Policies

Create two policies for the organization - one for pre-production and one for production:

**Pre-production policy** (`<org-name>-pre-secret-manager-policy`):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue",
        "secretsmanager:DescribeSecret",
        "secretsmanager:ListSecrets"
      ],
      "Resource": [
        "arn:aws:secretsmanager:eu-west-1:<account-id>:secret:hyperfy2-<org-name>-*-pre-*"
      ]
    }
  ]
}
```

**Production policy** (`<org-name>-pro-secret-manager-policy`):
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "secretsmanager:GetSecretValue",
        "secretsmanager:DescribeSecret",
        "secretsmanager:ListSecrets"
      ],
      "Resource": [
        "arn:aws:secretsmanager:eu-west-1:<account-id>:secret:hyperfy2-<org-name>-*-pro-*"
      ]
    }
  ]
}
```

### 2.3. Attach S3 Bucket Policy

Hyperfy2 applications need write access to S3. Attach the existing S3 bucket policy (e.g., `numinia-statics-bucket`) to the organization's IAM role.

### 2.4. Attach Policies to Role

Attach both secret manager policies and the S3 bucket policy to the IAM role.

## Step 3: Create AWS Secrets (for Hyperfy2 Applications)

If you're deploying Hyperfy2 applications for this organization, create the necessary secrets in AWS Secrets Manager.

### 3.1. Create Secrets

For each Hyperfy2 space you plan to deploy, create a secret following the pattern:
- Pre-production: `hyperfy2-<org-name>-<space-name>-pre`
- Production: `hyperfy2-<org-name>-<space-name>-pro`

Each secret should contain:
- JWT_SECRET
- ADMIN_CODE
- Any other application-specific credentials

### 3.2. Associate Secrets with Policy

After creating the secrets, verify they are accessible by the IAM policies you created. The policy ARN patterns (`hyperfy2-<org-name>-*-pre-*` and `hyperfy2-<org-name>-*-pro-*`) will automatically grant access to secrets matching these patterns.

## Step 4: Create S3 Folder Structure (for Hyperfy2)

Create the organization's folder structure in the S3 bucket:

```
hyperfy-spaces/<org-name>/<space-name>/<environment>/assets
```

For each space, you'll need folders for:
- Pre-production environment (typically `dev` folder)
- Production environment (typically `latest` folder)

Example:
```
hyperfy-spaces/myorg/world/dev/assets  (for pre-production)
hyperfy-spaces/myorg/world/latest/assets  (for production)
```

## Step 5: Configure Domain DNS and SSL Certificate

### 5.1. Point Domain Subdomain to Load Balancer

Before cert-manager can generate valid SSL certificates, you must configure the domain's DNS to point the subdomain to our Kubernetes load balancer. This step is essential for Let's Encrypt certificate validation.

**Important**: Point the organization's subdomain to the load balancer in your DNS hosting provider (Route53, Cloudflare, etc.). This allows cert-manager to successfully validate the domain and generate a valid SSL certificate through the HTTP-01 challenge.

### 5.2. Configure SSL Certificate in Cluster

Once the DNS is properly configured, declare the SSL certificate for the organization's domain in the cluster-issuer configuration:

Navigate to `infrastructure/networking/cert-manager/cluster-issuer.yaml` in the numinia-k8s repository and add a new ClusterIssuer for the organization:

```yaml
---
apiVersion: cert-manager.io/v1
kind: ClusterIssuer
metadata:
  name: letsencrypt-<org-name>-prod
spec:
  acme:
    server: https://acme-v02.api.letsencrypt.org/directory
    email: "notifications@numengames.com"
    privateKeySecretRef:
      name: letsencrypt-<org-name>-prod
    solvers:
    - http01:
        ingress:
          class: traefik
```

## Step 6: Update Kubernetes Configuration

### 6.1. Update Organization Directories

In the `<org-name>` directory you copied, update the following files:

**Namespace configuration** (`namespace.yaml` or in `org-config/`):
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: <org-name>
  labels:
    organization: <org-name>
    environment: production
```

**Service Account** (`org-config/overlays/service-account.yaml`):
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: <org-name>
  namespace: <org-name>
  annotations:
    eks.amazonaws.com/role-arn: arn:aws:iam::<account-id>:role/<org-name>-secret-manager-role
```

### 6.2. Register Organization in Main Kustomization

Add the organization to the main kustomization file:

In `clusters/prod-cluster/kustomization.yaml`, add:
```yaml
resources:
  - organizations/<org-name>
```

## Step 7: Deploy Applications to the Organization

Once the organization is set up with IAM roles, secrets, S3 folders, and SSL certificates, you can deploy applications to it. 

For detailed instructions on deploying applications (including Hyperfy2 spaces, ElizaOS agents, or custom applications), see the [Deploying an Application](deploying-application) guide.

The deployment guide covers:
- Creating application directory structure
- Configuring ConfigMaps with environment variables  
- Creating HelmRelease resources for Helm chart deployment
- Setting up Kustomization files for configuration management
- Registering applications in organization kustomization files
- Automatic deployment via Flux CD GitOps workflow

Example structure for a Hyperfy2 space:
```
<org-name>/
└── <space-name>/
    ├── pre/
    │   ├── configmap-values.yaml
    │   ├── helmrelease.yaml
    │   └── kustomization.yaml
    └── pro/
        ├── configmap-values.yaml
        ├── helmrelease.yaml
        └── kustomization.yaml
```

## Step 8: Commit and Deploy

After completing all configuration steps:

1. Commit your changes to the numinia-k8s repository
2. Flux CD will automatically detect the changes (within 1 minute)
3. The organization and spaces will be deployed automatically

```bash
git add organizations/<org-name>
git commit -m "feat: add <org-name> organization"
git push origin main
```

## Step 9: Verify Deployment

Verify that everything is working correctly:

```bash
# Check namespace creation
kubectl get namespace <org-name>

# Check service account
kubectl get serviceaccount <org-name> -n <org-name>

# Check Helm releases
flux get helmrelease -n <org-name>
```

## Troubleshooting

### Organization Not Appearing in Flux

If the organization is not being synced by Flux:
1. Check Flux logs: `flux logs -n flux-system`
2. Verify the kustomization path is correct
3. Ensure the directory structure exists in the repository

### Service Account Not Working

If the service account cannot access Secrets Manager:
1. Verify IAM role ARN is correct
2. Check IAM policy permissions for Secrets Manager
3. Ensure IRSA is properly configured on the EKS cluster

### SSL Certificates Not Provisioning

If certificates are not being created:
1. Check cert-manager logs: `kubectl logs -n cert-manager -l app=cert-manager`
2. Verify DNS records are correct in Route53
3. Check ClusterIssuer configuration

## Next Steps

Once your organization is set up:
- Deploy additional spaces by following the Hyperfy2 space configuration
- Configure domain mapping for your applications
- Set up monitoring and logging for the organization
- Review and adjust resource quotas as needed

For questions or issues, contact the infrastructure team.
