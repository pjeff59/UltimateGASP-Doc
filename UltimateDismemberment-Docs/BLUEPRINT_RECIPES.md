# Ultimate Dismemberment — Blueprint Recipes

Practical workflows for Unreal Engine 5.8+ using only Unreal Editor and Blueprints. These recipes complement the [Blueprint User Guide](USER_GUIDE.md); they use generic actors and assets that you provide.

## Before you begin

1. Add **Ultimate Dismemberment Component** to the character Blueprint.
2. Confirm **Dismemberment Enabled** is checked.
3. Use the plugin's default zones only when the target skeleton matches their bone names. Click **Populate Default Zones (bouton)** to create editable entries in the Blueprint. An empty **Dismemberment Zones** list is populated at runtime even when **Auto-Populate Default Zones** is disabled.
4. For hit-based damage, confirm the `Hit Result` contains a bone name mapped by **Root Bone** or **Child Bones**.
5. Keep optional Niagara systems, decals, sounds, and fragment meshes unassigned while validating damage logic; they are not required for these workflows.

Damage is first subtracted from the zone's health. The zone can be severed only when its health reaches zero and **Can Be Severed** is enabled. The hit that reaches zero must meet **Min Single Hit Damage To Sever** after **Global Limb Damage Multiplier**—unless the zone was already **Crippled**, in which case a later hit can sever it regardless of that threshold. If **Required Weapon Tags (optional)** is populated, the **Weapon Tags** pin must contain at least one matching tag.

## 1. Sever a hand with a hit-scan weapon

This example sends a point-damage event from a simple weapon trace and lets the target resolve the struck bone.

### Configure the target

1. In the character Blueprint, click **Populate Default Zones (bouton)** on the component.
2. Expand **Hand R** in **Dismemberment Zones** and confirm:
   - **Max Health** = `40`
   - **Min Single Hit Damage To Sever** = `10`
   - **Can Be Severed** = enabled
   - **Required Weapon Tags (optional)** = empty
3. Confirm **Global Limb Damage Multiplier** = `1.0`.

These are the default values for **Hand R**. A different zone or custom skeleton may use different values.

### Send point damage from the weapon

Use your weapon's existing firing event and line trace. The important wiring is:

```text
Fire
→ Line Trace By Channel
    Start = trace origin
    End = trace origin + forward direction × 5000
→ Branch (Return Value)
    True → Apply Point Damage
        Damaged Actor = Break Hit Result(Out Hit).Hit Actor
        Base Damage = 50
        Hit Info = Out Hit
        Hit From Direction = forward direction
        Event Instigator = firing controller
        Damage Causer = weapon actor
```

### Apply the dismemberment damage on the target

In the character's Event Graph:

```text
Event Point Damage
→ Apply Dismemberment Damage From Hit
    Target = Self
    Hit Result = Hit Info
    Damage = Damage
    Weapon Tags = empty
    Damage Causer = Damage Causer
```

The node reads the bone from **Hit Info** and selects **Hand R** when the hit resolves to `hand_r` or one of its mapped child bones. With these values, `50` damage removes the zone's `40` health and exceeds its `10` severance threshold. The returned severity is **Severed**; the component also fires **OnLimbSevered**.

If the returned severity is **None**, inspect the hit's bone name and the target's bone mapping before changing damage values. A hit on an unmapped bone cannot identify a zone.

## 2. Verify cumulative damage and the crippled state

This test uses **Apply Dismemberment Damage To Limb** because the target zone is already known; it does not need a `Hit Result`.

1. In **Dismemberment Zones**, configure **Upper Arm R**:
   - **Max Health** = `100`
   - **Min Single Hit Damage To Sever** = `40`
   - **Can Be Severed** = enabled
   - **Required Weapon Tags (optional)** = empty
2. Keep **Global Limb Damage Multiplier** at `1.0`.
3. From two test inputs or two steps in a test sequence, call **Apply Dismemberment Damage To Limb** on the same target:

```text
Test hit 1 → Apply Dismemberment Damage To Limb
    Target = test character
    Limb Type = Upper Arm R
    Damage = 80
    Weapon Tags = empty

Test hit 2 → Apply Dismemberment Damage To Limb
    Target = same character
    Limb Type = Upper Arm R
    Damage = 20
    Weapon Tags = empty
```

Provide a valid **Hit Location**, **Hit Direction**, and **Damage Causer** on both calls. The first hit leaves `20` health, below half of the maximum, so the zone becomes **Crippled** and fires **OnLimbCrippled**. The second hit reduces health to zero. Because the zone was already **Crippled**, it is then severed even though the second hit is below the configured `40` threshold. **OnLimbSevered** fires on that transition.

