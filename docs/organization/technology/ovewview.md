---
sidebar_position: 1
id: technical-overview
title: Technical Documentation
sidebar_label: Technical Overview
---

# Technical Overview

Numinia's technical infrastructure is built on open-source technologies, focusing on scalability, maintainability, and accessibility. Our architecture combines modern web technologies with cloud-native solutions to deliver immersive, gamified experiences.

## Core Platform

Our platform integrates three fundamental technologies: web-based 3D rendering through Three.js and WebGL for cross-platform compatibility, cloud infrastructure leveraging Kubernetes on AWS for deployment, and AI-powered digital agents for automation and assistance.

## Technology Stack

### Frontend Technologies

At the core of our frontend lies a Three.js-based engine implementation that powers our 3D experiences. We currently support multiple platforms, with Hyperfy 2 serving as our primary development environment. We're also preparing for integration with Oncyber once it becomes open source, and planning future support for Substrata.

### Backend Infrastructure

Our infrastructure follows Infrastructure as Code principles using Terragrunt built on top of the Terraform framework. This approach enables modular, version-controlled infrastructure management with reusable components across our platform.

The cloud architecture centers around Kubernetes for container orchestration, deeply integrated with AWS services. This foundation supports our microservices architecture, ensuring scalability and maintainability.

Our core services run on Node.js (22.x+), utilizing MongoDB for data persistence and incorporating real-time communication capabilities throughout the platform.

## System Components

Our platform integrates several core features including backend APIs, digital agent systems, a comprehensive gamification engine, and robust analytics capabilities. These components work together to create engaging, interactive experiences while maintaining performance and reliability.

## Documentation Structure

Our documentation is organized into three main areas:

### Platforms

Our [Platform Overview][platforms] covers implementation details for each supported engine. The [Hyperfy 2 documentation][hyperfy2] includes components, features, and best practices. Documentation for Oncyber and Substrata will be available as these platforms are integrated.

### Infrastructure

The [Infrastructure Guide][infrastructure] covers our Kubernetes setup, AWS architecture, and monitoring systems.

### Development

Our [Development Guide][development] provides comprehensive information about our APIs, integration procedures, and deployment processes.

## Contributing

Our [Contributing Guide][contributing] provides detailed information about development guidelines, documentation standards, security considerations, and best practices for contributing to the platform.

## Support

For technical support and inquiries, contact us at [tech@numengames.com](mailto:tech@numengames.com).

[platforms]: /docs/organization/technology/platforms/platforms-overview
[hyperfy2]: /docs/organization/technology/platforms/hyperfy2/overview
[infrastructure]: /docs/organization/technology/infrastructure/overview
[development]: /docs/organization/technology/development/overview
[contributing]: /docs/organization/contributing
