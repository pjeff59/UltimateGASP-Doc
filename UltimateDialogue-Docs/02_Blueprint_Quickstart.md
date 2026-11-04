# Blueprint quickstart

This guide creates two lines followed by an ending, then displays them with the supplied widget.

## 1. Create a StateTree

1. In the Content Browser, create a `StateTree` asset, for example `ST_Dialogue_Greeting`.
2. Open it and set its `Schema` to `Dialogue StateTree Schema`.
3. Add three dialogue states in this order: `Greeting`, `Reply`, `Finish`.
4. Add one enabled `Dialogue:` task to each state.

The first state with an enabled dialogue task is the entry point. The tasks and their links define the conversation flow.

## 2. Configure tasks and links

- In `Greeting`, add `Dialogue: Line`.
  - `Speaker Id`: `Guard`
  - `Text`: `Welcome. What brings you here?`
  - `Next State`: `Reply`
- In `Reply`, add `Dialogue: Line`.
  - `Speaker Id`: `Guard`
  - `Text`: `You may enter.`
  - `Next State`: `Finish`
- In `Finish`, add `Dialogue: End`.

Save the asset and resolve any compile errors. See the [StateTree authoring guide](03_Authoring_Dialogue_StateTrees.md) for other fields and task types.

## 3. Start the dialogue in a level

1. Open a level and place a `DialogueStarter` actor.
2. In its Details, assign `ST_Dialogue_Greeting` to `Dialogue State Tree`.
3. For this test, leave `Auto Start` and `Show Default Widget` enabled, and `Interaction Enabled` disabled.
4. Press `Play`.

The dialogue starts when play begins, displays the first line, and then advances to the next. The `Finish` state ends the conversation and closes the widget.

## 4. Start from a Blueprint

In a `Player Controller`, `Level Blueprint`, or another suitable Blueprint:

1. Use `Get Game Instance Subsystem` and select `Dialogue Subsystem`.
2. Call `Start Dialogue` with the StateTree.
3. Pass the returned instance to `Show Default Dialogue Widget`.

Use `Start Dialogue And Show Default Widget` to combine the start and display steps. Keep its `Out Instance` if you need to listen for events, pause, or cancel the conversation.

## 5. Supplied widget controls

- `E`, `Space Bar`, or `Enter`: advance a line or confirm the selected choice.
- `1` to `9`: select one of the first nine visible choices directly.
- Up/down arrows, D-pad, or stick: move the selection.
- Mouse hover and click: select a choice.
- Gamepad bottom face button: confirm.
- `Escape` or gamepad right face button: plays `SFX Back` if assigned; it does not cancel the conversation.

To cancel, explicitly call `Cancel` on the instance or `Cancel Active Dialogue` on `Dialogue Subsystem`.
