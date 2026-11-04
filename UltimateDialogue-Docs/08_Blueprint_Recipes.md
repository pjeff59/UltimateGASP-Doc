# Concrete Blueprint Recipes

These recipes extend the [first dialogue](02_Blueprint_Quickstart.md) and [progressive walkthrough](07_Progressive_Walkthrough.md) with reusable setups. Names in backticks match the tasks, nodes, events, and properties shown in Unreal Editor.

## Before you start

For each recipe that uses a StateTree:

1. Create a `StateTree` asset and select `Dialogue StateTree Schema` in `Schema`.
2. Create one state for each step below and add one enabled `Dialogue:` task to each state.
3. Confirm that the first dialogue state in StateTree order is the entry state.
4. Connect every `Next State`, `Target State`, `True State`, or `False State` field to a state with an enabled dialogue task.
5. Compile the StateTree and test every route, including false conditions and hidden choices.

Tasks without a UI prompt, such as `Dialogue: Trigger` and `Dialogue: Set Variable`, run immediately. `Max Auto Advance Steps` defaults to `128` to stop an accidental chain of invisible tasks.

## Recipe 1 — A guard filters choices by gate pass

This conversation changes its available response based on `Quest.HasGatePass`. Choosing to open the gate also raises a Blueprint event.

### 1. Set the variable before starting the dialogue

In the Blueprint that starts the conversation:

1. Call `Set Global Bool`.
2. Set `Name` to `Quest.HasGatePass`.
3. Set `Value` to `true` if the player already has the pass; otherwise set it to `false`.
4. Call `Start Dialogue And Show Default Dialogue Widget` with the StateTree.
5. Keep `Out Instance` and bind its `On Trigger` event before the player can select a response.

`Enable Global Variables` and `Fallback To Global Variables` are enabled by default. Global fallback lets conditions read this value as long as the instance does not contain a local variable with the same name.

### 2. Build the StateTree

Create these states and tasks:

| State | `Dialogue:` task | Setup and target |
|---|---|---|
| `Greeting` | `Line` | `Speaker Id`: `GateGuard`; `Speaker Display Name`: `Guard`; `Text`: `The gate is closed. Do you have a pass?`; `Next State`: `Offer`. |
| `Offer` | `Choice` | See the three entries below; leave `Choice Timeout` at `-1` to wait for the player. |
| `OpenGate` | `Trigger` | `Event Name`: `Gate.Open`; `Next State`: `Welcome`. |
| `Welcome` | `Line` | `Text`: `You may pass.`; `Next State`: `Finish`. |
| `AskAboutPass` | `Line` | `Text`: `The nearby depot can issue a pass.`; `Next State`: `Finish`. |
| `Leave` | `Line` | `Text`: `Come back whenever you are ready.`; `Next State`: `Finish`. |
| `Finish` | `End` | Ends the conversation. |

Add these entries to `Offer`:

| Text | `Condition` | `Target State` |
|---|---|---|
| `I have my pass.` | `Variable Name`: `Quest.HasGatePass`; `Value Type`: `Bool`; `Op`: `Equal`; `Compare Value`: `true`. | `OpenGate` |
| `Where can I get one?` | Same variable and type; `Op`: `Equal`; `Compare Value`: `false`. | `AskAboutPass` |
| `I will come back later.` | No condition. | `Leave` |

Entries whose conditions are false are not shown. The entry without a condition remains available. `On Choice Selected` and the index passed to `Advance` start at `0` in the list actually displayed, after filtering.

### 3. React to `Gate.Open`

In the Blueprint that starts the dialogue, use the instance's `On Trigger` event:

```text
On Trigger (Event Name)
  → Equal (Name): Gate.Open
  → Branch
      True → call your Blueprint gate-opening logic
```

Perform an immediate action here, such as enabling the gate. The task proceeds to `Welcome` right away; it does not wait for an animation or another asynchronous operation to finish.

### Checks

