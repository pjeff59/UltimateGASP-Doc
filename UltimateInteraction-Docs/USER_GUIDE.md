# Ultimate Interaction: Blueprint user guide

[Français](../FR/GUIDE_UTILISATEUR.md) | [Plugin home](../../README.md)

This guide is for building interactions in Unreal Editor with Blueprints. It describes the behavior of the current plugin, not the historical documents. Examples use your own input actions, meshes, widgets and game logic.

For detailed, reusable projects and failure cases, continue with the [step-by-step examples](EXAMPLES.md). They cover player input, widgets, doors, pickups, conditions, QTEs, inspection, contextual hands and saving.

## Contents

1. [Installation and preparation](#installation-and-preparation)
2. [Your first interaction](#your-first-interaction)
3. [Detection and settings](#detection-and-settings)
4. [Interaction types and events](#interaction-types-and-events)
5. [Prompts and progress](#prompts-and-progress)
6. [Reusable examples](#reusable-examples)
7. [Conditions, actions and presets](#conditions-actions-and-presets)
8. [Multiple hotspots](#multiple-hotspots)
9. [Inspecting an object](#inspecting-an-object)
10. [Quick-time events](#quick-time-events)
11. [Saving and loading](#saving-and-loading)
12. [Contextual interaction](#contextual-interaction)
13. [Troubleshooting and checklist](#troubleshooting-and-checklist)

## Installation and preparation

1. Close the editor and place the supplied `UltimateInteraction` folder in your project's `Plugins` folder. Keep the supplied folder structure intact.
2. Open the project, choose **Edit > Plugins**, find **Ultimate Interaction**, enable it and restart if prompted.
3. The plugin declares dependencies that must be available and enabled for it to load. If the editor reports a missing dependency or incompatible build, obtain a compatible complete distribution from the supplier; do not assume the plugin folder alone is sufficient.
4. Use a project with a playable, player-controlled character and an active input mapping. Create a test level or use your own level. No example map, input asset, mesh or widget is required from the plugin.
5. Create or reuse an **Input Action** of type **Digital (bool)**, here named `IA_Interact`. In an **Input Mapping Context**, map it to a key such as E. Activate that context for the local player using your project's existing input setup. Test that the action's Blueprint event fires before continuing.

The descriptor does not specify a minimum engine version. Use a distribution built for your project's engine version; do not assume compatibility with every version.

## Your first interaction

### 1. Set up the player

1. Open the playable character Blueprint. Add an **Interaction Detector Component** and an **Interaction Manager Component** to the same character. Compile.
2. The manager finds a detector on its owner at play start, or creates one if none is present. Adding it explicitly lets you edit its settings.
3. For the manual input path below, leave the manager's **Interact Action** and **Input Mapping Context** unassigned. Use one binding method only; otherwise the same key can start an interaction twice.
4. In the character Event Graph, add the `IA_Interact` event. On **Started**, call **Start Interaction** on the manager. On **Completed**, call **Stop Interaction** *only if* the detector's **Get Current Active Interaction** is valid and its **Interaction Type** is **Hold Interaction**. For an input that can be cancelled by its triggers, apply the same Hold-only release logic to **Canceled**.

```text
IA_Interact (Started) -> Interaction Manager: Start Interaction
IA_Interact (Completed or Canceled)
  -> Interaction Detector: Get Current Active Interaction
  -> if valid and Interaction Type is Hold Interaction
  -> Interaction Manager: Stop Interaction
```

These are wiring instructions, not text to paste into a graph. The manager advances an active hold automatically. The conditional release matters: **Stop Interaction** cancels an active QTE or custom interaction too.

**Optional automatic input:** instead of the manual events, assign **Interact Action** and optionally **Input Mapping Context** on the manager. The manager attempts to bind **Started** and **Completed** at play start using the local player controller. The controller and input component must already be available; if they are not, use the manual path. Automatic **Completed** calls **Stop Interaction** for every active type, so use the manual Hold-only path when QTEs or custom interactions must continue after release.

### 2. Create a test switch

1. Create an **Actor** Blueprint, for example `BP_TestSwitch`. Add a **Static Mesh** with a visible mesh and an **Interactable Component**.
2. Ensure the mesh has query collision and responds to the detector's **Detection Channel**, **Visibility** by default. Keep the path from player to switch clear.
3. Select the interactable. Set **Interaction Type** to **Simple Interaction**, **Interaction Prompt** to `Activate switch`, and **Interaction Distance** to `200`. The default **Require Look at** is enabled: face the target.
4. With the interactable selected, use its **Events** section to add **On Interaction Completed**. Connect it to **Print String** with `Switch activated`.
5. Compile, place the actor in the level, press **Play**, approach it, face it and press the mapped key. Check the message in the viewport.

This confirms detection and interaction, not an on-screen prompt. A generic interactable does **not** automatically create or show a prompt. Add one as described below.

## Detection and settings

Distances are in Unreal units, normally centimeters; times are seconds. Defaults below apply to a new component before a preset overrides them.

| Component | Setting | Default and effect |
|---|---|---|
| Detector | **Detection Radius** | `300`: search radius. |
| Detector | **Detection Frequency** | `0.1`: interval between searches; `0` searches every frame. |
| Detector | **Detection Channel** | **Visibility**: used for the search and line-of-sight test. |
| Detector | **Enable Extended Detection** | Off; when enabled, **Extended Detection Radius** defaults to `2000`. The target's own distance still applies. |
| Detector | **Interactable Actor Tags** | Empty: no tag filter. If populated, the target actor needs at least one listed Actor Tag. |
| Detector | **Enable Smart Fallback** | On; **Fallback Distance Bias** `60`, **Allow Near Fallback on Angle** on. |
| Interactable | **Interaction Distance** | `200`: maximum distance to its interaction point. |
| Interactable | **Interaction Angle** / **Require Look at** | `45` degrees / on; the player's control direction is used for a controlled pawn. |
| Interactable | **Priority** | **Normal**; other choices are **Low**, **High** and **Critical**. |
| Interactable | **Can Be Used Multiple Times** / **Cooldown Duration** | On / `0`: reusable immediately. Disable reuse for a one-shot interaction. |
| Interactable | **Tap to Toggle Hold** | Off: hold normally. When enabled with the manager, one press starts and another press cancels; release alone does not stop it. |
| Manager | **Block Duplicate Starts** / **Interaction Safety Lock Duration** | On / `0.12`: protects against rapid duplicate starts. |

The search and each target's distance must both reach the object; increasing only one may not be enough. Detection also checks angle, state and line of sight. Targets are ranked by priority and proximity; the fallback can affect the final choice. The detector's **On Best Interaction Changed**, **Get Available Interactions Sorted** and **Force Detection Update** are useful when building your own interface.

With a controlled pawn, detection uses the controller's direction, but the start validation uses the instigator actor's forward direction. A target can therefore be detected while looking at it with the camera but rejected on interaction if the character faces away. Test both directions; adjust character orientation or **Require Look at** to suit your game.

**On Player Enter Range** and **On Player Exit Range** describe targets becoming available or unavailable to a player-controlled detector. They are not equivalent to “this is the best target”: several targets can be available simultaneously. Availability does not guarantee that inventory or other start conditions will pass.

## Interaction types and events

| **Interaction Type** | Behavior |
|---|---|
| **Simple Interaction** | Completes immediately after a valid start. |
| **Pickup** | Completes immediately; your Blueprint must implement the reward and any removal. |
| **Hold Interaction** | The manager advances progress while active. Default **Hold Duration** is `1.0`. |
| **Progressive Interaction** | Each new start counts as one press. Default **Total Presses** is `5`, **Time Window per Press** is `1.0`, and **Reset on Timeout** is off. |
| **QTE Interaction** | Starts the configured quick-time event; see [Quick-time events](#quick-time-events). |
| **Custom** | Starts and remains active until your Blueprint calls **Complete Interaction** or **Cancel Interaction** on the component. |

The interactable exposes **On Interaction Started**, **On Interaction Progress**, **On Interaction Completed**, **On Interaction Failed** and **On Interaction Cancelled**. Select the component and add the corresponding event in its owning actor Blueprint. Progress is a value from `0` to `1`; break the supplied context to access **Instigator**, **Target**, **Interaction Type** and **Progress**. Handle cancellation separately from failure.

For **Hold Interaction**, releasing with **Stop Interaction** cancels and resets progress in the current implementation; do not rely on partial progress being resumed. **Can Interrupt** and **Interrupt Reset Speed** are exposed settings, but do not assume that changing them prevents cancellation or preserves progress. **Show Progress Bar** is a setting, not a widget creator.

For **Progressive Interaction**, a press starts and updates the interaction; another press is needed for each step. If **Reset on Timeout** is on, a late next press resets the count before being counted. When off, progress can decrease after the time window. The manager's duplicate-start lock can reject very rapid presses. **On Interaction Started** occurs on each accepted press, not only the first.

Feedback settings **Start Feedback**, **Complete Feedback** and **Fail Feedback** can reference sound, visual effects, force feedback or camera shake. Player and object animation montages are optional and need suitable skeletal meshes and animation setup. These options do not replace the game logic you connect to events.

## Prompts and progress

The manager does not create a HUD prompt or update it automatically. Its **Set Prompt Visible** node does not control a prompt in this version. Build a widget and manage its visibility and text yourself.

### Example: screen-space prompt for the best target

1. Create a Widget Blueprint, for example `WBP_Interaction`, with a text field and an optional progress bar. At play start, create it in the player Blueprint, add it to the viewport and hide it.
2. Bind an event to the detector's **On Best Interaction Changed**. If the supplied interactable is valid, read its **Interaction Prompt**, set the text and show the widget. If it is invalid, hide the widget.
3. To show progress, bind to **On Interaction Progress** on the relevant interactable and set the progress bar's percent. Unbind from the old target when the selected target changes. Clear or hide the bar on completion, cancellation or loss of the target.
4. Test with two nearby objects and while looking away. If the selected target changes during an active interaction, keep track of the active component separately from the best target.

For a world-space prompt, add a **Widget Component** to the actor and implement visibility with the range events. This may show multiple prompts; use the detector's best-target event if only one should be highlighted. Text and icons are data; a custom widget must actually display them.

## Reusable examples

Each example builds on the player setup above. Add the event to the object's interactable, not to an unrelated actor.

### 1. Toggle a door

Set **Simple Interaction** and prompt `Open or close`. Create a Boolean `IsOpen`, initially false. On **On Interaction Completed**, branch on `IsOpen`: play your opening Timeline when false and reverse it when true, then update the Boolean. Connect the Timeline output to **Set Relative Rotation** on the door mesh. Set a small cooldown if repeated presses should not interrupt the animation.

### 2. One-shot button

Set **Simple Interaction** and disable **Can Be Used Multiple Times**. On completion, change a light's visibility or material. After completion the component is **Completed** and cannot be selected again until you explicitly reset or re-enable it.

### 3. Generic collectible

Use an ordinary Actor with **Pickup**. On completion, use the context's **Instigator** to call your own Blueprint reward logic, then **Destroy Actor** only if the reward was actually granted. A generic **Pickup** component neither grants an item nor destroys the actor by itself. A separate inventory and its capacity rules are your responsibility.

**Alternative: a custom Pickup Base.** Create an Actor Blueprint derived from **Pickup Base**, set **Pickup Type** to **Custom**, disable **Auto Use Mesh from Definition** and assign **Manual Static Mesh**. Disable **Auto Configure Interaction** and enter **Manual Interaction Prompt**. Override **Give Item to Collector** in the Blueprint: perform your own reward logic and return true only when it succeeds. By default this custom function returns false. Use **On Pickup Collected** for successful collection; the underlying interaction's completion event alone does not prove that the reward succeeded.

The base actor has a world-space **PromptWidget3D**, but you must assign a widget class. For its automatic text update, name a Text Block `PromptText`; an Image named `InteractionIcon` is optional. Its **Required Tag for Widget Display** defaults to `CanShowPickupWidget`: add that Actor Tag to the character or clear the setting to remove this particular tag requirement. This tag is not required for generic interactables. The base actor destroys itself after a successful pickup by default, after `0.5` seconds; **Enable Respawn** is off by default. Configure and test these behaviors for your game.

### 4. Hold to open a container

Set **Hold Interaction**, **Hold Duration** `2.0` and prompt `Hold to open`. Use **On Interaction Progress** to drive a lid Timeline or progress bar; use **On Interaction Completed** to grant the contents. On **On Interaction Cancelled**, return the lid to its closed position. Test releasing early and holding for two seconds.

### 5. Accessibility toggle

On the container, enable **Tap to Toggle Hold**. Press once and release: the hold continues. Press again before completion to cancel it. Test with the same manager and ensure the UI tells the player about the alternate control.

### 6. Repeated-press mechanism

Set **Progressive Interaction**, **Total Presses** `5`, **Time Window per Press** `1.0`, and enable **Reset on Timeout**. On **On Interaction Progress**, multiply progress by your desired lift height and use **Set Relative Location**. On completion, enable the next part of your own puzzle. Pause for more than one second, then press again to test the reset.

### 7. Timed switch

Set **Simple Interaction**, keep reuse enabled and set **Cooldown Duration** to `3.0`. On completion turn a light on; use your own timer to turn it off. During cooldown the interaction cannot start, even if its prompt remains visible.

### 8. Manual custom interaction

Set **Custom**. On **On Interaction Started**, start your Blueprint sequence. At its successful end call **Complete Interaction** on the same component; on abort call **Cancel Interaction**. Do not call **Stop Interaction** automatically on input release if the sequence must continue.

### 9. Two-sided control panel

Add two interactables and two scene components to one actor. Assign an **Interaction Anchor** and distinct prompt to each interactable. Give each its own completion event and action, for example `Start machine` and `Stop machine`. Test which point is chosen from each side; see [Multiple hotspots](#multiple-hotspots).

### 10. Examine a clue

Use a simple interaction and the inspect components described below. Put text or a symbol on your own mesh, then rotate it during inspection. Bind **On Inspect Mode Started** and **On Inspect Mode Ended** to show and hide your own instructions.

## Conditions, actions and presets

### Conditions

On the interactable, **Requires Item**, **Required Item Name** and **Consume Item** are optional. Implement the **Interaction Inventory Provider** interface on the instigator or one of its components, with **Has Interaction Item** and **Consume Interaction Item**. The built-in required-item check requests quantity `1`; consumption occurs on completion, and a failed consumption fails the interaction.

**Important:** if no compatible inventory provider is found, the current implementation allows the required-item check and consumption to pass. An empty required item name also passes. Therefore, implement and test the interface; enabling the checkbox alone does not enforce a lock.

When **Can Interact** rejects a start, **On Interaction Failed** is not automatically broadcast for that rejection. To explain a locked target, call **Can Interact** with the actual instigator and display its **Out Fail Reason**, or provide your own feedback. A successful check alone does not guarantee that the manager's other start checks will pass.

**Additional Conditions** supports **Instigator Has Actor Tag**, **Owner Has Actor Tag**, **Inventory Item** and **Named Condition**. Set the appropriate name, quantity, target, optional **Invert Result** and **Fail Reason**. Actor tags are configured on the actor, not gameplay tags. For **Named Condition**, implement **Interaction Condition Provider** and **Evaluate Interaction Condition** on the selected actor or a component. An inventory or named condition with no matching provider also passes by default, so test missing providers explicitly. The **Inventory Item** additional condition checks availability; it does not consume the item.

Example locked door: add the inventory interface to the character, return whether `Key_A` is owned, configure **Requires Item** and `Key_A` on the door, then test with and without the key. If **Consume Item** is enabled, implement the actual removal and return whether it succeeded.

### Actions

**Interaction Actions** are optional entries with a phase: **On Started**, **On Completed**, **On Failed** or **On Cancelled**, and an **Owner** or **Instigator** target. Types include **Send Named Event**, **Set Actor Hidden In Game**, **Set Actor Tick Enabled**, **Set Actor Collision Enabled**, **Set Interaction Enabled** and **Destroy Actor**. Boolean actions use **Boolean Value**.

For **Send Named Event**, implement **Interaction Action Receiver** on the target actor or a component and handle **Handle Interaction Action** using the configured **Action Name** and context. If there is no receiver, nothing handles the action. **Set Interaction Enabled** applies to all interactables on the target actor; use the specific component's node if only one hotspot should change.

### Presets

Create a **Data Asset** of class **Interaction Preset**, configure the shared settings, then assign it to **Interaction Preset** on each interactable. **Apply Preset on Begin Play** is on by default and replaces the corresponding component settings at play start. Turn it off to manage application yourself; **Apply Interaction Preset** applies one explicitly. Check per-instance settings after applying the preset, especially prompts, conditions and actions.

## Multiple hotspots

On one actor, add a separate **Interactable Component** for each action. Set each component's **Interaction Anchor**, optional **Interaction Anchor Socket**, and **Interaction Point Offset**. With no valid anchor, the actor location is used. Place the visible mesh and collision so the detector can find the actor, and keep each point within its own interaction distance.

Each hotspot has separate type, prompt, priority, events and persistence settings. The actor is detected once and its interactables are evaluated separately. For inspection with several hotspots, use **Auto Bind Interactable Tag** on the inspect component and add the same **Component Tag** to the intended interactable; if no matching tag is found, the first interactable is used.

## Inspecting an object

1. On the player character, add an **Interaction Inspect Manager Component**. On the object, add an **Interaction Inspect Component** and an **Interactable Component** configured as **Simple Interaction**.
2. **Auto Bind to Interactable** is on by default: a completed interaction starts inspection. The inspector must resolve to a player and inspect manager. Alternatively disable auto-bind and call **Start Inspect** with the inspector actor, checking the Boolean return.
3. The object moves toward the camera. Default **Inspect Distance** is `110`, with minimum `70` and maximum `150`. Collision, physics and interaction detection are disabled during inspection by default and restored afterward; movement input remains enabled by default. **Restore Original Transform on Exit** is on.
4. Supply controls. In the player Blueprint, call **Add Inspect Rotation** with yaw and pitch input, **Add Inspect Zoom** with zoom input, and **Stop Inspect** to exit. Gate these controls with **Is Inspecting**. Alternatively assign the inspect manager's **Rotate Inspect Action** (Axis2D), **Zoom Inspect Action** (Axis1D), **Exit Inspect Action** and optional **Inspect Input Mapping Context**. Automatic bindings require an available player controller and input component at play start; test them in your project.
5. Use **On Inspect Mode Started** and **On Inspect Mode Ended** for your UI. The manager handles one currently inspected actor; wait for its return to finish before starting another test.

Use a movable object and test its collision, physics and original position before and after inspection. The inspect mode does not create instructions or a camera-control widget for you.

## Quick-time events

### Setup and input

1. Set an interactable's **Interaction Type** to **QTE Interaction**. Expand **QTE Settings > QTE Config** or assign a **QTE Profile** data asset; the assigned profile takes precedence. Default type is **Button Press**, **Total Duration** is `3.0`, and retries are off.
2. Use the manual player input path described earlier. After **Start Interaction** returns, retrieve the detector's **Get Current Active Interaction**, validate it, and call **Get Active QTE** on that interactable. The QTE is created after **On Interaction Started**, so it may not yet exist inside that event. Check validity and whether the QTE is active before forwarding inputs.
3. Create and activate your own input actions for the QTE. Connect them to **Process Input** with the configured action name, or **Process Directional Input** with the chosen direction. Create your own widget if the player needs instructions and progress.
4. Use the interactable's **On Interaction Completed**, **On Interaction Failed** and **On Interaction Cancelled** for outcomes. For QTE-specific data, bind to the active QTE's **On QTE Progress Update**, **On Step Completed**, **On QTE Completed** and **On QTE Failed** after acquiring it. Unbind when no longer needed. **On QTE Started** is emitted during start, before this post-start binding.

### Example: two-button sequence

Set **QTE Type** to **Sequence**, **Total Duration** to `5.0`, and create two **Sequence** entries with **Action Name** `A` and `B`. Leave each **Step Duration** at `0`. Map two separate input actions to keys. While this QTE is active, send `A` or `B` to **Process Input** from the corresponding action; do not also route those presses to **Start Interaction**. Press A then B within five seconds. A wrong action returns **Miss** without advancing; timeout fails the QTE. Use **On Interaction Completed** to unlock your own door.

### Supported behavior and limits

- **Button Press** completes on the first named input and scores timing around the middle of **Total Duration**. The action name is not checked by this type.
- **Button Mash** counts inputs until **Required Presses**; use a positive value. The action name is not checked.
- **Sequence** checks the configured action name in order. **Directional** checks each entry's **Is Directional** and **Direction** through **Process Directional Input**. Populate the sequence and use a positive total duration.
- **Timed Choice** completes on the first named input; implement your own choice validation and consequences. The supplied name does not select a built-in branch.
- QTE **Hold** advances with elapsed time and does not verify that a button remains held. Use **Hold Interaction** for a genuine held-input interaction.
- **Analog Stick** and **Rhythm** appear in the list but have no corresponding input handler in this version. Do not use them as ready-made QTEs.
- **Difficulty**, **Show Tutorial**, **Auto Trigger** and transition-related fields are exposed, but do not assume they automatically change timing, display a tutorial or chain interaction types. Configure timing and any transitions explicitly in your Blueprint.
- A retry, if enabled, restarts automatically until **Max Retries** is exhausted. A sequence's **Step Duration** affects timing evaluation, not an independent step timeout. The progress context is also updated by elapsed time, so do not treat it as a stable count of sequence steps; use **Current Step Index** or **Press Count**.

## Saving and loading

1. For each placed interactable to persist, enable **Persist State**. Optionally enter a unique **Save Key** per component and instance; explicit keys are especially important for actors created dynamically or renamed between sessions. An empty key uses the actor path and component name.
2. Once your interaction has fully completed, call **Save All Interactions to Slot**, for example with `InteractionsSlot` and user index `0`. Check its Boolean result and report failure to the player. Use a slot dedicated to interaction data.
3. After the level and its relevant actors are present and initialized, call **Load All Interactions from Slot** with the same slot and user index. Check the result; a missing slot returns false. The overall success does not guarantee that every individual key was found or restored.
4. Restore your own visual and gameplay state separately. The saved data contains interaction state, progress, press count and remaining cooldown, not your door rotation, reward, inventory, destroyed actors or other Blueprint variables. For a one-shot door, for example, after loading check its interactable's **Current State**; if **Completed**, set the mesh to its open rotation without granting the reward again.

Saving or loading is not automatic. Avoid persisting an active interaction as a resume mechanism: the saved data does not restore its instigator, input bindings or an active QTE. Test a new game, a missing slot, save/reload after completion, multiple actors and level changes.

For individual control, **Create Interaction Save Game Object**, **Save Interactable to Object**, **Load Interactable from Object**, **Capture Interactable State**, **Validate Saved State** and **Apply Captured State** are available. An object held in memory is not a disk save until you explicitly save it. Use **Get Resolved Save Key** to diagnose key collisions.

## Contextual interaction

The optional **Contextual Interaction Component** stores an actor, profile name, approach transform and left/right hand targets supplied to **Begin Contextual Interaction**. It exposes **Get Contextual Approach Transform**, **Get Contextual Left Hand Transform**, **Get Contextual Right Hand Transform**, **Get Left Hand IK Alpha**, **Get Right Hand IK Alpha**, **Uses Left Hand**, **Uses Right Hand**, **Clear Contextual Interaction** and **Cancel Contextual Interaction**.

Example: provide your own world-space hand targets for a lever, then read the transforms and alpha values in your animation Blueprint. Your Blueprint and animation graph must implement movement, animation and hand placement; the component does not do those steps automatically. Clear the context when finished or cancelled and test on the intended character skeleton.

For the **Pickup Base** alternative above, **Enable Contextual Hand Pickup** is off by default. To use it, add the contextual component to the collecting character, enable the option on the pickup and choose a **Contextual Hand Mode** other than **No Hand IK**. Implement **On Contextual Pickup Prepared** to start your own approach and animation using the supplied transforms. At the intended contact point, call **Complete Contextual Pickup** on the pickup and check its result; use **Cancel Contextual Pickup** on abort. **Attach Contextual Pickup to Hand** is optional and requires a suitable character mesh and socket. Configure **Contextual Max Hand Reach** and test obstacles: preparation may fail if the target is too far away or line of sight is blocked. No animation or hand placement is supplied automatically.

## Troubleshooting and checklist

| Symptom | Check |
|---|---|
| Plugin cannot be enabled | Confirm the complete compatible distribution and declared dependencies are present; restart after enabling. |
| Nothing is detected | Confirm the controlled player owns the detector, the target has an interactable and query collision responds to **Detection Channel**, both distances are sufficient, angle and line of sight are clear, and any Actor Tag filter matches. |
| Target is visible but interaction does not start | Check **Current State**, cooldown, conditions, interface implementations and **Start Interaction** return through **Start Interaction Safe** if you need a Boolean. A visible prompt does not prove conditions pass. |
| No prompt | Generic interactables do not create one. Check widget creation, visibility, event binding and the selected component's prompt text. |
| Hold stops immediately | Check the release wiring and input triggers. Call **Stop Interaction** on release for holds, not immediately after starting. |
| Repeated presses are ignored | Check **Interaction Safety Lock Duration**, **Total Presses**, timeout and input mapping. |
| QTE disappears on release | Use manual Hold-only release handling; do not send **Stop Interaction** on release while a QTE is active. Forward QTE actions to the active QTE. |
| Inspection starts but cannot exit | Supply an exit input and call **Stop Inspect**; verify player controller, manager and input mapping. |
| Reloaded object looks unchanged | Load after actors initialize, use unique matching keys, then restore visual and game state in your own Blueprint. |

For diagnosis, enable **Debug Draw Detection** and **Enable Debug Logs** on the detector, or **Enable Runtime Debug Overlay** on the manager. Disable verbose debugging after testing. Test near and far, looking toward and away from the target, blocked line of sight, competing hotspots, early release, completion, cancellation, missing conditions and save/load. Do not assume a successful editor test covers every input device or packaged build.
