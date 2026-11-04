# Explosions and Blueprint nodes

Ultimate Ballistics provides an explosion component, explosion Data Assets, and Blueprint library nodes for applying, querying, and previewing ballistic effects.

## Trigger an explosion from an actor

1. Add an `UBExplosionComponent` to an actor Blueprint.
2. Assign a `UBExplosionProfile` to **Profile**.
3. Call **Trigger Explosion** when the actor should detonate.
4. Bind **On Exploded** for a simple notification or **On Exploded Detailed** to receive the origin, affected actors, distances in centimeters, and instigator.

If **Profile** is not assigned, **Trigger Explosion** has no effect. Profile assets may provide **Explosion VFX** and **Explosion Cue**; those assets are optional.

## Apply an explosion without a component

Call **Apply Explosion** with a world context, profile, world-space origin, and instigator. It applies the profile's explosion effects and damage.

For staged behavior:

1. Call **Get Actors In Explosion Radius** to collect actors and their distances without applying blast damage or impulse.
2. Wait for the desired Blueprint delay or presentation event.
3. Call **Apply Damage To Actors** with the same profile, origin, instigator, actor array, and distance array.

Keep the actor and distance arrays paired by index. The query node can play the profile's explosion VFX and sound; it does not apply the blast damage or impulse at query time.

## Other Blueprint library nodes

| Blueprint node | Use |
|---|---|
| **Spawn Pooled Projectile From Socket** | Spawns a projectile at a socket on the owner's first Skeletal Mesh component and initializes its profile and velocity. The name is retained for compatibility; the node currently creates a fresh actor. |
| **Throw Projectile From Socket** | Builds a velocity from **Direction**, **Forward Speed**, and **Vertical Impulse**, then spawns the projectile. Direction is world-space; the vertical impulse follows world Z. |
| **Compute Throw Velocity** | Returns the velocity calculation used for a directional throw. |
| **Predict Trajectory** | Returns trajectory points using the profile's **Initial Speed**. |
| **Predict Trajectory With Speed** | Returns trajectory points using a supplied initial speed. |
| **Build Zeroing Table** | Returns drop values in centimeters for requested distances in meters and a zeroing distance in meters. |
| **Apply Directional Explosion And Get Affected** | Applies a directional/cone explosion and returns detailed hit information. |
| **Apply Chaos Radial Impulse** | Exposed Blueprint node with parameters for a field-system effect. Its current behavior does not apply the supplied origin, radius, strength, and velocity-change settings as a configured radial impulse; do not rely on those inputs for that result. |

`Predict Trajectory` and `Predict Trajectory With Speed` return points with **Location**, **Velocity**, and **Time**. They calculate a path; they do not test the path against world collision or reproduce impact, ricochet, penetration, or fuse events.

## Preview a trajectory in the editor

1. Place a `UBTrajectoryPreviewActor` in the level.
2. Assign its **Profile** and set **Start Location**, **Direction**, **Max Time** (default `3 seconds`), and **Time Step** (default `0.02 seconds`).
3. Select **Refresh Preview** in the actor's Details panel.

The preview is a visual aid. It is not a collision or damage simulation.

## Visualize a trajectory at runtime

Place a `UBTrajectoryVisualizer` actor, assign **Profile**, and select **Visualization Mode**:

- **Spline Mesh** (default) and **Instanced Meshes** use **Trajectory Mesh**; **Trajectory Material** is optional.
- **Niagara Ribbon** uses **Trajectory Niagara**.
- **Debug Lines** uses **Debug Line Color** and **Debug Line Thickness** and does not require a mesh or Niagara asset.

The visualizer defaults to **Max Time** `3 seconds`, **Time Step** `0.05 seconds`, **Auto Update** enabled, **Enable Collision Detection** enabled, **Trace Channel** Visibility, and **Show Impact Marker** enabled. It starts at the actor location and follows the actor's forward vector unless **Use Custom Start Location** or **Use Custom Direction** is enabled. When collision detection is enabled, the displayed path stops at the first blocking trace; the visualizer ignores its owner and any actors in **Ignored Actors**.

Blueprint nodes on the actor include **Update Trajectory**, **Set Visible**, **Add Ignored Actor**, **Clear Ignored Actors**, **Get Predicted Impact Location**, and **Get Trajectory Points**. The collision check only limits the displayed path; it does not simulate ricochet, penetration, fuses, or damage.

## Configure Chaos response

For explosion impulses on simulating-physics components, configure **Impulse Strength** and **Impulse Vel Change** in the `UBExplosionProfile`. **Apply Chaos Radial Impulse** is exposed, but its input values do not currently configure the applied field as their names imply; do not depend on it for a specifically tuned radial impulse.
