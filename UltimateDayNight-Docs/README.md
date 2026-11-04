# Ultimate Day Night — Blueprint User Guide

This guide explains how to install, configure, and use Ultimate Day Night from Unreal Editor and Blueprints. Blueprint node and editor names are kept in English so they can be found directly in the editor.

For complete, reusable graph recipes, see [Blueprint Examples](BLUEPRINT_EXAMPLES.md).

## 1. What the plugin does

Ultimate Day Night provides a world clock, optional calendar and season data, timed gameplay events, and controls for the scene's day/night lighting and sky. Configure a map with a `DayNightControllerActor`, then control the running system from Blueprints.

The basic setup does not require custom assets: the controller can use the project settings and automatically find or set up sky actors. Profiles, calendars, season profiles, and timed-event presets are optional.

## 2. Install and enable

1. Put the `UltimateDayNight` plugin folder in the project's `Plugins` folder. The `.uplugin` file must be at the root of that folder.
2. Open the project in Unreal Editor.
3. Open `Edit > Plugins`, search for `Ultimate Day Night`, enable it, and restart the editor if prompted.
4. Open `Project Settings` and search for `Ultimate Day Night Settings` to review the plugin defaults.

Use a plugin build compatible with the Unreal Engine version used by the project. If the editor reports a missing dependency or an incompatible build, resolve that reported issue before continuing.

## 3. First map setup

### Recommended: controller with automatic sky setup

1. In the Content Browser, create a Blueprint Class derived from `DayNightControllerActor`. Name it something descriptive, for example `BP_DayNightController`.
2. Drag the Blueprint into the level. Use one controller per map to keep its startup settings unambiguous.
3. In the controller's Details panel, leave `Allow Auto Spawn` enabled for the simplest setup.
4. Press Play. The controller applies its settings when play begins and asks the plugin to find or set up the sky actors.

`Allow Auto Spawn` is enabled by default. When it is disabled, the controller does not request automatic sky setup; use an explicitly configured sky setup instead.

### Optional: explicit sky components

For a map where you want direct control over the managed lights and atmosphere:

1. Create a Blueprint Class based on the `UDN Master Actor` class and place it in the level.
2. Configure its built-in sun, moon, sky light, sky atmosphere, fog, post-process, and cloud components in Details.
3. On the `DayNightControllerActor`, disable `Allow Auto Spawn` so it does not request a second automatic setup.
4. Play the level and check that the scene uses the intended components.

The master actor is optional. Do not add it just to get a basic cycle working.

## 4. Defaults and important settings

Search `Project Settings` for `Ultimate Day Night Settings`. The following defaults are particularly useful to know:

| Setting | Default | Effect |
|---|---:|---|
| `Start Time Of Day` | `8.0` | Initial clock time, in hours on a 0–24 clock. |
| `Time Scale` | `1.0` | Multiplier for the simulation speed. |
| `Seconds Per Game Minute` | `1.0` | At scale `1.0`, one real second advances one in-game minute. |
| `Require Controller Actor` | enabled | The world clock starts paused until a controller is placed in the map and applies its settings. |
| `Auto Spawn Sky Actors` | enabled | Allows automatic sky setup when the system is activated. |
| `Default Performance Profile` | `Balanced` | Initial performance profile. The available choices are `Simple`, `Balanced`, and `Production`. |
| `Use Simple Astronomy` | disabled | Optional approximate sun-position calculation using the configured location, time zone, and day of year. |

Other useful defaults are sunrise at `6:00`, sunset at `18:00`, and the debug overlay disabled. Settings such as `Disabled Maps`, optional curves, automatic light binding, and automatic skylight recapture can be changed when a map needs them; they are not required for the first setup.

On `DayNightControllerActor`, the initial values are `Allow Auto Spawn` enabled, `Pause Time` disabled, and `Time Scale` set to `1.0`. `Override Start Time` and `Override Start Date` are disabled. The controller's `World Profile`, `Calendar Asset`, and `Timed Event Presets` are empty until assigned.

### How the clock speed is calculated

The real-time interval is set by `Seconds Per Game Minute`; `Time Scale` multiplies it. With both at `1.0`, one real second advances one in-game minute. For example, a scale of `60` advances about one in-game hour per real second. `Pause Time` is a separate on/off control.

## 5. Control time from a Blueprint

`DayNightBPLibrary` provides world-context nodes that can be used in a Level Blueprint, Actor Blueprint, or Widget Blueprint. Search by the exact function name below; Unreal may display spaces between words in the node title.

### Set a starting time and speed

In the Level Blueprint:

1. Add `Event BeginPlay`.
2. Call `GetDayNightSubsystem`.
3. From the returned subsystem, call `SetTimeOfDayHours` on the subsystem, or use the library node `SetTimeOfDay`.
4. Call `SetTimeScale` with `30.0`.
5. Call `PauseTime` with `false`.

The world then starts at 08:00 and advances at 30 times its base rate. To move the clock forward immediately, call `AddHours` with `2.0`.

### Common time and state nodes

