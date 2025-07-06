---
sidebar_position: 3
id: base-app-component
title: Base App Component
sidebar_label: Base App
---

# Base App Component

The Base App component is designed as a foundational template for creating new Hyperfy applications. It provides essential infrastructure including node management, collision setup, visibility control, debug logging, and configuration management. This component serves as the starting point for developing custom interactive applications by providing common functionality that most apps require. **Important**: This component is a template/boilerplate - it provides the basic structure and utilities but requires custom logic to be added for specific functionality.

**[Download Component](http://statics.numinia.xyz/hyperfy-components/Base-20250706.hyp)**

## Overview

The BaseApp component provides a standardized foundation for building Hyperfy applications. It includes common utilities for node management, collision detection, visibility control, and debug logging. The component acts as a boilerplate template - it sets up the basic infrastructure that most applications need, allowing developers to focus on implementing their specific business logic rather than common setup tasks.

### Key Features

- **Node Management**: Automatic node discovery and validation in the 3D scene
- **Collision Setup**: Automatic collision detection and rigidbody creation for 3D models
- **Visibility Control**: Basic visibility state management for 3D objects
- **Debug System**: Comprehensive logging system for troubleshooting and development
- **Configuration Management**: Standardized configuration system with hints and validation
- **Error Handling**: Robust error handling with detailed logging
- **Extensible Architecture**: Easy to extend with custom logic and additional features

## Properties

### Basic Configuration

| Property       | Type    | Default | Description                                       |
| -------------- | ------- | ------- | ------------------------------------------------- |
| `appID`        | string  | `""`    | Unique identifier for this application            |
| `nodeName`     | string  | `""`    | The name of the 3D node in your scene to control  |
| `isDebugMode`  | boolean | `false` | Enable detailed logging for troubleshooting       |
| `hasCollision` | boolean | `true`  | Forces all meshes to have collision               |
| `isVisible`    | boolean | `true`  | Initial visibility state when the app starts      |

### Base Settings

| Property       | Type    | Default | Description                                       |
| -------------- | ------- | ------- | ------------------------------------------------- |
| `baseSection`  | section | -       | Base configuration section                        |

## Events

### Debug Events

| Event Name | Description                    | Parameters                        |
| ---------- | ------------------------------ | --------------------------------- |
| `log`      | Debug logging event            | `{ kind, timestamp, nodeName, message }` |

### Event Parameters

The debug log event includes these parameters:

| Parameter   | Type    | Description                                       |
| ----------- | ------- | ------------------------------------------------- |
| `kind`      | string  | Log level (info, warn, error, debug)             |
| `timestamp` | number  | Unix timestamp when the log was created          |
| `nodeName`  | string  | Name of the controlled 3D node                   |
| `message`   | string  | Log message content                               |

## Integration

The Base App component works by providing a foundation that other components and custom logic can build upon. It handles the common setup tasks like node discovery, collision setup, and debug logging, allowing developers to focus on implementing their specific functionality. The component can be extended by adding custom methods to the BaseApp class and implementing logic in the `_initClass()` method.

## Best Practices

- Use this component as a starting point for all new Hyperfy applications
- Enable debug mode during development to troubleshoot issues
- Extend the BaseApp class with custom methods for your specific functionality
- Implement your main logic in the `_initClass()` method
- Use the provided debug logging system for consistent error tracking
- Configure collision settings appropriately for your 3D models
- Set appropriate node names that match your scene structure 