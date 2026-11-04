# Blueprint events and variables

This reference covers the nodes and events available to your Blueprints.

## Start, display, and control

- `Get Game Instance Subsystem` → `Dialogue Subsystem` provides access to `Start Dialogue`.
- `Start Dialogue` starts a StateTree and returns an instance. In the default slot, starting a new dialogue cancels the previous one.
- `Show Default Dialogue Widget` creates the class in `Default Widget Class`, focuses it, sets UI input mode, and re-emits the current prompt.
- `Start Dialogue And Show Default Widget` combines starting and displaying, and returns `Out Instance`.
- `Advance` advances a line with index `-1`, or confirms a choice with its visible index starting at `0`.
- The Blueprint library provides `Pause Dialogue`, `Resume Dialogue`, and `Is Dialogue Paused`; the instance also exposes `Pause`, `Resume`, `Cancel`, and `Reemit Current Prompt`.
- The subsystem provides `Cancel Active Dialogue`, `Cancel Dialogue In Slot`, `Cancel All Dialogues`, `Get Active Dialogue`, `Get Dialogue In Slot`, `Is Any Dialogue Active`, and `Is Slot Active`.

`Start Dialogue In Slot` starts an instance in a named slot. A new dialogue replaces the active one in the same slot; different slots can contain multiple dialogues. The node does not automatically create a separate UI for each slot.

## Instance events

| Event | Data received | Use |
|---|---|---|
| `On Prompt` | `Prompt` | Update a UI. `Prompt` distinguishes a line from a choice and contains text, visible choices, and `Node Id`. |
| `On Choice Selected` | `Choice Index`, `Choice Text`, `Choice Node Id`, `Target Node Id` | React to the choice actually confirmed, regardless of how it was selected. |
| `On Trigger` | `Event Name`, `Node Id` | Run Blueprint logic associated with a `Dialogue: Trigger` task. |
| `On Emotion Changed` | `Speaker Id`, `Emotion`, `Intensity` | React to a `Dialogue: Emotion` task. It does not automatically change animation or facial expression. |
| `On Ended` | `Reason`, `Last Node Id` | Handle completion or diagnose why the dialogue stopped. |
| `On Finished` | None | End event without a reason; prefer `On Ended` in new Blueprints. |

The displayed `Reason` values are `Completed`, `Cancelled`, `Invalid Asset`, `Invalid Node`, `Invalid Choice`, and `Loop Guard`.

The start operation can emit `On Prompt` before your Blueprint finishes binding events. Call `Reemit Current Prompt` after binding. The standard widget does this when it is displayed.

## `DialogueStarter` events

The actor Blueprint exposes:

- `On Dialogue Started` with the instance, created widget, and active StateTree.
- `On Dialogue Choice Selected` with the index, text, choice-state ID, and target-state ID.
- `On Dialogue Ended` with the reason and last-state ID.
- `On Dialogue Trigger` with the event name and state ID.
- `On Player Enter Range` and `On Player Exit Range` when proximity interaction is enabled.

In interaction mode, `Try Interact` starts dialogue only when a player-controlled pawn is inside the radius. The plugin does not bind your input action to a key; call `Try Interact` from the Blueprint that handles player input.

## Local and global variables

### Local variables

For a local instance, use the `Set Bool`, `Set Int`, `Set Float`, `Set Name`, `Set String`, `Get Bool`, `Get Int`, `Get Float`, `Get FName`, `Get String`, `Has Variable`, and `Remove Variable` nodes with that instance. Local variables are shared only within that instance.

In `Dialogue: Set Variable`, choose a scope:

- `Local Instance`: available in the instance.
- `Global Dialogue Store`: stored in the shared global store and available to other dialogues.
- `Local + Global`: writes to both scopes.

### Global variables

The `Set Global Bool/Int/Float/Name/String` and `Get Global Bool/Int/Float/Name/String` nodes access the global store. `Enable Global Variables` and `Fallback To Global Variables` are enabled by default. Global fallback is used when the local instance does not contain that variable.

Global values are available across conversations in the current game session, but are not automatically saved between sessions. Use `Save Global Dialogue Variables` and `Load Global Dialogue Variables` to save or restore the store. The default slot name is `DialogueVars`; check the returned Boolean to handle a failure.

### Conditions

`Branch` and `Choice` entries can test a variable. A missing variable or a type mismatch makes the condition false. Available comparisons depend on the variable type; see [condition types](03_Authoring_Dialogue_StateTrees.md#choices-and-conditions).

## Trigger a Blueprint action

Add `Dialogue: Trigger` and set an `Event Name`. The instance emits `On Trigger`; your Blueprint can compare that name and call the desired gameplay logic.

For reusable logic, create a Blueprint class derived from `Dialogue Event Action`, implement `Execute Dialogue Event`, and assign the class to `Event Action Class`. This optional action runs when the task is reached. You can also listen to `On Trigger` at the same time.

## Concurrent slots

Named slots are optional. To display multiple conversations at once, manage each instance and its UI separately; the default widget does not automatically choose which slot should be visible.
