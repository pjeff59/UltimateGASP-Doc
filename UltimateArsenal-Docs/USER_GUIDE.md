# Ultimate Arsenal — Blueprint User Guide

This guide covers use in Unreal Editor and Blueprints. Names in backticks are
the class, property, node, or event names to search for in the editor.

For complete, worked examples with Blueprint node order, settings, and expected
results, continue with the [Blueprint recipes](BLUEPRINT_RECIPES.md).

## 1. Overview

Ultimate Arsenal provides Blueprint-accessible components for managing
weapons, ammunition, usable items, and equipment setups. The plugin does not
include meshes, Data Assets, animations, widgets, or input bindings: create
your own assets in the Content Browser and configure input in your project.

The minimum weapon workflow is:

1. Create an ammunition definition and a weapon definition.
2. Add **Weapon Manager** to a Character Blueprint.
3. Add ammunition, assign the weapon to a slot, then equip that slot.
4. Connect your Blueprint input actions to fire, aim, and reload.

## 2. Requirements and installation

- This repository targets Unreal Engine 5.8. Other versions require separate
  validation.
- Also enable the required companion modules `UltimateBallistics`,
  `UltimateGASP`, and `UltimateMelee` in the same project.
- The runtime descriptor lists Win64, Mac, and Linux. Console platforms are not
  declared as supported.

### Installation

1. Copy the `UltimateArsenal` folder into your project's `Plugins` folder. If
   the folder does not exist, create it beside the project file.
2. Open the project, go to **Edit > Plugins**, search for **Ultimate Arsenal**,
   enable it, and restart the editor if prompted.
3. Verify that the plugin and its companion modules are enabled. If the editor
   asks to compile modules, let the process finish before continuing.
4. In a Character Blueprint, open **Add Component** and search for
   `Weapon Manager`.

The plugin does not include ready-to-play example content. Create the required
assets and widgets in the Content Browser.

## 3. Create and equip a first hitscan weapon

### 3.1 Create the ammunition definition

1. In the Content Browser, choose **Add > Miscellaneous > Data Asset**.
2. Select **UA Ammo Definition** and create an asset such as `DA_Ammo_Test`.
3. In the asset details, set at least **Display Name** and **Ammo Type**. Add
   an icon if your interface needs one.

The initial values are **Ammo Type = None**, **Pickup Amount = 30**,
**Max Stack Size = 999**, and **Hide When Empty = disabled**. Choose a real
ammunition type before testing.

### 3.2 Create the weapon definition

1. Create another **Data Asset**, with class **UA Weapon Definition**, named
   for example `DA_Weapon_Test`.
2. Configure these fields in the displayed details categories:

| Category | Fields to set | Purpose |
|---|---|---|
| **Identity** | `Display Name`, `Weapon Type`, `Category` | Name and classifications used by gameplay and interfaces. The initial type is `None`; choose a suitable type. |
| **Visual** | `Weapon Mesh`, `Weapon Actor Class` | Weapon mesh and actor spawned at runtime. The actor class is required. Choose **UA Hitscan Weapon** for this test. |
| **Ammo** | `Ammo Definition`, `Magazine Size`, `Starting Ammo` | Ammunition asset, magazine capacity, and initial loaded amount. |
| **Fire Modes** | `Available Fire Modes`, `Default Fire Mode`, `Fire Rate` | Allowed modes, initial mode, and firing cadence. The initial available mode is `Semi-Auto`. |
| **Damage** | `Base Damage`, `Damage Type` | Base damage and damage type applied to the hit target. |
| **Visual \| Attachment** | `Equipped Socket Name` | Character skeleton socket for the equipped weapon, if you want to override the manager's socket. |
| **Visual \| Transform Offset - EQUIPPED / HOLSTERED** | location, rotation, and scale offsets | Visual adjustment while held or holstered. |
| **Animation** | `Fire Montage`, `Reload Montage`, `Equip Montage`, `Unequip Montage` | Optional montages. Leave them empty for the first test. |

Important initial weapon values are **Category = Ranged**,
**Hand Usage = Both Hands**, **Default Fire Mode = Semi-Auto**, **Magazine Size
= 30**, **Max Reserve Ammo = 120**, **Reload Time = 2.0**, **Base Damage =
25.0**, **Has Tactical Reload = enabled**, and **Supports ADS = enabled**. The
initial fire-mode list contains only `Semi-Auto`. For the standard weapon
firing interval, `Fire Rate` is rounds per minute: 600 corresponds to 10 shots
per second. Set the value to match the intended cadence.

Other useful settings: **Base Spread Degrees**, **Spread Increase Per Shot**,
and **Spread Recovery Rate** adjust spread; the **Recoil** fields configure
curves and multipliers. **ADS Widget Classes** is optional; every configured
widget class is shown at the same time while aiming. Sound, effects, animation,
charge, throwable, procedural-motion, and holstered-physics properties are
optional and do not provide the associated assets.

**UA Hitscan Weapon** traces on the **Visibility** collision channel. For the
shot to hit a target, that target must block this channel. The definition can
use `Base Damage` and `Damage Type`; the target must still handle damage
according to your game's health system.

