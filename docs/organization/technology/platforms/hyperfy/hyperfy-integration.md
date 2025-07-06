---
sidebar_position: 1
id: hyperfy-integration
title: Hyperfy Integration Guide
sidebar_label: Hyperfy Integration
---

# Hyperfy Development

Our work with Hyperfy focuses on three main areas: component development, platform maintenance through our fork, and space deployment using our customized Docker image.

## Current Implementation

### Platform Fork
We maintain a fork of Hyperfy where we:
- Keep the codebase up to date with upstream changes
- Implement specific optimizations for our use cases
- Prepare potential contributions back to the main repository

### Component Development
Our component development extends Hyperfy's base functionality with:
- Teleportation system
- Password protection for spaces
- Iframe integration
- Basic interaction features

### Deployment
Our deployment strategy is straightforward:
- Building and maintaining our custom Docker image
- Running spaces in our Kubernetes cluster using this image
- Managing space configurations and updates

## Implementation Status

- ✅ Platform fork maintenance
- ✅ Space deployment and management
- 🔄 Ongoing component development
- 🔄 Potential upstream contributions

For specific component documentation and implementation details, see our [Components Guide](components).
