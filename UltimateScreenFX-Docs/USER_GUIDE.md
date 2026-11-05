# Ultimate Screen FX — Blueprint user guide

[Français](../FR/GUIDE_UTILISATEUR.md) · [Plugin overview](../../README.md)

This guide covers the plugin as it is currently provided. It uses Unreal
Editor and Blueprints only. The examples create their own assets; they do not
depend on a demonstration map or an optional content pack.

## 1. Before you start

- Use a compatible Unreal Engine installation and a project that can load the
  supplied plugin. This project uses Unreal Engine 5.8; other engine versions
  have not been verified by this guide.
- Have a playable level, a local player controller, and an active camera.
- Screen effects are primarily applied to the first player controller. Do not
  assume independent effects for multiple local players.
- Test in **Play**, not just in the editor viewport. The plugin affects the
  rendered view; it does not change health, damage, game speed, or other game
  mechanics.

## 2. Install and enable

1. Close the editor. Place the supplied `UltimateScreenFX` folder in your
   project's `Plugins` folder, preserving its contents.
2. Open the project. If Unreal reports that plugin binaries are missing or
   incompatible, use a distribution built for the project's engine version.
3. Open **Edit > Plugins**, find **Ultimate Screen FX**, enable it, and restart
   when prompted.
4. Open **Edit > Project Settings**, find **Ultimate Screen FX**, and confirm
   **Enable Screen FX** is checked.
5. Start **Play**. In a Blueprint associated with the active level or player,
   add **Event BeginPlay** and connect it to **Play Heal Flash**. Leave
   **Intensity** at its default, compile, and play. You should see a brief
   flash. Remove the test node afterwards.

If you see no flash, check the Hit Flash and accessibility settings in section
3. A custom camera or existing post-process settings can also affect the
result; see section 13.

## 3. Project settings

Find **Ultimate Screen FX** in **Project Settings**. The values below are
plugin defaults, not a promise that an existing project has not overridden
them. Configured values are visible in your editor.

| Setting | Default | What to check |
|---|---:|---|
| **Enable Screen FX** | On | Main update switch. Restart and test after changing it. |
| **Default Preset** | Optional | A preset to load at startup. The supplied configuration may point to a project asset; verify that it exists or select your own. |
| **Global Intensity Multiplier** | 1 | Scales several effects; it is not a universal control over every custom material. |
| **Visual Profile** / **Context Profile** | Standard / Exploration | Adjust intensity and, for context, some visual settings. |
| **Enable Screen Shake** / **Shake Intensity Multiplier** | On / 1 | Enable and scale camera shake. |
| **Max Concurrent Shakes** | 5 | When full, a new shake removes the oldest. |
| **Enable Controller Rumble** | On | Attempts controller feedback for shakes; device support is required. |
| **Enable Color Grading** | On | Enables the built-in color-grading preset nodes. |
| **Enable Vignette** / **Base Vignette Intensity** | On / 0 | Controls the built-in vignette. |
| **Enable Hit Flash** / **Default Flash Duration** | On / 0.3 s | Controls the standard hit-flash nodes. |
| **Full Screen Flash** | Off | Standard hit flashes use the screen edges unless enabled. |
| **Enable Motion Blur** / **Default Motion Blur Amount** | On / 0.5 | Projects can override the default amount. |
| **Enable Status Effects** | On | Required for status effects and status-based recipes. |
| **Enable Cinematic Effects** | On | Required by the letterbox and fade nodes. |
| **Block Input During Fade** | On | Allows a fade requesting input blocking to ignore movement and look input. |
| **Update Rate** | 60 Hz | Limits effect updates; transitions complete on an update, so very low rates can look stepped. |
| **Effect Cull Distance** | 10000 | Nonzero shake epicenters beyond this distance from the player pawn are rejected. |
| **Enable Runtime Priority Guard** / **Max Active Effects** | On / 10 | Limits concurrent custom effects by priority; a low-priority request can be rejected. |
| **Enable Performance Guard** | On | Can reduce custom-effect intensity when performance drops. |
| **Reduced Effects Mode** / **Reduced Effects Multiplier** | Off / 0.3 | Optional reduction for affected effects. |
| **Disable Flashy Effects** | Off | Suppresses standard hit flashes and reduces some intensities. Do not assume it disables every custom or configured flash. |
| **Show Debug Overlay** / **Log Effects** | Off / Off | Optional runtime diagnostics. |

