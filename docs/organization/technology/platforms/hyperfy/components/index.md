---
sidebar_position: 2
id: hyperfy-components
title: Hyperfy Components
sidebar_label: Components
---

# Hyperfy Components

This section documents our custom Hyperfy components that extend the platform's functionality. Each component is designed to provide specific interactive capabilities within our 3D environments.

## Available Components

- **[Base App Component](./base-app.md)**: Foundational template for creating new Hyperfy applications with node management, collision setup, and debug logging
- **[Emitter Component](./emitter.md)**: Interactive zones that detect player interactions and emit signals for other components to handle
- **[Visibility Component](./visibility.md)**: Control visibility and collision states of 3D objects with synchronization and event-driven changes

## Component Structure

Each component documentation includes:

- **Overview**: General description and purpose
- **Key Features**: Main capabilities and functionality
- **Video Demo**: Interactive demonstration of the component in action
- **Properties**: Configurable parameters organized by category
- **Events**: Emitted signals and their parameters
- **Common Use Cases**: Typical applications and scenarios
- **Integration**: How the component works with other systems
- **Best Practices**: Guidelines for optimal usage

## Development Guidelines

When developing new components for Hyperfy:

1. **Clear Purpose**: Define the component's specific role and responsibilities
2. **Property Organization**: Group related properties into logical categories
3. **Event Documentation**: Document all emitted signals with clear parameter descriptions
4. **Use Case Examples**: Provide practical scenarios without code implementation
5. **Integration Clarity**: Explain how the component works with other systems
6. **Best Practices**: Include guidelines for optimal usage and performance

## Integration

These components are integrated into our Hyperfy fork and are available for use in all our 3D spaces. For implementation details, see our [Hyperfy Integration Guide](../hyperfy-integration).
