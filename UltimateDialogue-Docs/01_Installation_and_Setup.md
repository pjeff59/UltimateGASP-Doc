# Installation and setup

## Requirements

- Unreal Engine 5.7.
- An Unreal project with access to the Content Browser and Blueprints.
- Ultimate Dialogue in the project's `Plugins` folder or installed in the editor.

The plugin declares its dependency on StateTree, a feature included with Unreal Engine. No additional tools are required.

## Install and enable

1. If needed, place the `UltimateDialogue` folder in `<YourProject>/Plugins/`.
2. Open the project in Unreal Editor.
3. Open `Edit > Plugins`, search for `Ultimate Dialogue`, and enable it.
4. Restart the editor if prompted.
5. Confirm that the plugin folders and assets appear in the Content Browser.

## Project settings

Open `Edit > Project Settings`, then select the `Ultimate Dialogue` section. The following settings are available. Values shown are supplied defaults unless noted otherwise.

| Setting | Initial value | Effect |
|---|---:|---|
| `Default Widget Class` | `WBP_Dialogue_Ready` | Widget created by `Show Default Dialogue Widget`. If the class is missing or cannot be loaded, the base widget is used. |
| `Default Theme` | None | Optional shared theme used by the supplied widget unless a `Theme Override` is set. |
| `Speaker Database` | None | Optional source for speaker names, portraits, and related data. |
| `Enable Global Variables` | Enabled | Allows use of the global variable store. |
| `Fallback To Global Variables` | Enabled | A condition can look up a global variable if the corresponding local variable does not exist. |
| `Mirror Local Set Variables To Global` | Disabled | When enabled, local `Set Variable` tasks also copy their value to the global store. |
| `Max Auto Advance Steps` | `128` | Limits non-presenting tasks that run consecutively before a line, choice, or ending. |
| `SFX Move`, `SFX Confirm`, `SFX Back` | None | Optional sounds for interactions with the supplied widget. |
| `Enable Haptics` | Enabled | Allows haptic feedback from the supplied widget on supported platforms. |
| `Haptic Strength` | `0.25` | Haptic strength, from `0` to `1`. |
| `Haptic Duration` | `0.05` seconds | Haptic duration, from `0.01` to `1` second. |

The theme and speaker database are optional. Values set directly on a `Dialogue: Line` task take precedence over speaker database values.

## Recommended first setup

1. Keep `Default Widget Class` set to `WBP_Dialogue_Ready` to start.
2. Create a `DialogueTheme` asset if you want to change colors or scale, then assign it to `Default Theme`.
3. Create a `Speaker Database` only if multiple conversations should share speaker names and portraits.
4. Leave global variables enabled if conversations need shared progression. Call the explicit save and load nodes to keep those values between sessions.
5. Create your first StateTree by following the [quickstart](02_Blueprint_Quickstart.md).

## Disable or remove

To stop using the plugin, disable `Ultimate Dialogue` in `Edit > Plugins` and restart the editor if prompted. Before removing the project folder, remove references to its assets from levels, widgets, and Blueprints.