For accessibility, offer comfortable choices and test each effect yourself.
Settings do not guarantee that all authored materials or full-screen flashes
are photosensitivity-safe.

## 4. Finding and connecting Blueprint nodes

For simple effects, right-click in an Event Graph and search for the node by
name, for example **Play Damage Flash**. Keep **Context Sensitive** enabled
when possible. A Blueprint must have a valid game-world context.

For advanced controls:

1. Add **Get Screen FX Subsystem** in a Blueprint with a game-world context.
2. Drag from its return value and search for the desired node, for example
   **Play Custom Effect** or **Apply Preset**.
3. Connect the execution pins from your event to the action. The subsystem
   reference is the action's **Target**.
4. For reusable logic, promote the subsystem reference to a variable after
   checking it with **Is Valid**. Obtain it again in the appropriate game
   context if needed.

Some quick nodes and subsystem nodes have similar names. The examples say
which one to use; choose the node with the listed pins. Pin defaults are
starting points, not recommended values for every game.

## 5. First effects: feedback and shake

**Example A — receiving damage**

1. In the player's Blueprint, use **Event AnyDamage** or your own damage event.
2. Connect it to **Play Damage Flash**. Set **Intensity** to `0.4`.
3. Optionally connect **Show Damage Vignette** after it, with **Intensity**
   `0.4` and **Duration** `1.0`.
4. Test repeated hits. The vignette returns toward its base after its duration;
   this visual does not apply damage.

**Example B — healing and successful hits**

- From your heal event, call **Play Heal Flash** with **Intensity** `0.4`.
- From your confirmed-hit event, try **Play Damage Dealt Flash**, **Play
  Headshot Flash**, **Play Critical Hit Flash**, or **Play Shield Break Flash**.
  These are visual feedback only.

**Example C — local impact**

Call **Apply Impact Shake** with **Intensity** `1.0` and **Duration** `0.3`.
For a longer vibration, try **Apply Rumble Shake**. **Stop All Screen Shakes**
clears active transient shakes; an active preset can still supply an ambient
shake. Controller rumble depends on the device and project settings.

**Example D — a positioned explosion**

From your explosion event, call **Apply Explosion Shockwave**. Connect the
explosion's world location to **Epicenter**; try **Intensity** `1.5`,
**Radius** `1500`, and **Duration** `0.5`. The shake has spatial falloff;
the accompanying flash is not distance-scaled by this node. An epicenter at
the world origin is treated as an unspecified location for shake falloff.

For finer control, call **Apply Screen Shake Config** on the subsystem using
**Make Screen Shake Config**. Set **Shake Type**, **Intensity**, **Duration**,
**Frequency**, axes, and optional **Falloff Distance**. A zero duration does
not automatically expire; use **Stop All Screen Shakes** when appropriate.
For rotation-based movement, enable **Affect Rotation**. A zero falloff
distance disables distance falloff.

## 6. Color, vignette and camera effects

**Example E — an indoor look**

On entering your scene, call **Apply Warm Color Grading** with **Blend
Duration** `0.5`. On exit call **Reset Color Grading** with **Duration** `0.5`.
Reset returns to the current preset's base when one is active, not necessarily
to the original camera settings.

Other quick nodes include **Apply Cold Color Grading**, **Apply Cinematic Color
Grading**, **Apply Desaturated Color Grading**, **Apply Night Vision Color
Grading**, **Apply Thermal Vision Color Grading**, and **Apply Adrenaline Color
Grading**. Night and thermal nodes also set chromatic aberration; resetting
color grading alone does not reset that extra effect. Use **Set Chromatic
Aberration** on the subsystem with **Intensity** `0` to clear it.

