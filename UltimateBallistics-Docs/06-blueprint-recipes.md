# Ultimate Ballistics — Blueprint recipes

These recipes use assets created in your project and the Blueprint nodes
available in the plugin. The numeric values are test starting points, not
game-balance recommendations. They assume that the project already has an
input action for firing or interacting.

## 1. Fire a visible practice projectile from a muzzle

### Prepare the assets

1. Create a `UBProjectileProfile` Data Asset named `DA_TargetPractice`.
2. Set these values for a predictable first test:

   | Details field | Value |
   |---|---:|
   | `Projectile Type` | `Bullet` |
   | `Initial Speed` | `80000 cm/s` |
   | `Base Damage` | `30` |
   | `Max Life Seconds` | `5` |
   | `Trace Channel` | `Visibility` |
   | `Enable Ricochet` | Off |
   | `Enable Penetration` | Off |
   | `Explode On Impact` | Off |

3. Create `BP_TargetPracticeProjectile` as a child of `UBProjectileActor`.
   Add a Static Mesh component if the projectile should be visible in flight;
   the base actor does not supply a visible mesh.
4. Add a socket named `Muzzle` to the first Skeletal Mesh component on the
   actor that will own the projectile. This is the mesh searched by the socket
   launch node.

### Build the firing graph

On the owning Character Blueprint, connect an existing fire action:

```text
IA_Fire: Started
  → Get Mesh
  → Get Socket Transform
      Socket Name = Muzzle
      Transform Space = World
  → Break Transform
  → Get Forward Vector (from Rotation)
  → Vector * Float
      Float = 80000
  → Spawn Pooled Projectile From Socket
      Owner Actor = Self
      Projectile Class = BP_TargetPracticeProjectile
      Profile = DA_TargetPractice
      Socket Name = Muzzle
      Throw Velocity = multiplied vector
  → Is Valid (Return Value)
```

`Throw Velocity` is a world-space velocity. A non-zero value supplied here
overrides `Initial Speed`, so keep the two values aligned when this is a
straight-ahead shot. The helper uses the named socket on the owner's first
Skeletal Mesh component; if it cannot find that socket, it uses its documented
fallback position instead.

For the first test, place a target that blocks `Visibility`. If the projectile
appears but does not move, check that the profile and velocity are valid. If it
moves but does not hit, check the target's response to `Trace Channel`.

## 2. Make an arrow that drops and sticks on impact

Create `DA_PracticeArrow` from `UBProjectileProfile` and set:

| Details field | Value |
|---|---:|
| `Projectile Type` | `Arrow` |
| `Initial Speed` | `25000 cm/s` |
| `Base Damage` | `45` |
| `Gravity Scale` | `1.5` |
| `Trace Radius` | `2 cm` |
| `Max Life Seconds` | `8` |
| `Orient To Velocity` | On |
| `Stick On Hit` | On |
| `Enable Ricochet` | Off |
| `Enable Penetration` | Off |

Create `BP_PracticeArrow` as a child of `UBProjectileActor`, add an arrow mesh,
and assign `DA_PracticeArrow` to its `Projectile` component. On the Character,
use the view direction while spawning at the `Muzzle` socket:

```text
IA_Fire: Started
  → Get Control Rotation
  → Get Forward Vector
  → Throw Projectile From Socket
      Owner Actor = Self
      Projectile Class = BP_PracticeArrow
      Profile = DA_PracticeArrow
      Socket Name = Muzzle
      Direction = view forward vector
      Forward Speed = 25000
      Vertical Impulse = 0
  → Is Valid (Return Value)
```

The helper builds a world-space velocity from the normalized direction and
`Forward Speed`; the upward impulse is added along world Z. Gravity bends the
flight path. With `Stick On Hit` enabled, the arrow attaches to the hit
component and stops its projectile simulation. It remains until the profile's
`Max Life Seconds` expires. The target must block the profile's `Trace Channel`.

## 3. Make an impact rocket with a blast

### Create the two profiles

Create `DA_RocketBlast` from `UBExplosionProfile`:

| Details field | Value |
|---|---:|
| `Max Damage` | `100` |
| `Inner Radius` | `75 cm` |
| `Outer Radius` | `400 cm` |
| `Damage Falloff` | `Linear` |
| `Single Blast Damage Per Actor` | On |
| `Num Shrapnel` | `0` |

Create `DA_PracticeRocket` from `UBProjectileProfile`:

| Details field | Value |
|---|---:|
| `Projectile Type` | `Rocket` |
| `Initial Speed` | `30000 cm/s` |
| `Base Damage` | `20` |
| `Max Life Seconds` | `6` |
| `Enable Ricochet` | Off |
| `Explode On Impact` | On |
| `Explosion On Impact` | `DA_RocketBlast` |

Create `BP_PracticeRocket` as a child of `UBProjectileActor` and add a rocket
mesh. Reuse the firing graph from recipe 1, replacing the class and profile
with `BP_PracticeRocket` and `DA_PracticeRocket`, and use a `Throw Velocity`
of `Muzzle Forward Vector * 30000`.

On impact, `Base Damage` is applied to the directly hit actor and the
explosion profile applies its radial damage. A directly hit target can
therefore receive both. `Single Blast Damage Per Actor` prevents duplicate
blast damage from multiple components; it does not combine direct-hit damage
with blast damage. Targets must block `Visibility` for the projectile's
impact query and the explosion query.

