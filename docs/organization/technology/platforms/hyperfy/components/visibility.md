---
sidebar_position: 2
id: visibility-component
title: Visibility Component
sidebar_label: Visibility
---

# Visibility Component

The Visibility component is designed to control the visibility and collision states of 3D objects in real-time. It supports synchronized visibility changes across all connected clients, configurable delays for smooth transitions, and automatic collision management. The component can listen for custom events to trigger visibility changes and emit completion events for chaining actions. **Important**: This component only handles visibility and collision state management - it does not perform any other actions. You must associate other components or custom logic to trigger the visibility changes and create the desired behaviors.

**[Download Component](http://statics.numinia.xyz/hyperfy-components/Visibility-20250703.hyp)**

## Overview

The VisibilityController component allows you to dynamically control the visibility and collision states of 3D nodes in your scene. It supports both local and synchronized visibility changes, configurable delays for smooth transitions, and automatic collision setup for 3D models. The component acts as a pure state management system - it handles visibility and collision changes, but relies on other components or custom logic to trigger those changes and create the actual behaviors (showing/hiding objects, managing collision states, etc.).

### Key Features

- **Dual State Control**: Manage both visibility and collision states independently
- **Synchronized Changes**: Support for client-server synchronization across all connected players
- **Configurable Delays**: Add delays before applying visibility changes for smooth transitions
- **Event-Driven**: Listen for custom events to trigger visibility changes
- **Automatic Collision**: Automatic collision setup for 3D models with mesh detection
- **Completion Events**: Emit events when visibility changes are completed for action chaining
- **Debug System**: Comprehensive logging for troubleshooting visibility and collision issues

## Video Demo

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; margin: 20px 0;">
  <iframe 
    src="https://www.youtube.com/embed/your-video-id" 
    style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: 0;" 
    allowfullscreen>
  </iframe>
</div>

_Watch the Visibility component in action: demonstrating synchronized visibility changes, collision management, and event-driven state control._

## Properties

### Basic Configuration

| Property       | Type    | Default | Description                                       |
| -------------- | ------- | ------- | ------------------------------------------------- |
| `appID`        | string  | `""`    | Unique identifier for this visibility controller  |
| `nodeName`     | string  | `""`    | The name of the 3D node in your scene to control  |
| `isDebugMode`  | boolean | `false` | Enable detailed logging for troubleshooting       |
| `hasCollision` | boolean | `true`  | Forces all meshes to have collision               |
| `isVisible`    | boolean | `true`  | Initial visibility state when the app starts      |

### Signal Settings

| Property                           | Type    | Default | Description                                       |
| ---------------------------------- | ------- | ------- | ------------------------------------------------- |
| `visibilityControllerIsSync`       | boolean | `false` | Synchronize visibility changes across all clients |
| `visibilityControllerIsVisible`    | boolean | `true`  | Default visibility state for events               |
| `visibilityControllerHasCollision` | boolean | `true`  | Default collision state for events                |
| `visibilityControllerHasDelay`     | boolean | `true`  | Enable delay before applying visibility changes   |
| `visibilityControllerDelay`        | number  | `0`     | Time in seconds to wait before applying changes   |
| `visibilityControllerReceiver`     | string  | `"[]"`  | JSON array of event names to listen for          |
| `visibilityControllerEmitCompleted`| boolean | `true`  | Emit completion event when changes are finished   |

## Events

### Emitted Signals

| Event Name                    | Description                    | Parameters                        |
| ----------------------------- | ------------------------------ | --------------------------------- |
| `visibility-completed-{appID}`| Visibility change completed    | `{ playerId, timestamp }`         |

### Internal Events

| Event Name      | Description                    | Direction                        |
| --------------- | ------------------------------ | -------------------------------- |
| `visibility-sc` | Server to client visibility    | Server → Client                  |
| `visibility-cs` | Client to server visibility    | Client → Server                  |

### Event Parameters

When triggering visibility changes via events, you can pass these parameters:

| Parameter      | Type    | Description                                       |
| -------------- | ------- | ------------------------------------------------- |
| `isVisible`    | boolean | Whether the object should be visible              |
| `hasCollision` | boolean | Whether the object should have collision          |
| `isSync`       | boolean | Whether to synchronize across all clients         |
| `delay`        | number  | Delay in seconds before applying the change       |

## Common Use Cases

The Visibility component is perfect for creating dynamic environments that respond to player actions or game events. Common applications include:

- **Dynamic Doors**: Show/hide doors based on player proximity or interaction
- **Interactive Objects**: Reveal hidden objects when players solve puzzles
- **Environmental Effects**: Hide/show environmental elements based on time or conditions
- **Collision Management**: Enable/disable collision for objects based on game state
- **Synchronized Effects**: Ensure all players see the same visibility changes

## Integration

The Visibility component works by listening for custom events and applying visibility/collision changes to 3D nodes. For example, a door emitter might emit a "door-open" signal that the visibility component listens for to hide the door object. The component can also emit completion events that other components can listen for to chain additional actions.

## Best Practices

- Use synchronization for objects that all players should see change simultaneously
- Implement delays for smooth visual transitions and better user experience
- Configure collision states appropriately for your game mechanics
- Use completion events to chain multiple actions together
- Enable debug mode during development to troubleshoot visibility issues
- Set appropriate event receivers to avoid unnecessary processing
