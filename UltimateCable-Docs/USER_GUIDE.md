# Ultimate Cable — Blueprint user guide

Ultimate Cable provides a cable component that can be configured from Unreal Editor and controlled from Blueprints. This guide uses the names displayed by the editor; where a name is English, its purpose is explained in the surrounding text.

## 1. Install and enable the plugin

1. Close Unreal Editor before adding the plugin to a project.
2. Place the `UltimateCable` plugin folder in the project's `Plugins` folder. Create that folder if the project does not already have one.
3. Open the project.
4. In Unreal Editor, open **Edit > Plugins**, search for **Ultimate Cable**, and enable it.
5. Restart the editor if prompted.
6. Create an Actor Blueprint. In its **Components** panel, choose **Add** and search for `UltimateCableComponent`.

If the plugin does not appear in the plugin browser, check that its folder is directly inside the project's `Plugins` folder and reopen the project.

## 2. First cable

1. In the Actor Blueprint, add two `SceneComponent` components and name them `AnchorStart` and `AnchorEnd`.
2. Position the anchors where the cable should begin and end.
3. Add `UltimateCableComponent`.
4. In the component's Details panel, set `CableLength`, `NumSegments`, and `CableRadius`.
5. In the Event Graph, use `Event BeginPlay` and call `PinEndsToComponents` on the cable. Connect `AnchorStart`, `AnchorEnd`, and `None` for each socket if no socket is needed.
6. Compile the Blueprint, place an instance in the level, and press **Play**.

Set the segment count and length before attaching the ends. `SetNumSegments` can reinitialize the cable, so changing it after setup can reset its current shape.

### What the defaults do

The component starts with `CableLength = 100`, `NumSegments = 24`, `LocalEnd = (0, 0, -100)`, and `CableRadius = 2`. The first particle starts pinned at the component location; the other end is not attached until you add an attachment. Use `PinEndsToComponents` for a cable controlled by two endpoints.

Other important defaults:

| Setting | Default | Effect |
|---|---:|---|
| `Substep` | `0.008` seconds | Simulation time step. A smaller value can improve stability at higher cost. |
| `SolverIterations` | `8` | Constraint-solving work per simulation step. |
| `bEnableStiffness` | enabled | Adds the cable stiffness constraint. |
| `bUseXPBD` | enabled | Uses the XPBD solver option. |
| `XPBDComplianceDistance` / `XPBDComplianceStiffness` | `0` / `0` | Compliance values; zero is the stiffest setting. |
| `GravityScale` | `1` | Uses world gravity at its normal scale. |
| `bEnableCollision` | enabled | Enables cable collision checks. |
| `SweepRadius` / `CCDSubdivisions` | `2` / `6` | Collision sweep radius and subdivisions. |
| `TubeSides` / `UVTiling` | `6` / `1` | Rendered tube detail and lengthwise UV repetition. |
| `bUseDistanceLOD` | enabled | Allows the simulation time step to adapt with distance. |
| `bUseParallelSolver` / `ParallelMinSegments` | enabled / `16` | Allows parallel solving when the segment count reaches the threshold. |
| `UpdateMode` | `Full Simulation` | Runs the cable simulation. |
| `BreakTensionThreshold` / `TensionNotifyThreshold` | `0` / `0` | Global break and tension notifications are disabled at zero. |
| `bEnableSegmentBreaking` | disabled | Automatic segment breaking is off until enabled. |

`NumSegments` represents the number of spans between particles. For example, 24 segments use 25 particles. `CableRadius` controls visible thickness; `SweepRadius` is a separate collision setting.

## 3. Simulation presets

Call `SetSimulationPreset` and select one of the editor-visible values:

| Preset | Substep | Solver iterations | Stiffness | XPBD | CCD subdivisions |
|---|---:|---:|---|---|---:|
| `Fast` | `0.008` | `4` | off | off | `3` |
| `Stable` | `0.004` | `8` | on | off | `6` |
| `Cinematic` | `0.0025` | `14` | on | off | `10` |
| `Stable (XPBD)` | `0.004` | `8` | on | on | `6` |
| `Cinematic (XPBD)` | `0.0025` | `14` | on | on | `10` |

The XPBD presets also set compliance values. `Stable (XPBD)` uses `XPBDComplianceDistance = 0.000001` and `XPBDComplianceStiffness = 0.000002`; `Cinematic (XPBD)` uses `0.0000005` and `0.000001`. A preset changes simulation values; it does not attach endpoints or select a material.