The `Projectile` component's `On Projectile Exploded` event provides the
explosion origin, affected actors, their distances, and the instigator. Bind
it if the HUD needs to display which actors were affected.

### Optional variant: show the blast before applying damage

For a delayed blast effect, call `Get Actors In Explosion Radius` first, store
its actor and distance arrays, then apply them after a short delay:

```text
On Detonate
  → Set Saved Origin = Get Actor Location
  → Get Actors In Explosion Radius
      Profile = DA_RocketBlast
      Origin = Saved Origin
      Instigator Actor = Self
  → Set Saved Actors = Affected Actors
  → Set Saved Distances = Distances Cm
  → Delay = 0.25
  → Apply Damage To Actors
      Profile = DA_RocketBlast
      Origin = Saved Origin
      Instigator Actor = Self
      Target Actors = Saved Actors
      Distances Cm = Saved Distances
```

The query plays the profile's explosion VFX and cue immediately, but does not
apply damage or impulse. `Apply Damage To Actors` applies the stored blast damage and physics impulse
after the delay, along with any shrapnel configured in the profile. Keep the
actor and distance arrays paired by index; do not sort or filter one array
without applying the same change to the other.

## 4. Make a timed grenade with a Blueprint explosion

The time fuse can notify Blueprint at detonation time. For a timer-only
grenade, use that event to apply the explosion instead of enabling
`Explode On Impact`:

1. Create `DA_TimedGrenade` from `UBProjectileProfile` and set:

   | Details field | Value |
   |---|---:|
   | `Projectile Type` | `Grenade` |
   | `Initial Speed` | `1500 cm/s` |
   | `Base Damage` | `0` |
   | `Use Time Fuse` | On |
   | `Time To Detonate` | `3 seconds` |
   | `Max Life Seconds` | `6 seconds` |
   | `Explode On Impact` | Off |
   | `Enable Ricochet` | Off |

   The time starts when the projectile is spawned, not when it hits the
   ground. Keep `Max Life Seconds` greater than `Time To Detonate`.
2. Create `DA_GrenadeBlast` from `UBExplosionProfile` and choose the blast
   damage, radii, and optional VFX or cue.
3. Create `BP_TimedGrenade` as a child of `UBProjectileActor`. Add a visible
   mesh and a variable named `Blast Profile` of type `UBExplosionProfile`;
   set its default value to `DA_GrenadeBlast`.
4. On the Character, call `Throw Projectile From Socket` with
   `Projectile Class = BP_TimedGrenade`, `Profile = DA_TimedGrenade`,
   `Socket Name = Muzzle`, `Direction = view forward vector`,
   `Forward Speed = 1500`, and `Vertical Impulse = 350`.
5. In `BP_TimedGrenade`, add the `On Expired` event from its `Projectile`
   component and connect:

```text
Projectile: On Expired
  → Apply Explosion
      Profile = Blast Profile
      Origin = Get Actor Location
      Instigator Actor = Get Owner
  → Return To Pool
```

With `Explode On Impact` off, a collision stops the ballistic simulation but
leaves the fused projectile alive for its timer. If `Explode On Impact` is
enabled, it detonates on the first collision as well as enabling the built-in
time-fuse blast path; that is not a timer-only setup. `On Expired` also fires
at maximum lifetime, which is why the example sets the maximum lifetime to
six seconds and the fuse to three.

## 5. Place and disarm a proximity mine

1. Create `DA_MineBlast` from `UBExplosionProfile`. For a test, set
   `Max Damage = 80`, `Inner Radius = 50 cm`, `Outer Radius = 300 cm`, and
   `Damage Falloff = Linear`.
2. Place a `ProximityMine` actor in the level and assign `DA_MineBlast` to
   `Explosion Profile`. Set `Detection Radius = 250 cm`,
   `Minimum Detection Time = 0.2 seconds`, and `Explosion Delay = 0.75 seconds`.
   Leave `Is Armed` enabled.
3. Ensure the player Character's capsule generates overlap events. This mine
   detects the `Pawn` object channel.
4. In a Blueprint child of `ProximityMine`, implement `On Target Detected` and
   `On Countdown Started` to show a warning or update the interface.
5. To allow disarming, create a `UBTrapUtilityProfile` Data Asset named
   `DA_MineCounterplay`. Enable `Can Be Disarmed`, set
   `Disarm Duration Seconds = 0.5`, and assign it to the mine's
   `Trap Utility Profile`.
6. In the player's interaction Blueprint, cast the actor found by the
   interaction trace to `ProximityMine` and store it in a `Current Mine`
   variable. Then use that variable for the hold interaction:

```text
IA_Interact: Started
  → Is Valid (Current Mine)
  → Begin Disarm Trap
      Target = Current Mine
      Interacting Actor = Self
  → Branch (Return Value)
      True  → Show disarm progress
      False → Show "cannot disarm"

IA_Interact: Canceled
  → Is Valid (Current Mine)
  → Cancel Trap Interaction
```

`Begin Disarm Trap` returns false if the mine has no counterplay profile,
disarming is not enabled, the mine is not armed, or another interaction is
already running. On success, use `On Trap Disarm Started` for the progress UI
and `On Trap Disarmed By Actor` for completion feedback. Keep the disarm
duration shorter than the remaining explosion delay or the mine can trigger
before the interaction finishes. Leaving the detection
radius before the countdown begins clears the pending detection; leaving after
`On Countdown Started` does not cancel the explosion countdown.
