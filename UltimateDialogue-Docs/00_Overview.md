# Overview

Ultimate Dialogue lets you compose conversations with StateTree assets and control them from Blueprints. A dialogue can display lines and choices, evaluate conditions and variables, emit events, and then finish or be cancelled.

## What you can do

- Create lines, choices, conditional branches, and endings.
- Raise Blueprint events from dialogue steps.
- Use variables local to a conversation or global to the game.
- Set speaker names, portraits, and emotions.
- Select a supplied widget and theme, or customize the UI in UMG.
- Start dialogue automatically or when the player interacts nearby.
- Observe prompts, choices, events, and end reasons in a Blueprint.

## Guides

- Start with [installation and settings](01_Installation_and_Setup.md).
- Follow the [Blueprint quickstart](02_Blueprint_Quickstart.md).
- Review the [StateTree tasks](03_Authoring_Dialogue_StateTrees.md).
- Customize the [UI](04_UI_and_Themes.md) and [Blueprint events and variables](05_Blueprint_Reference.md).
- Use the [examples and troubleshooting guide](06_Examples_and_FAQ.md), followed by the [progressive walkthrough](07_Progressive_Walkthrough.md) and [concrete Blueprint recipes](08_Blueprint_Recipes.md).

## Compatibility and important limits

- The plugin is configured for Unreal Engine 5.7 and uses the StateTree feature supplied with the editor.
- A dialogue state should contain one enabled `Dialogue:` task. The first state with an enabled dialogue task, in StateTree order, is the entry point.
- Dialogue transitions use the `State` fields on `Dialogue:` tasks. Each required link must point to a state with an enabled dialogue task.
- The supplied widget handles its own keyboard, mouse, and gamepad controls. A custom widget must handle its own controls and focus behavior.
- Global values are saved across sessions only when you call the provided save and load nodes.
