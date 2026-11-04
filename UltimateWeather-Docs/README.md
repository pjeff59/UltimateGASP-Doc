# Ultimate Weather — Blueprint User Guide

This guide covers the features available to users in Unreal Editor and Blueprints. The [French guide](../FR/README.md) covers the same workflows.

## 1. What the plugin does

Ultimate Weather provides a world-level weather manager and Blueprint-accessible data for weather visuals and gameplay. You can:

- switch between weather presets with smooth or immediate transitions;
- optionally cycle through configured weather transitions or play a scripted weather front;
- read a forecast and current weather data in a HUD or gameplay Blueprint;
- change weather while a player-controlled Pawn overlaps a local zone;
- calculate local exposure using sky traces and shelter volumes;
- apply weather damage inside configured hazard volumes;
- optionally accumulate wetness and snow coverage, define spatial surface regions, and submit surface-deformation requests.

The weather manager is shared by the world; zones do not provide an independent climate per player. Weather impact values are data for your game to consume: for example, apply a movement penalty in your own Blueprint. Hazard damage is applied automatically only when its damage option is enabled.

## 2. Requirements and installation

- Use the Unreal Engine version compatible with the project containing this plugin (the current project setup uses Unreal Engine 5.8).
- Place the `UltimateWeather` plugin folder under the project’s `Plugins` directory, alongside `UltimateWeather.uplugin`.
- In Unreal Editor, open **Edit > Plugins**, find **Ultimate Weather**, enable it, and restart the editor if requested.
- The plugin descriptor enables the engine modules `Niagara` and `ProceduralMeshComponent` as dependencies. If Unreal reports one as unavailable, enable the corresponding engine plugin in **Edit > Plugins**.

The plugin does not include ready-made weather presets, Niagara systems, sounds, or project materials. Create or assign the assets you want to use.

## 3. Add a weather manager

For a level-wide setup:

1. In the Content Browser, choose **Add > Blueprint Class**.
2. Select `WeatherDirector` as the parent class and create a Blueprint, for example `BP_WeatherDirector`.
3. Place one instance in the level.
4. Open its defaults and configure the `WeatherManager` component described below.

`WeatherDirector` includes a `Weather Manager` component and is registered as the primary manager by default. It also follows the primary player by default (`bFollowPrimaryPlayer`), keeping its weather effects near the player. Set that option to false if the actor should remain in a fixed location.

Alternatively, add a `Weather Manager` component to a persistent actor in the level. Use one primary weather manager for the world.

## 4. Create and configure weather presets

Create a Data Asset based on `WeatherPreset` for each weather state:

1. In the Content Browser, choose **Add > Miscellaneous > Data Asset**.
2. Select `WeatherPreset`, name the asset, and open it.
3. Set `WeatherType`, `DisplayName`, and the visual/gameplay values needed by your game.
4. Create additional presets for the other states that you want to use.

The `WeatherType` choices shown in the editor are `Clear / Sunny`, `Cloudy`, `Overcast`, `Rain`, `Heavy Rain / Storm`, `Snow`, `Blizzard`, `Fog`, `Dense Fog`, `Sandstorm`, `Hail`, and `Custom`.

Important preset properties:

| Property | Purpose |
|---|---|
| `Visuals` | Fog, sunlight, cloud coverage, wetness target, and snow target. |
| `SkySettings` | Optional sky-related values for a compatible level setup. |
| `FXConfig` | Optional Niagara and audio effects. Assign your own assets. |
| `GameplayImpact` | General gameplay values such as movement multiplier, visibility range, ambient temperature, noise reduction, and environmental damage per second. These values do not automatically change your character’s movement or status systems. |
| `WindIntensity`, `WindDirection`, `WindForce` | Wind category and direction/force data for Blueprint use and configured effects. |
| `MinDuration`, `MaxDuration` | Automatic-cycle duration bounds, in seconds. Defaults are 120 and 600 seconds. |
| `PossibleTransitions` | Candidate weather states used by random automatic changes. |

Neutral gameplay multipliers are generally `1.0`; weather presets only have an effect on game systems that read and apply their values. Niagara and audio effects are optional.

## 5. Configure and control the manager

