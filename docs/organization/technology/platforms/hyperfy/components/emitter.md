---
sidebar_position: 1
id: emitter-component
title: Emitter Component
sidebar_label: Emitter
---

# Emitter Component

The Emitter component is designed to create interactive zones in 3D space capable of detecting player interactions and emitting signals. It supports both manual (key-press) and automatic (proximity) triggering modes with configurable cooldown, single-use options, and proximity zone events. The component includes a built-in UI system for automatic mode and custom signal emission with unique IDs. **Important**: This component only handles the trigger detection and event emission - it does not perform any actions itself. You must associate other components or custom logic to handle the emitted signals and create the desired behaviors.

**[Download Component](http://statics.numinia.xyz/hyperfy-components/Emitter-20250706.hyp)**

## Overview

The InteractiveEventEmitter component allows you to create dynamic interaction zones that can respond to player proximity and manual interactions. These zones emit custom signals that other components can listen to, making them ideal for creating complex interactive systems and game mechanics. **The component acts as a pure trigger system** - it detects interactions and emits events, but relies on other components or custom logic to handle those events and create the actual behaviors (opening doors, teleporting players, etc.).

### Key Features

- **Dual Interaction Modes**: Manual interaction with action buttons or automatic triggering by proximity
- **Flexible Event System**: Configurable cooldown periods, single-use options, and custom signal IDs
- **Proximity Detection**: Advanced proximity zone management with independent ENTER/LEAVE event controls
- **UI Integration**: Built-in UI display system for automatic mode with customizable positioning
- **Collision Support**: Automatic collision setup for 3D models with mesh detection
- **Debug System**: Comprehensive logging for troubleshooting and monitoring

## Video Demo

<div style="position: relative; padding-bottom: 56.25%; height: 0; overflow: hidden; max-width: 100%; margin: 20px 0;">
  <iframe 
    src="https://www.youtube.com/embed/your-video-id" 
    style="position: absolute; top: 0; left: 0; width: 100%; height: 100%; border: 0;" 
    allowfullscreen>
  </iframe>
</div>

_Watch the Emitter component in action: demonstrating manual and automatic interaction modes, proximity detection, and signal emission._

## Properties

### Basic Configuration

| Property                    | Type    | Default | Description                                       |
| --------------------------- | ------- | ------- | ------------------------------------------------- |
| `emitterHasCooldown`        | boolean | `false` | Enable cooldown system to prevent rapid-fire triggering |
| `emitterCooldown`           | number  | `0`     | Time in seconds before the emitter can be triggered again |
| `emitterEventLabel`         | string  | `""`    | Text displayed on the UI and action button |
| `emitterInteractionType`    | string  | `"key"` | KEY: Manual interaction with action button \| AUTO: Automatic trigger by proximity |
| `emitterIsSingleUse`        | boolean | `false` | If enabled, the emitter can only be triggered once and then becomes inactive |

### Distance Settings

| Property                    | Type    | Default | Description                                       |
| --------------------------- | ------- | ------- | ------------------------------------------------- |
| `emitterTriggerDistance`    | number  | `2`     | Distance in units where the action will be triggered (for both KEY and AUTO modes) |
| `emitterProximityDistance`  | number  | `3`     | Distance in units where the UI will be displayed (AUTO mode only) |
| `emitterUIRadiusY`          | number  | `1`     | Vertical offset for the UI display position (AUTO mode only) |

### Event Configuration

| Property                    | Type    | Default | Description                                       |
| --------------------------- | ------- | ------- | ------------------------------------------------- |
| `emitterSignalId`           | string  | `""`    | Unique identifier for the emitted signal (used in event naming) |
| `emitterEnableEnterEvent`   | boolean | `true`  | Send ENTER proximity zone event when player enters the proximity area |
| `emitterEnableLeaveEvent`   | boolean | `true`  | Send LEAVE proximity zone event when player exits the proximity area |

## Events

### Emitted Signals

| Event Name                             | Description                    | Parameters                   |
| -------------------------------------- | ------------------------------ | ---------------------------- |
| `{emitterSignalId}-{appID}`            | Main emitter signal            | `{ playerId, timestamp }`    |
| `emitter-enter-proximity-zone-{appID}` | Player enters proximity zone   | `{ playerId, timestamp }`    |
| `emitter-leave-proximity-zone-{appID}` | Player leaves proximity zone   | `{ playerId, timestamp }`    |

### Interaction Modes

#### Manual Mode (KEY)
- Player must press a key/button to trigger the emitter
- Action button appears when player is within trigger distance
- Button displays the configured event label

#### Automatic Mode (AUTO)
- Emitter triggers automatically when player enters trigger zone
- UI displays when player enters proximity zone
- Optional ENTER/LEAVE events can be emitted

## Common Use Cases

The Emitter component is perfect for creating interactive zones that respond to player presence or actions. Common applications include:

- **Automatic Doors**: Detect player proximity and emit signals for door controllers
- **Interactive Objects**: Require manual interaction for chests, switches, or collectibles
- **Teleporters**: Create travel points with anti-spam protection via cooldowns
- **Proximity Sensors**: Track player movement for environmental effects or security systems
- **Puzzle Triggers**: Activate puzzle elements when players enter specific zones
- **Resource Collection**: Set up collection points with single-use or renewable configurations

## Integration

The Emitter component works by emitting signals that other components or custom logic must handle to create the desired behaviors. For example, a door emitter might emit a "door-open" signal that a door controller component listens for to perform the actual door animation.

## Best Practices

- Use KEY mode for intentional interactions (chests, switches) and AUTO mode for environmental triggers (doors, sensors)
- Implement cooldowns to prevent spam in frequently used emitters
- Enable single-use mode for consumable items or one-time interactions
- Set appropriate trigger and proximity distances for your specific use case
- Disable unnecessary proximity events (ENTER/LEAVE) to reduce event traffic
