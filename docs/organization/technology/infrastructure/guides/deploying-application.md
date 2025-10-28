---
sidebar_position: 2
id: deploying-application
title: Deploying a New Application
sidebar_label: Deploying Application
---

# Deploying a New Application

This guide details the process of deploying applications (Hyperfy2 spaces, ElizaOS agents, or custom applications) within an existing organization. Our infrastructure uses Helm charts for application deployment with KNative for serverless capabilities.

## Prerequisites

Before deploying an application, ensure you have:

- An existing organization set up in the cluster (see [Creating Organization](creating-organization))
- AWS Secrets created for the application in Secrets Manager
- S3 folders created for the application (if using Hyperfy2)
- **Database schema created** (for Hyperfy2 applications) - See Step 1 below
- Domain DNS configured to point to the load balancer
- Access to our [numinia-k8s](https://github.com/numengames/numinia-k8s) repository
- The application's Docker image available

## Understanding the Deployment Structure

Each application deployment follows a standardized structure that supports both pre-production and production environments:

```
organizations/<org-name>/
└── <app-name>/                # Application name (e.g., "world", "eliza", etc.)
    ├── pre/                   # Pre-production environment
    │   ├── configmap-values.yaml
    │   ├── helmrelease.yaml
    │   └── kustomization.yaml
    └── pro/                   # Production environment
        ├── configmap-values.yaml
        ├── helmrelease.yaml
        └── kustomization.yaml
```

## Step 1: Create Database Schema (for Hyperfy2 Applications)

For Hyperfy2 applications, you need to create a dedicated database schema for each space in the shared RDS PostgreSQL database. The application will automatically create all necessary tables once it starts.

Connect to the RDS PostgreSQL database and create the schema:

```sql
-- Create schema for the application
CREATE SCHEMA IF NOT EXISTS <app-name>_<org-name>_<environment>;
```

Important notes:
- **Single RDS Instance**: All organizations and applications use the same RDS PostgreSQL instance
- **Schema Naming Convention**: `<app-name>_<org-name>_<environment>` (e.g., `world_r3s3t_pro`)
- **Automatic Table Creation**: The application automatically creates all required tables on first startup
- **Separate Schemas**: Each application environment gets its own schema for isolation
- **Connection Details**: Connection strings are stored in AWS Secrets Manager and accessed via service accounts
- **Existing Permissions**: The database user already has necessary permissions on the RDS instance

The DevOps team manages the RDS instance and access permissions. You only need to create the schema; the application handles the rest automatically.

## Step 2: Create Application Directory Structure

Create the directory structure for your application within the organization:

```bash
cd numinia-k8s/clusters/prod-cluster/organizations/<org-name>
mkdir -p <app-name>/pre
mkdir -p <app-name>/pro
```

## Step 3: Create ConfigMap Values

For each environment (pre/pro), create a `configmap-values.yaml` file that defines the application configuration:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: <app-name>-<org-name>-<environment>-values
  namespace: <org-name>-<environment>
  annotations:
    config-version: "1.0.0"
data:
  values.yaml: |
    awsSecretName: "hyperfy2-<org-name>-<app-name>-<environment>"
    orgName: "<org-name>"
    appName: "<app-name>"
    environment: "<environment>"
    namespace: <org-name>-<environment>
    serviceAccountName: "<org-name>-<environment>"
    world: "<app-name>"
    publicMaxUploadSize: "2000"
    publicWsUrl: "wss://<domain>/ws"
    publicApiUrl: "https://<domain>/api"
    assetsBaseUrl: "https://statics.numinia.xyz/hyperfy-spaces/<org-name>/<app-name>/<environment>/assets"
    clean: "true"
    tlsSecretName: "<org-name>-tls"
    resources:
      requests:
        cpu: "256m"
        memory: "256Mi"
      limits:
        cpu: "2048m"
        memory: "4096Mi"
    autoscaling:
      minScale: 0
      maxScale: 1
    domainMapping:
      enabled: true
      hostname: <domain>
      namespace: <org-name>-<environment>
```

Key configuration values:
- **awsSecretName**: Name of the secret in AWS Secrets Manager containing credentials
- **serviceAccountName**: Service account that provides IAM permissions for AWS resources
- **assetsBaseUrl**: S3 URL where the application stores assets (for Hyperfy2)
- **autoscaling**: KNative scale-to-zero configuration (minScale: 0, maxScale: 1)
- **domainMapping**: KNative domain mapping for automatic routing

## Step 4: Create HelmRelease

Create the `helmrelease.yaml` file that defines the Helm chart deployment:

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: <app-name>-<environment>
  namespace: <org-name>-<environment>
spec:
  interval: 5m
  releaseName: <app-name>-<environment>
  targetNamespace: <org-name>-<environment>
  install:
    remediation:
      retries: 3
  upgrade:
    remediation:
      retries: 3
  chart:
    spec:
      chart: ./clusters/prod-cluster/base/hyperfy2-knative
      sourceRef:
        kind: GitRepository
        name: flux-system-v0.4.7
        namespace: flux-system
      interval: 1m
      version: "0.4.7"
  valuesFrom:
    - kind: ConfigMap
      name: <app-name>-<org-name>-<environment>-values
      valuesKey: values.yaml
```

This HelmRelease:
- Uses the `hyperfy2-knative` Helm chart for KNative-based deployments
- References the ConfigMap values created in Step 3
- Automatic retry and remediation on failures
- Flux CD continuously monitors and reconciles the release

## Step 5: Create Kustomization

Create the `kustomization.yaml` file that ties everything together:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - configmap-values.yaml
  - helmrelease.yaml
```

This simple configuration includes both the ConfigMap and HelmRelease resources.

## Step 6: Register Application in Organization Kustomization

Register the application in the organization's kustomization files for each environment:

**For pre-production** (`pre-kustomization.yaml`):
```yaml
resources:
  - <app-name>/pre
```

**For production** (`pro-kustomization.yaml`):
```yaml
resources:
  - <app-name>/pro
```

This registration ensures Flux CD picks up and manages your application deployment.

## Deployment Process

Once all configuration files are created:

### 1. Verify Prerequisites

Ensure all prerequisites are met:
- **Database schema created** in PostgreSQL (for Hyperfy2 applications)
- AWS Secret exists in Secrets Manager with the correct name and database connection string
- S3 folders are created (for Hyperfy2 applications)
- DNS is configured to point to load balancer
- Organization is set up and registered

### 2. Commit and Push

Commit your changes to the repository:

```bash
git add organizations/<org-name>/<app-name>
git add organizations/<org-name>/pre-kustomization.yaml
git add organizations/<org-name>/pro-kustomization.yaml
git commit -m "feat: add <app-name> application to <org-name>"
git push origin main
```

### 3. Monitor Deployment

Flux CD will automatically detect the changes within 1 minute and deploy the application:

```bash
# Check Helm release status
flux get helmrelease -n <org-name>-pro

# Check application resources
kubectl get ksvc,deployment,svc -n <org-name>-pro

# Check KNative domain mapping
kubectl get domainmapping -n <org-name>-pro

# View application logs
kubectl logs -n <org-name>-pro -l serving.knative.dev/service=<app-name>-pro
```

### 4. Verify Scaling Behavior

With KNative configured (minScale: 0, maxScale: 1), verify the scaling behavior:

```bash
# Check when application scales to zero
kubectl get pods -n <org-name>-pro -w

# Access the application to trigger scale-up
curl https://<domain>/api

# Watch the pod start automatically (cold start takes a few seconds)
```

## Serverless Behavior with KNative

One of the key benefits of this deployment approach is automatic scale-to-zero:

- **Idle State**: Application scales to zero pods when no traffic arrives
- **On Demand**: First request triggers pod creation (cold start ~2-5 seconds)
- **Scaling**: Handles traffic up to the configured maxScale limit
- **Cost**: Only pay for resources consumed during active traffic

This is particularly valuable for web3 spaces and AI applications with intermittent traffic patterns.

## Application Types

### Hyperfy2 Spaces

For Hyperfy2 applications:
- Use `hyperfy2-knative` Helm chart
- Configure `assetsBaseUrl` to point to S3 folder
- Include S3 write permissions in the service account IAM role
- Typical resources: 256m-512Mi requests, 2048m-4096Mi limits

### ElizaOS Agents

For ElizaOS deployments:
- Use the appropriate ElizaOS Helm chart
- Configure AI/LLM API credentials in AWS Secrets
- May require different resource profiles based on model size

### Custom Applications

For other applications:
- Use or create appropriate Helm charts
- Ensure application supports KNative/Kourier for scale-to-zero
- Configure environment-specific settings in ConfigMap values

## Troubleshooting

### Application Not Starting

If the application fails to start:
1. Check Helm release status: `flux get helmrelease -n <org-name>-pro`
2. Check KNative Service status: `kubectl describe ksvc <app-name>-pro -n <org-name>-pro`
3. Review logs: `kubectl logs -n <org-name>-pro -l serving.knative.dev/service=<app-name>-pro`

### Certificate Issues

If SSL certificates are not provisioning:
1. Verify DNS is configured correctly
2. Check cert-manager logs: `kubectl logs -n cert-manager -l app=cert-manager`
3. Verify ClusterIssuer exists and is properly configured

### Cold Start Delays

If applications take too long to start:
1. Adjust KNative scale-down delay in ConfigMap
2. Consider setting minScale: 1 for frequently accessed applications
3. Optimize container image startup time

### AWS Secrets Access Issues

If application cannot access secrets:
1. Verify service account has correct IAM role annotation
2. Check IAM policies allow access to the specific secret ARN
3. Verify secret name matches the awsSecretName in ConfigMap

## Best Practices

When deploying applications:

- **Use consistent naming**: Follow the `<app-name>-<org-name>-<environment>` pattern
- **Configure proper resources**: Right-size based on application requirements
- **Enable scale-to-zero**: Use minScale: 0 for cost optimization on idle applications
- **Separate environments**: Maintain clear separation between pre and pro
- **Version control**: Use Git for all configuration changes
- **Monitor cold starts**: Balance between scale-to-zero and user experience
- **Regular updates**: Update Helm chart versions and configurations regularly

## Important Considerations

1. **KNative and Kourier**: Applications must be compatible with KNative/Kourier for automatic scaling
2. **Service Accounts**: Ensure service accounts have appropriate IAM roles for AWS resource access
3. **DNS Propagation**: Allow time for DNS changes to propagate before deploying
4. **Cold Starts**: Be aware of the 2-5 second cold start delay when scaled to zero
5. **Resource Limits**: Set appropriate limits to prevent resource contention
6. **Secrets Management**: Never commit secrets to Git; use AWS Secrets Manager
7. **GitOps Workflow**: All changes go through Git; Flux CD manages deployment
8. **Test in Pre**: Always test deployments in pre-production before production

## Related Documentation

- [Creating Organization](../guides/creating-organization.md) - Setting up a new organization
- [Database Access](../guides/database-access.md) - How applications connect to databases
- [SSL Certificates](../guides/ssl-certificates.md) - Certificate management
- [Kubernetes Platform](../platform/kubernetes.md) - KNative and serverless architecture
