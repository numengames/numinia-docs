# Hyperfy Components Documentation

This directory contains documentation for our custom Hyperfy components. Each component has its own dedicated page with comprehensive documentation including properties, events, and usage guidelines.

## Component Documentation Structure

Each component documentation follows this structure:

1. **Overview**: General description and purpose
2. **Key Features**: Main capabilities and functionality
3. **Video Demo**: Interactive demonstration (YouTube embed)
4. **Properties**: Configurable parameters organized by category
5. **Events**: Emitted signals and their parameters
6. **Common Use Cases**: Typical applications and scenarios
7. **Integration**: How the component works with other systems
8. **Best Practices**: Guidelines for optimal usage

## Adding New Components

When adding documentation for a new component:

1. Create a new `.md` file in this directory
2. Use the standard frontmatter structure:
   ```yaml
   ---
   sidebar_position: [number]
   id: [component-name]-component
   title: [Component Name] Component
   sidebar_label: [Component Name]
   ---
   ```
3. Follow the established documentation structure (8 sections)
4. Include a video demo (upload to YouTube and embed)
5. Update the main components index.md to include the new component
6. Update this README if needed

## Video Requirements

- **Format**: MP4 or WebM
- **Resolution**: 1920x1080 minimum
- **Duration**: 30-60 seconds for demonstrations
- **Content**: Show the component in action with clear examples
- **Upload**: Use YouTube and embed with responsive iframe

## Documentation Guidelines

All component documentation should:

- Follow the established 8-section structure
- Include clear property descriptions with types and defaults
- Document all emitted events and their parameters
- Provide practical use cases without code implementation
- Explain integration with other components
- Include performance considerations in best practices

## Contributing

When contributing to component documentation:

1. Follow the established 8-section structure
2. Verify property descriptions are accurate and complete
3. Ensure video demos are up to date and functional
4. Check that integration examples are current
5. Update related components if changes affect them
6. Test all documented functionality

## Current Components

- **[Base App](base-app)**: Foundational template for creating new Hyperfy applications with node management, collision setup, and debug logging
- **[Emitter](emitter)**: Interactive zones that detect player interactions and emit signals for other components to handle
- **[Visibility](visibility)**: Control visibility and collision states of 3D objects with synchronization and event-driven changes

## Future Components

Planned components for documentation:

- **Audio Component**: Sound management and spatial audio
- **Animation Component**: Timeline-based animations and transitions
- **Physics Component**: Advanced physics interactions and constraints
- **UI Component**: 3D user interface elements and interactions
