# Examples and troubleshooting

For complete StateTree and Blueprint graph setups, see the [concrete Blueprint recipes](08_Blueprint_Recipes.md).

## Example: conditional choice

1. Set the global variable before `Start Dialogue`, or call `Set Bool` on the instance while a line is visible and before the branch is reached.
2. Add `Dialogue: Branch` with `Use Condition` enabled.
3. In `Condition`, select the variable, type, operator, and comparison value.
4. Connect `True State` to the positive line and `False State` to the alternative.

If the value must exist before the first task runs, initialize a global variable before starting or place a `Set Variable` task before the branch.

## Example: respond to a choice

1. In a `Dialogue: Choice` state, add an entry for each response and connect each `Target State`.
2. In the target states, add the appropriate `Dialogue: Set Variable` or `Dialogue: Trigger` tasks.
3. In the Blueprint that starts the dialogue, bind `On Choice Selected` to receive the confirmed visible text and index.
4. Bind `On Trigger` if your logic should react to an event name from a branch.

Use `Scope = Global Dialogue Store` or `Local + Global` to write progression that lasts across conversations. Call the global save nodes if it must survive the game session.

## Example: start nearby

1. Assign the StateTree in the `DialogueStarter` Details panel.
2. Enable `Interaction Enabled` and set `Interaction Radius` (initial value: `200` cm).
3. Set `Auto Start` as desired; auto-start is suppressed while interaction is enabled.
4. Call `Try Interact` on the actor from the Blueprint that handles player input.
5. Use `On Player Enter Range` and `On Player Exit Range` to show or hide your own interaction prompt.

## FAQ and common issues

### Dialogue does not start

- Confirm that `Dialogue State Tree` references a compiled asset.
- Confirm that `Schema` is `Dialogue StateTree Schema`.
- Confirm that a state has an enabled `Dialogue:` task and that it is the intended entry state.
- Check required links and their targets. A target must contain an enabled `Dialogue:` task.
- With `DialogueStarter`, check `Auto Start` and `Interaction Enabled`. Auto-start only occurs when interaction is disabled.

### Dialogue starts but no UI appears

- With `DialogueStarter`, check `Show Default Widget`.
- From a Blueprint, call `Show Default Dialogue Widget` after `Start Dialogue`, or use `Start Dialogue And Show Default Widget`.
- Check `Default Widget Class`. A custom widget derived from `DialogueWidgetDefault` needs a `ChoicesBox` to display choices.

### My custom widget misses the first line

The initial prompt may be emitted before the widget binds. Call `Reemit Current Prompt` after `Bind To Dialogue`. The standard display helper already re-emits it.

### A line does not advance automatically

`Auto Advance After` defaults to `-1`. Without `Voice Over`, the player must confirm manually. A value of `0` also disables automatic advance.

### A choice is hidden or cannot be selected

Check `Enabled`, the `Condition`, and initialized variables. Indices are assigned after filtering; index `0` is the first visible choice. Keyboard keys `1` to `9` cover only the first nine choices.

### A timeout selects an unexpected response

`Choice Timeout` is disabled at `-1`. When enabled, `Choice Default Index` refers to the filtered visible list and starts at `0`.

### `Escape` does not close the conversation

This is the supplied widget's expected behavior. `Escape` and the gamepad right face button only play the optional `SFX Back` sound. Call `Cancel` on the instance or `Cancel Active Dialogue`.

### A `Trigger` event does not run my logic

Check that the Blueprint listens to `On Trigger` on the right instance and compares the exact `Event Name`. For `Event Action Class`, confirm that a class is assigned and its `Execute Dialogue Event` is implemented.

### `Try Interact` does nothing

`Interaction Enabled` must be on, and a player-controlled pawn must be within the radius. Call `Try Interact` from your input logic; the plugin does not choose the project's interaction key.

### The dialogue ends with `Loop Guard`

Too many tasks without a visible prompt are chained together. Check the routes from `Branch`, `Jump`, `Random`, and `Trigger`; increase `Max Auto Advance Steps` only when the longer chain is intentional.

## Keep in mind

- `Dialogue: Comment` is for annotations; do not put it on a playable path.
- An emotion is sent through `On Emotion Changed`; it does not automatically change an animation or portrait.
- Dialogues in different slots can coexist, but their widgets must be managed separately.
- `On Ended` reports why the dialogue ended; `On Finished` does not provide a reason.