For a custom look, use **Set Color Grading Config** with **Make Color Grading
Config**. The subsystem also offers **Set Vignette Intensity**, **Set Motion
Blur Amount**, **Set Chromatic Aberration**, **Set Bloom Intensity**, **Set
Depth of Field**, and **Disable Depth of Field**. For example, set motion blur
to `0` over `0.3` seconds, or enable depth of field with **Focal Distance**
`1000`; choose values appropriate to the scene.

The built-in vignette rendering uses its intensity and pulse; do not rely on
its color, shape, or softness fields to produce a colored or shaped vignette.
Use a suitable post-process material for a specific appearance. The motion
blur configuration currently applies its **Amount**; do not rely on its other
configuration fields changing per-object blur or maximum distortion.

## 7. Status effects and ready-made recipes

**Example F — temporary poison**

Call **Apply Poisoned Effect**, with **Intensity** `0.5` and **Duration** `4`.
It expires automatically. To clear it early, call **Clear Status Effect** and
select **Poisoned - Green tint + vignette**. Applying the same status again
replaces its previous configuration and restarts its timer.

**Example G — entering and leaving water**

On entry, call **Apply Underwater Effect**. Its default **Duration** is `0`,
so it stays until cleared. On exit, call **Clear Status Effect** and select
**Underwater - Blue tint + blur**. For other direct status effects, a duration
of `0` also means no automatic expiry.

Try **Apply Drunk Effect**, **Apply Dizzy Effect**, **Apply Burning Effect**,
**Apply Frozen Effect**, or **Apply Stunned Effect** for other visual states.
For custom control use **Apply Status Effect** or **Apply Status Effect
Config** on the subsystem, and **Is Status Effect Active** to query it.
**Clear All Status Effects** removes all statuses, not only the caller's.

**Example H — short gameplay feedback**

Call **Apply Sprint Burst Effect** for a dash, **Apply Glitch Effect** for a
failed interaction, or **Apply Low Health Effect** for danger. These recipes
use timed status effects. Their zero or negative duration does not make them
permanent. **Apply Bullet Time Effect** and **Enable Focus Mode** change
visuals only; implement any actual slowdown separately.

Other quick nodes include **Apply Dream Effect**, **Apply Radiation Effect**,
**Apply Underwater Panic Effect**, **Apply Magic Charge Effect**, **Apply
Paranormal Effect**, **Apply Stealth Focus Effect**, **Apply Magic Buff
Effect**, **Apply Possession Effect**, and **Apply Combat Panic Effect**.
**Apply Vehicle Crash Effect** accepts an impact location and radius.
**Apply Boss Reveal Effect** and **Apply Photo Mode Framing Effect** provide
framing; hide the letterbox and reset the color grading when your sequence
needs to end. Photo framing also changes motion blur and vignette: restore
them with **Set Motion Blur Amount** and **Reset Vignette** as needed.

Status types are shared: if two systems apply the same type, clearing it
clears the shared status rather than just one source. A status may also
override part of a preset while it is active.

## 8. Cinematic framing and fades

**Example I — frame a scene**

1. Call **Show Cinematic Letterbox** with **Height** `0.12` and **Duration**
   `0.5`. Height is the fraction of screen height used by each bar.
2. After the scene, call **Hide Cinematic Letterbox** with **Duration** `0.5`.
3. For a different bar color, use **Show Cinematic Letterbox Config** and
   **Make Letterbox Config**. Supply the fade-out duration to the hide node
   separately.

**Example J — fade between scenes**

Call **Fade To Black** with **Duration** `1` and **Block Input** enabled.
When your transition is ready, call **Fade From Black** with **Duration** `1`.
Input is released when the fade-in is requested, not when it finishes.
**Is Fading** reports whether the transition is in progress, not whether the
screen remains black afterwards. Use **Clear All Effects** if your flow is
interrupted and needs cleanup. The input-blocking setting and a valid player
controller are required for blocking.

## 9. Presets

A **Screen FX Preset** is a reusable collection of color grading, vignette,
motion blur, and ambient shake settings.

**Create one**

1. In the Content Browser, choose **Add > Miscellaneous > Data Asset** and
   select **Screen FX Preset**. Name it `DA_IndoorLook`.
