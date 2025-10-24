---
sidebar_position: 2
id: hyperfy-components
title: Hyperfy Components
sidebar_label: Components
---

# Hyperfy Components

This section documents our custom Hyperfy components that extend the platform's functionality. Each component is designed to provide specific interactive capabilities within our 3D environments.

## Available Components

### Core Animation Components

- **[Base App Component](./base-app.md)**: Foundational template for creating new Hyperfy applications with node management, collision setup, and debug logging
- **[Visibility Component](./visibility.md)**: Control visibility and collision states of 3D objects with synchronization and event-driven changes
- **[Rotate Component](./rotate.md)**: Smooth 3D object rotation animations with configurable speed, delays, and event-driven control
- **[Translate Component](./translate.md)**: Smooth 3D object translation animations with configurable speed, delays, and event-driven control

### Interactive Components

- **[Emitter Component](./emitter.md)**: Interactive zones that detect player interactions and emit signals for other components to handle
- **[Teleport Component](./teleport.md)**: Flexible teleportation functionality that moves players to specific destinations through events
- **[Redirect Component](./redirect.md)**: External URL redirection when triggered by events, opening websites in new browser windows/tabs
- **[Insert Password Component](./insert-password.md)**: Secure password-protected interactions with password input UI and validation

### UI and Communication Components

- **[Dialog Component](./dialog.md)**: Interactive dialog system for character conversations with multiple lines, character names, and auto-advance
- **[Keyboard Component](./keyboard.md)**: Virtual keyboard interface with letter and number keypads for text input
- **[Password Manager Component](./password-manager.md)**: Secure password management with input UI, validation, and keyboard integration
- **[Privacy Policy Component](./privacy-policy.md)**: Clean interface for displaying privacy policy information and legal documents

### System Components

- **[Logs Component](./logs.md)**: Centralized logging system that handles debug messages across all components
- **[Respawn Component](./respawn.md)**: Intelligent player respawn system that tracks positions and prevents falls into void areas

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