## 4. Update modes

`UpdateMode` can be set to:

- `Full Simulation` — updates the simulated cable. This is the component default.
- `Visual Only` — rebuilds a visual curve between the endpoints using `VisualSag` and `VisualSagDirection`; it does not run the normal simulation.
- `Frozen Snapshot` — stops the simulation and keeps the current shape. Attached particles continue to follow their scene-component anchors.

Use `SetUpdateMode` to change modes at runtime. Use `RebuildVisualOnlyCable` after changing the visual-only sag settings. Automatic tension breaks and automatic segment breaks are evaluated during `Full Simulation`; manual segment break and repair functions are separate.

## 5. Blueprint nodes and events

The most useful nodes are grouped below by task. Add a reference to the cable component in the Blueprint graph, then drag from it to search for its component nodes.

### Setup and attachments

- `SetCableLength`
- `SetNumSegments`
- `ResetCable`
- `PinParticleToComponent`
- `UnpinParticle`
- `PinEndsToComponents`
- `SetSlidingGrip`
- `ClearSlidingGrip`
- `SetSimulationPreset`
- `ApplyPresetAsset`
- `ApplyBreakProfileAsset`
- `SetUpdateMode`
- `RearmCable`

Sockets are optional. `PinParticleToComponent` associates a particle with a scene component or one of its sockets. `SetSlidingGrip` sets the component's single sliding grip; `ClearSlidingGrip` removes it.

### Forces and collision

- `AddParticleImpulse`
- `AddImpulse`
- `SetParticlePosition`
- `SetWindForce`
- `ClearWindForce`
- `AddActorsToCollisionIgnore`
- `ClearCollisionIgnore`

`SetWindForce` adds a dedicated wind contribution; `ClearWindForce` removes that contribution without changing `UniformForce`.

### Queries

- `GetParticlePositions`
- `GetParticlesCount`
- `GetCurrentSegmentLengths`
- `GetCurrentTotalLength`
- `GetTensionRatio`
- `GetSegmentTensionRatio`
- `GetParticleVelocity`
- `GetNearestParticleIndex`
- `GetClosestPointOnCable`
- `GetPositionAlongCable`
- `SlideAlongCable`
- `IsAnySegmentColliding`
- `IsSegmentBroken`
- `GetBrokenSegmentIndices`
- `IsBroken`

Particle and segment indices start at zero. `GetTensionRatio` compares the current intact cable length with its rest length: `1` is at rest length and a value above `1` indicates a stretched cable.

### Breaking and repairing

- `BreakSegment`
- `RepairSegment`
- `RepairAllSegments`
- `BreakClosestSegment`
- `BreakIfOverTension`

Component events:

- `OnCableBreak` — fires when the global tension ratio reaches the positive `BreakTensionThreshold`; the last particle is unpinned.
- `OnTensionChanged` — fires when the ratio crosses `TensionNotifyThreshold` in either direction. Its outputs are `OldRatio` and `NewRatio`. It is not a continuous per-frame notification.
- `OnCableSegmentBroken` — provides `SegmentIndex` and `BreakWorldLocation`.
- `OnCableSegmentRepaired` — provides `SegmentIndex` and the updated world location.

The global break and tension-notification thresholds default to zero, which disables those behaviors. Automatic segment breaking is also disabled by default.

When using the corresponding preconfigured gameplay actor, these additional events are available:

- Grapple: `OnGrappleAttached`, `OnGrappleDetached`, and `OnGrappleFullyRetracted`.
- Tripwire: `OnTripwireTriggered` and `OnTripwireReset`, fired when the collision state changes.
- Winch: `OnReachedMinLength` and `OnReachedMaxLength`, fired when the configured length limit is reached.

### Materials

- `SetOverrideMaterial`
- `ApplyMaterialByPath`
- `ApplyDefaultRopeMaterial`
- `Load And Apply Cable Material Async`

You can assign a material in the component's material slot or use `SetOverrideMaterial`. `Load And Apply Cable Material Async` applies a material when loading succeeds and exposes `Completed` and `Failed` execution pins. `ApplyDefaultRopeMaterial` only succeeds if `/UltimateCable/Materials/M_CableRope` exists in the project.

## 6. Reusable Blueprint examples