- If neither conditional response is shown, check the spelling of `Quest.HasGatePass`, its `Bool` type, and that it was set before the dialogue started.
- A local variable with the same name takes precedence over the global variable.
- If the player selects a response but the gate does not react, confirm that `On Trigger` is bound on the right instance and compares the exact name `Gate.Open`.

## Recipe 2 — Save dialogue progression between sessions

Global variables are shared across conversations in a session, but they are not saved automatically. This recipe loads the store before the first dialogue and saves it immediately after changing progression.

### 1. Load before starting

In the Blueprint that initializes the game, before calling `Start Dialogue`:

```text
Event BeginPlay
  → Load Global Dialogue Variables (Slot Name: DialogueVars)
  → Branch (Return Value)
      True  → allow dialogues to start
      False → handle the failure; initialize starting values only when a new game is intended
```

The default slot name is `DialogueVars`. A `false` return means loading failed; it cannot by itself distinguish a first playthrough from an unreadable progress file. For an existing save, report the failure instead of silently overwriting the values.

To initialize a new game, call `Set Global Bool` with `Name = Quest.HasGatePass` and `Value = false`, then allow dialogues to start.

### 2. Save a change

In the StateTree for the quest that grants the pass, add a `Dialogue: Trigger` task with `Event Name = Quest.GatePassGranted`. When the corresponding gameplay action succeeds:

```text
On Trigger (Event Name = Quest.GatePassGranted)
  → run and verify the pass-granting gameplay action
  → Branch (grant succeeded)
      True  → Set Global Bool (Quest.HasGatePass, true)
            → Save Global Dialogue Variables (Slot Name: DialogueVars)
            → Branch (Return Value)
                False → display or log the save failure
      False → do not set Quest.HasGatePass to true
```

Call `Save Global Dialogue Variables` after changing a value you need to keep. Check its Boolean return value; writing can fail. If you use `Dialogue: Set Variable` instead, select `Scope = Global Dialogue Store` or `Local + Global`, then save the store after the task.

### Checks

- Loading must finish before starting a dialogue whose conditions depend on these values.
- Keep variable names and types consistent across loading, conditions, and writes.
- The global store does not replace real quest or inventory state: set the Boolean only after confirming the corresponding gameplay action.

## Recipe 3 — Start by pressing a key near an actor

`DialogueStarter` provides proximity detection, but it does not choose the project's interaction key.

### 1. Configure the actor

1. Place `DialogueStarter` in the level and assign the StateTree to `Dialogue State Tree`.
2. Set `Auto Start` to `false`.
3. Leave `Show Default Widget` enabled to display the widget selected by `Default Widget Class`.
4. Enable `Interaction Enabled` and set `Interaction Radius`, for example to `250` cm for this test.

With `Interaction Enabled`, auto-start is suppressed. A player-controlled pawn must enter the radius.

### 2. Connect proximity and player input

In a Blueprint that can reference this `DialogueStarter`:

1. Bind `On Player Enter Range` to show a hint such as `Talk`.
2. Bind `On Player Exit Range` to hide the hint.
3. From the input event already configured in the project for the interaction key, call `Try Interact` on this `DialogueStarter` reference.

`Try Interact` starts the dialogue only when `Interaction Enabled` is on and a player-controlled pawn is in range. The plugin does not bind a key automatically. For multiple actors, your Blueprint must choose and call the intended actor reference; these events do not select the nearest target.

### Checks

- Test in play mode with a pawn actually controlled by the player, not only with an actor simulated in the editor.
- Confirm that the radius is at least `50` cm and that `Dialogue State Tree` is assigned.
- If the dialogue starts but the UI is missing, check `Show Default Widget` and `Default Widget Class`.

## Recipe 4 — Custom UI with choices and line voice-over

This recipe creates a UMG interface independent of the supplied widget. It displays lines and visible choices, and plays the voice assigned to a line.

### 1. Create the two widgets

Create `WBP_Dialogue_Custom`, derived from `DialogueWidget`, with:

- a `TextBlock` named `SpeakerLabel`;
- a `TextBlock` named `LineLabel`;
- a `VerticalBox` named `ChoicesList`;
- a `DialogueInstance` variable of type `Dialogue Instance`, exposed on widget creation;
- an `AudioComponent` variable to keep track of the current sound.