On the `Weather Manager` component:

1. Add your assets to `AvailablePresets`.
2. Assign `InitialPreset`.
3. Set `AvailableWeatherFronts` if you plan to play fronts.
4. Choose whether `bAutoWeatherCycle` should be enabled. It is **true by default**; checks occur every `AutoCycleCheckInterval` seconds (60 by default), after the current preset’s duration has elapsed.
5. For random cycling, configure valid `PossibleTransitions` on the active preset. For a fixed sequence, disable automatic cycling or use a weather front.

Useful Blueprint nodes (search these names in the Blueprint action menu):

- `Get Weather Manager` — find the manager for the current world.
- `Set Weather` — transition to a preset; default transition duration is 30 seconds.
- `Set Weather By Type` / `Set Weather By Tag` — find an assigned preset and transition to it.
- `Force Weather` — apply a preset immediately.
- `Set Auto Weather Cycle` — enable or disable random cycling at runtime.
- `Get Current Weather Type`, `Get Current Preset`, `Is Transitioning`, `Get Transition Progress`.
- `Get Current Gameplay Impact`, `Get Current Visuals`, `Get Wind Info`, `Get Ambient Temperature`.
- `Set Wind Direction`, `Set Wind Force`, `Clear Wind Overrides`.

### Example: clear weather, then a storm warning

```text
Event BeginPlay
  -> Get Weather Manager
  -> Set Auto Weather Cycle(false)
  -> Set Weather(DA_Weather_Clear, 0)

Event StormWarning
  -> Get Weather Manager
  -> Set Weather(DA_Weather_HeavyRain, 25)
```

In this example, the zero-second initial transition applies the preset immediately. The later call blends to the storm preset over 25 seconds. Use `Force Weather` when an immediate runtime override is more appropriate.

## 6. Scripted fronts and forecasts

Create a Data Asset based on `WeatherFrontPreset`. Add ordered `Stages` and set each stage’s `TargetWeather` (or `PresetOverride`), `StageDuration`, and `TransitionDuration`. `StageDuration` is the time spent in the stage, excluding the transition into it. `ForecastConfidence` is a display value from 0 to 1.

1. Add the front to the manager’s `AvailableWeatherFronts`.
2. From a Blueprint event, call `Apply Weather Front` with the front asset.
3. Use `Cancel Weather Front` if gameplay should stop the sequence early.
4. Bind `On Weather Front Stage Changed` or `On Weather Front Finished` for UI and level logic.
5. Read `Get Forecast Entries`; `Format Forecast Entry` can format an entry for display. Refresh a forecast-driven widget after `On Forecast Updated`.

The manager has four forecast entries by default. Forecast times are estimates, especially when they include future random cycling. A front can loop its stages and can resume the automatic cycle after completion; these are optional settings on the front asset.

## 7. Local weather zones

Place a `WeatherZone` in the level and configure its box volume:

- assign `ZonePreset`;
- set `TransitionDuration` (20 seconds by default);
- adjust `Priority` if zones overlap;
- leave `bAffectOnlyPlayerControlledPawns` enabled to affect only player-controlled Pawns;
- leave `bRevertOnExit` and `bPersistPreviousWeather` enabled if the previous weather should be restored when leaving.

When eligible zones overlap, the zone with the higher `Priority` wins. The zone needs a valid preset and an overlap with the target Pawn. `On Actor Entered` and `On Actor Left` can drive additional Blueprint behavior.

For a non-rectangular region, use `WeatherSplineZone`. Edit its spline points and dimensions (`ZoneHeight` and `HeightOffset`), then call `Rebuild Zone` after changing the shape at runtime. `Is Zone Shape Valid` and `Contains World Location` are available for checks.

## 8. Shelter and actor exposure

Add a `Weather Exposure` component to an actor whose Blueprint needs to read local conditions. By default, it checks sky visibility every 0.2 seconds using the `Visibility` trace channel and uses `WeatherShelterZone` volumes.

Place a `WeatherShelterZone` around a building, tunnel, or partial cover. Tune:

