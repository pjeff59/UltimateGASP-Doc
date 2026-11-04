# Troubleshooting and limitations

## The projectile does not move

- Check that **Profile** is assigned and **Initial Speed** is non-zero.
- Check that the input to **Fire In Direction** is a non-zero world-space direction.
- If you use **Spawn Pooled Projectile From Socket**, pass a world-space velocity in **Throw Velocity**. A normalized direction alone has a speed of about `1 cm/s`.
- When you use **Spawn Actor From Class**, initialize the spawned component with **Initialize With Profile** or **Initialize With Profile And Velocity** before firing.

## Impact effects or launch sound are missing

- Check that the profile has the relevant **Launch Cue**, **Trail VFX**, or **Impact VFX** assigned.
- If the profile is selected dynamically after spawn, use **Initialize With Profile** instead of setting **Profile** after Begin Play. The initialization node also starts the profile's launch and trail effects.
- Per-surface entries require a matching physical material and surface type on the hit mesh.
- The base projectile actor has no visible mesh. Add a mesh to a child Blueprint or assign a trail effect if you need an in-flight visual.

## The projectile passes through a target

- Confirm that the target blocks the profile's **Trace Channel**. The default is **Visibility**.
- For swept collision, review **Trace Radius**. A larger radius can help detect thin targets but may change the feel of impacts.
- If **Trace Complex** is enabled, check the target's complex collision setup.

## Damage is not received

- Confirm **Base Damage** is greater than zero for a direct impact, or **Max Damage** is greater than zero for an explosion.
- Make sure the target is configured to receive damage and handles **Event Point Damage** or the relevant damage event in its Blueprint.
- For radial damage, ensure the target is within **Outer Radius** and is detectable by the explosion query.
- If **Single Blast Damage Per Actor** is disabled, a target with multiple overlapping components can receive blast damage more than once. Enable it when one blast should damage each actor only once.

## The projectile launches from an unexpected position

- The socket helper searches the owner's first Skeletal Mesh component for **Socket Name**. Confirm that the spelling and socket are correct.
- Without a matching socket, the helper uses its documented owner-relative fallback (`40 cm` forward and `40 cm` upward). Add the socket or provide a different spawn workflow if that fallback is unsuitable.
- **Direction** and **Throw Velocity** are world-space values; convert local-space vectors before passing them to the node.

## A fuse or trap does not trigger

- Fuses are off by default. Enable the required fuse option and set its radius or time.
- Check **Arm Delay Seconds** and the profile's **Explosion On Impact** assignment.
- For traps using the shared explosion flow, assign **Explosion Profile** and ensure the trap is armed. After a trigger, call **Reset Trap** to reset the state; the trap is armed again by this reset only when **Can Retrigger** is enabled.
- Counterplay nodes require a `UBTrapUtilityProfile` with the corresponding permission enabled.

## Performance and behavior limits

- Runtime projectile reuse is disabled. **Spawn Pooled Projectile From Socket** creates a fresh actor, **Prewarm Pool** has no effect, and returning a projectile does not provide active-pool reuse. Do not budget for projectile pooling based on the node name.
- Adaptive substepping is enabled by default, with up to `8` steps per tick and a maximum step distance of `200 cm`. More steps can improve collision sampling at a CPU cost.
- **Predict Trajectory** and the editor preview actor do not query world collision. **UBTrajectoryVisualizer** can stop the displayed path at a blocking trace, but that trace does not simulate ricochet, penetration, fuses, or damage.
- Per-surface effects require physical materials configured on the hit assets.
- The plugin is designed for single-player use; its projectile, damage, explosion, and trap behavior is not replicated for multiplayer.
- The **Team ID** field on the shared trap base is not currently used to filter triggers.
- Niagara systems, sounds, decal materials, and projectile meshes are supplied by your project; assigning none of them leaves the corresponding presentation absent.