### 3.3 Add components and prepare sockets

1. Open your Character Blueprint and add **Weapon Manager**.
2. Check the skeleton sockets. The manager defaults to `hand_r` for the
   equipped weapon, then `spine_03`, `hip_pistol`, `spine_02`, and `hip_pouch`
   for holstered slots. Add these sockets or adapt the configuration to your
   skeleton.
3. In the Event Graph, use the `Weapon Manager` component reference.

### 3.4 Initialize the weapon in Blueprint

On `Event BeginPlay`, connect the nodes in this order. Example: a 10-round
magazine and 20 rounds in reserve.

```text
Event BeginPlay
  → Add Ammo For Definition
      Ammo Definition = DA_Ammo_Test
      Amount = 30
  → Assign Weapon To Slot
      Weapon Definition = DA_Weapon_Test
      Slot = Slot 1
      Auto Equip = false
  → Equip Weapon In Slot
      Slot = Slot 1
      Play Animation = false
```

Connect the execution pins. Check the **Return Value** booleans from
`Assign Weapon To Slot` and `Equip Weapon In Slot` while debugging. Assignment
transfers up to `Starting Ammo` from the ammunition pool into the magazine;
the remainder stays in reserve. If the pool has no ammunition, the magazine
can remain empty.

### 3.5 Connect gameplay input

Ultimate Arsenal does not define keys or input mappings. Create the actions you
want in your project, then connect them as follows:

```text
Fire — button pressed   → Weapon Manager: Start Fire
Fire — button released  → Weapon Manager: Stop Fire
Reload                  → Weapon Manager: Reload
Aim — button pressed    → Weapon Manager: Start Aim
Aim — button released   → Weapon Manager: Stop Aim
Change fire mode        → Weapon Manager: Switch Fire Mode
```

To equip a specific slot, call `Equip Weapon In Slot` with `Slot 1`, `Slot 2`,
`Slot 3`, or `Slot 4`. `Cycle Next Weapon Slot` and
`Cycle Previous Weapon Slot` move to the next occupied slot. Make sure the
requested slot contains a weapon.

## 4. Route actions by current equipment (optional)

Add **Equipment State Component** if the same
input should perform different actions depending on the active weapon or item.
**Auto Detect Components** and **Auto Update Equipment State** are enabled by
default; **Auto Aim Before Fire** and **Enable Debug Logs** are disabled.

For a ranged weapon, you can connect:

| Gameplay action | Blueprint call |
|---|---|
| Primary attack pressed | `Route Primary Attack` with `Pressed = true` |
| Primary attack released | `Route Primary Attack` with `Pressed = false` |
| Secondary attack / aim | Ranged weapon: `Route Secondary Attack` with `Pressed = true`, then `false`; usable item: press to toggle |
| Reload | `Route Reload Or Interact` |
| Special action | `Route Special Action` with `Pressed = true` |

For ranged weapons, `Route Primary Attack` starts and stops firing,
`Route Secondary Attack` starts and stops aiming, `Route Reload Or Interact`
reloads, and `Route Special Action` switches to the next fire mode. For a
usable item, primary press calls `Use Item` and secondary press calls
`Toggle Item State`; release does not toggle it. For a throwable, the primary
route forwards only the press to `Start Fire`. If the throwable uses charge,
connect press and release directly to `Start Fire` and `Stop Fire` so the
charge can be released. Releasing a ranged attack through
`Route Primary Attack` can also cancel an unfinished `Burst`; see the
[Blueprint recipes](BLUEPRINT_RECIPES.md) for mode-specific input wiring.
Keep the required components on the same Character and call
`Detect Components` after adding components dynamically.

## 5. Inventory, arsenals, and items

### Weapon inventory and arsenals

**Favorites Manager** manages weapon, gadget,
and consumable inventories, plus four equipment setups named `Arsenal 1`
through `Arsenal 4`. Add it to the same Character as **Weapon Manager**.

To grant and equip a weapon through the inventory:

```text
Event BeginPlay
  → Add Weapon To Inventory
      Weapon Definition = DA_Weapon_Test
      Auto Assign Slot = true
  → Equip Weapon In Slot
      Slot = Slot 1
      Play Animation = false
```

Adding the weapon to the inventory puts the initial total
`Starting Ammo + Starting Reserve Ammo` into the manager's pool; slot
assignment then fills the magazine up to `Starting Ammo`. Do not add that same
total again with `Add Ammo For Definition` unless you intend to grant extra
ammunition.

Use `Save Current Arsenal` and `Load Arsenal` with `Arsenal 1`, `Arsenal 2`,
`Arsenal 3`, or `Arsenal 4` to save and restore an equipment setup.
**Auto Save Arsenal** is enabled by default. Definitions must remain in the
inventory to be restored. `Load Arsenal` does not grant ammunition; if some
entries are unavailable, the rest are still applied, `On Arsenal Changed` is
broadcast, and `Load Arsenal` returns `false`. Arsenal setups are initialized
in memory at startup; use your project's Save Game workflow to retain them
between sessions.

