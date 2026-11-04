# Projectile profiles and behavior

`UBProjectileProfile` is the Data Asset that configures a projectile's ballistics, collision queries, damage, effects, and optional behaviors. Assign it to the `Projectile` component of a projectile actor or pass it to a spawn/initialization node.

## Common profile settings

The values below are the profile defaults. Unreal Editor may hide conditional settings until their enabling option is selected.

| Details label | Default | Effect |
|---|---:|---|
| **Projectile Type** | Bullet | Descriptive projectile category. |
| **Mass Kg** | `0.02` | Mass used by the ballistic drag calculation, in kilograms. |
| **Diameter Cm** | `0.9` | Projectile diameter used for the drag cross-section, in centimeters. |
| **Drag Coefficient** | `0.3` | Drag coefficient. |
| **Initial Speed** | `80000` | Initial speed in centimeters per second. |
| **Gravity Scale** | `1.0` | Multiplier applied to world gravity. |
| **Max Life Seconds** | `5.0` | Time before the projectile expires. |
| **Trace Channel** | Visibility | Channel used to detect projectile impacts. The target must block this channel. |
| **Trace Radius** | `5.0` | Radius, in centimeters, used for swept collision traces. |
| **Trace Complex** | Off | Whether traces use complex collision. |
| **Enable Ricochet** | On | Enables ricochet behavior. |
| **Ricochet Angle Threshold Deg** | `20` | Incident-angle threshold, in degrees. |
| **Ricochet Energy Retention** | `0.6` | Fraction of energy retained after a ricochet. |
| **Enable Penetration** | Off | Enables material penetration behavior. |
| **Base Damage** | `30` | Direct impact damage. |
| **Orient To Velocity** | On | Requests orientation along the velocity for projectile types such as arrows and rockets. |
| **Stick On Hit** | Off | Requests that the projectile stick to the impact surface. |
| **Explode On Impact** | Off | Enables impact detonation when an **Explosion On Impact** profile is assigned. |
| **Enable Substepping** | On | Enables adaptive simulation substeps. **Max Step Distance Cm** defaults to `200`; **Max Substeps Per Tick** defaults to `8`. |
| **Enable Wind** | Off | Enables the configured world-space wind vector and optional turbulence. |
| **Enable Near Miss** | Off | Enables near-miss feedback. **Near Miss Radius** defaults to `60 cm`; the cooldown defaults to `0.3 seconds`. |
| **Limit Impact Decals** | On | Limits active impact decals. **Max Impact Decals** defaults to `32`. |

`Enable Ricochet` is on by default. Turn it off when a projectile should stop, stick, penetrate, or explode rather than ricochet. Penetration, fuses, wind, and near-miss feedback are optional and disabled by default.

## Configure penetration

1. Enable **Enable Penetration** on the projectile profile.
2. Configure **Max Penetration Cm** and **Energy Loss Per Cm** for the fallback behavior.
3. Optionally create a `UBPenetrationTable` Data Asset, add entries for the physical surface types used by your target assets, and assign it to **Penetration Table**.
4. Assign the corresponding physical materials to the target meshes and test with the profile's **Trace Channel**.

Penetration is disabled by default. The profile also exposes optional angle-based penetration curves; leave **Use Penetration Angle Curve** off unless those curves have been configured.

## Configure fuses

- **Use Time Fuse** is off by default. When enabled, **Time To Detonate** defaults to `3 seconds`.
- **Use Proximity Fuse** is off by default. When enabled, **Proximity Radius** defaults to `250 cm` and **Proximity Channel** defaults to Pawn.
- **Arm Delay Seconds** defaults to `0.1 seconds`.

The time fuse counts from projectile spawn, not from impact. Its `On Expired`
event can drive a custom Blueprint explosion. For the built-in time-fuse blast,
enable **Explode On Impact** and assign an asset to **Explosion On Impact**.
That also makes the projectile explode on its first collision, so it is not a
timer-only setup. See the
[timed grenade recipe](06-blueprint-recipes.md) for a timer-only Blueprint
workflow.

The proximity fuse uses **Explosion On Impact** when it triggers. If that field
is disabled for editing, temporarily enable **Explode On Impact**, assign the
profile, and then turn **Explode On Impact** off again to avoid contact
detonation. Proximity checks use **Proximity Channel** (Pawn by default) while
the projectile is simulating. The movement trace runs first, so a blocking
impact can end the projectile before the proximity check; an impact that stops
the ballistic simulation also stops further proximity checks. Do not use this
fuse as a substitute for a placed mine.

## Configure impact effects

The optional profile assets are **Trail VFX**, **Impact VFX**, **Launch Cue**, **Impact Cue**, and **Flyby Cue**. You provide and assign these assets.

For per-surface impact overrides:

1. Assign a physical material with a surface type to the target mesh.
2. Add that same surface type to **Surface Impact FX** in the profile.
3. Assign any desired impact VFX, sound, and decal material in that entry.

When a matching surface entry contains an asset, it overrides that effect from the general profile settings. A decal entry also supplies **Decal Size** (default `12 cm`) and **Decal Lifetime** (default `10 seconds`). If **Draw Debug Impact Sphere** is enabled, a temporary debug sphere is drawn at the impact point.

`Every Nth As Tracer` is informational; it does not select projectiles automatically. When initializing a projectile after spawn, call **Set Is Tracer** before **Initialize With Profile** or **Initialize With Profile And Velocity** to request the profile's **Tracer VFX**.

## Explosion profile defaults

Create a `UBExplosionProfile` Data Asset for projectiles that explode or for explosion components and traps.

| Details label | Default | Effect |
|---|---:|---|
| **Max Damage** | `100` | Maximum blast damage. |
| **Inner Radius** | `100 cm` | Damage remains at its maximum inside this radius. |
| **Outer Radius** | `600 cm` | No radial damage is applied at or beyond this radius. |
| **Damage Falloff** | Linear | Falloff between the inner and outer radii: Linear, Inverse, or None. |
| **Single Blast Damage Per Actor** | Off | If enabled, blast damage is applied once per actor even if several of its components overlap. |
| **Impulse Strength** | `500000` | Radial impulse strength for overlapping simulating-physics components. |
| **Impulse Vel Change** | Off | Whether the impulse is applied as a velocity change. |
| **Num Shrapnel** | `0` | Number of simulated fragments; zero disables shrapnel. |
| **Shrapnel Range** | `1200 cm` | Maximum fragment trace range when shrapnel is enabled. |
| **Shrapnel Damage** | `20` | Damage per fragment hit. |
| **Shrapnel Trace Channel** | Visibility | Channel used by fragment traces. |

Keep **Inner Radius** less than or equal to **Outer Radius**. **Single Blast Damage Per Actor** is off by default; when enabled, it does not combine shrapnel hits or change per-component physics impulses.

## Blueprint projectile events

The `Projectile` component exposes these assignable events:

- **On Hit** — provides the impact hit result.
- **On Expired** — notifies that the projectile's configured lifetime has elapsed.
- **On Projectile Exploded** — provides the explosion origin, affected actors, their distances in centimeters, and the instigator.
- **On Impact Resolved** and **On Returned To Pool** — additional diagnostics/lifecycle events.

The component also exposes **Fire In Direction**, **Initialize With Profile**, **Initialize With Profile And Velocity**, and **Set Is Tracer**. Profile assignment after spawning should use an initialization function rather than changing the component's **Profile** property after Begin Play.
