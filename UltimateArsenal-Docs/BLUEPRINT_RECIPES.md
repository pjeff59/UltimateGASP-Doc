# Ultimate Arsenal — Blueprint recipes

This companion guide is for building complete, testable workflows in Unreal
Editor. It complements the [user guide](USER_GUIDE.md), which covers
installation and the main settings. Names in backticks are the labels to look
for in the editor.

The plugin supplies management components and Blueprint-callable behavior, not
ready-made weapon meshes, animation montages, input actions, UI, or target
characters. Create those assets in your own project. The recipes below use
generic asset names that you can replace.

## 1. Shared setup: a Character that can equip a weapon

For the first test, add **Weapon Manager** to the Character Blueprint. Add
**Favorites Manager** only if you want its inventory and Arsenal 1–4 loadouts;
it is not needed for direct weapon-slot setup. Add **Item Manager** for usable
items, **Skin Manager** for weapon skins, and the backpack component only when
using backpacks.

Create these assets in the Content Browser:

| Asset type | Example asset | Minimum purpose |
|---|---|---|
| **UA Ammo Definition** | `DA_Ammo_Test` | Identifies the ammunition and its type. |
| **UA Weapon Definition** | `DA_Weapon_Test` | Defines the weapon, its actor class, mesh, ammunition, and behavior. |

For a first weapon, set these fields on `DA_Weapon_Test`:

| Details category | Fields | Example |
|---|---|---|
| **Identity** | `Display Name`, `Weapon Type`, `Category` | `Test Rifle`, a suitable weapon type, `Ranged` |
| **Visual** | `Weapon Mesh`, `Weapon Actor Class` | Your skeletal mesh, `UA Hitscan Weapon` |
| **Ammo** | `Ammo Definition`, `Magazine Size`, `Starting Ammo`, `Starting Reserve Ammo` | `DA_Ammo_Test`, 10, 10, 20 |
| **Fire Modes** | `Available Fire Modes`, `Default Fire Mode`, `Fire Rate` | `Semi-Auto`, `Semi-Auto`, 600 |
| **Damage** | `Base Damage`, `Damage Type` | A test value and the desired damage type |
| **Visual \| Attachment** | `Equipped Socket Name` | A socket on the Character skeleton, if overriding the default |

`Fire Rate` is rounds per minute in the firing interval calculation: 600 means
one shot every 0.1 seconds for an automatic mode. A hitscan weapon traces on
**Visibility**; a target must block that channel. `Starting Ammo` is the
magazine target when the weapon is assigned, but the available ammunition pool
limits how many rounds can be loaded. Add reserve ammunition before assigning
the weapon.

The default equipped socket is `hand_r`. Holstered weapons use the manager's
slot sockets, which default to `spine_03`, `hip_pistol`, `spine_02`, and
`hip_pouch` for Slots 1–4. Make these sockets on your skeleton or change the
component's socket settings to match it. Use the definition's equipped and
holstered transform offsets to align the mesh after the correct socket exists.

### Give and equip the test weapon

In the Character Blueprint Event Graph, use the `Weapon Manager` component
reference:

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

Check the `Return Value` from `Assign Weapon To Slot` and
`Equip Weapon In Slot`. For a test with an inventory, use **Favorites Manager**
and `Add Weapon To Inventory` instead; that path grants the definition's
starting magazine and reserve ammunition. Do not grant the same initial
ammunition a second time.

## 2. Connect input and make a live ammunition display

The plugin does not create input mappings. Create your own Enhanced Input
actions, for example `IA_Fire`, `IA_Reload`, and `IA_Aim`, then connect their
events:

```text
IA_Fire: Started    → Weapon Manager: Start Fire
IA_Fire: Completed  → Weapon Manager: Stop Fire
IA_Reload: Started  → Weapon Manager: Reload
IA_Aim: Started     → Weapon Manager: Start Aim
IA_Aim: Completed   → Weapon Manager: Stop Aim
```

For a single fire action, connect `Started` and `Completed` separately.
`Start Fire`/`Stop Fire` let automatic and charge modes respond to both press
and release. `Switch Fire Mode` cycles through the modes listed in the weapon
definition; it does not add modes that are absent from that list.

### Configure and test the fire modes

Add only the modes you want players to use to `Available Fire Modes`, then
choose the initial one with `Default Fire Mode`. `Switch Fire Mode` moves to
the next entry in that list.

