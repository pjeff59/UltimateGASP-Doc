# Blueprint Node Reference

Node display names are written as they appear in Blueprint graphs. Search for
`UA` or `UltimateAudio` in the Blueprint node menu. The examples below assume
that the calling Blueprint has a valid world context, such as a Level
Blueprint, Actor, or PlayerController.

## Audio manager and event dispatchers

| Node or event | Purpose |
| --- | --- |
| `Get Audio Manager` | Gets the game-instance audio manager. Use it when you need to bind to its event dispatchers. |
| `Initialize Audio` | Explicitly initializes audio through the manager. Usually unnecessary for basic playback. |
| `OnTrackChanged` | Broadcasts the new playlist track index and display name. |
| `OnMusicStopped` | Broadcasts when managed music stops. |
| `OnMusicStateChanged` | Broadcasts the previous and newly selected music state. |
| `OnAudioFloatParameterChanged` | Broadcasts a named parameter and its new value. |
| `OnEnvironmentProfileChanged` | Broadcasts the previous and new environment profile. |

Bind once from a persistent or well-defined Blueprint. Avoid binding again on
every widget open or level tick without unbinding, or the same event can be
handled multiple times.

## Music playback and playlists

| Node | Behavior |
| --- | --- |
| `UA Play Music` | Plays one sound and crossfades from the active music. A fade of `-1` uses the project default. It leaves playlist/state-stack playback. |
| `UA Play Playlist` | Starts a `Music Playlist`, following its loop and shuffle settings. |
| `UA Stop Music` | Fades out music and clears the active music-state flow. |
| `UA Pause Music` / `UA Resume Music` | Pauses and resumes music; playlist auto-advance pauses too. |
| `UA Is Music Playing` | Pure query for managed music playback. |
| `UA Get Current Track Name` | Gets the explicit track name, or the sound asset name if no name is set. |
| `UA Get Current Track Progress` | Gets normalized progress from `0` to `1`; remains fixed while paused. |
| `UA Get Current Track Duration` | Gets duration in seconds, or `-1` when unknown. |
| `UA Next Playlist Track` / `UA Previous Playlist Track` | Moves through the active playlist. |
| `UA Jump To Track` | Jumps to the requested zero-based track index. |

`UA Play Music`, `UA Play Playlist`, and `UA Stop Music` are direct playback
operations that exit the state-driven flow; do not use them for a temporary
override if you need to preserve the music-state stack.

## Named music states

| Node | Behavior |
| --- | --- |
| `UA Register Music State` | Registers or replaces one runtime state. |
| `UA Register Music State Collection` | Registers the states in a collection. `Clear Existing` optionally clears the previous registry first. |
| `UA Set Music State` | Registers and immediately activates one state. |
| `UA Activate Music State` | Makes a registered state the new base and clears temporary stack entries. Returns success/failure. |
| `UA Push Music State` | Adds a registered state as a temporary override. Returns success/failure. |
| `UA Pop Music State` | Removes the named state, or the logical top state when no name is provided. |
| `UA Clear Music State Stack` | Removes all temporary state entries and reevaluates playback. |
| `UA Unregister Music State` | Removes a state from the runtime registry. |
| `UA Clear Music State` | Clears the current state without necessarily stopping the playlist. |
| `UA Get Current Music State` / `UA Get Previous Music State` | Gets the current or previous selected state name. |
| `UA Is Music State Active` | Tests whether a state is currently selected, not merely present but hidden in the stack. |
| `UA Is Music State Registered` | Tests whether the state name has been registered. |
| `UA Get Music State Stack` | Gets the current stack entries. Use this to detect a state hidden by a higher-priority state. |
| `UA Get Music State Priority` | Gets the priority registered for a state. |

Resolution rules:

1. The highest priority in the active stack wins.
2. The most recently pushed state wins a priority tie.
3. `Activate` establishes a new base and clears the temporary stack.
4. Pair each temporary push with a pop. Multiple pushes of the same name are
   not automatically deduplicated.
