# Ultimate Melee — Blueprint user guide

This guide covers the current user-facing workflows for Ultimate Melee in Unreal Editor. It is written for Blueprints; use the node and asset names shown in the editor.

## 1. Requirements and installation

- Use Unreal Engine 5.7 or later.
- Place the `UltimateMelee` plugin folder in your project's `Plugins` folder.
- Open the project, choose **Edit > Plugins**, search for **Ultimate Melee**, enable it, and restart the editor when prompted.
- The combat component uses `Gasp_Vitals_Component` for stamina. Confirm that this component is available in the project's **Add Component** list before configuring combat. Without it, setup validation reports the missing component; by default the combat system does not block play and stamina checks are bypassed. In the plugin settings, **Block Combat On Misconfiguration** can be enabled to make an invalid setup stop combat.

## 2. How the setup fits together

| Place | Add or assign | Purpose |
|---|---|---|
| Character Blueprint | `Melee Combat Component` and `Gasp_Vitals_Component` | Receives combat inputs and owns combat state and stamina. |
| Weapon Actor Blueprint | `Melee Weapon Component`, a mesh, and a weapon data asset | Supplies attack data and hit detection. |
| Weapon data asset | At least one step in the chain used by an input | Defines damage, timing, animation, and optional effects. |

`Equip Weapon` takes the **weapon actor**, not the data asset. It registers the actor's `Melee Weapon Component`; it does not spawn or visually attach the actor. Spawn or attach the weapon in your own Blueprint workflow, then pass its actor reference to `Equip Weapon`.

## 3. Create and configure a weapon

### 3.1 Create the data asset

In the Content Browser, choose **Add > Miscellaneous > Data Asset**, select **Melee Weapon Data Asset**, and create an asset such as `DA_TrainingBlade`.

Useful starting values (the values in the second column are the plugin defaults):

| Property | Default | Example |
|---|---:|---:|
| `Weapon Name` | Empty | Training Blade |
| `Weapon Type` | `One Handed` | `One Handed` |
| `Base Damage` | 25 | 25 |
| `Attack Speed` | 1.0 | 1.0 |
| `Range` | 150 cm | 150 cm |
| `Poise Break` | 10 | 10 |
| `Stamina Cost` | 20 | 12 |

Add one or more entries to `Light Combo Chain`. For each entry, set `Attack Montage` if the step should play an animation. The default numeric windows are `Combo Input Window Start` 0.4 s, `Combo Input Window End` 0.8 s, `Hitbox Active Start` 0.3 s, and `Hitbox Active End` 0.6 s. Adjust these times to the montage; they are not universal timing recommendations.

For a three-step training combo, add three entries and assign a montage to each. Keep `Damage Multiplier` at 1.0 for the first pass, then adjust individual entries only after the hit timing works.

### 3.2 Add the weapon component

1. Create or open a weapon Actor Blueprint and add a mesh.
2. Add `Melee Weapon Component`.
3. Assign the data asset to `Weapon Data`.
4. For `Socket Trace`, add the weapon mesh socket names to `Hitbox Socket Names` in the data asset. The asset's list takes precedence when it is not empty; otherwise the component's `Hitbox Socket Names` list is used.
5. Choose `Hit Detection Type`. The default is `Socket Trace`; the component's default `Hit Trace Channel` is `Pawn`, and its `Trace Radius` is 5 cm.
6. For `Collision Based`, assign the relevant mesh components to `Hitbox Meshes` and configure their collision to produce the intended contacts.

The named sockets must exist on the weapon mesh. The hit trace channel and the target's collision response must allow the query to reach the target.

### 3.3 Equip the actor

Once the weapon actor exists, call `Equip Weapon` on the character's `Melee Combat Component` and connect the weapon actor to its `Weapon Actor` input.

```text
Event BeginPlay (or your weapon-ready event)
  -> Get Melee Combat Component
  -> Equip Weapon (Weapon Actor = spawned or referenced weapon actor)
```

For a weapon change, call `Unequip Weapon` before replacing it, then call `Equip Weapon` with the new actor. To replace only its data at runtime, call `Set Weapon Data` on the weapon component.

## 4. Integrate the system into a Character Blueprint

This is the full player-character setup, from the Components panel to a weapon ready for input.

### 4.1 Prepare the Character Blueprint

