# Getting Started — Blueprint Only

This quick start takes you from enabling the plugin to hearing your first
sound. It assumes Unreal Engine 5.8 and does not require C++ work.

## 1. Install and enable the plugin

1. Place the packaged `UltimateAudio` plugin in your project's `Plugins`
   folder, for example `YourProject/Plugins/UltimateAudio`.
2. Open the project in Unreal Editor.
3. Open **Edit > Plugins**, find **Ultimate Audio**, enable it, and restart
   the editor if prompted.
4. If Unreal reports that the plugin module is not built for this engine
   version, use a plugin package built for UE 5.8 or follow the installation
   instructions provided with your plugin package.

Do not copy the repository's `/Game/Addons/UltimateAudio` example assets into
your project unless you also want those examples. They are project sample
content, not a required part of the runtime plugin.

## 2. Configure audio categories

Open **Edit > Project Settings > Plugins > UltimateGASP > Ultimate Audio**.
The plugin supplies four categories:

| Category | Typical use | Setup note |
| --- | --- | --- |
| `Music` | Background and combat music | Its volume is applied directly to UltimateAudio's music playback. |
| `SFX` | World and gameplay sound effects | Assign a `SoundClass` if you want category volume, mute, or ducking control. |
| `UI` | Menu hover and click sounds | Assign a `SoundClass` and use the category-specific SFX node. |
| `Dialogue` | Voice and dialogue sounds | Assign a `SoundClass` and use the category-specific SFX node. |

Set the default category volumes first. Assign a `SoundClass` for `SFX`, `UI`,
and `Dialogue` before relying on their sliders, mute state, or ducking. A
`SoundMix` is optional and is only needed when your mix design uses one.

## 3. Create a test sound

Use a `SoundWave`, `Sound Cue`, or `MetaSound` that is already imported into
your project and can be heard when played directly.

From a Level Blueprint:

```text
Event BeginPlay
-> UA Play SFX 2D (TestSound, Volume = 1.0, Pitch = 1.0)
```

Press **Play**. If the test sound plays, the plugin is enabled and the basic
Blueprint-to-audio path is working.

For a first music test, create a `Music Playlist` asset, add one or more valid
tracks, then use:

```text
Event BeginPlay
-> UA Play Playlist (DA_Playlist_Test)
```

Alternatively, play one sound directly with `UA Play Music`.

## 4. Create UltimateAudio data assets

The project may expose shortcuts at **Content Browser > Right-click >
UltimateGASP > Ultimate Audio** when the optional UltimateContentManager
editor integration is installed and enabled. You can also create the data
assets through the Content Browser's asset creation menu by searching for the
UltimateAudio asset type.

Start with:

- `Music Playlist` for a track or a group of tracks.
- `Music State Collection` for named gameplay music states and priorities.
- `Audio Category Library` for named or randomized SFX cues.
- `Audio Environment Profile` for zone ambience, a `SoundMix`, and a
  parameter preset.
- `Audio Parameter Preset` for reusable named float values.

See [Data assets and music states](03_Data_Assets_and_States.md) for field-by-
field setup and examples.

## 5. Build the first gameplay transition

Create a `Music State Collection` with an `Exploration` state and a `Combat`
state. Give `Combat` a higher priority. In a Blueprint that has a valid world
context:

```text
Event BeginPlay
-> UA Register Music State Collection (DA_MusicStates_Global, Clear Existing = true)
-> UA Activate Music State (Exploration)
```

When combat starts:

```text
Combat Started
-> UA Push Music State (Combat)
```

When the encounter is actually over:

```text
Combat Ended
-> UA Pop Music State (Combat)
```

Do not push once per enemy unless you also track each enemy's matching exit.
For encounters with several enemies, aggregate their state in a central
Blueprint and push/pop the shared combat state only when the encounter changes
from inactive to active, then back to inactive.

## 6. Open the project documentation

The bilingual entry point is [Documentation/README.md](../README.md). The
English guides continue with the
[complete Blueprint integration workflow](04_Blueprint_Game_Integration.md)
and the [Blueprint node reference](02_Blueprint_Node_Reference.md).

## Quick validation checklist

- The plugin is enabled and the editor restarted if required.
- A known-good `SoundWave`, `Sound Cue`, or `MetaSound` is assigned.
- The master audio output is audible.
- The relevant `SoundClass` is assigned for `SFX`, `UI`, and `Dialogue`
  categories.
- The `Music State Collection` is registered before requesting a named state.
- Every pushed temporary state has a matching pop.