5. When the stack is emptied, the manager can restore the music context that
   existed before the first override.

## Ambience and environment profiles

| Node | Behavior |
| --- | --- |
| `UA Play Ambiance` / `UA Stop Ambiance` | Starts or fades out the separate ambience layer. Configure the sound asset itself to loop if required. |
| `UA Set Ambiance Volume` / `UA Get Ambiance Volume` | Sets or queries the independent ambience volume. |
| `UA Is Ambiance Playing` | Queries whether the managed ambience component is playing. |
| `UA Apply Environment Profile` | Applies one profile's `SoundMix`, ambience, and optional parameter preset. |
| `UA Clear Environment Profile` | Clears the current profile and fades as requested. It does not restore a previous profile automatically. |
| `UA Push Sound Mix` / `UA Pop Sound Mix` | Pushes or removes an explicit Unreal `SoundMix`. |

Only one environment profile is current. For overlapping zone triggers, use a
central zone director that chooses which profile should be applied; do not let
each trigger clear the profile unconditionally on exit.

## Sound effects and cue libraries

| Node | Behavior |
| --- | --- |
| `UA Play SFX 2D` | Plays a non-spatialized sound in the `SFX` category. |
| `UA Play SFX 2D Category` | Plays a non-spatialized sound in an explicit category such as `UI` or `Dialogue`. |
| `UA Play SFX At Location` | Plays a spatialized sound at a world position. |
| `UA Play SFX Attached` | Plays a sound attached to a scene component or socket. |
| `UA Stop SFX` | Stops/fades one active SFX returned by a managed `UA Play SFX` node. Returns false for an invalid, unmanaged, or no-longer-playing component. |
| `UA Stop All Audio` | Stops music, ambience, and SFX managed by UltimateAudio. It does not force-stop unrelated audio systems. |
| `UA Play Cue By Name` | Plays a named entry from an `Audio Category Library`, including its category and variation settings. |
| `UA Play Random Cue` | Chooses an eligible cue using weight, cooldown, and recent-history rules. |

The two cue-playback nodes are 2D. Use the SFX location or attached nodes for
spatial audio.

## Categories, mute, ducking, and save/load

| Node | Behavior |
| --- | --- |
| `UA Set Category Volume` / `UA Get Category Volume` | Sets or gets the runtime volume for `Music`, `SFX`, `UI`, or `Dialogue`. Keep slider values in the `0`–`1` range. |
| `UA Set Category Muted` / `UA Is Category Muted` | Mutes/unmutes without replacing the stored slider value. |
| `UA Begin Ducking` | Lowers a target category. `-1` uses the project default ratio and attack time. |
| `UA End Ducking` | Restores a ducked category. `-1` uses the project default release time. |
| `UA Save Audio Settings` | Saves category volumes and mute values to a SaveGame slot. Default slot: `AudioPrefs`. |
| `UA Load Audio Settings` | Resets categories to project defaults, then applies the slot if it exists. Load once at startup. |

`Music` volume and ducking are applied directly to managed music playback.
Category control for `SFX`, `UI`, and `Dialogue` requires the appropriate
`SoundClass` assignment in project settings.

## Runtime float parameters

| Node | Behavior |
| --- | --- |
| `UA Set Audio Float Parameter` | Stores a named float and, by default, applies it to active managed components. |
| `UA Get Audio Float Parameter` | Returns whether the value exists and outputs its value. |
| `UA Has Audio Float Parameter` | Tests whether a value is stored. |
| `UA Clear Audio Float Parameter` | Removes one stored value; optionally sends zero to active components first. |
| `UA Clear All Audio Float Parameters` | Removes all stored values; optionally sends zero to active components first. |
| `UA Apply Audio Parameter Preset` | Applies values and target-layer options from a preset asset. |
| `UA Push Audio Float Parameters To Component` | Applies stored values to an externally created audio component. |

The exact parameter name must also exist in the Sound Cue or MetaSound. Stored
values remain until replaced or explicitly cleared. Applying a preset does not
clear names omitted by that preset.
