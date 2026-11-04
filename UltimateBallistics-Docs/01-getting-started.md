# Installation and first projectile

This guide takes you from enabling the plugin to firing a projectile from a Blueprint.

## 1. Install and enable

1. Close Unreal Editor.
2. Copy the `UltimateBallistics` plugin folder into your project's `Plugins` folder. The plugin descriptor should be at `YourProject/Plugins/UltimateBallistics/UltimateBallistics.uplugin`.
3. Open the project. If Unreal reports a missing required dependency, install and enable the matching dependency files supplied with your plugin distribution.
4. In Unreal Editor, open **Edit > Plugins**, search for **Ultimate Ballistics**, enable it, and restart the editor when prompted.

If the plugin cannot load, check that you installed the complete package for the same Unreal Engine version as the project. Do not combine plugin files from different builds.

## 2. Create a projectile profile

1. In the Content Browser, choose **Add > Miscellaneous > Data Asset**.
2. Select `UBProjectileProfile` and create an asset such as `DA_Projectile_Default`.
3. Set **Initial Speed** to a non-zero value. Its unit is centimeters per second.
4. Set **Base Damage** if the projectile should damage targets.
5. Save the asset.

The default profile already has an initial speed of `80000 cm/s`, base damage of `30`, world gravity scale of `1`, and a maximum lifetime of `5 seconds`. Ricochet is enabled by default; see the [profile reference](02-projectiles-and-profiles.md) before testing impacts.

## 3. Create a visible projectile Blueprint

The base projectile actor includes a collision component and a `Projectile` component, but no visible projectile mesh.

1. Create a Blueprint child of `UBProjectileActor`.
2. Add a Static Mesh component if the projectile needs a visible body. Assign a mesh from your project and position it so its forward axis matches the projectile's forward direction.
3. Select the `Projectile` component and assign the profile to **Profile** in the Blueprint defaults.
4. Compile and save the Blueprint.

You can add optional trail, impact, and launch assets to the profile. Leaving them empty does not prevent the projectile from moving or applying damage.

## 4. Fire from a weapon Blueprint

For the simplest reliable workflow, call **Spawn Pooled Projectile From Socket** from the weapon or owning actor Blueprint:

1. Add your fire action to the weapon Blueprint's existing input or gameplay event.
2. Call **Spawn Pooled Projectile From Socket**.
3. Set **Owner Actor** to the weapon or character, **Projectile Class** to your projectile Blueprint, **Profile** to the Data Asset, and **Socket Name** to a socket on the owner's first Skeletal Mesh component (for example, `Muzzle`).
4. Connect a world-space velocity to **Throw Velocity**. For a forward shot, multiply the muzzle's world-space forward vector by the desired speed in centimeters per second. The profile's **Initial Speed** is not substituted for a non-zero **Throw Velocity**.
5. Use the returned projectile actor if you need to access its `Projectile` component or bind to its events.

The socket helper uses a matching socket on the owner's first Skeletal Mesh component. If the owner has no such mesh or the socket name does not exist, it falls back to a position `40 cm` forward and `40 cm` upward from the owner.

The name **Spawn Pooled Projectile From Socket** is retained for Blueprint compatibility. It currently spawns a new actor; it does not reuse an inactive projectile.

## Alternative launch workflows

- **Throw Projectile From Socket** takes a world-space **Direction**, **Forward Speed**, and **Vertical Impulse**, builds the velocity, and spawns the projectile. A positive vertical impulse points upward along world Z.
- If you use **Spawn Actor From Class** instead, either assign the profile in the projectile Blueprint defaults or call **Initialize With Profile** on the spawned `Projectile` component before firing. Then call **Fire In Direction** with a non-zero direction. **Fire In Direction** uses the profile's **Initial Speed**; it has no speed-override input.
- **Initialize With Profile And Velocity** is available when you need to supply a complete velocity vector.

## First test checklist

- The profile is assigned and **Initial Speed** is greater than zero.
- The firing direction and velocity are in world space and are not zero.
- The impact target blocks the profile's **Trace Channel**.
- The target can receive damage if you expect damage.
- A visible mesh or trail is assigned if you expect to see the projectile in flight.

Continue with [Projectile profiles and behavior](02-projectiles-and-profiles.md).