| Mode | What to configure and expect |
|---|---|
| `Semi-Auto` | Each `Start Fire` request produces one shot. |
| `Full-Auto` | Hold the input between `Start Fire` and `Stop Fire`. Shots repeat using `Fire Rate`; 600 means one shot every 0.1 seconds. |
| `Burst` | One `Start Fire` begins a burst up to `Burst Count` shots (default 3). `Stop Fire` cancels the pending burst timer, so do not call it immediately on release if the entire burst should finish. |
| `Charge` | `Start Fire` begins charging; `Stop Fire` releases it. A shot requires at least 10% of `Max Charge Time`. The charge multiplier interpolates from `Min Charge Multiplier` to 1.0. |

For an input shared by every mode, send `Started` to `Start Fire`. On
`Completed`, call `Stop Fire` for `Full-Auto` and `Charge`; for `Burst`, omit
that call until the burst should be cancelled. `Get Current Fire Mode` is
available on the current weapon for a mode check. `Route Primary Attack` sends
the release to `Stop Fire` for ranged weapons, so use mode-aware input handling
if you need bursts to finish after button release.

To keep a custom ammunition counter in sync:

1. In your HUD widget, create a function or event that accepts `Current Ammo`
   and `Reserve Ammo` and updates the text.
2. From the Character's `Weapon Manager`, bind to `On Weapon Equipped`. Its
   `Weapon` output is the newly equipped weapon actor.
3. Bind that actor's `On Ammo Changed` event to the widget update event.
4. Immediately read `Get Current Ammo` and `Get Reserve Ammo` from the new
   weapon and update the widget. This provides an initial value before the
   next ammo change.
5. When switching weapons, stop listening to the previous weapon's event
   before binding to the new one.

The weapon also exposes `On Weapon Fired`, `On Weapon Reloaded`,
`On Weapon Unequipped`, `On Weapon State Changed`, `On Fire Mode Changed`, and
`On Aim State Changed`. Use those dispatchers for UI or animation logic; no HUD
is created automatically.

## 3. Replace hitscan with a travelling projectile

Duplicate the weapon definition or create a second one. Keep the ammo,
inventory, slot, and input workflow from recipe 1, but change:

1. Set `Weapon Actor Class` to **UA Projectile Weapon**.
2. Assign a valid `Projectile Profile` in the weapon definition. A projectile
   weapon without this profile cannot spawn its shot.
3. Optionally set `Projectile Actor Class` to a custom projectile actor. If
   left empty, the built-in default projectile actor is used.
4. Set the mesh muzzle socket and weapon offsets to match the new mesh.
5. Assign the definition to another slot and call `Equip Weapon In Slot` to
   test it.

For a shotgun-style projectile spread, `Pellets Per Shot` controls the number
of projectiles spawned per trigger pull, and `Base Spread Degrees` provides
the default pellet spread angle. Test the profile's trajectory, impact, and
damage settings in a simple enclosed test area before tuning the weapon's
damage and cadence.

### Optional: show a predicted projectile arc

To use trajectory preview, choose **UA Projectile Weapon With Prediction** as
the `Weapon Actor Class` instead:

1. Assign a `Trajectory Visualizer Class` in the weapon actor Blueprint's
   defaults.
2. Leave `Enable Trajectory Prediction` enabled to use the preview.
3. `Show Prediction Only When Aiming` is enabled by default. Keep it enabled
   for an aim-to-preview flow, or disable it to preview whenever the weapon is
   equipped.
4. Bind the same aim input to `Start Aim` and `Stop Aim`.

The prediction settings do not replace the required `Projectile Profile`. If
the visualizer class is empty, the weapon has no preview actor to display.

## 4. Set up a throwable with charge and trajectory preview

Create a weapon definition for the throwable. Set `Weapon Type` to
`Throwable`, choose **UA Throwable Weapon With Prediction** as `Weapon Actor
Class`, and assign the mesh, ammo definition, and `Projectile Profile`.
`Projectile Profile` is required for the throw. `Projectile Actor Class` is
optional.

For a simple, uncharged throw:

1. Keep `Semi-Auto` in `Available Fire Modes` and set it as the default.
2. Set `Throwable Use Charge` off.
3. Bind fire press/release to `Start Fire` and `Stop Fire`. A press throws the
   item; a manual reload is not used for throwables.
4. Give ammunition to the manager's pool. After a throw, one unit is drawn
   from that reserve automatically when available.

For a charged throw:

