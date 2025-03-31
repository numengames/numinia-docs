---
sidebar_position: 2
id: kubernetes-deployments
title: Kubernetes Deployments
sidebar_label: Deployments
---

# Kubernetes Deployments

This section provides comprehensive guides for deploying and managing applications and organizations in our Kubernetes infrastructure. These guides are designed to help you understand our GitOps-based workflow and the specific patterns we follow for deployments.

## Contents

- **[Deploying New App](deploying-new-app)**: Step-by-step guide for creating and deploying a new application to our Kubernetes cluster.
- **[Creating Organization](creating-organization)**: Comprehensive process for setting up a new organization within our multi-tenant infrastructure.

## Deployment Philosophy

Our deployment approach is built on several core principles:

- **GitOps-First**: All deployments are managed through Git repositories
- **Infrastructure as Code**: All resources are defined in version-controlled configuration files
- **Standardized Patterns**: Consistent deployment patterns across all applications
- **Automated Workflows**: Flux CD automates the deployment process
- **Multi-Tenant Isolation**: Organizations are securely isolated from each other

## Recommended Reading

Before deploying your first application, we recommend:

1. Reviewing our [Kubernetes Overview](../kubernetes-overview) to understand the cluster architecture
2. Familiarizing yourself with [SSL Certificate Management](../security/ssl-certificates)
3. Understanding how we structure configurations in our Git repositories 