### Example A — Two-anchor rope

Create `AnchorStart` and `AnchorEnd` scene components, then:

```text
Event BeginPlay
  -> SetSimulationPreset(Cable, Stable (XPBD))
  -> SetNumSegments(Cable, 24, true)
  -> SetCableLength(Cable, 800, true)
  -> PinEndsToComponents(Cable, AnchorStart, None, AnchorEnd, None)
```

Use a length at least as large as the distance between the anchors when you want slack. A shorter rest length pulls the simulated cable taut.

### Example B — Lightweight decorative line

For a cable that only needs a visual sag:

```text
Select cable component in Details
  -> UpdateMode = Visual Only
  -> VisualSag = 40
  -> VisualSagDirection = (0, 0, -1)
  -> bEnableCollision = false
```

If you change these values at runtime, call `SetUpdateMode(Visual Only)` or `RebuildVisualOnlyCable` to rebuild the visual points. This mode does not produce physical movement.

### Example C — Grapple cable that retracts

Keep a float variable `GrappleLength` only if you need to show the current length elsewhere. Configure the anchors when the grapple attaches, then shorten the cable on each tick:

```text
On grapple attach
  -> SetupGrapple(Cable, StartComponent, AnchorComponent, AnchorSocket, 700)

Event Tick(Delta Seconds)
  -> UpdateRetract(Cable, Delta Seconds, 900, 120)
```

`SetupGrapple` attaches both ends and sets the rest length. `UpdateRetract` shortens that length to its minimum. These helper nodes do not move or pull the player; connect your own character movement logic if that is required.

### Example D — Zipline path and moving object

Create a float variable `T` initialized to `0`, two endpoint components, and a mover component:

```text
Event BeginPlay
  -> SetupZipline(Cable, AnchorA, AnchorB, 0)

Event Tick(Delta Seconds)
  -> SlideAlongCable(Cable, T, 300, Delta Seconds)
  -> if successful: SetWorldLocation(Mover, OutWorldPos)
```

With `LengthOverride = 0`, `SetupZipline` uses the distance between the endpoints. `SlideAlongCable` updates the normalized `T` value between `0` and `1`; it returns a position but does not move the object for you. The resulting movement follows the cable's segment interpolation and is not guaranteed to have constant speed along a curved path.

### Example E — Tripwire alarm

Set up the cable and query collision from the Blueprint:

```text
Event BeginPlay
  -> SetupTripwire(Cable, AnchorA, AnchorB, 1.5, 2, true)

Event Tick
  -> CheckTripwireTriggered(Cable)
  -> compare result with previous trigger state
  -> on false-to-true: play alarm and set trigger state
  -> on true-to-false: reset trigger state
```

`SetupTripwire` enables collision and sets the cable length from the anchor distance. The query reports whether any selected cable segment is colliding; it is not an overlap event for a particular actor. Make sure the cable can collide with the objects you intend to detect.

### Example F — Break a cable at the hit location

For a manually controlled puzzle or damage response:

```text
On cable hit
  -> BreakClosestSegment(Cable, Hit Location, true)

OnCableSegmentBroken(SegmentIndex, BreakWorldLocation)
  -> play break sound at BreakWorldLocation
  -> update puzzle state
```

To repair every broken segment when an interaction succeeds:

```text
On repair interaction
  -> RepairAllSegments(Cable)
```

`BreakClosestSegment` chooses the nearest segment to the supplied world position. Automatic segment breaking is separate: enable `bEnableSegmentBreaking`, set `SegmentBreakThresholdRatio`, and, if desired, configure `MaxAutoBrokenSegments`.

### Example G — React to cable tension

Set `TensionNotifyThreshold` to a positive ratio, such as `1.1`, and bind `OnTensionChanged`:

```text
OnTensionChanged(OldRatio, NewRatio)
  -> if NewRatio >= threshold: start taut-cable audio
  -> otherwise: stop taut-cable audio
```

The event fires on threshold crossings, so use `GetTensionRatio` when you need to query the current value at another point in the Blueprint.

### Example H — Turn wind on and off

```text
On storm begins
  -> SetWindForce(Cable, (1, 0, 0), 300)

On storm ends
  -> ClearWindForce(Cable)
```

The wind direction is normalized by the node. Its force is added separately from `UniformForce`.

### Example I — Change material without blocking on its load

