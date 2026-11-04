# Ultimate Day Night — Blueprint Examples

These recipes use only the plugin's exposed Blueprint nodes and events. Keep one `DayNightControllerActor` in the level, and create the optional Data Assets referenced in a recipe before assigning them. In graphs, search for the function names as written; Unreal may add spaces in the displayed node title.

## Shared setup: get the world subsystem

In an Actor Blueprint, Level Blueprint, or Widget Blueprint, add `GetDayNightSubsystem` where a recipe needs the system. The function-library nodes such as `SetTimeOfDay`, `AddHours`, and `GetCurrentDateTime` can also be called directly; their world context is supplied by the owning Blueprint.

## 1. Set a playable starting time

**Goal:** start a test map at 08:00 and make one in-game hour pass in about two real seconds.

In the Level Blueprint:

1. Add `Event BeginPlay`.
2. Call `SetTimeOfDay` with `8.0`.
3. Call `SetTimeScale` with `30.0`.
4. Call `PauseTime` with `false`.

The default `Seconds Per Game Minute` is `1.0`, so scale `30.0` advances 30 in-game minutes per real second. Adjust either speed setting if that is too fast or too slow.

## 2. Make a simple time-control widget

**Goal:** give a tester buttons for pause, resume, and advancing one hour.

1. Create a Widget Blueprint with three buttons: `Pause`, `Resume`, and `+1 Hour`.
2. On the `Pause` button's `OnClicked`, call `PauseTime(true)`.
3. On the `Resume` button's `OnClicked`, call `PauseTime(false)`.
4. On `+1 Hour`'s `OnClicked`, call `AddHours(1.0)`.
5. Add a `TextBlock`; on a one-second timer, call `GetCurrentDateTime`, read its `Hour` and `Minute` fields, and update the text.

The widget uses calls into the current world; it does not need to own a separate clock.

## 3. Turn a porch light on at dusk

**Goal:** switch a light on at dusk and off at dawn without checking the clock every frame.

In a light-manager Actor Blueprint:

1. On `Event BeginPlay`, call `GetDayNightSubsystem`.
2. Bind to `OnDayPhaseChanged`.
3. In the bound event, use `Switch on EDayPhase`.
4. For `Dusk` and `Night`, set the light component's visibility to true.
5. For `Dawn` and `Day`, set visibility to false.

For a single event at the configured sunset time, bind to `OnSunsetExact` instead. Use `OnSunriseExact` to turn the light off at the configured sunrise time.

## 4. Open and close a shop by schedule

**Goal:** make a shop available from 08:00 to 20:00.

Create a `DayNightTimedEventPresetDataAsset` with two enabled rules:

| Property | Rule 1 | Rule 2 |
|---|---|---|
| `Event Tag` | `ShopOpen` | `ShopClose` |
| `Trigger Hour` | `8.0` | `20.0` |
| `Trigger Once Per Day` | enabled | enabled |

Add the preset to the controller's `Timed Event Presets` array. In the shop Blueprint, bind to `OnTimedEventTriggered`; compare `Event Tag` with `ShopOpen` and `ShopClose`, then toggle the shop's interaction and visible state.

## 5. Run a festival on a special date

**Goal:** trigger a festival opening only at 10:00 on a named calendar date.

1. In the active `DayNightCalendarDataAsset`, add a `Special Dates` entry with a unique `Tag`, display name, month, and day.
2. In a timed-event preset, add an enabled rule with `Event Tag` set to `FestivalStart` and `Trigger Hour` set to `10.0`.
3. Enable `Require Special Date` and set `Required Special Date Tag` to the calendar entry's tag.
4. Assign the calendar to the controller or its world profile and add the event preset to `Timed Event Presets`.
5. Bind the festival manager to `OnTimedEventTriggered`. When `Event Tag` equals `FestivalStart`, enable the decorations and festival interactions.

The special-date condition is optional; the tag must match the one in the calendar.

## 6. Grow crops when a new day starts

**Goal:** update crop growth once per in-game day.

1. In a farming-manager Actor Blueprint, get the subsystem on `Event BeginPlay`.
2. Bind to `OnDayChanged`.
3. In the bound event, iterate over the crop actors managed by the Blueprint.
4. Call the crop's own `Advance Growth Stage` event or function.
5. If growth depends on the season, call `GetCurrentSeasonName` and choose the appropriate growth amount.

Use the day-change event rather than running this loop on Tick.

## 7. Make an interaction available only during business hours

**Goal:** allow an interaction from 09:00 inclusive to 17:00 exclusive.

