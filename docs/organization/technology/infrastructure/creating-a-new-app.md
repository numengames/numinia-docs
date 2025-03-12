---
sidebar_position: 6
id: creating-a-new-app
title: Creating a New App
sidebar_label: Creating a New App
---

# Creating a New App guide

This guide details the process of setting up a new application within our infrastructure, using a real-world example based on our Crimson Tower implementation. Our infrastructure is designed to provide a standardized, secure, and scalable environment for deploying applications while maintaining flexibility for different use cases.

## Prerequisites

Before creating a new application, ensure you have:

- Access to the organization's namespace in Kubernetes
- Docker image of your application
- Required environment variables and secrets in AWS Secrets Manager
- Access to our [numinia-k8s](https://github.com/numengames/numinia-k8s) repository

Our infrastructure uses a GitOps approach, where all configuration is stored in version control. This ensures consistency, traceability, and the ability to roll back changes when needed.

## Application Structure

The application structure follows a standardized pattern that promotes maintainability and separation of concerns. Each application resides in its own directory within the organization's folder:

```bash
organizations/<org-name>/
└── <app-name>/
    └── base/
        ├── config.env           # Environment variables
        ├── deployment.yaml      # Application deployment
        ├── kustomization.yaml   # Kustomize configuration
        ├── route.yaml          # Traefik IngressRoute
        ├── secret-provider.yaml # AWS Secrets configuration
        └── service.yaml        # Service configuration
```

This structure allows us to manage each application independently while maintaining a consistent organization-wide standard. The `base` directory contains all the fundamental configurations needed for the application to run in our Kubernetes environment.

## Step-by-Step Setup

### 1. Configure Environment Variables

Environment variables are crucial for application configuration. We use a `config.env` file to maintain these variables in a centralized location. This approach allows us to modify configuration without changing the application code or Kubernetes manifests.

```env
# POD SPECS #
CPU=125m
PORT=3000
MEMORY=256Mi
LIMITS_CPU=250m
SAVE_INTERVAL=60
LIMITS_MEMORY=512Mi

# APP SPECS #
ENVIRONMENT=PRO
DOMAIN=<your-domain>
STORAGE_PATH=/mnt/persistent-storage
COMMIT_HASH=<your-commit-hash>

# PUBLIC URLS #
PUBLIC_API_URL=https://<your-domain>/api
PUBLIC_WS_URL=https://<your-domain>/ws
PUBLIC_ASSETS_URL=https://<your-domain>/assets
```

The environment variables are divided into three main categories:
- Pod specifications: Define resource allocation and operational parameters
- Application specifications: Set core application configuration
- Public URLs: Configure external access points

### 2. Create Deployment Configuration

The deployment configuration is the heart of your application setup. It defines how your application runs in Kubernetes:

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: <app-name>-deployment
  namespace: <org-name>
  labels:
    environment: pro
    app: <app-name>
spec:
  replicas: 1
  selector:
    matchLabels:
      app: <app-name>
      environment: pro
  template:
    metadata:
      labels:
        app: <app-name>
        environment: pro
    spec:
      serviceAccountName: aws-secret-manager-sa
      volumes:
        - name: persistent-storage
          persistentVolumeClaim:
            claimName: <org-name>-shared-pvc
        - name: secrets-store
          csi:
            driver: secrets-store.csi.k8s.io
            readOnly: true
            volumeAttributes:
              secretProviderClass: <app-name>-aws-secrets
      containers:
        - name: <app-name>
          image: <your-image>:<tag>
          volumeMounts:
            - name: persistent-storage
              mountPath: /mnt/persistent-storage
              subPath: <app-name>
            - name: secrets-store
              mountPath: "/mnt/secrets-store"
              readOnly: true
          env:
            - name: PORT
              value: ${PORT}
            - name: DOMAIN
              value: ${DOMAIN}
            - name: STORAGE_PATH
              value: ${STORAGE_PATH}
            # Add other environment variables from config.env
          resources:
            requests:
              memory: ${MEMORY}
              cpu: ${CPU}
            limits:
              memory: ${LIMITS_MEMORY}
              cpu: ${LIMITS_CPU}
```

Key aspects of the deployment configuration include:
- Service account integration for AWS Secrets Manager access
- Persistent storage configuration using shared volumes
- Resource management to ensure optimal performance
- Environment variable injection from config.env
- Container configuration and image specification

### 3. Configure AWS Secrets

Security is paramount in our infrastructure. We use AWS Secrets Manager to handle sensitive information:

```yaml
# secret-provider.yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: <app-name>-aws-secrets
  namespace: <org-name>
spec:
  provider: aws
  parameters:
    objects: |
      - objectName: "<app-name>-pro"
        objectType: "secretsmanager"
        jmesPath:
          - path: SECRET_KEY
            objectAlias: SECRET_KEY
  secretObjects:
    - secretName: aws-secrets-<app-name>
      type: Opaque
      data:
        - objectName: SECRET_KEY
          key: SECRET_KEY
```

This configuration enables secure access to secrets while maintaining separation between infrastructure and application concerns. The secrets are mounted as files in the container, providing a secure way to access sensitive information.

### 4. Create Service

The service configuration exposes your application within the Kubernetes cluster:

```yaml
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: <app-name>-service
  namespace: <org-name>
  labels:
    app: <app-name>
    environment: pro
spec:
  selector:
    app: <app-name>
  ports:
    - protocol: TCP
      port: 3000
      targetPort: 3000
```

Services act as stable endpoints for your application, enabling internal communication and load balancing.

### 5. Configure Traefik Route

Traefik handles external access to your application through IngressRoutes:

```yaml
# route.yaml
apiVersion: traefik.io/v1alpha1
kind: IngressRoute
metadata:
  name: <app-name>-route
  namespace: <org-name>
spec:
  entryPoints:
    - websecure
  routes:
    - match: "Host(`<your-domain>`)"
      kind: Rule
      services:
        - name: "<app-name>-service"
          port: 3000
  tls:
    secretName: <org-name>-tls
```

This configuration provides:
- Automatic SSL/TLS certificate management
- Domain-based routing
- Integration with our existing security measures

### 6. Setup Kustomization

Kustomize helps us manage environment-specific configurations:

```yaml
# kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
namespace: <org-name>
resources:
- deployment.yaml
- service.yaml
- route.yaml
- secret-provider.yaml
configMapGenerator:
- behavior: create
  envs:
  - config.env
  name: <app-name>-config
replacements:
- source:
    kind: ConfigMap
    name: <app-name>-config
    fieldPath: data.MEMORY
  targets:
  - select:
      kind: Deployment
      name: <app-name>-deployment
    fieldPaths:
    - spec.template.spec.containers.[name=<app-name>].resources.requests.memory
# Add similar replacements for other environment variables
```

This powerful tool allows us to:
- Generate ConfigMaps from environment files
- Replace values dynamically
- Maintain consistent configurations across environments

### 7. Register in Organization's Kustomization

After setting up the application's kustomization, you need to register it in the organization's main kustomization file. This ensures that Flux CD picks up and manages your application:

```yaml
# organizations/<org-name>/kustomization.yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
- <app-name>/base  # Add this line to include your new application
```

This step is crucial as it integrates your application into the organization's GitOps workflow, allowing Flux CD to automatically detect and apply any changes to your application configuration.

## Application Deployment

The deployment process follows our GitOps workflow, where Flux CD automatically reconciles the state of our cluster with the configuration in our Git repository:

1. Create AWS Secrets:
   ```bash
   # Create secrets in AWS Secrets Manager
   aws secretsmanager create-secret --name "<app-name>-pro" --secret-string '{"SECRET_KEY":"your-secret-value"}'
   ```

2. Associate the Secret with the Secret Manager Policy:
   After creating the secret, it's crucial to associate it with our secret-manager-policy. This policy allows our Kubernetes cluster to access and manage the secrets. The policy includes permissions for the following actions:
   - secretsmanager:GetSecretValue
   - secretsmanager:DescribeSecret
   - secretsmanager:ListSecrets

   The policy is already configured with the necessary ARN patterns:
   ```
   arn:aws:secretsmanager:eu-west-1:241533135482:secret:hyperfy2-*
   ```

   Make sure your secret name follows the pattern defined in the policy to ensure proper access.

3. Push Changes to Remote Repository:
   After completing all the configuration steps (1-7), commit and push your changes to the remote repository:
   ```bash
   git add organizations/<org-name>/<app-name>
   git add organizations/<org-name>/kustomization.yaml
   git commit -m "feat: add <app-name> application"
   git push origin main
   ```

   Flux CD will automatically detect these changes and start the reconciliation process, deploying your application to the cluster.

4. Monitor Deployment Status:
   You can verify the deployment status using:
   ```bash
   # Check Flux reconciliation status
   flux get kustomizations -n flux-system

   # Check application resources
   kubectl get deployment,svc,ingressroute,secretproviderclass -n <org-name>
   ```

Remember that with our GitOps approach, you should never apply changes directly to the cluster. All changes should be made through Git, allowing Flux CD to manage the deployment process automatically.

## Best Practices

Our best practices are derived from real-world experience and are designed to ensure reliable operation:

- Use environment variables for configuration to maintain flexibility
- Store sensitive data in AWS Secrets Manager for security
- Implement proper resource limits to prevent resource contention
- Use comprehensive logging for troubleshooting
- Leverage Kustomize for configuration management
- Follow consistent naming conventions
- Utilize shared storage with proper isolation