Use **Get Limb Health Percent**, **Get Limb State**, or **Get Dismemberment Status Summary** to display the result while testing. A zone already in the **Severed** state ignores further damage.

## 3. Connect bleeding to a character health variable

The component calculates bleed damage; the character Blueprint decides how that damage changes the game's health.

### Configure a predictable test

1. In the **Hand R** zone, set **Bleed Damage Per Second** = `3` and **Bleed Duration (s)** = `10`.
2. In **Settings**, keep **Bleeding Enabled** and **Apply Continuous Bleed Damage** enabled.
3. Set **Bleed Damage Tick Interval (s)** = `1` and **Natural Cauterization Time (s, 0 = limb duration)** = `0`.
4. For a test, sever **Hand R** with recipe 1, or use **Force Sever Limb** on a disposable test character. The force node bypasses normal health, damage, and weapon-tag checks; do not use it as a regular weapon attack.

### Apply the callback to your health

1. In the character Blueprint, open **Class Settings > Implemented Interfaces** and add **DismembermentInterface**.
2. Add the **OnDismembermentApplyBleedDamage** interface event.
3. Connect its `Bleed Damage` input to your existing health logic. For a simple standalone test, create `Current Health` and `Max Health` variables and use:

```text
OnDismembermentApplyBleedDamage (Bleed Damage, Limb Type)
→ Current Health - Bleed Damage
→ Clamp (Float), Min = 0, Max = Max Health
→ Set Current Health
→ Branch (Current Health <= 0 AND NOT bIsDefeated)
    True → Set bIsDefeated = true
         → call the character's existing defeat/death event
```

With the test values, a fully severed zone applies `3` damage per one-second tick. A wounded or crippled zone can bleed at a reduced rate. The callback is not invoked for continuous health damage unless **Apply Continuous Bleed Damage** is enabled.

The `bIsDefeated` guard prevents later bleed ticks from calling the death event again. Reset this variable in your respawn flow.

Optionally bind **OnBleedingStarted** to show a status indicator and **OnBleedingStopped** to remove it. Call **Cauterize Limb** for **Hand R** to stop its bleeding without restoring the severed limb.

## 4. Handle a critical severance without duplicate death logic

Use this workflow when a critical zone should trigger your game's own defeat behavior.

1. In the zone definition, enable **Is Critical Limb (death if severed)**. The default **Head** zone is critical; custom zones are not critical unless you enable this property.
2. In **Settings**, keep **Dispatch Critical Limb Hook** enabled. **Request Owner Death On Critical Limb** is disabled by default.
3. Implement **DismembermentInterface** on the character Blueprint.
4. Add **OnDismembermentOwnerDefeated** and use its summary to enter the project's defeat flow:

```text
OnDismembermentOwnerDefeated (Summary, Damage Causer)
→ Branch (Summary: Is Defeated)
    True → Set the character's defeated state
         → Stop All Effects (on death) [optional]
         → call the game's existing defeat flow
```

The component also broadcasts **OnCriticalLimbSevered** and attempts to pass the defeat state to a compatible project health component. If no compatible health path is found, **Request Owner Death On Critical Limb** controls whether the actor is destroyed as a fallback. Avoid calling the same health/death action from multiple hooks; test the chosen path with the character's actual health setup.

Use **OnCriticalLimbSevered** for a local reaction such as an animation or UI update. Use **OnDismembermentOwnerDefeated** as the centralized character-defeat hook.

## 5. Restore the character after respawn

Call **Reset Dismemberment State** from the character's existing respawn or reinitialization event:

```text
Character respawned
→ Reset Dismemberment State
    Restore Limb Health = true
    Clear Fragments = true
    Clear Effects = true
→ Set bIsDefeated = false
→ restore the game's overall health separately
```

This restores the configured limb health and states, makes hidden bones visible again, removes active fragments and stump caps, and stops bleeding effects. It resets dismemberment state only; it does not restore the character's separate overall health variable.

## Troubleshooting this test

| Result | Check |
|---|---|
| Damage event does not fire | Confirm the weapon uses **Apply Point Damage** and targets the actor returned by the trace. |
| Severity is **None** | Confirm **Hit Info** contains a bone name and that bone maps to a configured zone. |
| Zone is damaged but not severed | Check remaining zone health, **Can Be Severed**, the final-hit threshold, **Global Limb Damage Multiplier**, and any required weapon tags. |
| Bleed events fire but character health is unchanged | Confirm **DismembermentInterface** is implemented, continuous bleed damage is enabled, and its input reaches the health logic. |
| Critical severance runs death logic twice | Use one defeat path and check both the interface callback and compatible health integration. |
| Bones remain hidden after respawn | Call **Reset Dismemberment State** after the mesh is available and keep all three reset options enabled. |