```text
On visual theme changes
  -> Load And Apply Cable Material Async(Cable, MaterialSoftReference)
      Completed -> continue with the new material
      Failed -> keep the existing material or show a fallback
```

The node applies the loaded material to the cable. If using the synchronous `ApplyMaterialByPath` node instead, ensure the asset exists at the requested path.

### Example J — Freeze and resume a cable

```text
When the scene reaches its desired pose
  -> SetUpdateMode(Cable, Frozen Snapshot)

When motion should resume
  -> SetUpdateMode(Cable, Full Simulation)
```

The snapshot stops the normal simulation. Anchored particles still track their attached components.

## 7. Presets and reusable Data Assets

`ApplyPresetAsset` applies a `Cable Preset Data Asset`, including segment count, length, simulation, collision, rendering, and its optional material. `CaptureFromComponent` copies the current component values into that asset. The asset does not store endpoint attachments, wind, update mode, optimization settings, or `VelocityDamping`.

`ApplyBreakProfileAsset` applies the automatic segment-break, repair, global break, and tension-notification values from a `Cable Break Profile Data Asset`. The component's own defaults remain in effect until you change them or apply a profile.

When applying a preset with a material, the material reference may be loaded synchronously. Prefer `Load And Apply Cable Material Async` for a material you want to load asynchronously.

## 8. Performance and important limitations

- `NumSegments` affects both simulation and rendered detail. Start with the default `24` and increase it only when the shape needs more detail.
- Collision is enabled by default. Turn it off for decorative cables that do not need collision.
- `bUseParallelSolver` is enabled by default, but parallel solving is used only when the segment count reaches `ParallelMinSegments` (default `16`).
- `bUseDistanceLOD` is enabled by default. `bSkipUpdateWhenNotVisible` and `bSkipUpdateWhenOwnerNotRecentlyRendered` are optional and default to off.
- Automatic tension breaking and automatic segment breaking require `Full Simulation`. Manually calling `BreakSegment` remains available as a separate gameplay action.
- Segment auto-repair requires both `bAutoRepairBrokenSegments = true` and a positive `BrokenSegmentRepairDelay`.
- `CableRadius` is the visible tube thickness; it does not set the collision sweep radius.
- The grapple, zipline, tripwire, and wind helpers configure or query the cable. They do not implement a complete character movement, alarm, or force-response system.
- No built-in network synchronization workflow is provided.

## 9. Troubleshooting

### The cable does not appear

- Check that the component is visible and has at least one material. A material can be assigned in its slot or through `SetOverrideMaterial`.
- Check that its endpoints are not at the same position and that the cable has at least one segment.
- If using `Visual Only`, check `VisualSagDirection` and call `RebuildVisualOnlyCable` after changing visual settings.

### The cable does not follow one or both anchors

- Call `PinEndsToComponents` with valid `SceneComponent` references.
- Supply the exact socket name if using a socket; use `None` when attaching to the component origin.
- Set the segment count before creating attachments if you also call `SetNumSegments`.

### The cable looks too stretched or too slack

- Compare the distance between the anchors with `CableLength`.
- Use `GetTensionRatio` to inspect the current ratio.
- Try a different simulation preset or adjust `Substep` and `SolverIterations`.

### Collision does not detect an object

- Check `bEnableCollision`, `SweepRadius`, and `CCDSubdivisions`.
- Confirm that the object is available to the cable's collision checks and is not in `CollisionIgnoreActors`.
- For a tripwire, use `CheckTripwireTriggered` and remember it reports cable-segment collision rather than an actor-specific overlap.

### The cable never breaks automatically

- For a global break, set `BreakTensionThreshold` above `0`; the threshold is a cable tension ratio.
- For segment breaks, enable `bEnableSegmentBreaking` and check `SegmentBreakThresholdRatio` and `MaxAutoBrokenSegments`.
- Automatic checks run in `Full Simulation`. Manual `BreakSegment` and `BreakClosestSegment` do not require automatic breaking to be enabled.

### Automatic repair never happens

- Enable `bAutoRepairBrokenSegments`.
- Set `BrokenSegmentRepairDelay` to a value greater than `0`.

### The material does not change

- Confirm the material reference points to a valid material.
- Use the `Completed` and `Failed` pins of `Load And Apply Cable Material Async`.
- `ApplyDefaultRopeMaterial` requires the project asset `/UltimateCable/Materials/M_CableRope`.