In the interaction Blueprint, call `IsWithinTimeWindow` with `Start Hour = 9.0` and `End Hour = 17.0`, then connect its Boolean result to a `Branch`.

- `True`: show or execute the interaction.
- `False`: hide it or display a “come back later” message.

Check again when the player interacts; a prompt that was visible earlier may no longer be valid.

## 8. Advance to the next morning when sleeping

**Goal:** advance the clock to 06:00 and allow crossed time-based events to run.

1. Call `GetTimeOfDay`.
2. If the current time is earlier than `6.0`, calculate `Hours To Advance = 6.0 - Current Time`.
3. Otherwise calculate `Hours To Advance = 30.0 - Current Time` (the remaining hours to midnight plus six).
4. Call `AddHours` with the result.
5. Optionally call `PauseTime(true)` after the transition if the morning should remain frozen.

Replace `6.0` with the desired wake-up time. `AddHours` advances through the intervening time; direct `SetTimeOfDay` is better reserved for repositioning the clock during setup or testing.

## 9. React to a full moon

**Goal:** enable a night-time gameplay modifier only during a full-moon phase.

1. On `Event BeginPlay`, get the subsystem and bind to `OnMoonPhaseChanged`.
2. In the bound event, switch on the `EDayNightMoonPhase` value.
3. On `Full Moon`, enable the modifier.
4. On the other phase values, disable it.
5. To initialize the state immediately after binding, query `GetMoonPhaseType` and apply the same switch once.

For a continuous intensity instead of a phase check, query `GetMoonLightFactor`.

## 10. Display date, time, and season in a HUD

**Goal:** show a readable clock and calendar without updating every frame.

1. Create a Widget Blueprint with separate text blocks for date, time, and season.
2. On `Event Construct`, set a `Set Timer by Event` interval to `1.0` second.
3. In the timer event, call `GetCurrentDateTime`, read its `Year`, `Month`, `Day`, `Hour`, and `Minute` fields, and format them for the date and clock text blocks.
4. Call `GetCurrentSeasonName` and update the season text.
5. Clear the timer when the widget is removed.

Use `OnDateChanged` or `OnSeasonChanged` instead when the widget should update only when those values change.

## 11. Save and restore the whole temporal state

**Goal:** continue a saved session with its previous clock, date, profile, and pause state.

1. In a SaveGame Blueprint, add a variable of type `DayNightSnapshot`.
2. Before saving, call `MakeDayNightSnapshot` and assign its result to that variable.
3. Save the SaveGame object using the normal Unreal save flow.
4. After loading, call `RestoreDayNightSnapshot` and pass the saved snapshot.
5. Set `bRefreshSky` to true when the restored time should immediately refresh the managed sky. Choose `bTriggerEvents` according to whether the restore should produce transition events.

The snapshot records the temporal context supported by the plugin. It does not replace saving unrelated gameplay state such as actor inventories or quest progress.

## 12. Build a quick QA time-jump panel

**Goal:** test the scene lighting at representative times.

Create four buttons and connect their `OnClicked` events to `SetTimeOfDay` with `6.0`, `12.0`, `18.0`, and `0.0`. Add a `Refresh` button that calls `RefreshSkyState` after a deliberate visual setup change. Add a status text block and update it from `GetRuntimeInspectorSummary`.

Use `SetTimeOfDay` for quick visual checks. To test a timed-event rule, advance through its hour with `AddHours` or let the clock run instead.

## 13. Configure a seasonal daylight change

**Goal:** use shorter winter daylight and longer summer daylight.

1. Create a `DayNightCalendarDataAsset` and add season entries with their month/day ranges.
2. For the winter season, assign a `DayNightSeasonProfileDataAsset` with a negative `Sunset Hour Offset` and, if desired, a positive `Sunrise Hour Offset`.
3. For summer, use the opposite offsets.
4. Assign the calendar to the controller's `Calendar Asset` or to the selected world profile.
5. During play, read `GetCurrentSeasonName` or bind to `OnSeasonChanged` to drive any additional gameplay presentation.

Season profiles can also reference optional sun, moon, fog, and exposure curves. Tune those curves in the asset only when the scene needs them.

## Testing checklist

- Confirm the controller is in the level and the clock is not paused.
- Test one direct clock position with `SetTimeOfDay`.
- Test one event by crossing its scheduled hour with `AddHours`.
- Confirm the active calendar/profile on the controller matches the assets used by the recipe.
- Use `GetRuntimeInspectorSummary` or `ValidateRuntimeEnvironment` if the time and visual state do not agree.