1. Open a Blueprint based on `Character`.
2. In the **Components** panel, add `Melee Combat Component` and `Gasp_Vitals_Component`.
3. Select the character's `Mesh`. Assign a skeletal mesh and an Animation Blueprint that can play montages compatible with that mesh's skeleton.
4. Leave the combat component's values at their defaults for the first test. The `Apply Combat Profile (Arena)` call in the graph below applies the profile's configured timing and poise values; if you need custom values, apply the profile first and set your overrides afterward.
5. In the Content Browser, right-click the attack Animation Sequence and choose **Create > Create AnimMontage**, or select an existing montage that uses the character mesh's skeleton. Assign the montage to `Attack Montage` on the corresponding combo step.
6. Open the Animation Blueprint's AnimGraph and route the final pose through a `Slot` node whose slot name matches the montage slot. Compile the Animation Blueprint. Without a matching slot, a montage may be requested but not appear on the character.
7. Assign a playable `Dodge Montage` in the weapon data asset if the character should dodge; a usable animation instance is required for dodge.

The character mesh's hand socket and the weapon's hit-trace sockets have different purposes: the hand socket attaches the weapon actor to the character, while `Hitbox Socket Names` on the weapon mesh define where weapon traces are made.

### 4.2 Prepare the weapon actor

1. Create a weapon Actor Blueprint, for example `BP_TrainingBlade`.
2. Add the weapon mesh and a `Melee Weapon Component`.
3. Assign `DA_TrainingBlade` to the component's `Weapon Data`.
4. Configure the hit-detection mode and sockets as described in section 3. The socket names must exist on the weapon mesh.
5. Save and compile the weapon Blueprint.

### 4.3 Spawn, attach, and equip it

In the Character Blueprint's Event Graph, use the following order. Replace the example class and socket with the assets used by your project.

```text
Event BeginPlay
  -> Melee Combat Component: Apply Combat Profile (Arena)
  -> Spawn Actor from Class (BP_TrainingBlade)
  -> Is Valid (Spawn Actor Return Value)
       Is Valid
         -> Attach Actor To Component
              Target = Spawn Actor Return Value
              Parent = Character Mesh
              Socket Name = an existing hand socket on this mesh
              Location Rule = Snap to Target
              Rotation Rule = Snap to Target
              Scale Rule = Keep World
         -> Melee Combat Component: Equip Weapon
              Weapon Actor = Spawn Actor Return Value
         -> Melee Combat Component: Refresh Combat Setup Validation
         -> Branch (Condition = Melee Combat Component: Is Combat Setup Valid)
              False -> Print String ("Check the combat setup and required vitals component")
       Is Not Valid
         -> Print String ("Weapon actor could not be spawned")
```

Attach before calling `Equip Weapon`, so the actor is in its intended place when combat starts. Adjust the attached actor's relative transform if the mesh orientation requires it. If the weapon actor is already created by another Blueprint, skip `Spawn Actor from Class` and pass that actor reference to `Equip Weapon` after it is ready.

`Apply Combat Profile` is an explicit setup step: the component starts with `Arena` selected, but does not automatically call this node during `BeginPlay`. Calling it applies profile-driven timing and poise values, so apply it before any per-character overrides. To change weapons later, call `Unequip Weapon`, attach the replacement actor, and then call `Equip Weapon` with its reference.

### 4.4 Confirm the first attack

Before adding more combat logic, verify the basic chain:

1. The spawned weapon actor is valid and has `Melee Weapon Component`.
2. Its `Weapon Data` contains at least one entry in `Light Combo Chain`.
3. The Character's combat component has registered the weapon through `Equip Weapon`.
4. The attack montage is compatible with the Character mesh's skeleton and Animation Blueprint.
5. A target is in range and in front of the attacker; the weapon sockets, trace channel, and target collision response are configured.

Use `Is Combat Setup Valid` to check the setup state. If values or components change at runtime, call `Refresh Combat Setup Validation` before checking again.

## 5. Connect your input events

Call the combat input nodes from the input events already used by your project:

| Input event | Blueprint node |
|---|---|
| Light attack pressed | `Input Light Attack` |
| Heavy attack pressed | `Input Heavy Attack` |
| Special attack pressed | `Input Special Attack` |
| Block started | `Input Block` with `Pressed` = `true` |
| Block completed or canceled | `Input Block` with `Pressed` = `false` |
| Dodge pressed | `Input Dodge`, passing a `Vector2D` direction |

