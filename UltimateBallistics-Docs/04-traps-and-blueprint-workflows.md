# Traps and Blueprint workflows

The plugin includes concrete trap actors that can be placed in a level or used as parents for project-specific Blueprint children. Choose a concrete actor class; `UBTrapBase` and several category base classes are abstract.

## Common explosive-trap setup

For a trap that uses the shared `UBTrapBase` explosion flow:

1. Place a concrete trap actor in the level.
2. Assign a `UBExplosionProfile` to **Explosion Profile**.
3. Configure **Is Armed**, **Arming Delay**, **Can Retrigger**, and **Destroy Delay After Explosion** as needed.
4. Use the trap's built-in trigger or call **Trigger Trap** from a Blueprint event.
5. Bind **On Trap Triggered**, **Before Trap Triggered**, **After Trap Triggered**, **On Trap Armed**, or **On Trap Disarmed** as needed.

The shared base defaults to armed, with no arming delay, no retrigger, no player-only restriction, and a `2 second` destroy delay after an explosion. The inherited **Trigger Trap** requires an **Explosion Profile**; specialized trap actors can have their own effects and requirements.

## Available trap groups

Search for the concrete actor classes in the Blueprint parent picker or placeable actor list:

- **Projectile traps:** `ArrowTrap` and `GrenadeTrap`. Configure the projectile class, launch behavior, and reload/volley settings shown on the selected actor.
- **Utility traps:** `AlarmTrap`, `FlashTrap`, `EMPTrap`, and `FoamTrap`. Their shared **Utility Trap** settings include detection/effect radii, trigger delay, overlap activation, line-of-sight checks, and whether to affect only the triggering actor. Optional activation VFX and sound can be assigned.
- **Area-effect traps:** `GasTrap`, `PoisonGasTrap`, `FlameTrap`, and `ElectricTrap`. Their **Area Effect** settings include effect radius and duration, damage per second, tick interval, particles, and looping sound.
- **Mechanical traps:** `BearTrap`, `NetTrap`, and `SpikeTrap`.
- **Other concrete trap actors:** `ProximityMine`, `TimedExplosive`, `RemoteExplosive`, `TripwireTrap`, `LaserTripwire`, `LaserGrid`, `Claymore`, `ExplosiveBarrel`, `PressurePlate`, `BouncingBetty`, `SentryGun`, `GasCanister`, and `StickyBomb`.

The available properties and Blueprint events depend on the selected concrete class. For example, utility traps provide **Execute Utility Effect** and **BP On Utility Effect Applied**; area-effect traps provide **Start Effect**, **Stop Effect**, **BP On Effect Started**, and **BP On Effect Ended**. Check the actor's Details panel and Blueprint event menu for its class-specific options.

## Disarm, sabotage, recovery, and relay

For the shared counterplay and relay options:

1. Create a `UBTrapUtilityProfile` Data Asset.
2. Assign it to **Trap Utility Profile** on the trap.
3. Enable **Can Be Disarmed**, **Can Be Sabotaged**, or **Can Be Recovered** as appropriate, and set their durations.
4. To link traps, configure **Trap Channel**, **Dispatch Relay On Triggered**, **Relay Target Channel**, and **Relay Action** on the profile.
5. From a Blueprint interaction, call **Begin Disarm Trap**, **Begin Sabotage Trap**, or **Recover Trap** and check the returned Boolean. Call **Cancel Trap Interaction** if an in-progress interaction is interrupted.

**Begin Sabotage Trap** triggers the trap after its configured sabotage delay. **Recover Trap** is available only when allowed by the profile and the trap is no longer armed without having been triggered. **Trigger Linked Traps** dispatches the configured relay action to matching traps; a positive **Relay Range** limits the receiving distance.

The trap base also exposes **Arm Trap**, **Disarm Trap**, **Reset Trap**, **Can Trigger**, **Should Trigger For Actor**, and **Is Trap Interaction In Progress**.

## Reusable Blueprint pattern

For an interaction-driven trap:

1. Store a reference to the target trap actor when the player starts interacting.
2. Call **Can Trigger** or **Is Trap Interaction In Progress** if the interaction needs a state check.
3. Call the appropriate trap action and branch on its Boolean result.
4. Use the trap's Blueprint event or **On Trap Triggered** to update the interface, play a project-specific animation, or record the result.
5. If the interaction is canceled before its timer completes, call **Cancel Trap Interaction**.

Trap effects are not universal player-status systems. For example, utility and area-effect actors expose events and settings that your character Blueprint can use to apply project-specific UI or gameplay responses.
