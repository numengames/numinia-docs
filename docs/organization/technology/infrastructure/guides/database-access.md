---
sidebar_position: 4
id: database-access
title: Database Access Patterns
sidebar_label: Database Access
---

# Database Access from Kubernetes Applications

This guide explains how applications running in our Kubernetes cluster access external database resources. Unlike traditional deployments that use persistent volumes, our applications connect to Amazon RDS PostgreSQL databases through secure, managed connections.

## Database Architecture

Our platform uses RDS PostgreSQL instances managed by the DevOps team. Each organization has access to its own database resources, with connection strings and credentials stored securely in AWS Secrets Manager. This approach provides several advantages over on-cluster databases: better isolation, independent scaling, automatic backups, and multi-AZ high availability managed by AWS.

The database instances reside in private subnets of our VPC, accessible only from within the network. Applications running in Kubernetes pods connect to these databases using connection strings retrieved from Secrets Manager, with SSL/TLS encryption for all connections.

## Setting Up Database Access

### Prerequisites

Before configuring database access for an application, ensure you have:

- Organization namespace created in the cluster
- Service account with IAM role configured for the organization
- Database credentials provided by the DevOps team
- Access to AWS Secrets Manager

### Step 1: Store Database Credentials in Secrets Manager

Create a secret in AWS Secrets Manager containing the database connection information. The secret should include:

- Database hostname or endpoint
- Port number
- Database name
- Username
- Password
- Connection string (optional, can be constructed from components)

Name the secret following the pattern: `{organization}-{environment}-database-credentials`

Example secret content:
```json
{
  "host": "your-database-cluster.xxxxx.eu-west-1.rds.amazonaws.com",
  "port": "5432",
  "database": "your_database_name",
  "username": "your_db_user",
  "password": "your-secure-password"
}
```

### Step 2: Configure Secret Access in Kubernetes

Applications access database credentials through AWS Secrets Manager using a SecretProviderClass resource. This configuration mounts the secrets from Secrets Manager as files or environment variables in the pod.

Create a `secret-provider.yaml` file:

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: {app-name}-database-secrets
  namespace: {org-name}
spec:
  provider: aws
  parameters:
    objects: |
      - objectName: "{organization}-{environment}-database-credentials"
        objectType: "secretsmanager"
        jmesPath:
          - path: host
            objectAlias: DB_HOST
          - path: port
            objectAlias: DB_PORT
          - path: database
            objectAlias: DB_NAME
          - path: username
            objectAlias: DB_USER
          - path: password
            objectAlias: DB_PASSWORD
  secretObjects:
    - secretName: database-credentials-{app-name}
      type: Opaque
      data:
        - objectName: DB_HOST
          key: host
        - objectName: DB_PORT
          key: port
        - objectName: DB_NAME
          key: database
        - objectName: DB_USER
          key: username
        - objectName: DB_PASSWORD
          key: password
```

### Step 3: Mount Secrets in Application Pod

Reference the SecretProviderClass in your deployment to mount the database credentials:

```yaml
spec:
  template:
    spec:
      serviceAccountName: {org-name}
      volumes:
        - name: database-secrets
          csi:
            driver: secrets-store.csi.k8s.io
            readOnly: true
            volumeAttributes:
              secretProviderClass: {app-name}-database-secrets
      containers:
        - name: {app-name}
          volumeMounts:
            - name: database-secrets
              mountPath: "/mnt/secrets-store"
              readOnly: true
          env:
            - name: DATABASE_URL
              value: "postgresql://$(cat /mnt/secrets-store/username):$(cat /mnt/secrets-store/password)@$(cat /mnt/secrets-store/host):$(cat /mnt/secrets-store/port)/$(cat /mnt/secrets-store/database)?sslmode=require"
```

### Step 4: Configure Application to Use Database

Configure your application to read connection information from the mounted secrets. Most applications can construct a connection string from individual components, or use a pre-formatted connection string if stored in the secret.

For PostgreSQL, include the `sslmode=require` parameter to enforce encrypted connections. This ensures that all database communications are protected, even within the VPC.

## Connection Security

All database connections enforce SSL/TLS encryption through the `sslmode=require` parameter. This ensures that data in transit between application pods and RDS instances is encrypted, even though traffic stays within the private subnets of our VPC.

The databases themselves are configured with encryption at rest, providing comprehensive data protection both in transit and at rest. AWS manages the encryption keys and certificate rotation automatically.

Access control is enforced at multiple levels: network security groups restrict access to authorized sources within the VPC, IAM policies control access to Secrets Manager credentials, and database-level access controls manage user permissions.

## Multi-Tenant Database Isolation

In our multi-tenant architecture, each organization maintains isolation at the database level. Organizations may have their own database schemas, dedicated databases, or in some cases separate RDS instances, depending on security and performance requirements.

Connection strings are organization-specific and stored in Secrets Manager with appropriate access controls. Service accounts and IAM roles ensure that applications can only access credentials for their designated organization's databases.

## Troubleshooting Database Connections

### Connection Refused Errors

If you encounter connection refused errors:

1. Verify the database endpoint is correct in Secrets Manager
2. Check that the database security group allows connections from the Kubernetes cluster CIDR
3. Ensure the database is running and accessible from the VPC
4. Verify SSL/TLS configuration if using encrypted connections

### Authentication Failures

For authentication failures:

1. Confirm credentials in Secrets Manager are current and valid
2. Check that the service account has permissions to access the secret
3. Verify IAM policies include Secrets Manager access for the organization
4. Ensure the database user exists and has appropriate permissions

### Network Timeouts

Network timeouts may indicate:

1. Database is in a different VPC or region than the cluster
2. Security groups are blocking traffic between subnets
3. NAT gateway configuration for private subnet egress
4. Database instance status and availability

## Best Practices

When configuring database access:

- Always use SSL/TLS encryption for database connections
- Store credentials in Secrets Manager, never in code or ConfigMaps
- Use service accounts with IAM roles for secure credential access
- Implement connection pooling to manage database connections efficiently
- Monitor database connection metrics and pool utilization
- Use parameterized queries to prevent SQL injection
- Implement retry logic with exponential backoff for transient failures
- Keep database credentials rotated regularly through Secrets Manager

## Related Documentation

- [AWS Services - RDS PostgreSQL](../aws-services#database-rds-postgresql) - Overview of RDS configuration
- [Creating Organization](creating-organization) - Organization setup including service accounts
- [Deploying Application](deploying-application) - Application deployment patterns