| Node | Use |
|---|---|
| `SetTimeOfDay` / `GetTimeOfDay` | Set or read the time in hours. |
| `SetTimeScale` / `GetTimeScale` | Set or read the simulation-speed multiplier. |
| `PauseTime` / `IsTimePaused` | Pause, resume, or query the clock. |
| `AddHours` | Advance or rewind the clock by a number of hours. |
| `GetDayPhase` | Read `Night`, `Dawn`, `Day`, or `Dusk`. |
| `GetNightFactor` / `GetVisibilityFactor` | Read normalized night and visibility values for gameplay decisions. |
| `SkipToNextPhase` | Move to the next day phase; choose whether to trigger transitions. |
| `RefreshSkyState` | Reapply the current time and profile to the managed sky. |

Use `SetTimeOfDay` to position the clock directly, for example while testing a look. Use normal time progression or `AddHours` when gameplay needs to pass through time and scheduled events.

## 6. Profiles, calendars, and seasons

These Data Assets are optional. To create one, use the Content Browser's `Add > Miscellaneous > Data Asset` command and choose the matching asset class:

| Asset class | What it configures |
|---|---|
| `DayNightWorldProfileDataAsset` | Time speed, optional starting date/time, sunrise/sunset, astronomy, calendar, curves, cloud settings, and visual overrides. |
| `DayNightCalendarDataAsset` | Month lengths and names, weekday names, season ranges, and special dates. |
| `DayNightSeasonProfileDataAsset` | Optional sunrise/sunset offsets and visual curves for a season. Assign it to a season entry in a calendar. |
| `DayNightTimedEventPresetDataAsset` | A reusable collection of timed gameplay rules; see the next section. |

Assign a world profile to the controller's `World Profile` field. A profile only applies its starting date or time when its corresponding `Apply Start Date When Profile Is Applied` or `Apply Start Time When Profile Is Applied` option is enabled. Its calendar is used unless the controller has an explicit `Calendar Asset`; the controller's `Time Scale` is then applied.

In a calendar, define the months in order, supply the weekday names in their intended order, and add season date ranges. Each season can reference a season profile. Add a special date with a tag, display name, month, and day; query it with `IsSpecialDate`.

At runtime, the relevant nodes include `ApplyWorldProfile`, `SetCalendarAsset`, `GetCurrentDateTime`, `SetCurrentDateTime`, `AddDays`, `AddMonths`, `GetDayOfYear`, `GetDayOfWeekName`, `GetCurrentSeasonName`, and `SetSeasonOverride`.

## 7. Scheduled events

Use a timed-event preset when several gameplay Blueprints need to respond to named times:

1. Create a `DayNightTimedEventPresetDataAsset`.
2. Add rules. Set `Enabled`, `Event Tag`, and `Trigger Hour`.
3. Leave `Trigger Once Per Day` enabled unless a rule must repeat during the same day.
4. Add conditions only when needed: `Require Phase`, `Require Day Of Week`, `Require Day Of Month`, `Require Month`, `Require Season`, or `Require Special Date`.
5. Add the preset to the controller's `Timed Event Presets` array. Alternatively, register it in a Blueprint with `RegisterTimedEventPreset`.
6. Bind to the subsystem delegate `OnTimedEventTriggered` and branch on its `Event Tag` value.

`Group Tag` lets a Blueprint call `TriggerEventGroupNow` to emit all enabled rules in that group immediately. `HasTriggeredEventToday` can check whether an event tag has already fired.

### Example: shop opening hours

Create two enabled rules in one preset:

| Event Tag | Trigger Hour | Once per day |
|---|---:|---|
| `ShopOpen` | `8.0` | enabled |
| `ShopClose` | `20.0` | enabled |

Add the preset to the map controller. In the shop Blueprint, bind to `OnTimedEventTriggered`, compare `Event Tag` with `ShopOpen` and `ShopClose`, then enable or disable the shop's interaction and visuals.

## 8. Progressive Blueprint examples

### A. Turn lights on at dusk

In a light-manager Actor Blueprint, use `Event BeginPlay` to get the subsystem and bind to `OnDayPhaseChanged`. Switch on the phase: turn the lights on for `Dusk` or `Night`, and off for `Dawn` or `Day`. This responds to phase transitions without polling the clock every frame.

### B. Advance crops once per day

In a farming manager Blueprint, bind to `OnDayChanged`. In the bound event, loop over the crop actors and increment each crop's growth stage. If growth depends on the season, also read `GetCurrentSeasonName` and apply the matching rule.

### C. Show time in a widget

Create a Widget Blueprint with a `TextBlock`. On `Event Construct`, start a `Set Timer by Event` at a modest interval such as one second. In the timer event, call `GetTimeOfDay` and `GetCurrentDateTime`, format the values, and update the text. Stop or clear the timer when the widget is removed.

### D. Skip to morning when sleeping

When the player confirms sleep, call `AddHours` with the number of hours needed to reach the desired morning time. This advances through the intervening clock time. Use `SetTimeOfDay` instead only when you intentionally want to reposition the display clock directly.