2. Open it. Adjust **Color Grading**, **Vignette**, and **Motion Blur**.
   Set **Ambient Shake > Intensity** to `0` if you do not want a continuous
   shake. Set **Transition Duration** to `0.5`, **Priority** to `50`, and
   **Duration** to `0` for an indefinite preset. Save the asset.
3. In a Blueprint, call **Apply Effect Preset** and assign your asset.
   Later, call **Remove Effect Preset** with that same asset.

**Example K — two environments**

Create `DA_IndoorLook` with priority `50` and `DA_AlertLook` with priority
`80`. Apply the indoor preset when entering the area, then the alert preset
when the alert starts. Removing the alert preset reveals the remaining indoor
preset. The highest priority wins; at equal priority, the most recently
applied preset wins. Reapplying the same asset replaces its existing entry
and restarts its duration.

On the subsystem, **Apply Preset** also has **Persistent**, which prevents
duration expiry. **Set Preset Paused** pauses its expiry timer; it does not
hide the preset. **Apply Preset Replacing Tag**, **Is Preset Tag Active**,
**Is Preset In Stack**, **Get Active Preset**, and **Clear Preset Stack** offer
additional control. **Clear Effect Preset Stack** optionally keeps the
configured default preset; **Clear All Effects** clears the stack entirely.

The default preset and example assets are optional: do not depend on a
particular project content path being present. Validate your own preset in
the target project.

## 10. Zones

**Example L — an area with a preset**

1. Create a Blueprint class based on **Screen FX Zone** and place an instance
   in the level. Select its **Trigger Volume** and resize it.
2. In the instance details, keep **Enabled** and **Affects Only First Local
   Player** checked. Assign your own **Preset** and keep **Remove Preset On
   Exit** checked.
3. Optionally add entries to **Status Effects On Enter**. **Clear Entered
   Status Effects On Exit** controls their removal. Optional **Play Shake On
   Enter** and **Play Flash On Enter** are off by default.
4. Ensure the player pawn and trigger generate overlap events and their
   collision responses allow overlap. Play, walk in, then walk out.

The zone checks the first player's pawn by default. If **Affects Only First
Local Player** is disabled, any actor that generates a qualifying overlap can
trigger it; this does not create per-player effects.

**Important:** zones do not own separate copies of presets or status types.
Exiting one can remove the same preset or status used elsewhere. Overlapping
zones and pawns with multiple overlapping components can also cause premature
cleanup. Avoid sharing these effects between zones, or implement your own
Blueprint overlap and ownership logic. Test possession changes and level
transitions. **Apply Zone Effects** and **Clear Zone Effects** can be called
manually, but manual application does not mark the player as inside.

## 11. Custom effects

A **Screen FX Effect Asset** can combine optional post-process material layers
with native vignette or chromatic aberration. Materials and textures are not
required for the following example.

**Example M — create a reusable native effect**

1. Create a Data Asset of type **Screen FX Effect Asset**, named
   `DA_ChromaticPulse`.
2. Enable **Use Native Chromatic Aberration**, set **Chromatic Aberration** to
   `3`, **Duration** to `2`, and **Fade Out Duration** to `0.2`. Save.
3. In a Blueprint, get the subsystem and call **Play Custom Effect**, using
   the asset as **Effect**. Set **Intensity** to `1`, leave **Duration
   Override** at `0`, and leave **Loop Until Stopped** off.
4. Save the returned **Return Value** as an integer handle. `-1` means the
   effect was not launched. **Is Custom Effect Active** can check the handle.

**Example N — hold an effect until an ability ends**

Call **Play Custom Effect** with **Loop Until Stopped** on and store its
handle. On the ability's end or cancellation, call **Stop Custom Effect**
with that handle. The configured fade-out is applied; repeated stop requests
do not restart it. For groups, assign **Tags** in the asset and use **Stop
Custom Effects by Tag** or **Is Custom Effect Tag Active**. **Stop Custom
Effects** stops all custom effects.

