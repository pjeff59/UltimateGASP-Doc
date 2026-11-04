# Blueprint Recipes and Troubleshooting

These are short patterns to adapt to your own gameplay events. Node names are
shown as they appear in Blueprint.

## Recipes

### Start exploration music and switch to combat

```text
Event BeginPlay
-> UA Register Music State Collection (DA_MusicStates_Global, true)
-> UA Activate Music State (Exploration)

Encounter Started
-> UA Push Music State (Combat)

Encounter Ended
-> UA Pop Music State (Combat)
```

### Play a UI click

```text
Button OnClicked
-> UA Play SFX 2D Category (Snd_UI_Click, UI, 1.0, 1.0)
```

Assign a `SoundClass` to the `UI` category if the UI slider or mute control
must affect this sound.

### Play an impact at a world position

```text
Hit Event
-> UA Play SFX At Location (Snd_Impact, HitLocation, 1.0, RandomPitch)
```

Use an attenuation asset on the sound when distance-based falloff is
required.

### Play a footstep attached to a moving character

```text
Footstep Anim Notify
-> UA Play SFX Attached (Snd_Footstep, CharacterMesh, FootSocket)
```

### Display the current track

```text
Get Audio Manager
-> Bind Event to OnTrackChanged
-> Update the track-name text in your widget
```

Use `UA Get Current Track Name` for the displayed name. For a progress bar,
query `UA Get Current Track Progress` on a UI timer rather than updating every
gameplay frame.

### Duck music during dialogue

```text
Dialogue Started
-> UA Begin Ducking (Music, 0.3)
-> UA Play SFX 2D Category (VoiceLine, Dialogue)

Dialogue Ended
-> UA End Ducking (Music)
```

### Apply an underwater context

```text
Player Entered Water
-> UA Apply Environment Profile (DA_AudioEnv_Underwater)
-> UA Set Audio Float Parameter (Underwater, 1.0, true)

Player Left Water
-> UA Set Audio Float Parameter (Underwater, 0.0, true)
-> UA Apply Environment Profile (DA_AudioEnv_Exterior)
```

Reapply the intended land profile explicitly; clearing a profile does not
restore the previous one or automatically remove its float parameters.

### Create a day/night base with temporary combat

Register states such as `Village_Day` and `Village_Night` at low priority,
then `Village_Panic` and `Village_Combat` at higher priorities. Treat the
ambience as an environment profile, separate from the music playlist:

```text
Day phase changed
-> Audio Director updates the selected base state/profile

Combat begins
-> UA Push Music State (Village_Combat)

Combat ends
-> UA Pop Music State (Village_Combat)
```

If day/night can change while combat is active, let one director manage both
the base and overlays. Do not use `UA Activate Music State` for each phase
change while temporary overrides are active, because activation clears the
state stack.

## Troubleshooting

| Symptom | Check |
| --- | --- |
| No sound plays | Confirm the plugin is enabled, the sound asset is valid, the master output is audible, and the node is called from a Blueprint with world context. Test `UA Play SFX 2D` first. |
| Category slider does not affect SFX/UI/Dialogue | Assign the matching `SoundClass` in **Project Settings > Plugins > UltimateGASP > Ultimate Audio** and verify the sound uses that class. `Music` is applied directly. |
| Playlist does not advance | Auto-advance uses the sound's reported duration. For a MetaSound with unknown duration, call `UA Next Playlist Track` from your own event. |
| Fade is too short or too long | Adjust `Default Music Fade Time` or pass a fade value to the relevant node/track. |
| Combat music starts but never ends | Verify every successful `UA Push Music State` has one matching pop. Guard against duplicate enemy notifications and aggregate encounter state centrally. |
| State request has no effect | Register its collection first and verify the `FName` spelling/case and the bool output of `UA Activate Music State` or `UA Push Music State`. |
| A lower-priority state does not appear active | It can remain in the stack while masked. Use `UA Get Music State Stack` to inspect the entries. |
| Random cue does not play | Confirm the library entry and sound are assigned, `Weight` is greater than zero, and the cue is not on cooldown. |
| Ambience stops or overlaps unexpectedly | Check the profile's stop options, whether the sound asset loops, and which profile the central zone director currently selects. |
| Float parameter has no audible effect | The Sound Cue/MetaSound must use the same exact parameter name; verify the preset target checkboxes or the node's apply-to-active option. |
| Ducking has no effect on a category | Check that ducking is enabled. `SFX`, `UI`, and `Dialogue` also need the matching `SoundClass`; `Music` is applied directly. |
| A sound keeps playing after Stop All Audio | That node only stops sounds managed by UltimateAudio. Stop audio started by other systems through those systems. |
| Options reset after a map change | Load the save slot once at startup, save after changes, and avoid reloading on each map or widget open. |

## Before shipping

- Test category defaults, slider changes, mute, and save/load.
- Test playlists with both finite-duration sounds and any MetaSounds with
  unknown duration.
- Exercise nested state transitions and every push/pop exit path.
- Test overlapping zone triggers through a central profile selector.
- Confirm each Sound Cue/MetaSound consumes the parameter names sent by
  Blueprint.
- Remember that UltimateAudio does not add multiplayer replication.