Ultimate Melee provides the combat action nodes, not keyboard or controller mappings. Create or reuse your project's input events in the Character Blueprint. For example:

```text
IA_LightAttack (Started)       -> Melee Combat Component: Input Light Attack
IA_HeavyAttack (Started)       -> Melee Combat Component: Input Heavy Attack
IA_SpecialAttack (Started)     -> Melee Combat Component: Input Special Attack
IA_Block (Started)             -> Melee Combat Component: Input Block (Pressed = true)
IA_Block (Completed/Canceled)  -> Melee Combat Component: Input Block (Pressed = false)
IA_Dodge (Started, Vector2D)   -> Melee Combat Component: Input Dodge (Direction)
```

These `IA_...` names are examples for your own input events, not assets supplied by the plugin.

Call the attack input node directly when you want the combo input buffer to work. During an attack, `Input Light Attack`, `Input Heavy Attack`, and `Input Special Attack` can buffer a valid follow-up while the combo window is open. `Can Attack` is useful for UI feedback or a separate immediate-action decision; placing it as a gate before every attack input also prevents inputs from being buffered while the character is attacking.

`Try Start Light Attack` is the immediate-start alternative: its Boolean result indicates whether a light attack actually began at that moment.

For dodge, connect a non-zero two-dimensional movement direction. The selected montage can use the sections `Forward`, `Backward`, `Left`, `Right`, or `Default` when those sections exist.

## 6. Tune attacks and animation windows

### Numeric windows

By default, the time fields on each combo entry control when a follow-up is accepted and when hit detection is active. Set the times against the corresponding attack montage.

### Animation notifies

To drive timing from the montage instead:

1. Add **Melee: Open Combo Window** and **Melee: Close Combo Window** at the start and end of the follow-up input window.
2. Add **Melee: Hitbox Active** as a notify state over the frames that should detect hits.
3. Enable `Use Combo Notifies Only` and/or `Use Hitbox Notifies Only` on that combo entry to make the corresponding notify the sole source for that window.
4. Optionally add **Melee: Try Combo Transition** where a buffered follow-up should be processed.

With the `Only` options disabled, numeric timings remain active; a hitbox notify can add an active interval. Ensure notify timing and numeric fields are not unintentionally fighting each other.

## 7. Defense, profiles, and optional movement

- **Block and parry:** hold `Input Block` with `Pressed` = `true`, then release it with `false`. A hit received during the opening parry window can be parried; there is no separate `Input Parry` node. `Input Riposte` can be called when a riposte is available, for example after handling `On Melee Parried`.
- **Defense arc:** `Defense Arc Degrees` is 180 by default. Change it on the combat component to adjust the horizontal block/parry arc; 360 covers all horizontal directions.
- **Automatic defense:** `Enable Automatic Defense` is enabled by default on the combat component, but it is evaluated only for pawns controlled by a non-player controller. Disable it if your own Blueprint logic should decide every defensive action.
- **Combat profiles:** the component's default profile is `Arena`. Call `Apply Combat Profile` to select `SoulsLike`, `HackAndSlash`, or `FightingGame`. These profiles tune combat timing and related values; adjust the result in play rather than assuming a profile fits every animation.
- **Native attack movement:** `Enable Native Attack Movement` is off by default. When enabled, a combo step can use montage root motion (`Use Root Motion`) or a local `Manual Movement` distance in centimeters. The montage and its animation data must contain root motion for that option to move the character. Manual movement uses a capsule sweep and can stop short when blocked.

## 8. Targeting and Blueprint events

### Targeting

- Call `Set Target` with a target actor or `Find Best Target` to acquire one.
- Target validation requires a melee or vitals component by default (`Require Combat Target` is enabled). Disable it when generic actors should be valid targets.
- Override `Is Target Allowed` in a child Blueprint when the game needs additional rules.
- A set target is also checked against attack range and the forward attack arc. A target is not required for attacks unless the global setting **Require Target For Attack** is enabled.

### Events for gameplay and UI

Useful assignable events on `Melee Combat Component` include:

- `On Combo Started`, `On Combo Extended`, and `On Combo Ended` for combo counters or interface feedback.
- `On Melee Hit` for hit feedback.
- `On Melee Blocked` and `On Melee Parried` for defense feedback.
- `On Poise Break` and `On Combat State Changed` for reactions to combat state.

`On Melee Hit` reports a resolved contact, not necessarily health damage. Inspect the event's `Hit Result`: `Was Ignored`, `Was Blocked`, `Was Parried`, `Damage` (calculated damage), and `Applied Damage` (damage actually reported as applied).

## 9. Reusable examples

### Example A — a three-hit training blade

1. Create `DA_TrainingBlade` with the example stats above.
2. Add three entries to `Light Combo Chain`; assign an attack montage to each.
3. Set a hitbox interval that overlaps each blade contact. Start with numeric windows, then use notifies if animation edits make fixed times inconvenient.
4. Equip the weapon actor on `BeginPlay`.
5. Bind the light-attack event to `Input Light Attack`.
6. In the character's `On Combo Extended` event, update a combo counter; reset it on `On Combo Ended`.

### Example B — a heavy strike with a distinct cost

1. Add one or more entries to `Heavy Combo Chain`.
2. Give the first heavy step a slower montage and a `Stamina Cost Multiplier` above 1.0, such as 1.5.
3. Bind the heavy-attack event to `Input Heavy Attack`.
4. Use `On Melee Hit` and check `Applied Damage` before playing a confirmed-hit effect.

The montage, damage, and cost are data-asset choices; the example values are starting points, not required balance values.

### Example C — a target dummy

1. Add `Melee Combat Component` and `Gasp_Vitals_Component` to a test Character.
2. Place the dummy within the weapon's configured `Range` and in front of the attacker.
3. Call `Set Target` on the attacker with the dummy.
4. Bind `On Melee Hit` on the attacking character's combat component to display the `Hit Result` values while testing.
5. If the dummy should be a valid target without either recognized component, disable `Require Combat Target` on the attacking component.

### Example D — a pawn that defends automatically

Use a pawn controlled by a non-player controller and leave `Enable Automatic Defense` enabled to test the optional automatic response. To keep all decisions in your own Blueprint logic, disable that property and call `Input Block` or `Input Dodge` from your own conditions.

## 10. Common problems

| Symptom | Check |
|---|---|
| Attack input does nothing | The weapon actor is valid, it has `Melee Weapon Component`, `Weapon Data` is assigned, and the requested attack has an entry in its matching chain. Check `Can Attack` and setup validation. |
| The montage plays but no hit is detected | Confirm the montage's hit window, the chosen `Hit Detection Type`, socket spelling and existence, `Hitbox Meshes` when using `Collision Based`, trace channel, and target collision response. |
| Dodge is refused | Assign a playable `Dodge Montage` to the weapon data asset and ensure the character mesh has a usable animation instance. Check stamina, recovery, stagger, and dodge cooldown as well. |
| Block does not start | Release any previous block input and check `Can Block`, stamina availability, recovery, stagger, and whether an attack is still committed. |
| A target cannot be selected | Confirm it is within the requested acquisition range and, if enabled, has a melee or vitals component. Check `Is Target Allowed` and line-of-sight options. |
| Combat setup is reported invalid | Add `Gasp_Vitals_Component`, or change the project's missing-vitals validation policy if a stamina-free setup is intentional. |
| The hit window feels late or early | Match the numeric window values to the montage, or use the combo and hitbox notifies with the relevant `Only` settings. |

## 11. Important behavior and limits

- Before applying a profile, the component defaults include a 1.5 s combo reset time, a 0.3 s input buffer, 50 maximum poise, a 10-per-second poise regeneration rate, and a 180-degree defense arc. `Arena` is the initially selected profile, but its settings are applied only when `Apply Combat Profile` is called.
- `Can Attack` reports whether a new attack can start under current combat conditions; it does not guarantee that the selected attack chain contains a step. `Try Start Light Attack` reports whether an immediate light attack started.
- A dodge without a playable dodge montage is refused; no dodge stamina is consumed and no dodge invulnerability window begins.
- `Equip Weapon` registers the weapon component but does not create, attach, or visually position the weapon actor.
- Test timing, collision responses, montage sections, and movement with the actual character and weapon assets used by the project.