1. Enable `Throwable Use Charge`.
2. Add `Charge` to `Available Fire Modes` and choose it as `Default Fire Mode`.
3. Set `Max Charge Time` to the desired hold duration.
4. Keep fire bound to both press (`Start Fire`) and release (`Stop Fire`).
   Holding starts charge; releasing throws.
5. Bind aim to `Start Aim`/`Stop Aim`. With the prediction subclass and a
   configured `Trajectory Visualizer Class`, aiming shows the trajectory.
6. Assign `Prepare Throw Montage` and `Throw Montage` if you have them. Set
   `Throw Spawn Delay` to the release point in the throw montage; leave it at
   zero for an immediate spawn.

The preview variant has `Enable Trajectory Prediction` enabled by default,
but still needs a `Trajectory Visualizer Class`. `Trajectory Resume Extra
Delay` controls how long after a throw the preview waits before returning
while still aiming. A throwable does not use a manual reload animation.

## 5. Make a consumable that restores health

This workflow uses an item actor so the effect can be configured in a
Blueprint:

1. Create a **UA Item Definition**, for example `DA_Bandage`. Set `Item Type`
   to `Consumable`, enable `Is Consumable`, enable `Destroy On Use`, and set
   `Use Duration` to zero for an immediate use.
2. Create a Blueprint child of **UA Usable Item Base**, for example
   `BP_Bandage`, and assign it to `Equipped Actor Class`.
3. In `BP_Bandage` class defaults, set `Effect Amount` to the desired total
   healing amount, `Use Native Gasp Healing` on, `Effect Duration` to zero,
   `Consume On Use` on, and `Can Be Reused` off.
4. Add **Item Manager** to the Character. The Character also needs its Vitals
   component for native healing.
5. On a pickup or test input, call `Add Item To Inventory` with `DA_Bandage`
   and the desired count. On the use input, call `Use Item` with
   `DA_Bandage`.
6. Check the `Return Value` from both calls. Bind `On Item Used` or
   `On Item Removed` to refresh your UI.

With `Use Native Gasp Healing` enabled, `Effect Amount` must be positive and
the Character must have Vitals. The built-in healing still calls
`On Apply Effect`; do not apply the same healing again in that event.

To make healing-over-time, set `Effect Duration` and `Effect Tick Rate` to
positive values. The configured total `Effect Amount` is divided across the
effect's ticks, and `On Apply Effect` is called on each tick. Use
`On Apply Effect` for custom Blueprint behavior when native healing is off.
With a `Use Montage`, `On Use Item` indicates the use request, while the
effect is applied only when the montage finishes normally. An interrupted
montage does not apply the effect.

### Optional: equip and toggle a flashlight

Use this workflow for a flashlight, binoculars, or another item that stays
equipped and switches between active and inactive states:

1. Create a **UA Item Definition**, for example `DA_Flashlight`. Set `Item
   Type` to `Equippable`, `Category` to `Tool`, and `Equipped Actor Class` to
   a Blueprint child of **UA Usable Item Base**. Set `Equipped Socket Name` to
   a socket on the Character skeleton if the default attachment is unsuitable.
2. Create `BP_Flashlight` as that child Blueprint. Add a Spot Light component
   and make it invisible by default. In class defaults, enable `Can Toggle`.
   `Toggle On Montage`, `Toggle Off Montage`, and `Overlay Widget Class` are
   optional.
3. Implement `On Toggle State`. Feed its `New State` input into the Spot
   Light's `Set Visibility` node. The event fires after each successful
   toggle, so the component follows the item's active state.
4. Add **Equipment State** to the Character alongside **Weapon Manager**.
   Leave `Auto Detect Components` and `Auto Update Equipment State` enabled
   for the default automatic routing.
5. On the Character's startup setup, call `Assign Item To Slot` on **Weapon
   Manager** with `DA_Flashlight`, `Slot 3`, and `Auto Equip` enabled. Check
   the return value.
6. Connect the toggle input's `Started` event to `Route Secondary Attack`
   with `Pressed` set to true. For a ranged weapon the same route starts aim;
   for the equipped usable item it calls `Toggle Item State`. The release is
   not needed to toggle the item.

`Route Primary Attack` on a usable item calls `Use Item`; it does not toggle
the flashlight. `Assign Item To Slot` creates the equipped actor in Weapon
Manager's slots, but does not add an item stack to **Item Manager**. Add it
there separately only if the game also needs a stack or an item-inventory UI.

## 6. Unlock a skin and persist it with your Save Game

### Configure skin entries

In the weapon definition's `Available Skins` array:

