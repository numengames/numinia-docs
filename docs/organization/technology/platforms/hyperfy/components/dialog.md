---
sidebar_position: 8
id: dialog-component
title: Dialog Component
sidebar_label: Dialog
---

# Dialog Component

The Dialog component provides an interactive dialog system for character conversations with support for multiple lines, character names, auto-advance, and text splitting. Think of it as a comprehensive conversation system that can display text-based dialogues with characters, supporting complex conversations and seamless integration with other components.

**Important**: This component creates a floating dialog UI that appears when triggered and can display multiple lines of text with character names. It supports both manual advancement and auto-advance modes, with full control over appearance and behavior.

**[Download Component](http://statics.numinia.xyz/hyperfy-components/dialog.hyp)**

## What Does It Do?

The Dialog component is like having a conversation system for your 3D world. You can:
- **Display character conversations** with multiple lines of text
- **Show character names** above dialog text for immersion
- **Control text advancement** manually or automatically
- **Customize dialog appearance** with colors, fonts, and sizing
- **Sync conversations** across all players in multiplayer environments
- **Chain actions** by triggering other components when dialogs complete
- **Handle complex conversations** with multiple phrases and characters

**Important**: This component creates a floating dialog interface that appears when activated. Players can advance through dialog lines manually or automatically, and the component handles text display without automatic splitting - users control dialog length.

## Video Demo

<div style={{position: "relative", paddingBottom: "56.25%", height: 0, overflow: "hidden", maxWidth: "100%", margin: "20px 0"}}>
  <iframe 
    src="https://www.youtube.com/embed/your-video-id" 
    style={{position: "absolute", top: 0, left: 0, width: "100%", height: "100%", border: 0}} 
    allowfullscreen>
  </iframe>
</div>

_Watch the Dialog component in action: character conversations, text advancement, customizable UI, and synchronized effects across all players._

## Properties

### Basic Settings

| Property       | Type    | Default | Description                                       |
| -------------- | ------- | ------- | ------------------------------------------------- |
| `App ID`       | text    | Auto    | Unique name for this component (auto-generated)   |
| `Node Name`    | text    | `""`    | **Required**: Name of the 3D object in your scene |
| `Debug Logs`   | toggle  | `false` | Show detailed information for troubleshooting     |
| `Add Collision`| toggle  | `true`  | Make the object solid (players can't walk through)|
| `Start Visible`| toggle  | `true`  | Should the object be visible when the scene loads?|

### Dialog Settings

| Property                    | Type    | Default | Description                                       |
| --------------------------- | ------- | ------- | ------------------------------------------------- |
| `Sync Changes`              | toggle  | `false` | Should all players see dialog changes at the same time? |
| `Use Delay`                 | toggle  | `false` | Add a wait time before showing dialogs? |
| `Delay (sec)`               | number  | `0`     | How many seconds to wait before showing dialog (0.1 to 60) |
| `Dialog Width`              | number  | `400`   | Width of the dialog box in pixels (10 to 1200) |
| `Dialog Height`             | number  | `200`   | Height of the dialog box in pixels (5 to 800) |
| `Dialog Scale`              | number  | `1.0`   | Scale factor for the dialog (0.1 to 2.0) |
| `Dialog Background`         | color   | `#00000080` | Background color with transparency |
| `Dialog Border`             | color   | `#FFFFFF80` | Border color with transparency |
| `Border Radius`             | number  | `10`    | Corner rounding in pixels (0 to 20) |

### Text Settings

| Property                    | Type    | Default | Description                                       |
| --------------------------- | ------- | ------- | ------------------------------------------------- |
| `Text Color`                | color   | `#FFFFFF` | Color of the dialog text |
| `Font Size`                 | number  | `16`    | Size of the dialog text in pixels (8 to 24) |
| `Font Weight`               | select  | `normal` | Normal or bold text |
| `Show Character Name`        | toggle  | `true`  | Display character name above dialog |
| `Character Name`             | text    | `""`    | Name of the character speaking |
| `Name Label Color`           | color   | `#FFFF00` | Color of the character name |
| `Name Label Size`            | number  | `18`    | Size of the character name in pixels (8 to 24) |

### Content Settings

| Property                    | Type    | Default | Description                                       |
| --------------------------- | ------- | ------- | ------------------------------------------------- |
| `Auto Advance`              | toggle  | `false` | Automatically advance to next dialog line |
| `Dialog Duration`           | number  | `3`     | Time to wait before auto-advancing (1 to 30 seconds) |
| `Default Dialog Phrases`    | textarea| `[]`    | Default phrases to show when no content provided |

### Event Settings

| Property                    | Type    | Default | Description                                       |
| --------------------------- | ------- | ------- | ------------------------------------------------- |
| `Accept Any Event`          | toggle  | `false` | Listen to any signal sent to this component? |
| `Event Listeners`           | textarea| `[]`    | List of events that trigger dialog changes |

## How Events Work

The Dialog component listens for "signals" (events) from other components. When it receives a signal, it can show/hide dialogs or advance through dialog lines. The component uses its unique App ID (generated by Hyperfy) to identify itself in the event system.

### Event Format

In the `Event Listeners` field, you can specify events like this:

```json
[
  {
    "id": "a1b2c3d4e",
    "actions": [
      {
        "type": "next-dialog",
        "params": {
          "dialogText": "Hello, welcome!",
          "characterName": "Guide",
          "delay": 1.5
        }
      }
    ]
  }
]
```

**Note**: When referencing this component in events from other components, use the App ID that Hyperfy generated for this component. Also, when duplicating components, be sure you make the app unique, so it has a new AppID.

### Action Types

| Action | Description | Parameters |
|--------|-------------|------------|
| `next-dialog` | Show dialog or advance to next line | `dialogText`, `dialogPhrases`, `characterName`, `delay` |
| `hide-dialog` | Hide the currently visible dialog | None |

### Event Parameters

When an event triggers a dialog change, you can control:

| Parameter      | Type    | Description                                       |
| -------------- | ------- | ------------------------------------------------- |
| `dialogText`   | string  | Single dialog text to show |
| `dialogPhrases`| array   | Multiple dialog phrases to show sequentially |
| `characterName`| string  | Character name to display |
| `delay`        | number  | Seconds to wait before showing dialog |

## Common Use Cases

### 1. **Character Conversations**
- Object: An NPC character
- Event: Player approaches character
- Action: Character starts conversation with multiple lines

### 2. **Tutorial Messages**
- Object: A tutorial trigger
- Event: Player enters tutorial area
- Action: Tutorial dialog appears with instructions

### 3. **Story Progression**
- Object: A story trigger
- Event: Player completes objective
- Action: Story dialog advances the narrative

### 4. **Help System**
- Object: A help button
- Event: Player needs assistance
- Action: Help dialog appears with guidance

### 5. **Interactive Narratives**
- Object: Multiple story elements
- Event: Player explores different areas
- Action: Different dialogs reveal story elements

## Step-by-Step Setup

### 1. **Add the Component**
- Drag the Dialog component into your scene
- Select the 3D object you want to associate with dialogs

### 2. **Configure Basic Settings**
- Set the `Node Name` to match your 3D object's name
- Choose if the object should `Start Visible`
- Decide if it should `Add Collision` (be solid)

### 3. **Set Up Dialog Appearance**
- Configure `Dialog Width` and `Dialog Height` for your UI
- Set `Dialog Background` and `Dialog Border` colors
- Choose `Text Color` and `Font Size` for readability

### 4. **Configure Character Settings**
- Enable `Show Character Name` if you want character names
- Set `Character Name` for the default character
- Choose `Name Label Color` and `Name Label Size`

### 5. **Set Up Content Behavior**
- Enable `Auto Advance` for automatic progression
- Set `Dialog Duration` for auto-advance timing
- Add `Default Dialog Phrases` for fallback content

### 6. **Configure Events**
- In `Event Listeners`, add the events that should trigger dialogs
- Use the JSON format shown above
- Replace event names with actual events from other components

### 7. **Test Your Setup**
- Enable `Debug Logs` to see what's happening
- Trigger your events and watch dialogs appear
- Test both manual and auto-advance modes

## Tips for Designers

### **Writing Good Dialog**
- Keep dialog text concise and engaging
- Use character names to create immersion
- Test dialog length to ensure it fits the UI
- Consider the pacing of conversations

### **Planning Your Dialog System**
- Think about when players should see dialogs
- Consider timing and delays for dramatic effects
- Plan for both single-player and multiplayer scenarios
- Design clear visual cues for dialog triggers

### **Testing Your Setup**
- Always test with multiple players
- Use debug logs to troubleshoot issues
- Test edge cases (what happens if players disconnect?)
- Verify dialog text displays correctly

### **Performance Considerations**
- Dialogs are lightweight - minimal performance impact
- Consider the impact on player experience
- Test with different screen sizes and devices

## Troubleshooting

### **Dialog Not Showing**
- Check that the event is being sent correctly
- Verify that the `Event Listeners` format is correct
- Enable `Debug Logs` to see what's happening
- Test with `Accept Any Event` enabled

### **Text Display Issues**
- Check that dialog text is not too long for the UI
- Verify that text formatting is appropriate
- Ensure special characters are properly handled
- Test with different font sizes

### **Character Names Not Showing**
- Ensure `Show Character Name` is enabled
- Check that `Character Name` is set
- Verify `Name Label Color` and `Name Label Size` settings
- Test with different character names

### **Auto-Advance Not Working**
- Ensure `Auto Advance` is enabled
- Check that `Dialog Duration` is greater than 0
- Verify that multiple dialog phrases are provided
- Test with different duration settings

### **Events Not Triggering**
- Check that the event `id` matches exactly
- Verify the Event Listeners JSON format is correct
- Test with `Accept Any Event` enabled
- Use debug logs to see event reception

## Best Practices

- **Start Simple**: Test with basic dialogs before adding complexity
- **Use Character Names**: Always specify character names for better immersion
- **Control Dialog Length**: Users are responsible for keeping dialog text appropriate
- **Set Appropriate Delays**: Use delays sparingly and only when needed
- **Test Dialog Display**: Verify that dialog text displays correctly
- **Use Auto-Advance Sparingly**: Only use auto-advance for non-interactive sequences
- **Provide Manual Control**: Allow users to advance dialogs manually for important content
- **Configure Event Listeners**: Use specific event listeners for better control
- **Use Accept Any Event**: Enable this for simple setups where any component can trigger dialogs
- **Test Thoroughly**: Always test with multiple players and scenarios