### Create a usable item

1. Create a **UA Item Definition**. Set **Item Type**, **Category**,
   **Display Name**, and the required inventory options.
2. For a Blueprint effect, create a Blueprint child of **UA Usable Item Base**
   and assign it to **Equipped Actor Class** in the item definition.
3. In the child Blueprint, implement **On Apply Effect** to apply the desired
   effect. **On Use Item** is available for behavior associated with using the
   item; **On Toggle State** is available for toggleable items.
4. Add **Item Manager** to the Character, then use
   `Add Item To Inventory` and `Use Item` with the definition.

Without a compatible **Equipped Actor Class**, `Use Item` can use the basic
inventory path, but that path does not apply a custom gameplay effect by
default. An assigned actor must be a child of **UA Usable Item Base**.

Optional usable-actor settings include **Use Montage**, **Effect Amount**,
**Effect Duration**, **Effect Tick Rate**, **Consume On Use**, **Can Be Reused**,
**Use Cooldown**, **Can Toggle**, and related widgets/effects. With no montage,
the effect starts immediately; with a montage, it starts when the montage
finishes normally. An interrupted montage does not apply the effect.

Initial item values include **Item Type = None**, **Category =
Tool**, **Is Stackable = disabled**, **Max Stack Size = 1**, **Weight = 0.5**,
**Is Consumable = disabled**, and **Use Duration = 0**. Set **Item Type** and
consumption options explicitly for each item. **Item Manager** starts
with **Max Inventory Slots = 20** and **Max Carry Weight = 50**.

**Use Native Gasp Healing** is disabled by default. If enabled, the Character
must have a Vitals component, and **Effect Amount** must be positive. The
**On Apply Effect** event is still called; do not apply the same healing a
second time in that event. For separate health regeneration,
`Start Health Regeneration` also requires this component and positive `Regen
Rate` and `Duration` values; use `Stop Health Regeneration` to cancel it.

### Additional options

- **Ammunition**: `Get Current Weapon` returns the equipped actor; on that
  actor, `Get Current Ammo` reads the magazine. `Get Ammo Count For Definition`
  reads reserve ammunition for a specific asset;
  `Get Total Ammo Count For Definition` includes magazines and reserve.
- **Custom interface**: use the weapon manager's `On Weapon Equipped`,
  `On Weapon Unequipped`, and `On Ammo Inventory Changed` events; the item
  manager's `On Item Added`, `On Item Removed`, and `On Item Used` events; and
  the arsenal manager's `On Inventory Changed`, `On Favorite Slot Changed`,
  `On Slot Ammo Changed`, `On Slot Item Count Changed`, and `On Arsenal Changed`
  events. The plugin does not create a HUD automatically.
- **Ammunition reserves**: weapons referencing the same exact
  **UA Ammo Definition** asset share its reserve pool. Different assets have
  separate pools even when their **Ammo Type** is the same.
- **Skins**: `Available Skins` configures materials and skin identifiers in a
  definition. Index 0 is the default skin. On the weapon actor, `Apply Skin`
  and `Get Current Skin Index` change and read the index.
- **Drop**: `Drop Current Weapon` requires **Weapon Pickup Class** to reference
  a class implementing **UA Weapon Pickup Interface**. Implement
  `Initialize Weapon Pickup` and return `true` only after successful
  initialization. On failure, the call returns `false` and keeps the equipped
  weapon.

## 6. Troubleshooting and limitations

| Symptom | Check |
|---|---|
| Component or Data Asset class is missing | Check installation, plugin and required-module activation, then restart the editor. |
| `Assign Weapon To Slot` returns `false` | Check the definition, Character owner, and especially **Weapon Actor Class**. The class must not be empty. |
| Weapon is invisible or misplaced | Check `Weapon Mesh`, `Equipped Socket Name`, skeleton sockets, and transform offsets. |
| Weapon does not fire or magazine is empty | Check the equipped slot, `Ammo Definition`, `Starting Ammo`, and node order. Add ammo to the pool before assigning the weapon. |
| Hitscan does not hit | Check that the target blocks **Visibility** and that the trace is unobstructed. Configure the target's damage response separately. |
| Aim or animations have no visible effect | Check **Supports ADS**, widget classes if used, montages, and sockets. No HUD or input mappings are supplied automatically. |
| `Use Item` returns `false` | Check that the item is in the inventory, **Equipped Actor Class** is a child of **UA Usable Item Base**, and any configured montage can play. |
| `Drop Current Weapon` fails | Check **Weapon Pickup Class**, the interface, and `Initialize Weapon Pickup`'s result. The weapon is retained on failure. |
| `Load Arsenal` returns `false` | At least one saved definition is missing from inventory or could not be assigned. The result can be partial; check `On Arsenal Changed`. |

The components provide management and Blueprint events, not game assets
(weapons, animations, widgets, input actions, targets, or effects) or a
ready-made progression save system. Arsenal setups are not persisted between
sessions without the project's save workflow.
