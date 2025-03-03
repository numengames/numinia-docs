---
sidebar_position: 2
id: supported-platforms
title: Supported Platforms
sidebar_label: Supported Platforms
---

# Supported Platforms

We leverage existing 3D platforms to build our immersive educational experiences. Our expertise lies in two main areas: creating specialized components and developing the cloud infrastructure necessary to manage and scale these platforms. Here's an overview of how we work with different platforms:

## Platform Comparison

When developing a project, selecting the right underlying platform is crucial. We've carefully evaluated different engines based on their capabilities and deployment options. Some platforms allow us to manage them in our cloud infrastructure, while others are accessed through their own platforms. The following comparison highlights the key characteristics of each platform we support:

| Feature                    | Hyperfy 2          | Substrata _(In Progress)_ | AWE (Oncyber) |
|---------------------------|--------------------|-----------------------|---------------|
| **Status**                | Production Ready   | In Development       | Available     |
| **Focus**                 | General Purpose    | Performance Critical | Social Spaces |
| **Deployment**            | Web & VR           | Web, Mobile, VR      | Web           |
| **Performance**           | High               | Ultra High           | High          |
| **Component Support**     | In Development     | In Development       | Platform Only |
| **Documentation**         | In Progress        | In Progress          | In Progress   |
| **Use Case**             | General Purpose    | Specialized Apps     | Social Spaces |
| **Infrastructure**        | Custom Cloud       | Custom Cloud         | Platform Managed |

## Hyperfy 2

We've chosen Hyperfy 2 as our primary development platform for its robust capabilities in creating interactive 3D spaces. Our work with Hyperfy 2 operates on two levels:

1. **Cloud Infrastructure**: We develop and maintain the complete cloud architecture needed to deploy and scale Hyperfy-based experiences.
2. **Component Development**: We create specialized components and tools that enhance the platform's capabilities for educational and interactive experiences.

### Our Components and Tools
- Educational interaction components
- Custom UI elements for learning experiences
- Collaboration tools and utilities
- Integration with educational platforms
- Analytics and progress tracking tools
- Cloud management and scaling solutions

### Implementation Status
- 🔄 Component development in progress
- ✅ Cloud infrastructure operational
- 🔄 Documentation in development
- 🔄 Examples in preparation

[Learn more about our Hyperfy 2 implementation][hyperfy2]

## Substrata

We're currently developing both infrastructure and components for Substrata, focusing on specialized educational use cases that require high performance. Our work includes:

1. **Cloud Architecture**: Building the infrastructure to deploy and manage Substrata-based experiences.
2. **Performance Components**: Creating optimized components for complex educational scenarios.

### Planned Components
- High-performance simulation tools
- Advanced visualization components
- Specialized educational interfaces
- Performance-optimized interactions
- Technical learning utilities
- Cloud deployment tools

### Implementation Status
- 🔄 Initial component development
- 🔄 Cloud infrastructure in progress
- 🔄 Documentation in development
- 📋 Examples in preparation

[Track our Substrata implementation progress][substrata]

## AWE (Oncyber)

We utilize AWE through Oncyber's platform for creating social spaces and experiences. Unlike our other supported platforms, AWE is accessed directly through Oncyber's infrastructure:

### Platform Usage
- Creation of social spaces through Oncyber's platform
- Limited to platform-provided features
- No custom cloud deployment options
- Access to platform's built-in tools

### Implementation Status
- ✅ Platform access ready
- 🔄 Space creation in progress
- ℹ️ Uses platform documentation
- ℹ️ Platform-managed deployment

## Implementation Resources

While each platform provides its own documentation for basic component development and implementation, we focus on documenting our cloud architecture and custom solutions:

### Cloud Architecture
Our cloud infrastructure is organized into two main layers:

#### Base Architecture
- Common infrastructure components
- Core networking setup
- Shared security policies
- Monitoring and logging infrastructure
- Base scaling configurations
- Standard deployment pipelines

#### Engine-Specific Implementations
- **Hyperfy 2**: Specialized container configuration and scaling policies
- **Substrata**: Performance-optimized infrastructure setup
Each engine has its dedicated configuration folder with specific adjustments and optimizations.

### Custom Components
- Documentation of our component libraries
- Integration examples with our cloud infrastructure
- Performance optimization guidelines
- Component testing and validation

### Additional Resources
- Video tutorials on our YouTube channel
- Case studies of successful implementations
- Technical blog posts and articles
- Community contributions and feedback
- Troubleshooting guides for common scenarios

For detailed information about our cloud architecture and custom solutions, visit:
- [Base Architecture Guide][base-architecture]
- [Hyperfy 2 Infrastructure][hyperfy2]
- [Substrata Infrastructure][substrata]
- [Custom Components][components]

[hyperfy2]: /docs/organization/technology/platforms/hyperfy2/overview
[substrata]: /docs/organization/technology/platforms/substrata/overview
[components]: /docs/organization/technology/platforms/components
[base-architecture]: /docs/organization/technology/infrastructure/base-architecture
[integration]: /docs/organization/technology/platforms/integration