Also create `WBP_Dialogue_ChoiceRow`, a Widget Blueprint containing a `Button` and a `TextBlock`. Add:

- an integer variable named `ChoiceIndex`;
- an event dispatcher named `ChoiceClicked` with one integer parameter;
- a `Setup Choice` function that receives the index and text, stores them, and updates the `TextBlock`;
- on `Button.OnClicked`, a call to `ChoiceClicked` with `ChoiceIndex`.

### 2. Start and bind the UI

In the Blueprint that starts the dialogue:

```text
Start Dialogue (StateTree) → Instance
Create Widget (WBP_Dialogue_Custom, DialogueInstance = Instance)
  → Add to Viewport
  → Bind To Dialogue (Instance)
  → Reemit Current Prompt (Instance)
```

Keep the same instance in `WBP_Dialogue_Custom`. `Reemit Current Prompt` matters because the first prompt may have been emitted before the widget was created and bound. Also manage focus and input mode for this custom interface.

### 3. Display each prompt

Implement `On Prompt Received` in `WBP_Dialogue_Custom`:

```text
On Prompt Received (Prompt)
  → Clear Children (ChoicesList)
  → Branch (Prompt.bIsChoice)
      False → display Prompt.CurrentLine.SpeakerDisplayName (or Speaker Id when empty) and Prompt.CurrentLine.Text
      True  → For Each Loop (Prompt.Choices)
                → Create Widget (WBP_Dialogue_ChoiceRow)
                → Setup Choice (Array Index, Array Element.Text)
                → Bind Event to ChoiceClicked
                → Add Child to Vertical Box (ChoicesList)
```

Create a `Handle Choice Clicked(Index)` event in the parent widget and bind every row to it. Its body calls `Advance` on `DialogueInstance` with `Index`. `Prompt.Choices` already contains the available choices; use the `Array Index` of that visible list, not the index from the unfiltered StateTree entries.

Implement `On Dialogue Finished` to stop the current sound if necessary, then call `Remove From Parent`.

### 4. Play voice and set automatic advance

The `Voice Over` field on `Dialogue: Line` provides a sound and its duration, but the plugin does not play that sound automatically. In the `False` path of `Prompt.bIsChoice`:

1. Stop the previous `AudioComponent` if it is still valid.
2. If `Prompt.CurrentLine.VoiceOver` is valid, call `Spawn Sound 2D` with that value and store the returned `AudioComponent`.
3. Leave `Auto Advance After` at `-1` to use the `Voice Over` duration. Without a `Voice Over`, the player must advance manually.

A positive `Auto Advance After` value overrides the voice duration. A value of `0` also waits for manual confirmation. If an explicit duration is shorter than the sound, stop the sound when the prompt changes so it does not continue over the next line.

### Checks

- If the text does not update, check that `Bind To Dialogue` receives the instance returned by `Start Dialogue`.
- If clicks select the wrong response, pass the index from `Prompt.Choices`, not the original entry index before filtering.
- If the sound does not play, check `Voice Over` and the `Spawn Sound 2D` logic: assigning the asset does not play it by itself.
- The supplied widget handles its own controls; a custom widget must provide its own navigation, confirmation, cancellation, and focus handling.

## Mini recipe — Choice timeout

In a `Dialogue: Choice` task, for example set:

- `Choice Timeout`: `8.0` seconds;
- `Choice Default Index`: `0`.

After eight seconds, the runtime confirms the default choice from the filtered list shown to the player. `Choice Timeout = -1` disables the timeout. If filtering leaves no entries available, no automatic choice can be confirmed.

## Next steps

- For conditions and state links, see [Authoring a dialogue with StateTree](03_Authoring_Dialogue_StateTrees.md).
- For instance nodes, variables, and events, see [Blueprint events and variables](05_Blueprint_Reference.md).
- For common issues, see [Examples and troubleshooting](06_Examples_and_FAQ.md).
