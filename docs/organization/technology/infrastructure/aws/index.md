---
sidebar_position: 1
id: aws-infrastructure
title: AWS Infrastructure
sidebar_label: AWS Infrastructure
---

# AWS Infrastructure

This section documents our AWS infrastructure, explaining the main components and how they are configured to support our applications.

## Contents

- **[AWS Architecture Overview](aws-architecture-overview)**: Overview of our base AWS architecture, including VPC, subnets, and access strategy.

- **Storage**:
  - **[S3 & CloudFront CDN](storage/s3-cloudfront-cdn)**: Implementation of a CDN using S3 and CloudFront to deliver static assets.

## Related Guides

- To understand how this infrastructure is implemented with IaC, see the [Terragrunt](/docs/organization/technology/infrastructure/terragrunt/terragrunt-overview) section.
- For details on deploying applications on this infrastructure, visit the [Kubernetes](/docs/organization/technology/infrastructure/kubernetes/kubernetes-overview) section. 