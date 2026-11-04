# UltimateCable — Changelog

---

## v2.0 — AAA Improvement Pass

### Critical Bug Fixes

#### 🔴 Simulation never advanced (TickComponent)
**Bug:** `TickComponent` updated attachment pins and called `MarkRenderDynamicDataDirty()` but **never called `SubstepSim()`**. The cable particle positions were permanently frozen; no physics ever ran.

**Fix:** Added a fixed-timestep accumulator loop in `TickComponent`:
```
SubstepAccumulator += DeltaTime;
while (SubstepAccumulator >= ActiveSubstep) {
    SubstepSim(ActiveSubstep);
    SubstepAccumulator -= ActiveSubstep;
}
```
The accumulator is clamped to `ActiveSubstep * 8` to prevent spiral-of-death on heavy frames.

#### 🔴 `bSkipUpdateWhenNotVisible` permanently disabled tick
**Bug:** When the cable became non-visible, `SetComponentTickEnabled(false)` was called. The tick was never re-enabled, so moving the cable back into view had no effect.

**Fix:** Changed to `return;` only. The tick remains enabled; the cable resumes simulation as soon as it is visible again.

#### 🟠 `UpdateLODs` corrupted `Substep` property
**Bug:** `UpdateLODs` wrote a scaled substep back into the `Substep` UPROPERTY. After moving far away, the base substep was permanently increased (e.g. `0.008 → 0.032`). Moving close again used the already-corrupted value.

**Fix:** Added `BaseSubstep` private field (set once in `OnRegister`). `ComputeLODSubstep()` scales `BaseSubstep` read-only and returns the active value. `Substep` is never touched at runtime by LOD.

#### 🟠 `ApplyWind` overwrote `UniformForce`
**Bug:** `UUltimateCableGameplayLibrary::ApplyWind` did `Cable->UniformForce = Direction * Strength`, destroying any existing `UniformForce` value the user had set.

**Fix:** Added dedicated `WindForce` private field and `SetWindForce` / `ClearWindForce` BP functions. `ApplyWind` now routes through `SetWindForce`. The integrator sums `UniformForce + WindForce + Gravity` — the two forces are independent.

#### 🟠 `bSkipUpdateWhenOwnerNotRecentlyRendered` was never checked
**Bug:** The property existed and showed in the Details panel but the corresponding code was missing from `TickComponent`.

**Fix:** Added null-safe check:
```cpp
if (bSkipUpdateWhenOwnerNotRecentlyRendered) {
    AActor* Owner = GetOwner();
    if (Owner && !Owner->WasRecentlyRendered(0.1f)) return;
}
```

#### 🟠 `GetTensionRatio` in library used wrong denominator
**Bug:** `GetTensionRatio` divided by `CableLength` (the design-time constant). After `SetCableLength(NewLen, /*bReinitialize=*/false)` the rest lengths were out-of-sync and the ratio was wrong.

**Fix:** `GetTensionRatio()` is now a method on `UUltimateCableComponent` that sums `SegmentRestLengths` for the true rest length. Library delegates to the component method.

#### 🟡 `ATripwireCableActor::Tick` fired no events
**Bug:** `bTriggered` was updated every tick but Blueprint had no way to react to the moment of transition (event/signal). Game logic had to poll every tick.

**Fix:** Added edge detection and `DYNAMIC_MULTICAST_DELEGATE` events:
- `OnTripwireTriggered` — fires once when cable **first** detects a collision
- `OnTripwireReset` — fires once when cable **stops** colliding

---

### New Features

#### `UCablePresetDataAsset`
New `UPrimaryDataAsset` subclass that stores a complete cable configuration:
- All simulation parameters (Substep, SolverIterations, XPBD settings, GravityScale)
- Collision settings (SweepRadius, CCDSubdivisions)
- Rendering settings (CableRadius, TubeSides, UVTiling, Material soft reference)
- Optional preset shortcut (`bUsePreset + SimulationPreset`)

**API:**
```blueprint
CablePreset->ApplyToComponent(MyCable)   // Apply stored config to any cable
CablePreset->CaptureFromComponent(Cable) // Capture current cable state into asset
```

#### `OnCableBreak` / `OnTensionChanged` Delegates
- `OnCableBreak` — fires when tension ratio exceeds `BreakTensionThreshold`. Automatically unpins the last particle. Re-arm with `RearmCable()`.
- `OnTensionChanged(OldRatio, NewRatio)` — fires on edge when tension crosses `TensionNotifyThreshold`.

#### New Component BP Functions
| Function | Category | Description |
|---|---|---|
| `GetTensionRatio()` | Query | `actual_length / rest_length` (1.0 = slack, >1 = taut) |
| `GetParticleVelocity(Index)` | Query | Verlet velocity estimate for a single particle |
| `GetNearestParticleIndex(WorldPos)` | Query | Index of particle closest to a world point |
| `SetWindForce(Direction, Strength)` | Forces | Additive wind — independent of UniformForce |
| `ClearWindForce()` | Forces | Removes the wind force contribution |
| `ApplyPresetAsset(Preset)` | Config | Apply a `UCablePresetDataAsset` to this cable |
| `IsBroken()` | Query | True after `OnCableBreak` fires |
| `RearmCable()` | Config | Reset break state, re-pin last particle |

#### New Library BP Functions
| Function | Description |
|---|---|
| `ClearWind(Cable)` | Clears wind set by `ApplyWind` |
| `GetParticleVelocity(Cable, Index)` | Wraps component method |
| `GetNearestParticleIndex(Cable, WorldPos)` | Wraps component method |
| `GetCableTensionRatio(Cable)` | Wraps corrected component method |
| `ApplyPresetAsset(Cable, Preset)` | Apply a preset data asset |

#### `AGrappleCableActor` Events
- `OnGrappleAttached` — fires in `ActivateGrapple`
- `OnGrappleDetached` — fires in `CancelGrapple`
- `OnGrappleFullyRetracted` — fires once when cable retracts to minimum length (100 uu)

#### `AWinchCableActor` Events
- `OnReachedMinLength` — fires once when winding in reaches `MinLength`
- `OnReachedMaxLength` — fires once when winding out reaches `MaxLength`
- Both events re-arm automatically when the direction reverses

---

### Architecture notes

**Fixed-timestep integration** eliminates simulation instability at low framerates. The substep accumulator ensures physics advances at a constant rate regardless of rendering FPS.

**Wind / Uniform Force separation** means designers can use `UniformForce` as a steady environmental force zone and `SetWindForce` for per-cable dynamic wind without the two clobbering each other.

**LOD substep scaling** is now non-destructive: `BaseSubstep` captures the design-time value and the active substep returned by `ComputeLODSubstep()` is only used for the current tick. The Details panel value never drifts.