- `ExposureMultiplier` (0 = protected, 1 = exposed; default 0);
- `FXVisibilityMultiplier` (default 0.2);
- `AudioOcclusionMultiplier` (default 0.4);
- `TemperatureOffset` (default +4 degrees);
- `ShelterState` (default `Under Cover`);
- `Priority` when shelter volumes overlap.

Useful nodes include `Get Current Exposure State`, `Is Outdoors`, `Get Effective Weather Intensity`, `Get Effective Ambient Temperature`, `Get Effective Gameplay Impact`, and `Force Refresh Exposure`. Bind `On Exposure Changed` and `On Shelter State Changed` to refresh a status display.

The manager can attenuate local effects using the exposure state. A Niagara system that should fade gradually must consume the configured intensity parameter (`User.WeatherIntensity` by default); an effect that ignores it cannot show a partial intensity fade. Ensure the visibility trace channel and level collision correctly represent roofs and ceilings, or use explicit shelter volumes for important interiors.

For a periodically refreshed status snapshot, add a `Weather Runtime Status` component. It refreshes every 0.25 seconds by default and can use a `Weather Exposure` component when present. `Get Current Status Effects`, `Get Current Debug Info`, `On Status Changed`, and `On Debug Info Changed` can drive HUD indicators; these values do not apply status effects to your game by themselves.

### Example: weather status on a HUD

```text
On Exposure Changed
  -> Get Current Exposure State
  -> Update shelter label and exposure indicator

On Weather Changed (Weather Manager)
  -> Get Effective Gameplay Impact (Weather Exposure)
  -> Update temperature / visibility indicators
```

## 9. Weather hazard zones

Place a `WeatherHazardZone` and configure its bounds and activation:

- `ActiveWeatherTypes` selects the weather types; an empty list accepts any type.
- `bRequireSevereWeather` is optional and false by default.
- `bEnabled` and `bOnlyAffectPlayerControlledPawns` are true by default.
- `DamagePerSecondOverride` uses the preset’s `EnvironmentalDamagePerSecond` when it is zero or less.
- `DamageMultiplier` defaults to 1; `HazardTickInterval` defaults to 0.5 seconds.
- `bApplyDamage` defaults to true. `bUseWeatherExposure` defaults to true and reduces damage according to the actor’s exposure.

`Filter Damage For Actor` is the Blueprint override point for resistance or immunity; it runs before damage is applied. `On Hazard Tick` is a notification after the calculated damage has been applied (when `bApplyDamage` is enabled), so do not use it to cancel that damage. If applying custom damage yourself, disable `bApplyDamage` to avoid double application. `On Actor Entered`, `On Actor Left`, `Is Hazard Active`, and `Set Hazard Enabled` support additional gameplay.

### Example: cold area

1. Set the cold preset’s `EnvironmentalDamagePerSecond` to the desired rate.
2. Set `ActiveWeatherTypes` to `Blizzard`, and enable `bRequireSevereWeather` if only severe blizzards should count.
3. Leave `DamagePerSecondOverride` at 0 to use the preset value.
4. Use `Filter Damage For Actor` to return a reduced value for insulated equipment.
5. Place a shelter volume over a safe hut; exposure reduces the hazard damage there.

## 10. Optional surface weather and deformation

### Wetness and snow coverage

The manager’s `SurfaceSettings` is disabled by default. To use it, enable `SurfaceSettings > Enabled`, assign a compatible `Surface Material Parameters` collection, and ensure the project’s materials read that collection. This is an optional, world-level effect; it does not automatically modify every material.

`Enable Surface Materials` attempts to activate a compatible collection and returns success or failure. `Disable Surface Materials`, `Get Surface State`, `Set Surface State`, and `Refresh Surface Materials` are available in Blueprint. `Set Surface State` sets global wetness and snow coverage in the 0–1 range. Accumulation is controlled by `bAccumulate`, wetting/drying rates, snow buildup/melt rates, and `MeltTemperature`. Preset wetness and snow values are targets; accumulation is gradual.

### Spatial surface regions and movable actors

The `Surface Environment` setting on `WeatherZone`, `WeatherSplineZone`, and `WeatherWeightmapZone` can define a local surface state independently of whether that zone overrides the weather preset:

- `Disabled` leaves the region without a surface state.
- `InitialState` starts from the specified `Wetness` and `SnowCoverage`, then lets weather and surface rates change them.
- `PersistentBase` behaves similarly but keeps the specified values as minimum coverage.

Both coverage values are in the 0–1 range; `BlendDistance` is an inward blend in world centimeters (300 cm by default). `Surface State Key` can identify a zone when saving/restoring state. Call `Rebuild Surface Environment` after changing its surface settings. For a `WeatherSplineZone`, call `Rebuild Zone` after changing spline geometry at runtime.

`WeatherWeightmapZone` uses a box for its bounds and a binary row-major mask. `Configure Mask` expects a complete 2–1009 by 2–1009 grid; zero cells are excluded and nonzero cells are included. Check the returned success and message, or query `Is Mask Valid`.

Use `Get Spatial Surface Subsystem` to access `Get Surface State At Location`, `Get Surface Climate At Location`, `Refresh Surface Zones`, `Set Zone Surface State`, `Capture Surface State`, and `Restore Surface State`. Check the success/message outputs of operations that can fail. The captured data can be stored in your own save workflow. Spatial surface data does not create precipitation or make arbitrary project materials compatible.

For an actor that moves through different climates, add `Weather Actor Surface` to its Blueprint or use `Enable Actor Surface Weather`. It stores an individual surface state and can respect shelter exposure. The actor’s mesh materials must be compatible; call `Refresh Material Bindings` after changing meshes or materials and check its result.

For vegetation interaction, place one `FoliageInteractionActor` in the world, assign its `InteractionCollection`, and use materials that read that collection. `RecoveryDuration` controls how quickly the recorded interaction fades (0.8 seconds by default). The actor does not make an incompatible foliage material react automatically.

### Footprints and other surface marks

Surface deformation requires a configured deformation setup and compatible project materials. On `WeatherDirector`, assign the required deformation presets and material references before using deformation nodes. A `Deformation Source` component can submit traces from a character or vehicle; use a socket for a foot or wheel, and enable continuous mode for wheel tracks.

For a simple Blueprint test, call `Get Deformation Subsystem`, then `Apply Footprint` at a world-space ground location. `Trace And Deform`, `Apply Wheel Track`, `Apply Projectile Impact`, and `Apply Explosion Crater` are alternatives. A false return from an apply node means the request was not accepted; check that the subsystem and its required configuration are available. The visible result also depends on the project’s compatible materials.

## 11. Common problems and checks

| Symptom | Check |
|---|---|
| Weather does not change | Confirm the manager is in the level, `InitialPreset` is assigned, and the target preset is in `AvailablePresets`. |
| Random cycling appears idle | Confirm `bAutoWeatherCycle` is enabled, the preset duration has elapsed, and `PossibleTransitions` contains valid targets. |
| A zone has no effect | Confirm its preset, box bounds, Pawn overlap/collision, and `bAffectOnlyPlayerControlledPawns` setting. Check overlapping zones’ priorities. |
| The HUD has no forecast | Read `Get Forecast Entries`, populate stages/presets, and react to `On Forecast Updated`. |
| The player is still exposed indoors | Check `Weather Exposure`, shelter bounds/priority, `bUseShelterZones`, and the sky-trace collision channel. |
| Hazard does not damage | Check that the weather matches `ActiveWeatherTypes`, the target overlaps the volume, and the resolved damage rate is greater than zero. |
| Niagara does not fade under cover | Make sure the Niagara system consumes the manager’s intensity parameter; assigning an effect alone does not guarantee partial fading. |
| Surface materials do not react | Verify that surface weather is enabled, the collection is assigned and compatible, and the material actually reads its parameters. |
| Deformation requests are accepted but not visible | Verify the subsystem configuration and material/render-target setup used by the project. |

## 12. Important limits

- Weather, zones, and the shared surface state are managed at world level, not independently per player.
- General gameplay multipliers are data only; connect them to movement, stamina, visibility, or other game systems in your Blueprint.
- Visual effects, sounds, and material responses require compatible assets and configuration in the project.
- Sky exposure is inferred from collision traces. Explicit shelter volumes are more predictable for complex interiors.