1. Keep the first entry as the default appearance. Index 0 is the fallback
   appearance.
2. Give every entry a unique, stable `Skin Id`; do not change an ID after
   shipping save data.
3. Add its `Material Overrides`. Set `Unlocked By Default` only for skins
   everyone should have.

Add **Skin Manager** to the Character. When an unlock condition succeeds,
connect:

```text
Unlock Skin
  Weapon Definition = DA_Weapon_Test
  Skin Id = Skin_Blue

Set Active Skin
  Weapon Definition = DA_Weapon_Test
  Skin Id = Skin_Blue
```

Call `Is Skin Unlocked` before enabling a selection button. `Get Unlocked Skin
Indices` supplies the indices for the weapon's available entries. `Set Active
Skin` returns false if the ID is missing or locked. A successful change is
applied to matching weapon actors already managed by the Character, including
holstered actors, and broadcasts `On Active Skin Changed`.

### Save and restore skin state

The Skin Manager keeps this data in memory; it does not create or write a
Save Game automatically.

1. Create a Save Game Blueprint and add an array variable of the plugin's
   weapon-skin save-data struct.
2. When saving, call `Get Save Data` on Skin Manager, set the array on your
   Save Game object, then use Unreal's `Save Game To Slot`.
3. On startup, load your Save Game and call `Load From Save Data` before
   presenting skin choices.
4. If weapons already exist when the data is loaded, call `Apply Active Skin
   To Weapon Actor` for each one. Loading data does not repaint existing
   weapon actors by itself.
5. Save again after `On Skin Unlocked` or `On Active Skin Changed`, according
   to your progression rules.

## 7. Attach holstered weapons to a backpack

### Prepare the definition and sockets

1. Create a **UA Backpack Definition**.
2. Assign a backpack skeletal mesh to `Backpack Mesh`.
3. On the Character skeleton, create or choose the `Attach Socket On
   Character` socket. `spine_03` is the default.
4. On the backpack mesh, create sockets for the weapon slots you want to move.
   In `Backpack Sockets By Slot`, map `Slot 1`–`Slot 4` to those exact socket
   names. A slot without a configured socket continues using the Character's
   holster socket.
5. Use `Slot Transform Overrides` only if a weapon needs a backpack-specific
   transform; otherwise its own holstered offsets are used.

### Equip and remove

Add **Backpack** to the Character that already has **Weapon Manager**. Call
`Equip Backpack` with the definition. Once the backpack is equipped, holstered
weapons are redirected to its matching sockets. `Remove Backpack` returns
those weapons to the Character's holster sockets.

`Default Backpack Definition` is not equipped automatically unless
`Auto Equip Default Backpack` is enabled. Its default is off.

For animated equip, assign `Equip Montage`, `Unequip Montage`, and
`Equip Hand Socket`. The default equip timing attaches to the hand at montage
start and snaps to the back at 80% of the montage. The default removal timing
detaches at the start and finishes at the end. Use a timing value of `-1` to
trigger that phase manually from an animation notify with `Notify Equip
Attach To Hand`, `Notify Equip Complete`, `Notify Unequip Start`, or
`Finish Unequip`.

`Remove Backpack` with `Destroy Actor` false detaches the actor; it does not
make it collectible by itself. To spawn a pickup, set `Dropped Pickup Class`
and handle `On Backpack Dropped` to pass the backpack definition to your
pickup Blueprint.

## 8. Build a bare-handed combo

### Configure the style

Create a **Bare Hands Style Data Asset** and assign:

- An ordered `Light Combo Chain` of attack montages.
- `Heavy Attack Montage` and, if used, `Special Attack Montage` and
  `Dodge Montage`.
- `Base Damage`, attack and dodge stamina costs, `Attack Range`, and
  `Attack Speed Multiplier`.

Add **Bare Hands Combat Component** to a Character and assign the style to
`Current Style`, or add it to `Available Styles`. If `Current Style` is empty,
the first available style is selected at startup.

The component's default hit sockets are `hand_r`, `hand_l`, `foot_r`, and
`foot_l`; the hit radius defaults to 30 cm and the trace channel defaults to
**Visibility**. Ensure those sockets exist on the Character skeleton and
targets block the chosen channel.

### Mark the active hit frames and combo window

For each attack montage:

1. Add **Activate Limb for Hit Detection** at the start of the active strike.
   Set `Limb Type` to the striking hand or foot and `Activate` on.
2. Add a second notify at the end of the strike with the same limb and
   `Activate` off.
3. For a light combo, add montage notify events at the intended buffer window
   and call `Open Combo Window`, then `Close Combo Window`. Call
   `Process Buffered Input` at the transition point so a buffered light-attack
   input can start the next montage.

Bind light, heavy, special, guard, and dodge inputs to `Input Light Attack`,
`Input Heavy Attack`, `Input Special Attack`, `Input Guard`, and `Input
Dodge`. `Input Guard` changes the component's guard state; it does not reduce
incoming damage by itself. `Input Counter` is exposed, but does not currently
execute a counter attack. Dodge exposes an invincibility state for its
configured duration; your game's damage handling must check that state if it
should block damage.

Bind `On Hit Landed` to show hit feedback or update a combo UI, and
`On Combo Progress`/`On Finisher Activated` to update combo indicators. A
successful hit applies point damage to the target, so the target must handle
damage in its own health logic. If the Character has a Vitals component with
stamina enabled, attack and dodge costs are checked and consumed; without an
active stamina system, those actions are allowed without a stamina deduction.

## 9. Preview weapons and skins in a widget

1. Create a render target asset and a Blueprint child of **UA Weapon Showroom**.
   Assign the render target to `Render Target`.
2. For `Transparent Background`, use a render target format with alpha, such as
   `RTF_RGBA8` or `RTF_RGBA16f`.
3. Place the showroom actor in the level, or otherwise keep a valid reference
   to it from your widget. Display its render target in a UMG Image.
4. When opening the picker, call `Set Weapon` with a weapon definition. This
   loads the weapon mesh and resets preview skin, zoom, and rotation.
5. For a selected skin, call `Set Skin` with its index. The showroom itself
   does not enforce unlock rules; filter the choices using Skin Manager before
   letting the player select an index.
6. Connect horizontal drag delta to `Add Rotation Delta`. Connect mouse wheel
   delta multiplied by `-1` to `Add Zoom`.

`Auto-Frame On Set Weapon` is enabled by default. `Zoom FOV` starts at 30;
`Zoom Min (FOV)` and `Zoom Max (FOV)` bound zoom. If the target changes at
runtime, call `Apply Render Target`. If the preview is clipped, increase
`Framing Padding` or call `Auto Frame Weapon`.

## 10. Arsenal loadouts are not Save Games

Add **Favorites Manager** to the Character alongside **Weapon Manager**.
`Save Current Arsenal` stores the current equipment in one of `Arsenal 1`
through `Arsenal 4`; `Load Arsenal` restores entries that are still in the
inventory. It does not grant missing weapons or ammunition. A partial restore
still applies the available entries and returns false. `Auto Save Arsenal` is
enabled by default, but the Arsenal data is in memory only. Use your own Save
Game workflow if loadouts must survive a restart.

## 11. Troubleshooting by symptom

| Symptom | Check |
|---|---|
| Weapon assignment returns false | The weapon definition and `Weapon Actor Class` must be valid; check that the requested slot is allowed and the owner is a Character. |
| Weapon is invisible or offset | Check `Weapon Mesh`, the equipped socket, socket spelling, and equipped transform offsets. |
| Hitscan misses a visible target | The target must block **Visibility**; check for intervening collision. |
| Projectile weapon fires but no projectile appears | Assign a `Projectile Profile`; check the profile and optional projectile actor class. |
| Throwable cannot be aimed or thrown | Check loaded/reserve ammo, `Projectile Profile`, fire mode, and whether charge mode is configured on both the definition and inputs. |
| Trajectory is not shown | Assign a `Trajectory Visualizer Class`, enable prediction, and call `Start Aim` if the preview is aim-only. |
| Item use fails or heals twice | Confirm the item is in **Item Manager**, the actor class inherits **UA Usable Item Base**, Vitals exists for native healing, and custom healing does not duplicate native healing. |
| Skin button does not apply a skin | Check that `Skin Id` matches an entry, the entry is unlocked, and `Material Overrides` are assigned. |
| Restored skin is not visible | Restore skin data before equipping, or call `Apply Active Skin To Weapon Actor` for an already existing weapon actor. |
| Holstered weapon stays on the Character | Check the backpack's slot-to-socket map and that the named socket exists on its skeletal mesh. |
| Bare-handed attack never hits | Check active-limb notifies, limb sockets, trace channel blocking, and target damage handling. |
| Combo does not continue | Open the combo window during the montage and call `Process Buffered Input` after buffering the next light attack. |
