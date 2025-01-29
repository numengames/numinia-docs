---
sidebar_position: 1
title: Overview
description: Introduction to Hyperfy 2 infrastructure
---

# Infrastructure Overview

Our Hyperfy 2 infrastructure implementation focuses on providing a scalable and maintainable environment for running distributed services. This section provides a high-level overview of our infrastructure components and guides you to detailed documentation.

## Architecture Principles

- **Provider Agnostic**: Core infrastructure defined through Terraform modules
- **Scalability**: Designed to handle varying loads efficiently
- **Security**: Following cloud security best practices
- **Cost Optimization**: Resource utilization monitoring and optimization
- **Maintainability**: Modular design for easy updates and modifications

## Key Components

### Service Architecture
We use containerized services running on AWS ECS, with each service managing multiple processes through PM2. For detailed architecture diagrams and specifications, see our [Architecture Guide](/docs/hyperfy-2/infrastructure/architecture).

### Resource Management
Our infrastructure utilizes shared resources including:
- Centralized storage (EFS)
- Load balancing
- Service discovery
- Configuration management

Learn more in our [Resource Management Guide](/docs/hyperfy-2/infrastructure/resources).

### Deployment Strategy
Services are deployed through an automated pipeline, from GitHub to production. For implementation details, check our [Deployment Guide](/docs/hyperfy-2/infrastructure/deployment).

## Documentation Structure

- **[AWS Setup](/docs/hyperfy-2/infrastructure/aws-setup)**: Initial cloud infrastructure setup
- **[Architecture](/docs/hyperfy-2/infrastructure/architecture)**: Detailed system design and diagrams
- **[Terraform Modules](/docs/hyperfy-2/infrastructure/terraform-modules)**: Infrastructure as Code implementation
- **[Deployment](/docs/hyperfy-2/infrastructure/deployment)**: Service deployment process
- **[Resources](/docs/hyperfy-2/infrastructure/resources)**: Shared resource management
- **[Monitoring](/docs/hyperfy-2/infrastructure/monitoring)**: System monitoring and logging

## Getting Started

1. Review the [Architecture Guide](/docs/hyperfy-2/infrastructure/architecture) for system understanding
2. Follow the [AWS Setup Guide](/docs/hyperfy-2/infrastructure/aws-setup) for initial configuration
3. Use the [Deployment Guide](/docs/hyperfy-2/infrastructure/deployment) for service implementation
