# Ultimate Interaction: step-by-step Blueprint examples

[User guide](USER_GUIDE.md) | [Exemples en français](../FR/EXEMPLES.md) | [Home](../../README.md)

This collection provides independent recipes and variations. Names such as `BP_Door`, `IA_Interact`, `BlueKey` and `OpenDoor` are **examples you create**, not supplied assets. Bold labels identify editor or plugin elements. Diagrams describe connections; they are not text to paste into a graph.

## Shared preparation

First follow [Your first interaction](USER_GUIDE.md#your-first-interaction). For all examples:

1. Add an **Interaction Detector Component** and **Interaction Manager Component** to the playable character. Activate your **Input Mapping Context** and verify that `IA_Interact`, a Boolean **Input Action**, receives the chosen key.
2. Use manual input: **Started** calls **Start Interaction**; **Completed** and, if necessary, **Canceled** call **Stop Interaction** only for an active **Hold Interaction**. Leave the manager's **Interact Action** and **Input Mapping Context** empty to avoid double binding.
3. Give each object a mesh with query collision responding to **Visibility**, and an **Interactable Component**. For example, leave **Detection Radius** at `300` and **Interaction Distance** at `200` unless stated otherwise. Place the objects in the level.
4. In an object Blueprint, select its interactable and add its events from **Events**. A component reference also lets you call its nodes. `Self` means the Blueprint running the graph: in the character it is the character, in the door it is the door.
5. Compile, save and test with **Play**. Look **and** face toward the target: detection and start validation do not always use the same direction.

These steps describe what you must build; graphs, variables, widgets, inputs and animations are not created automatically. **On Interaction Completed** is broadcast before the component's final state update: do not assume **Current State** is already **Completed** inside that event, and save only after the interaction has fully finished.

## 1. Input, display and diagnosis

### Example 01 — Connect press and release

**Goal:** use one key for a door and a hold without cancelling a QTE.

1. In the character, connect `IA_Interact` **Started** to **Start Interaction** on the manager.
2. From **Completed**, call **Get Current Active Interaction** on the detector. Use **Is Valid** on the result; if valid, read its **Interaction Type** and compare it with **Hold Interaction**.
3. If the comparison is true, call **Stop Interaction** on the manager. Otherwise do nothing. Repeat this wiring on **Canceled** if your triggers can cancel the input.
4. To diagnose a refusal, you may replace **Start Interaction** with **Start Interaction Safe** and display its Boolean result. `False` indicates a refusal, not necessarily its cause.

**Test:** the door responds to a press; a hold cancels on release; releasing the start key must not cancel a QTE. Do not also assign the action in the manager's input settings.

### Example 02 — One on-screen prompt for the best target

**Goal:** display the best target's text without showing every nearby prompt.

1. Create `WBP_Prompt` with a **Text Block** `PromptLabel`. Enable **Is Variable** for it. In the widget, create your own `ShowPromptText` function taking a **Text**, updating the Text Block and making the interface visible.
2. In the character's **BeginPlay**, use **Create Widget**, store the reference as `PromptUI`, then **Add to Viewport** and hide it.
3. Bind an event to the detector's **On Best Interaction Changed** dispatcher. In that event, check whether the supplied interactable is valid. If valid, read **Interaction Prompt** and pass the text to `ShowPromptText`. Otherwise hide `PromptUI`.
4. Place a door with text `Open` and a button with `Activate`. Move between them and look away.

**Test:** the text follows the best target and disappears when none remains. An actor's range events do not prove it is the best target. If a prompt remains visible for an unusable object, inspect its conditions and cooldown: detection does not check every start condition.

### Example 03 — Explain why a door is locked

**Goal:** display a message when a detected target refuses interaction.

1. In the character, before calling **Start Interaction**, get **Get Current Interactable** from the manager and validate the reference.
2. On that reference, call **Can Interact** with the character as **Instigator**. If false, display **Out Fail Reason** in your widget; if true, call **Start Interaction Safe**.
3. Also check **Start Interaction Safe**'s Boolean result: even after a positive **Can Interact**, distance, orientation, a lock or a target change can prevent the start.

**Test:** with a correctly configured required-item condition, try without the key and then with it. A **Can Interact** refusal does not automatically broadcast **On Interaction Failed**. Do not use that event alone to display the locked-door message.

### Example 04 — Diagnose detection and orientation

1. Temporarily enable **Debug Draw Detection** and **Enable Debug Logs** on the detector, plus **Enable Runtime Debug Overlay** on the manager.
2. Test `150` units from a target set to a distance of `200`, with no wall between them. Then move more than `200` units away.
3. Look at the target with the camera while the character faces the opposite direction. Compare with the camera and character both facing it.
4. For an object usable regardless of orientation, disable **Require Look at** on its interactable and repeat the test.

**Result:** a target detected using the control direction can be rejected at start using the character's forward direction. Also check the radius, distance, collision channel and obstacles. Disable detailed diagnostics after testing.

## 2. Objects and mechanisms

### Example 05 — Door that opens and closes

**Create:** `BP_Door`, a scene component used as a pivot, a `Door` mesh parented to the pivot and an interactable. Place the pivot at the hinge and position the mesh relative to it.

1. Choose **Simple Interaction**, set **Interaction Prompt** to `Open or close`, and leave **Can Be Used Multiple Times** enabled.
2. Create `IsOpen` (Boolean, false) and a Timeline `TL_Door` with a float track from `0` to `90` between `0` and `0.6` seconds.
3. On **On Interaction Completed**, **Branch** on `IsOpen`: if false, **Play** the Timeline and set `IsOpen` true; if true, **Reverse** it and set the Boolean false.
4. On the Timeline's **Update**, use its value as yaw for **Set Relative Rotation** on the pivot. Adjust the sign for the hinge. If the animation should not be interrupted, set **Cooldown Duration** at least as long as the animation.

**Tests:** open, close, press twice rapidly and try from the other side. `IsOpen` and rotation are your Blueprint data: they are not automatically saved.

### Example 06 — Button that toggles a light

1. Create `BP_Button` with a mesh, a light component and a **Simple Interaction** interactable.
2. On **On Interaction Completed**, read your Boolean `LightOn`, invert it, then use **Set Visibility** on the light component with the new value.
3. Add a short **Cooldown Duration**, such as `0.5`, if multiple presses should not be accepted immediately.

**Tests:** two spaced presses toggle the light twice; the component does not restart during cooldown. The prompt may remain visible during that period.

### Example 07 — One-shot lever and reset

1. Create `BP_Lever` with **Simple Interaction** and disable **Can Be Used Multiple Times**.
2. On **On Interaction Completed**, rotate the mesh, then call your `ActivateMechanism` event using a valid reference to your mechanism.
3. After the interaction finishes, the component becomes **Completed**. For another puzzle cycle, call **Reset Saved State** on that component, then also restore the rotation and mechanism variables yourself.

**Tests:** a second press before reset does not restart the lever; after reset, it works again. Resetting the component does not reset your Blueprint variables.

### Example 08 — Generic collectible and full inventory

1. Create `BP_Item` with a mesh and an interactable of type **Pickup**. In the character, create integer variables such as `ItemCount` and `Capacity`.
2. Create your own character Blueprint function `TryAddItem`: if `ItemCount < Capacity`, increment the count and return true; otherwise return false.
3. In `BP_Item`, on **On Interaction Completed**, break the context, get **Instigator**, identify your character and call `TryAddItem`. If true, call **Destroy Actor** on the item; if false, display `Inventory full` and keep it.
4. Keep the interactable reusable so the player can retry after freeing a space.

**Tests:** with a free space, the count increases once and the item disappears. At capacity, no item is added or destroyed; after freeing a space, another press succeeds. Interaction completion itself does not grant an item.

### Example 09 — Custom Pickup Base with respawn

1. Create `BP_Collectible` derived from **Pickup Base**. In its settings, choose **Pickup Type** `Custom`, disable **Auto Use Mesh from Definition**, and assign **Manual Static Mesh**.
2. Disable **Auto Configure Interaction** and enter **Manual Interaction Prompt**. Override **Give Item to Collector**: call your own add logic on `Collector` and return its result. Without custom logic, a `Custom` pickup returns false.
3. Enable **Enable Respawn** and set **Respawn Delay** to `10`. Use **On Pickup Collected** for a success sound or message; do not treat interaction completion alone as proof that the item was granted.
4. For the built-in prompt, assign a widget class to `PromptWidget3D` and include a Text Block named `PromptText`. Add the Actor Tag `CanShowPickupWidget` to the character or clear **Required Tag for Widget Display**.

**Tests:** after a successful add, the object disappears and reappears after the delay; if the add fails, it must not be treated as collected. Test respawn and visuals with your mesh and widget.

### Example 10 — Hold to open a chest with progress

1. Create `BP_Chest` with a lid mesh, a **Hold Interaction** interactable, **Hold Duration** `2.0`, and **Can Be Used Multiple Times** disabled.
2. On **On Interaction Progress**, use `Progress` to calculate an angle from `0` to `90` and apply it to the lid with **Set Relative Rotation**. If you have a widget, send it the same value.
3. On **On Interaction Completed**, grant the contents through your own logic. On **On Interaction Cancelled**, return the lid and display to zero; grant no reward.

**Tests:** releasing before two seconds closes the chest; holding to completion opens it and grants the contents once. The plugin resets progress when cancelled: do not assume partial progress resumes.

### Example 11 — Accessible two-press hold

1. Duplicate the chest from example 10 and enable **Tap to Toggle Hold**.
2. Display your own instruction: `Press to start, press again to cancel`.
3. With the manager, press and release: progress continues. Press a second time before completion: the interaction cancels. If necessary, wait until the short duplicate-start lock expires before the second press.

**Tests:** release alone must not cancel; the second press must cancel; progress to `1` completes the interaction. This mode applies to the manager's **Hold Interaction**, not QTE **Hold**.

### Example 12 — Raise a gate with repeated presses

1. Create `BP_Gate` with **Progressive Interaction**, **Total Presses** `5`, **Time Window per Press** `1.0`, and **Reset on Timeout** enabled.
2. Store the mesh's initial relative location in `ClosedPosition`. On **On Interaction Progress**, multiply `Progress` by `200`, add it to the Z coordinate of `ClosedPosition`, and apply the relative location.
3. On **On Interaction Completed**, place the gate permanently at the top and trigger your puzzle action. Each accepted press emits **On Interaction Started**: do not play a one-time start sequence without your own guard.

**Tests:** five presses less than one second apart complete the interaction; after a pause of more than one second, the next press starts again at one out of five. Test at a pace slower than the manager's safety lock. With **Reset on Timeout** disabled, progress may decrease without a progress event for each decrease: read **Get Current Progress** to update a display that must follow the decay.

### Example 13 — Two-lever machine

1. Create `BP_Machine` with two meshes, two anchor scene components and two interactables. Assign each interactable an anchor and a distinct prompt.
2. Create `LeverAActive` and `LeverBActive`, both false. A's completion event sets A true; B's sets B true.
3. After each update, call your `CheckMachine` function. If both Booleans are true, activate your machine light or animation.

**Tests:** A alone and B alone do not start the machine; A then B, or B then A, starts it. Both points must be detectable and individually in range. If the puzzle must persist, also save your two Booleans.

## 3. Conditions, actions and selection

### Example 14 — Door with a consumable key

1. On the character, add the **Interaction Inventory Provider** interface in **Class Settings > Interfaces**. Create an integer `BlueKeyCount`, initially `0`.
2. Implement **Has Interaction Item**: for the name `BlueKey`, return whether `BlueKeyCount` is at least `Quantity`; return false for other names. Implement **Consume Interaction Item**: if the quantity is available, subtract it and return true; otherwise return false.
3. On the door, enable **Requires Item**, set **Required Item Name** to `BlueKey`, and enable **Consume Item**. On **On Interaction Completed**, play the opening from example 05.
4. Provide a way to give the character a key, such as example 08's item adapted to `BlueKeyCount`. Use example 03 to explain a refusal.

**Tests:** without a key the door does not start; with a key it opens and the count decreases by one; failed consumption must not open it. Without a compatible provider, or with an empty name, the check passes by default: that configuration is not a lock.

### Example 15 — Named condition: power restored

1. In `BP_Door`, add **Interaction Condition Provider** to the interfaces and create `PowerOn` (Boolean).
2. Implement **Evaluate Interaction Condition**: for **Condition Name** `PowerOn`, return the Boolean; return false for other names.
3. Add an entry to the interactable's **Additional Conditions**: **Type** `Named Condition`, **Target** `Owner`, **Condition Name** `PowerOn`, and a **Fail Reason** such as `Power is off`.
4. In your switch, set `PowerOn` on the door to true using a valid reference. Display the reason using example 03.

**Tests:** with power off, interaction is refused; with power on, it can start. Also test a missing interface: a named condition without a provider passes by default.

### Example 16 — Named action at the right phase

1. On an actor, implement **Interaction Action Receiver** and the **Handle Interaction Action** event.
2. In its interactable's **Interaction Actions**, add an entry: **Phase** `On Completed`, **Type** `Send Named Event`, **Target** `Owner`, **Action Name** `EnableIndicator`.
3. In **Handle Interaction Action**, compare the received name with `EnableIndicator`, then turn on your own indicator. Use the context if you need the instigator.

**Tests:** the indicator turns on after successful completion, not at start. Try another name and a target without the interface: no named action is handled. **Set Interaction Enabled** applied to an actor affects all its interactables; address a specific component to change only one.

### Example 17 — Shared preset for several buttons

1. Create a **Data Asset** of class **Interaction Preset**, such as `DA_Button`. Choose **Simple Interaction**, **Priority** `Normal`, and a common cooldown.
2. Assign the preset to **Interaction Preset** on two interactables. Keep **Apply Preset on Begin Play** enabled to apply it at start.
3. Give each actor its own **On Interaction Completed** logic. If the texts should differ, change them after preset application in your logic, or use distinct presets.

**Tests:** both buttons receive shared settings at start but execute their own events. A component prompt or condition set before play can be replaced by the preset.

### Example 18 — Priority and fallback between two objects

1. Place two visible interactables in range: a door with **Priority** `High` and a button with **Priority** `Normal`.
2. Temporarily disable **Enable Smart Fallback** on the detector. Observe the selected target, then swap the priorities.
3. Re-enable **Enable Smart Fallback**, move significantly closer to the button and compare the result with **Fallback Distance Bias** `60`.

**Tests:** without fallback, available candidates are sorted first by priority, then distance. With fallback, the nearest candidate can replace a higher-priority target when the distance difference exceeds the bias. A failed condition does not necessarily exclude a target from detection.

## 4. Inspection and QTEs

### Example 19 — Inspect a book

1. Add an **Interaction Inspect Manager Component** to the character. Create `BP_Book` with its root and mesh set to **Movable**, a **Simple Interaction** interactable and an **Interaction Inspect Component**.
2. Keep **Auto Bind to Interactable** enabled; if there are several interactables, give the one used for inspection a **Component Tag** and set **Auto Bind Interactable Tag** to the same name.
3. Create `IA_ExitInspect` and connect its event to **Stop Inspect** on the manager. Ensure its mapping is active while inspecting.
4. Press the interaction key, then the exit key. The object moves toward the camera, then returns to its position if **Restore Original Transform on Exit** remains enabled.

**Tests:** detection is suspended and then restored by default; the book recovers its position, collision and physics settings. Use **On Inspect Mode Started** and **On Inspect Mode Ended** to show and hide your own instructions.

### Example 20 — Rotate and zoom the inspected object

1. Using example 19's setup, create an **Axis2D** rotation input and an **Axis1D** zoom input, active while inspecting.
2. In the character, when **Is Inspecting** is true, connect rotation X to **Yaw Input** and rotation Y to **Pitch Input** of **Add Inspect Rotation**. Connect the zoom value to **Add Inspect Zoom**.
3. On the object's component, try **Inspect Distance** `110`, **Min Inspect Distance** `70`, and **Max Inspect Distance** `150`. Adjust **Inspect View Offset** if framing is poor.

**Tests:** rotate on both axes, zoom to both limits, exit and check the return. If normal camera controls use the same inputs, manage their priority or suspension in your Blueprint.

### Example 21 — QTE: two-button sequence

1. Create two distinct character input actions `IA_QTE_A` and `IA_QTE_B` and activate their mappings. Configure the object as **QTE Interaction**, **QTE Type** `Sequence`, **Total Duration** `5.0`.
2. Add two **Sequence** entries with **Action Name** `A` then `B` and **Step Duration** `0`.
3. After **Start Interaction**, get **Get Current Active Interaction** on the detector and then **Get Active QTE** on the interactable. Validate the references and active state. Store the QTE reference for input.
4. On `IA_QTE_A`, if the QTE is active, call **Process Input** with `A`; on `IA_QTE_B`, send `B`. Do not also call **Start Interaction** from those presses.
5. On the interactable, connect **On Interaction Completed** to opening your lock and **On Interaction Failed** to a failure message. Do not rely on **Stop Interaction** on release for the outcome.

**Tests:** A then B in under five seconds succeeds; B first returns **Miss** without advancing; waiting for timeout fails. The QTE component may not exist during **On Interaction Started**: retrieve it after the start call returns.

### Example 22 — QTE: directions

1. Reuse example 21's active-QTE retrieval. Choose **QTE Type** `Directional` and create two steps: **Is Directional** enabled with **Direction** `Up`, then `Right`.
2. Map two separate actions. While the QTE is active, the first calls **Process Directional Input** with `Up`, the second with `Right`.
3. Display your own `Up, then right` instruction and use **On Interaction Completed** for success.

**Tests:** up then right succeeds before time expires; right first produces **Miss** without advancing. An empty step list is not a playable sequence.

### Example 23 — QTE: rapid presses

1. Configure **QTE Type** `Button Mash`, **Required Presses** `8`, and **Total Duration** `4.0`.
2. With a dedicated input action, call **Process Input** on the active QTE for each press. This type does not check the supplied action name.
3. To display the counter, get **Get Current Context** from the QTE and read **Press Count** and **Time Remaining**; update your widget while the QTE is active.

**Tests:** eight presses before timeout succeed; fewer than eight fail at timeout. Context progress is also updated by elapsed time: use the counter to display the number of presses.

### Example 24 — QTE: choice and consequences

1. Configure **QTE Type** `Timed Choice` and **Total Duration** `5.0`. Create two input actions and a `Choice` variable on your actor or manager.
2. For the first input, set `Choice` to `Left` **before** calling **Process Input**; for the second, set `Choice` to `Right` before the call. Then send a corresponding name to the active QTE.
3. In **On Interaction Completed**, read `Choice` and play the consequence you programmed. In **On Interaction Failed**, handle time expiring without a choice.

**Tests:** each key triggers your respective branch; pressing nothing fails. The built-in type accepts the first named input without validating choices or selecting a branch: that validation and those consequences are yours to implement.

## 5. Saving and advanced logic

### Example 25 — Saved one-shot door

1. Configure a **Simple Interaction** door, disable **Can Be Used Multiple Times**, enable **Persist State**, and give it a unique **Save Key** such as `EntranceDoor_01`.
2. On interaction completion, open the mesh. After the interaction has fully returned, trigger **Save All Interactions to Slot** from your menu or checkpoint, using slot `Interactions` and index `0`. Check its Boolean and report failure.
3. On load, wait until the level and relevant actors are initialized, then call **Load All Interactions from Slot** with the same parameters. If the slot is missing, handle a new game.
4. After loading, have your door read **Current State**: if **Completed**, set the mesh directly to its open position without replaying the reward or completion event.

**Tests:** open, save, quit and load; a new game without a slot; two doors with distinct keys; a failed save. Overall load success does not guarantee every key was present. Rotation and your gameplay variables require separate restoration.

### Example 26 — Capture an interaction in memory

1. Create a variable of type **Interaction Saved State** on your manager. When the interaction is not active, call **Capture Interactable State** and store the result.
2. To restore that snapshot on the same component, call **Validate Saved State**, then **Apply Captured State** only if validation succeeds. Restore the mesh and your variables separately.
3. For a save object, **Create Interaction Save Game Object**, **Save Interactable to Object**, and **Load Interactable from Object** let you manipulate individual states; to survive closing the game, explicitly save the object to a slot.

**Tests:** capture a state, change the interaction, then restore it; try an invalid state. Do not treat an in-memory capture as a disk save or a way to resume an active QTE.

### Example 27 — Custom interaction with success or cancellation

1. Create a terminal with **Interaction Type** `Custom`. On **On Interaction Started**, open your input widget and retain a reference to the interactable.
2. When your own validation succeeds, call **Complete Interaction** on that reference, then close the widget. If the player exits, call **Cancel Interaction** and close it.
3. Handle the reward or opening on **On Interaction Completed** and cleanup on **On Interaction Cancelled**. Do not wire terminal exit to releasing the initial interaction key.

**Tests:** correct validation, cancellation, and a repeated attempt after closing. A `Custom` interaction stays active until your Blueprint completes or cancels it.

### Example 28 — Pick up with a hand and a contact point

1. Reuse example 09's custom **Pickup Base**. Add a **Contextual Interaction Component** to the character. On the pickup, enable **Enable Contextual Hand Pickup**, choose **Contextual Hand Mode** `Right Hand`, and check **Contextual Max Hand Reach**.
2. Implement **On Contextual Pickup Prepared**: use the supplied transforms to start your own approach and animation. The component supplies context, but does not move the character or build the IK graph for you.
3. In your animation Blueprint, get the character's contextual component and read **Get Contextual Right Hand Transform** and **Get Right Hand IK Alpha** for your own hand-placement logic. Adapt coordinate spaces to your chosen animation node.
4. At your contact point, optionally call **Attach Contextual Pickup to Hand**, check its result, and use an actual socket or bone. Then call **Complete Contextual Pickup** and check its result. On abort, call **Cancel Contextual Pickup**.

**Tests:** visible object in reach, object too far away, obstacle, missing socket, and full inventory. Successful collection clears the context; refused preparation or item grant must not be reported as success.

## Final test checklist

- Test a single target, two nearby targets, an obstructed target and an out-of-range target.
- Test camera and character facing together, then in different directions.
- Test press, early release, cancellation, double press and restart after cooldown.
- Test conditions with and without a provider, required item, quantity and consumption.
- Test QTE success, wrong input, timeout, cancellation and retries if enabled.
- Test missing save, successful or failed write, unique keys, reload and visual restoration.
- Test inspection and contextual pickup with your own widgets, animations, meshes and inputs.