**Duration Override** greater than zero replaces the asset duration; `0` uses
the asset duration. An asset duration of `0` is not a substitute for **Loop
Until Stopped**. **Fade In Duration**, **Fade Out Duration**, and optional
curves shape the timeline. A natural completion can include the configured
fade-out after its duration.

For a material layer, create or select a material whose **Material Domain** is
**Post Process**, add it under **Material Layers**, and set **Weight**.
Configure the material's own graph to produce the appearance and to use any
parameters you need. Scalar, vector, texture, and animated scalar entries are
optional. **Blend Mode** is passed to the material as a scalar parameter; it
does not create blending behavior unless the material uses it.

The automatically written material parameters are `Intensity`, `Progress`,
`Time`, `ImpactDirection`, and `BlendMode`. A material must define and use
them for them to matter. Animated scalar entries can overwrite a parameter
set through Blueprint on subsequent updates. Use **Set Custom Effect Scalar
Parameter For Handle**, **Set Custom Effect Vector Parameter For Handle**, or
**Set Custom Effect Texture Parameter For Handle** for one active effect;
the versions without **For Handle** address all active custom effects.

**Priority** and **Exclusive Tag** are optional. Effects sharing the exclusive
tag are stopped when a valid replacement is admitted. With the priority
guard enabled, a lower-priority request may be rejected with handle `-1`;
an existing effect can instead be cleared to make room for an admitted one.
An effect with no material and no enabled native feature is also rejected.

## 12. Events and cleanup

From a valid subsystem reference, bind a custom event to the desired event
dispatcher. For example, bind **On Custom Effect Finished**, save the handle
from **Play Custom Effect**, and compare the event's **Handle** with the saved
handle before updating your UI.

| Event | Information |
|---|---|
| **On Custom Effect Started** | Handle and effect name. |
| **On Custom Effect Updated** | Handle, effect name, progress, and effective intensity at an update. |
| **On Custom Effect Finished** | Handle, name, and reason: **Completed**, **Stopped**, or **Cleared**. |
| **On Custom Effect Ended** | Compatibility event carrying the same end information. Bind one of the two end events unless you intentionally need both. |
| **On Preset Changed** | Previous and new active presets. |
| **On Screen FX Started** / **On Screen FX Ended** | General notifications, not a matched start/end pair for every kind of effect. Use the custom-effect events for reliable custom-effect lifecycle handling. |

Effects started from an update callback are first updated on the next pass.
Use **Clear All Effects** when intentionally resetting the whole system: it
clears active custom effects, statuses, shakes, presets, framing, and fade
state. It is not a selective reset and does not restore arbitrary camera
post-process settings.

## 13. Troubleshooting and limitations

| Symptom | Check or action |
|---|---|
| No effect or node | Plugin enabled, editor restarted, valid game-world context, local player and active camera. Search from a subsystem reference for subsystem nodes. |
| Flash missing | **Enable Hit Flash** and **Disable Flashy Effects**; inspect whether a configured flash is full-screen or edge-only. |
| Effect is weaker than expected | Intensity, profile, accessibility settings, performance guard, material graph, and active presets or statuses. |
| Custom effect returns `-1` | Assign a usable layer or enable a native feature; check the priority limit. A material must be a working post-process material. |
| Custom effect ends immediately | Set a positive asset duration or enable **Loop Until Stopped**. A duration override of `0` means “use asset duration.” |
| Preset seems ignored | Check priority, apply order, active statuses, context profile, and whether the asset exists. |
| Effects disappear when leaving a zone | Another zone or system may share the same preset or status. Avoid sharing or manage ownership explicitly. |
| Player cannot move after a fade | Request **Fade From Black** or use **Clear All Effects**, and check other systems that may independently block input. |
| Screen changes or loses existing camera settings | The plugin writes post-process settings to the active camera and may override settings or blendables already on it. Test camera changes and compatibility in your project; clearing effects does not restore the previous camera configuration. |

The plugin controls one main local view and has no guarantee of per-player
isolation. Changing cameras, overlapping zones, pausing, level transitions,
custom materials, accessibility, and performance should all be tested in
your own project. Full-screen fades can be affected by both camera and overlay
rendering. Do not assume that turning off the main update switch restores
previously modified camera settings.
