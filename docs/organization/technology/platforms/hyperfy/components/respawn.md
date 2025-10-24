---
sidebar_position: 13
id: respawn-component
title: Respawn Component
sidebar_label: Respawn
---

# Respawn Component

The Respawn component tracks player positions and automatically returns them to their last safe position when they fall below a configurable threshold. Think of it as an intelligent safety system that prevents players from falling into the void or getting stuck in inaccessible areas.

**Important**: This component continuously monitors player heights and movement patterns to detect falls and identify safe positions. It uses intelligent algorithms to determine when a player is actually falling versus normal movement, and teleports them back to their last known safe position.

**[Download Component](http://statics.numinia.xyz/hyperfy-components/respawn.hyp)**

## What Does It Do?

The Respawn component is like having an intelligent safety net for your 3D world. You can:
- **Prevent players from falling into the void** in elevated worlds
- **Create safety nets** for platforming challenges
- **Ensure players don't get stuck** in inaccessible areas
- **Track player movement patterns** to detect falls intelligently
- **Identify safe positions** automatically based on player stability
- **Provide audio feedback** when players are respawned
- **Customize fall detection** with configurable thresholds

**Important**: This component uses intelligent fall detection that analyzes player movement patterns rather than just height thresholds. It waits for players to be stable before saving safe positions and only respawns when it detects actual falling behavior.

## Video Demo

<div style={{position: "relative", paddingBottom: "56.25%", height: 0, overflow: "hidden", maxWidth: "100%", margin: "20px 0"}}>
  <iframe 
    src="https://www.youtube.com/embed/your-video-id" 
    style={{position: "absolute", top: 0, left: 0, width: "100%", height: "100%", border: 0}} 
    allowfullscreen>
  </iframe>
</div>

_Watch the Respawn component in action: intelligent fall detection, safe position tracking, automatic respawning, and audio feedback._

## Properties

### Basic Settings

| Property       | Type    | Default | Description                                       |
| -------------- | ------- | ------- | ------------------------------------------------- |
| `App ID`       | text    | Auto    | Unique name for this component (auto-generated)   |
| `Node Name`    | text    | `"RespawnZone"` | Name of the 3D object in your scene |
| `Debug Logs`   | toggle  | `false` | Show detailed information for troubleshooting     |
| `Add Collision`| toggle  | `true`  | Make the object solid (players can't walk through)|
| `Start Visible`| toggle  | `true`  | Should the object be visible when the scene loads?|

### Respawn Settings

| Property                    | Type    | Default | Description                                       |
| --------------------------- | ------- | ------- | ------------------------------------------------- |
| `Fall Y Threshold`          | number  | `4`     | Y coordinate difference to consider as a fall (triggers respawn) |
| `Position Tracking Interval`| number  | `1`     | How often player positions are recorded in seconds |
| `Height History Length`     | number  | `5`     | Number of height samples to keep for analyzing movement |
| `Height Sampling Rate`      | number  | `0.2`   | How often to sample player movement in seconds (lower = more frequent) |
| `Safe Position Save Delay`  | number  | `1.5`   | Time to wait before saving a position as safe |
| `Significant Movement Threshold` | number | `0.1` | Minimum movement distance to consider as significant |

### Feedback Settings

| Property                    | Type    | Default | Description                                       |
| --------------------------- | ------- | ------- | ------------------------------------------------- |
| `Enable Sound`              | toggle  | `true`  | Play a sound when a player is respawned |
| `Sound URL`                 | text    | `""`    | URL to the sound file to play when respawning |
| `Sound Volume`              | number  | `0.5`   | Volume of the respawn sound (0-1) |

## How It Works

The Respawn component uses intelligent algorithms to detect falls and identify safe positions:

### 1. **Position Tracking**
- Continuously samples player height at configurable intervals
- Maintains a history of height changes for analysis
- Tracks player movement patterns over time

### 2. **Fall Detection**
- Analyzes height history to detect falling trends
- Only respawns when more than 75% of recent height changes are downward
- Prevents respawns during normal activities like walking down stairs

### 3. **Safe Position Identification**
- Waits for players to be stable (minimal height changes) for a configured time
- Ensures players aren't falling when saving a safe position
- Requires significant movement from the previous safe position

### 4. **Respawn Logic**
- Teleports players back to their last known safe position
- Plays audio feedback when respawning
- Logs respawn events for debugging

## Common Use Cases

### 1. **Elevated Worlds**
- Object: A respawn zone in an elevated world
- Event: Player falls below the world
- Action: Player is teleported back to last safe position

### 2. **Platforming Challenges**
- Object: A safety net for platforming areas
- Event: Player falls off platforms
- Action: Player is respawned to continue the challenge

### 3. **Void Prevention**
- Object: A respawn system for void areas
- Event: Player falls into void
- Action: Player is teleported to safety

### 4. **Inaccessible Area Prevention**
- Object: A respawn system for restricted areas
- Event: Player gets stuck in inaccessible area
- Action: Player is teleported to accessible location

### 5. **Safety Net Systems**
- Object: A general safety net for the world
- Event: Player falls or gets stuck
- Action: Player is respawned to safety

## Step-by-Step Setup

### 1. **Add the Component**
- Drag the Respawn component into your scene
- Select any 3D object (position doesn't matter)

### 2. **Configure Basic Settings**
- Set the `Node Name` to match your 3D object's name
- Choose if the object should `Start Visible`
- Decide if it should `Add Collision` (be solid)

### 3. **Set Up Respawn Parameters**
- Configure `Fall Y Threshold` (how far down triggers respawn)
- Set `Position Tracking Interval` (how often to check positions)
- Adjust `Height Sampling Rate` (how frequently to sample movement)

### 4. **Configure Safe Position Detection**
- Set `Height History Length` (how many samples to analyze)
- Configure `Safe Position Save Delay` (how long to wait before saving safe position)
- Adjust `Significant Movement Threshold` (minimum movement to consider significant)

### 5. **Set Up Audio Feedback**
- Enable `Enable Sound` if you want audio feedback
- Set `Sound URL` to your respawn sound file
- Configure `Sound Volume` for appropriate audio level

### 6. **Test Your Setup**
- Enable `Debug Logs` to see what's happening
- Test fall detection with different scenarios
- Verify that safe positions are being saved correctly

## Tips for Designers

### **Configuring Fall Detection**
- Start with default values and adjust based on your world
- Higher `Fall Y Threshold` = less sensitive to small falls
- Lower `Height Sampling Rate` = more responsive detection
- Test with different types of movement (stairs, slopes, platforms)

### **Planning Your Respawn System**
- Think about where players should be respawned
- Consider the layout of your world and safe areas
- Plan for both single-player and multiplayer scenarios
- Design clear visual cues for respawn zones

### **Testing Your Setup**
- Always test with multiple players
- Use debug logs to troubleshoot issues
- Test edge cases (what happens if players disconnect?)
- Verify that respawn behavior feels natural

### **Performance Considerations**
- Position tracking runs continuously - consider impact on performance
- Higher sampling rates provide better detection but use more CPU
- Consider the impact on mobile devices
- Test with different numbers of players

## Troubleshooting

### **Players Not Being Respawned**
- Check that `Fall Y Threshold` is appropriate for your world
- Verify that `Height Sampling Rate` is not too high
- Enable `Debug Logs` to see what's happening
- Test with different fall scenarios

### **Respawn Too Sensitive**
- Increase `Fall Y Threshold` to make it less sensitive
- Adjust `Height History Length` to require more consistent falling
- Check that `Significant Movement Threshold` is appropriate
- Test with normal movement (stairs, slopes)

### **Safe Positions Not Being Saved**
- Check that `Safe Position Save Delay` is not too short
- Verify that `Significant Movement Threshold` is appropriate
- Ensure players are stable before positions are saved
- Use debug logs to see position saving behavior

### **Audio Not Playing**
- Check that `Enable Sound` is enabled
- Verify that `Sound URL` is correct and accessible
- Test with different sound files
- Check that `Sound Volume` is greater than 0

### **Performance Issues**
- Reduce `Height Sampling Rate` to decrease CPU usage
- Increase `Position Tracking Interval` to reduce frequency
- Consider the impact on mobile devices
- Test with different numbers of players

## Best Practices

- **Start with Defaults**: Use default values and adjust based on your world
- **Test Thoroughly**: Always test with multiple players and scenarios
- **Use Debug Logs**: Enable debug mode during development
- **Consider World Layout**: Adjust thresholds based on your world's design
- **Test Edge Cases**: Verify behavior with different movement types
- **Monitor Performance**: Watch for performance impact with many players
- **Provide Audio Feedback**: Use sound to inform players of respawns
- **Document Settings**: Keep notes on your respawn configurations
- **Test on Mobile**: Verify behavior on different devices
- **Consider Accessibility**: Ensure respawn behavior is fair for all players

## Advanced Features

### **Intelligent Fall Detection**
The component doesn't just respawn players when they go below a certain height. Instead, it analyzes the player's movement history to determine if they're actually falling:
- Tracks the player's height over time
- Analyzes the trend to determine if the player is falling
- Prevents respawns during normal activities like walking down stairs

### **Stable Position Detection**
The component intelligently determines when a position should be considered "safe" for respawning:
- Waits until the player has been stable for a configured time
- Ensures the player isn't falling when saving a safe position
- Requires significant movement from the previous safe position

### **Configurable Audio Feedback**
- Plays customizable sound when respawning
- Supports different sound files and volumes
- Provides audio feedback for respawn events
