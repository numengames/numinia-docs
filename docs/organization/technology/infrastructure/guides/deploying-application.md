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
- Access to our [numinia-k8s](https://github.com/numengames/numinia-k8s) repository
- Access to AWS Console (Secrets Manager, S3, EC2)
- Access to the organization's domain DNS management

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

## Step 2: Create S3 Folder Structure (for Hyperfy2 Applications)

Create the application's folder structure in the S3 bucket and populate it with the necessary assets.

### 2.1. Create the folder structure

In the `numinia-statics` S3 bucket, create the following path:

```
hyperfy-spaces/<org-name>/<app-name>/<environment>/assets
```

Repeat for each environment:

```
hyperfy-spaces/<org-name>/<app-name>/dev/assets     (pre-production)
hyperfy-spaces/<org-name>/<app-name>/latest/assets  (production)
```

### 2.2. Copy assets from a reference world

The folder must contain the required base assets before the application starts. Use an existing deployed world as a reference:

1. Navigate to another world's folder in the same bucket (e.g., `hyperfy-spaces/<org-name>/<reference-world>/`)
2. Copy all its contents into the new application's folder, preserving the folder structure
3. If the reference world has little content, the copied assets will be the minimal set required for the application to boot correctly

> **Note**: The schema and S3 structure must exist before creating the AWS Secret in the next step, as their values are required in the secret configuration.

## Step 3: Create AWS Secret in Secrets Manager

Each Hyperfy2 application requires a secret in AWS Secrets Manager containing its credentials and configuration. Create one secret per environment.

### 3.1. Open Secrets Manager

In the AWS Console, go to **Secrets Manager → Secrets → Store a new secret**.

### 3.2. Configure the secret

- **Secret type**: `Other type of secret`
- **Key/value pairs**: Use an existing secret from the same organization as reference (open it in a separate tab). Add the same set of keys, filling in the values for this application. Some keys correspond to services that may not be integrated yet — add those keys but leave their value empty.
- **Encryption key**: Leave as default (`aws/secretsmanager`)

Click **Next**.

### 3.3. Name the secret

Follow the naming convention:

```
hyperfy2-<org-name>-<app-name>-<env>
```

Examples:
- `hyperfy2-numen-games-world-pre`
- `hyperfy2-numen-games-world-pro`

Leave **Description**, **Tags**, and **Resource permissions** empty.

Click **Next**.

### 3.4. Disable automatic rotation

Leave **Automatic rotation** disabled.

Click **Next**, review the configuration, and **Store** the secret.

Repeat for each environment (pre and pro).

## Step 4: Configure DNS

The application's subdomain must be pointed to the cluster load balancer before the Kubernetes deployment is merged, so that cert-manager can complete the SSL certificate validation.

### 4.1. Find the load balancer address

The load balancer address is shared across all applications in the cluster. You can find it by:

- **Referencing existing DNS entries**: Check the DNS records already configured for other applications in the same organization — they all point to the same load balancer.
- **AWS Console**: Go to **EC2 → Load Balancers** and locate the cluster's load balancer to get its DNS name.

### 4.2. Add DNS records

In the organization's external DNS provider (not Route53), add **one record per environment**. Each environment has its own subdomain:

| Environment | Type | Name | Value |
|---|---|---|---|
| Pre-production | `CNAME` | `pre.<app-name>.<org-domain>` | Load balancer DNS name |
| Production | `CNAME` | `<app-name>.<org-domain>` | Load balancer DNS name |

Both records point to the **same** load balancer — environment routing is handled inside the cluster by KNative DomainMapping.

### 4.3. Wait for DNS propagation

After saving the DNS record, wait a few minutes for propagation before merging the Kubernetes changes in the next steps. Merging before DNS resolves correctly may cause cert-manager to fail the certificate challenge.

## Step 5: Create Application Directory Structure

Create the directory structure for your application within the organization:

```bash
cd numinia-k8s/clusters/prod-cluster/organizations/<org-name>
mkdir -p <app-name>/pre
mkdir -p <app-name>/pro
```

## Step 6: Create ConfigMap Values

For each environment (pre/pro), create a `configmap-values.yaml` file that defines the application configuration.

> **Naming conventions used below**
> - `<environment>` is `pre` or `pro` (used in resource names and the AWS Secret name).
> - `<assets-folder>` is `dev` for pre-production and `latest` for production (used only in the S3 path).
> - `<k8s-namespace>` is `<org-name>-pre` for pre and `<org-name>` (no suffix) for pro. The same value is used as `serviceAccountName`.
> - `<hostname>` is `pre.<app-name>.<org-domain>` for pre and `<app-name>.<org-domain>` for pro.

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: <app-name>-<environment>-values
  namespace: <k8s-namespace>
  annotations:
    config-version: "1.0.0"
data:
  values.yaml: |
    awsSecretName: "hyperfy2-<org-name>-<app-name>-<environment>"
    orgName: "<org-name>"
    appName: "<app-name>"
    environment: "<environment>"
    namespace: <k8s-namespace>
    serviceAccountName: "<k8s-namespace>"
    image:
      repository: "<image-repository>"
      tag: "<image-tag>"
    world: "<environment>-<app-name>"
    publicMaxUploadSize: "2000"
    publicWsUrl: "wss://<hostname>/ws"
    publicApiUrl: "https://<hostname>/api"
    assetsBaseUrl: "https://statics.numinia.xyz/hyperfy-spaces/<org-name>/<app-name>/<assets-folder>/assets"
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
      hostname: <hostname>
      namespace: <k8s-namespace>
```

Key configuration values:
- **awsSecretName**: Name of the secret in AWS Secrets Manager (Step 3). The chart uses this to create an `ExternalSecret` that materializes the secret as a Kubernetes secret consumed by the pod at runtime.
- **serviceAccountName**: Service account that provides IAM permissions (via IRSA) for accessing AWS Secrets Manager and S3. It is created at the organization level — see [Creating Organization](creating-organization).
- **image**: Container image for the application. Optional — if omitted, the chart's default is used. To pin a specific image and tag, copy the values from an existing application in the same organization.
- **world**: Internal Hyperfy2 world identifier. By convention it is prefixed with the environment (`pre-` or `pro-`) so logs and metrics are unambiguous across environments sharing the same database instance.
- **assetsBaseUrl**: S3 URL where the application reads/writes assets. The folder name (`dev`/`latest`) does **not** match the `<environment>` value (`pre`/`pro`) — keep this mapping in mind.
- **autoscaling**: KNative scale-to-zero configuration (minScale: 0, maxScale: 1).
- **domainMapping**: KNative DomainMapping that routes the public hostname to this service.

## Step 7: Create HelmRelease

Create the `helmrelease.yaml` file that defines the Helm chart deployment:

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: <app-name>-<environment>
  namespace: <k8s-namespace>
spec:
  interval: 5m
  releaseName: <app-name>-<environment>
  targetNamespace: <k8s-namespace>
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
        name: flux-system
        namespace: flux-system
      interval: 1m
  valuesFrom:
    - kind: ConfigMap
      name: <app-name>-<environment>-values
      valuesKey: values.yaml
```

> **Pinning a chart version**: If an application needs to stay on a specific chart revision (e.g., to avoid picking up changes pushed to the base chart), reference a versioned `GitRepository` (such as `flux-system-v0.4.7`) and add `version: "0.4.7"` under `chart.spec`. By default, omit both fields so the application tracks the live base chart.

This HelmRelease:
- Uses the `hyperfy2-knative` Helm chart for KNative-based deployments
- References the ConfigMap values created in Step 6
- Automatic retry and remediation on failures
- Flux CD continuously monitors and reconciles the release

## Step 8: Create Kustomization

Create the `kustomization.yaml` file that ties everything together:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - configmap-values.yaml
  - helmrelease.yaml
```

## Step 9: Register Application in Organization Kustomization

Register the application in the organization's kustomization files for each environment. Each environment has its own kustomization directory at the organization root: `pre-kustomization/` and `pro-kustomization/`. Add a new entry to the `resources` list inside each `kustomization.yaml`.

**For pre-production** (`organizations/<org-name>/pre-kustomization/kustomization.yaml`):
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: <org-name>-pre
resources:
  - ../org-config/overlays/pre
  # ... existing applications ...
  - ../<app-name>/pre
```

**For production** (`organizations/<org-name>/pro-kustomization/kustomization.yaml`):
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: <org-name>
resources:
  - ../org-config/overlays/pro
  # ... existing applications ...
  - ../<app-name>/pro
```

Resource paths are relative to the kustomization file (hence the `../` prefix). This registration ensures Flux CD picks up and manages your application deployment.

## Deployment Process

Once all configuration files are created:

### 1. Commit and Push

Commit your changes to the repository:

```bash
git add organizations/<org-name>/<app-name>
git add organizations/<org-name>/pre-kustomization/kustomization.yaml
git add organizations/<org-name>/pro-kustomization/kustomization.yaml
git commit -m "feat: add <app-name> application to <org-name>"
git push origin main
```

> **Note**: Ensure DNS propagation has completed (Step 4) before merging changes. Flux CD will begin deploying within 1 minute of the merge, and cert-manager will attempt SSL certificate validation immediately.

### 2. Monitor Deployment

Flux CD will automatically detect the changes and deploy the application. The commands below use `<k8s-namespace>` — replace it with `<org-name>-pre` for pre-production or `<org-name>` for production:

```bash
# Check Helm release status
flux get helmrelease -n <k8s-namespace>

# Check application resources
kubectl get ksvc,deployment,svc -n <k8s-namespace>

# Check KNative domain mapping
kubectl get domainmapping -n <k8s-namespace>

# View application logs
kubectl logs -n <k8s-namespace> -l serving.knative.dev/service=<app-name>-<environment>
```

Allow a few minutes after merging for the application to be fully operational.

### 3. Verify Scaling Behavior

With KNative configured (minScale: 0, maxScale: 1), verify the scaling behavior:

```bash
# Check when application scales to zero (idle pods terminate after the
# Knative scale-down window).
kubectl get pods -n <k8s-namespace> -w

# Trigger a cold start by opening the application URL in a browser
# (Hyperfy2 needs a websocket handshake — a plain GET is not enough).

# Watch the pod start automatically (cold start takes a few seconds).
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

If the application fails to start (replace `<k8s-namespace>` with `<org-name>-pre` for pre-production or `<org-name>` for production):
1. Check Helm release status: `flux get helmrelease -n <k8s-namespace>`
2. Check KNative Service status: `kubectl describe ksvc <app-name>-<environment> -n <k8s-namespace>`
3. Review logs: `kubectl logs -n <k8s-namespace> -l serving.knative.dev/service=<app-name>-<environment>`
4. Identify the failing component, apply the fix, and trigger a reconciliation:
   ```bash
   flux reconcile helmrelease <app-name>-<environment> -n <k8s-namespace>
   ```

### ExternalSecret Not Syncing

If the pod fails to start with errors about missing environment variables, the `ExternalSecret` may not have synced from AWS Secrets Manager:
1. Check the ExternalSecret status: `kubectl describe externalsecret -n <k8s-namespace>`
2. Verify the AWS secret name matches `awsSecretName` in the ConfigMap exactly.
3. Verify the IAM role attached to the service account allows `secretsmanager:GetSecretValue` for that secret ARN — see [Creating Organization](creating-organization).
4. Force a refresh by annotating the ExternalSecret: `kubectl annotate externalsecret <name> -n <k8s-namespace> force-sync=$(date +%s) --overwrite`

### Certificate Issues

If SSL certificates are not provisioning:
1. Verify DNS is configured correctly and has propagated
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

1. **Order matters**: Complete Steps 1–4 (DB schema, S3, Secret, DNS) before creating the Kubernetes configuration. The secret requires values from both the database and S3, and DNS must propagate before the deployment is merged.
2. **KNative and Kourier**: Applications must be compatible with KNative/Kourier for automatic scaling
3. **Service Accounts**: Ensure service accounts have appropriate IAM roles for AWS resource access
4. **DNS Propagation**: Allow time for DNS changes to propagate before deploying
5. **Cold Starts**: Be aware of the 2-5 second cold start delay when scaled to zero
6. **Resource Limits**: Set appropriate limits to prevent resource contention
7. **Secrets Management**: Never commit secrets to Git; use AWS Secrets Manager
8. **GitOps Workflow**: All changes go through Git; Flux CD manages deployment
9. **Test in Pre**: Always test deployments in pre-production before production

## Related Documentation

- [Creating Organization](../guides/creating-organization.md) - Setting up a new organization
- [Database Access](../guides/database-access.md) - How applications connect to databases
- [SSL Certificates](../guides/ssl-certificates.md) - Certificate management
- [Kubernetes Platform](../platform/kubernetes.md) - KNative and serverless architecture
