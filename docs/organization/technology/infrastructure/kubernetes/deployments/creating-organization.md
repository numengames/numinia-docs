---
sidebar_position: 2
id: creating-organization
title: Creating a New Organization
sidebar_label: Creating Organization
---

# Deploying a New Organization

This guide details the process of setting up a new organization within our infrastructure. Each organization is isolated within our Kubernetes cluster while sharing the core infrastructure components.

## Prerequisites

Before creating a new organization, ensure you have:

- Access to our Kubernetes cluster
- Access to our [numinia-k8s](https://github.com/numengames/numinia-k8s) repository
- Access to our [numinia-terragrunt](https://github.com/numengames/numinia-terragrunt) repository
- The organization's domain information
- Required resource quotas defined

## Infrastructure Setup

### 1. Create KMS Key

First, create a KMS key for the organization's EFS encryption. This key will ensure all data stored in the EFS volume is encrypted at rest.

Navigate to the KMS directory:

```bash
cd numinia-terragrunt/infrastructure/production/eu-west-1/numinia/kms
mkdir <org-name>-efs-key
cd <org-name>-efs-key
```

Create the KMS configuration:

```hcl
# terragrunt.hcl
include "root" {
  path = find_in_parent_folders("root.hcl")
}

terraform {
  source = "tfr:///terraform-aws-modules/kms/aws?version=1.5.0"
}

inputs = {
  description = "KMS key for <org-name> EFS encryption"
  
  # Key configuration
  enable_key_rotation = true
  
  # Alias
  aliases = ["<org-name>-efs"]
  
  # Tags
  tags = {
    Environment = "production"
    Organization = "<org-name>"
    ManagedBy = "terragrunt"
  }
}
```

Apply the KMS configuration:

```bash
terragrunt init
terragrunt plan
terragrunt apply
```

Note the KMS key ARN from the output as it will be needed for the EFS configuration.

### 2. Create EFS Volume

Next, create the organization's EFS volume using Terragrunt. Navigate to the appropriate directory in the numinia-terragrunt repository:

```bash
cd infrastructure/production/eu-west-1/organizations/<org-name>/efs
```

Create the EFS configuration:

```hcl
# terragrunt.hcl
include "root" {
  path = find_in_parent_folders("root.hcl")
}

terraform {
  source = "tfr:///terraform-aws-modules/efs/aws?version=1.1.1"
}

dependency "vpc" {
  config_path = "../../../numinia/vpc"
}

inputs = {
  # Organization-specific name
  name = "org-<name>-efs"

  # Mount targets in private subnets
  mount_targets = {
    "eu-west-1a" = {
      subnet_id = dependency.vpc.outputs.private_subnets[0]
    }
    "eu-west-1b" = {
      subnet_id = dependency.vpc.outputs.private_subnets[1]
    }
    "eu-west-1c" = {
      subnet_id = dependency.vpc.outputs.private_subnets[2]
    }
  }

  # Security group configuration
  security_group_description = "EFS security group for organization <name>"
  security_group_vpc_id     = dependency.vpc.outputs.vpc_id
  security_group_rules = {
    vpc = {
      description = "NFS from VPC"
      cidr_blocks = [dependency.vpc.outputs.vpc_cidr_block]
    }
  }

  # EFS configuration
  encrypted         = true
  performance_mode  = "generalPurpose"
  throughput_mode   = "bursting"

  # Tags
  tags = {
    Organization = "<name>"
    Environment  = "production"
  }
}
```

Apply the configuration:

```bash
terragrunt init
terragrunt plan
terragrunt apply
```

## Kubernetes Setup

### 1. Create Organization Directory Structure

First, create the directory structure for the new organization in the `numinia-k8s` repository:

```bash
cd numinia-k8s/clusters/prod-cluster/organizations
mkdir -p <org-name>/{base,org-config}
```

This creates:
- `<org-name>/`: Main organization directory
- `base/`: For base resources (namespace, storage, quotas)
- `org-config/`: For organization-specific configurations

### 2. Register Organization in Flux

Next, register the organization in the Flux kustomization configuration. Add two new Kustomization entries to `clusters/prod-cluster/flux-kustomization/kustomization.yaml`:

```yaml
# Organization applications kustomization
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: <org-name>-apps
  namespace: <org-name>
spec:
  interval: 5m0s
  path: ./clusters/prod-cluster/organizations/<org-name>
  prune: true
  sourceRef:
    kind: GitRepository
    name: numinia-k8s
    namespace: flux-system

# Organization configuration kustomization
---
apiVersion: kustomize.toolkit.fluxcd.io/v1
kind: Kustomization
metadata:
  name: org-config-<org-name>
  namespace: kube-system
spec:
  interval: 5m0s
  path: ./clusters/prod-cluster/organizations/<org-name>/org-config
  prune: true
  sourceRef:
    kind: GitRepository
    name: numinia-k8s
```

### 2.1. Create ClusterIssuer

Create a ClusterIssuer for SSL certificate management. Navigate to the infrastructure directory:

```bash
cd numinia-k8s/clusters/prod-cluster/infrastructure/networking/cert-manager
```

Add a new ClusterIssuer to `cluster-issuer.yaml`:

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

### 3. Configure Organization

In the `org-config` directory, create the following configuration files:

```bash
cd <org-name>/org-config
mkdir storage
```

#### 3.1 Create Namespaces Configuration

Create `namespaces.yaml` to define the organization's namespace:

```yaml
# namespaces.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: <org-name>
  labels:
    organization: <org-name>
```

#### 3.2 Configure RBAC

Create `rbac.yaml` to define role-based access control:

```yaml
# rbac.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: <org-name>-role
  namespace: <org-name>
spec:
  rules:
  - apiGroups: [""]
    resources: ["pods", "services", "configmaps", "secrets"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: <org-name>-binding
  namespace: <org-name>
spec:
  roleRef:
    apiGroup: rbac.authorization.k8s.io
    kind: Role
    name: <org-name>-role
  subjects:
  - kind: ServiceAccount
    name: default
    namespace: <org-name>
```

#### 3.3 Configure SSL Certificate

Create `cert.yaml` for SSL certificate management:

```yaml
# cert.yaml
apiVersion: cert-manager.io/v1
kind: Certificate
metadata:
  name: <org-name>-tls
  namespace: <org-name>
spec:
  secretName: <org-name>-tls
  dnsNames:
  - "<domain-name>"
  - "<subdomain>.<domain-name>"
  issuerRef:
    name: letsencrypt-<org-name>-prod
    kind: ClusterIssuer
```

#### 3.4 Configure Storage

In the `storage` directory, create the following files:

```bash
cd storage
```

1. Create the storage class configuration:
```yaml
# storage-class.yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: <org-name>-efs-sc
provisioner: efs.csi.aws.com
parameters:
  provisioningMode: efs-ap
  fileSystemId: ${EFS_ID} # From Terragrunt output
  directoryPerms: "700"
```

2. Create the persistent volume configuration:
```yaml
# persistent-volume.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: <org-name>-efs-pv
spec:
  capacity:
    storage: 1Ti
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: <org-name>-efs-sc
  csi:
    driver: efs.csi.aws.com
    volumeHandle: ${EFS_ID}::${ACCESS_POINT_ID}
```

3. Create the shared PVC configuration:
```yaml
# shared-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: <org-name>-shared-pvc
  namespace: <org-name>
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: <org-name>-efs-sc
  resources:
    requests:
      storage: 1Ti
```

4. Create the storage kustomization file:
```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- storage-class.yaml
- persistent-volume.yaml
- shared-pvc.yaml
```

This storage configuration:
- Creates an EFS storage class specific to the organization
- Sets up a persistent volume using the EFS filesystem
- Configures a shared PVC for the organization's applications
- Uses Kustomize to manage all storage resources together

#### 3.5 Create Kustomization

Create `kustomization.yaml` to manage all these resources:

```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- namespaces.yaml
- rbac.yaml
- cert.yaml
- storage/
```

### 4. Configure Resource Quotas

Set up resource limits for the organization:

```yaml
# resourcequota.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: org-quota
  namespace: org-<name>
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    persistentvolumeclaims: "10"
```

### 5. Setup Base Kustomization

Create the base kustomization file:

```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - namespace.yaml
  - storage.yaml
  - resourcequota.yaml
```

## Post-Setup Verification

After creating the organization structure:

1. Verify EFS creation:
   ```bash
   aws efs describe-file-systems --query 'FileSystems[?Name==`org-<name>-efs`]'
   ```

2. Apply Kubernetes configurations:
   ```bash
   kubectl apply -k organizations/<org-name>/base
   ```

3. Verify storage setup:
   ```bash
   kubectl get sc,pv -n org-<name>
   ```

4. Check resource quotas:
   ```bash
   kubectl describe resourcequota -n org-<name>
   ```

## Best Practices

- Use clear, consistent naming conventions
- Document all organization-specific requirements
- Implement proper RBAC configurations
- Set up monitoring from day one
- Maintain separate configuration repositories
- Ensure EFS backups are configured
- Monitor EFS performance metrics

## Support

For assistance with organization setup:
1. Review our internal documentation
2. Contact the infrastructure team
3. Submit issues in our repositories

Remember to update the organization's documentation with any specific requirements or configurations. 