### E. Save and restore the temporal state

In a SaveGame Blueprint, add a variable of type `DayNightSnapshot`. Before saving, call `MakeDayNightSnapshot` and store the result. After loading, call `RestoreDayNightSnapshot` with that value and choose whether to refresh the sky and trigger events. The snapshot includes the date/time, time scale, pause state, active world profile, calendar override, astronomy setting, moon-cycle offset, and season override.

## 9. Blueprint delegates

The subsystem exposes these commonly used delegates:

| Delegate | Payload / use |
|---|---|
| `OnTimeOfDayChanged` | New clock time. |
| `OnTimeScaleChanged` | New time-scale value. |
| `OnPauseChanged` | New paused state. |
| `OnDayPhaseChanged` | New `EDayPhase` value. |
| `OnSunrise`, `OnSunset` | Sunrise and sunset transitions. |
| `OnSunriseExact`, `OnSunsetExact` | Exact configured sunrise and sunset boundaries. |
| `OnDateChanged`, `OnDayChanged` | Updated date or new day. |
| `OnSeasonChanged` | New season name. |
| `OnMoonPhaseChanged` | New moon phase. |
| `OnSpecialDateStarted` | Tag of a special calendar date. |
| `OnTimedEventTriggered` | Tag of a timed rule. |
| `OnWorldProfileChanged` | The applied world profile. |

Bind once when the owning Blueprint is initialized, and unbind when the Blueprint's lifetime requires it. Prefer transition delegates for event-driven gameplay rather than checking the same condition every frame.

## 10. Clouds and sky controls

The controller's `Cloud Settings` and the world profile's cloud settings can select:

- `Use Project Settings`: use the project's automatic cloud choice.
- `Disabled`: do not manage clouds.
- `Volumetric Clouds`: use volumetric cloud settings.
- `Classic Clouds`: use the classic cloud dome and its material settings.

Cloud appearance fields such as coverage, density, brightness, speed, and tint are available for tuning. The controller only applies its cloud settings when it is explicitly overriding the profile's cloud configuration.

Use `AutoSetupSkyActors` to request automatic discovery/setup, and `RefreshSkyState` after deliberate Blueprint changes to the sky. The subsystem also exposes getters and setters for the sun light, moon light, sky light, sky atmosphere, height fog, and volumetric clouds.

## 11. Diagnostics

For a quick runtime check, call `GetRuntimeInspectorSummary`. For a validation result and its message, call `ValidateRuntimeEnvironment`. `GetRuntimeDomainHealth` provides status for the time, sky, scheduler, and profile areas.

The project setting `Disabled Maps` can disable the system for specific map names. If a map appears inactive, check that list as well as `Require Controller Actor` and the controller's `Pause Time`.

## 12. Troubleshooting

### The clock does not advance

- Confirm a `DayNightControllerActor` is placed in the level when `Require Controller Actor` is enabled.
- Check the controller's `Pause Time` and the Blueprint calls to `PauseTime`.
- Check that `Time Scale` is greater than zero.
- Check whether the current map is listed under `Disabled Maps`.

### The clock changes but the lighting does not

- Confirm `Allow Auto Spawn` is enabled, or place and configure the optional `UDN Master Actor`.
- Check that the selected sun light and other managed components are present and enabled in the level.
- Call `RefreshSkyState` after changing the setup at runtime.
- Use `GetRuntimeInspectorSummary` or `ValidateRuntimeEnvironment` to inspect the active setup.

### A scheduled event is missing

- Confirm the preset is assigned to `Timed Event Presets` or registered with `RegisterTimedEventPreset`.
- Check that the rule is enabled, has an `Event Tag`, and has the intended `Trigger Hour`.
- Check all enabled date, season, phase, and special-date conditions against the active calendar.
- Advance through the rule's time with normal progression or `AddHours`. A direct `SetTimeOfDay` reposition is not a substitute for passing through scheduled times.

### The wrong season or date appears

- Confirm the intended calendar is assigned to the controller or its world profile.
- Remember that the controller's explicit `Calendar Asset` takes precedence over the profile's calendar.
- Check month lengths, season start/end dates, and the date assigned to the controller.

### A large time scale is unexpectedly fast

`Time Scale` multiplies the rate set by `Seconds Per Game Minute`. Reduce either value, or pause the clock while the map is inactive.

## 13. Important behavior and limits

- With `Require Controller Actor` enabled (the default), a map without a controller starts paused and does not automatically set up its sky.
- Automatic sky setup depends on the map's settings and on the actors available in that level. Use the optional master actor when you need explicit component control.
- `SetTimeOfDay` positions the clock directly. For gameplay that should pass through scheduled times, use normal progression or `AddHours`.
- Calendar, world, season, and timed-event assets are optional; without assigning them, do not expect custom calendar rules or scheduled events.
- Simple astronomy is optional and approximate. It is not enabled by default.
- Day/night rendering depends on the scene's lighting, atmosphere, cloud, fog, and exposure configuration. The plugin's time controls do not replace level-specific visual tuning.
