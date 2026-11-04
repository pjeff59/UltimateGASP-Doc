# UMG Audio Options — Blueprint Setup

This guide builds an options widget using the plugin's provided
`UA_AudioOptionsWidget` parent class. You can do the setup in the UMG Designer
and Event Graph; no C++ changes are required.

## 1. Create the widget Blueprint

1. In the Content Browser, create a **Widget Blueprint**.
2. In the Parent Class picker, select `UA_AudioOptionsWidget`.
3. Name the widget, for example `WBP_AudioOptions`.
4. Add and arrange the controls in the UMG Designer.

The parent class recognizes the widget variable names below. Keep the names
exact, including capitalization, and enable **Is Variable** for each widget.
You may change the layout and styling freely.

## 2. Add the required controls

| Exact widget name | Widget type | Purpose |
| --- | --- | --- |
| `MusicSlider` | Slider | Music category volume |
| `SFXSlider` | Slider | SFX category volume |
| `UISlider` | Slider | UI category volume |
| `DialogueSlider` | Slider | Dialogue category volume |
| `BtnPlay` | Button | Start or replay the configured startup playlist |
| `BtnStop` | Button | Stop managed music |
| `BtnNext` | Button | Advance the playlist |
| `BtnPrev` | Button | Go to the previous track |
| `DuckingCheck` | Check Box | Toggle music ducking for the sample controls |
| `CueNameBox` | Editable Text Box | Enter the name of a cue to test |
| `BtnPlayCueByName` | Button | Play the entered cue |
| `BtnPlayRandomCue` | Button | Play a random eligible cue |

Set each slider's minimum to `0` and maximum to `1`. This gives the range
expected by category volume controls.

## 3. Assign optional startup assets

Select the widget or its instance and set:

- `StartupPlaylist`: an optional `Music Playlist`. It is only started when no
  music or playlist is already active, so reopening the options menu does not
  replace gameplay music.
- `CueLibrary`: an optional `Audio Category Library` used by the cue test
  buttons.

## 4. Add the widget to the screen

From a PlayerController, HUD, or another UI-owning Blueprint:

```text
Create Widget (WBP_AudioOptions)
-> Add To Viewport
```

For a pause/options screen, also set the input mode and cursor visibility in
your owning UI Blueprint according to your game's menu flow.

## 5. Add custom controls if needed

If you build your own widget instead of using the provided parent, wire the
controls with these nodes:

```text
Music Slider OnValueChanged(Value)
-> UA Set Category Volume (Music, Value)

SFX Slider OnValueChanged(Value)
-> UA Set Category Volume (SFX, Value)

Mute Button
-> UA Set Category Muted (Music, true)
```

Use `UA Get Category Volume` to initialize a slider and
`UA Set Category Muted` to toggle mute without overwriting the slider value.
For SFX, UI, and Dialogue category changes to affect sounds, assign their
`SoundClass` in Ultimate Audio project settings.

Save when the player confirms changes:

```text
Apply Button OnClicked
-> UA Save Audio Settings (AudioPrefs)
```

Load `AudioPrefs` once when the game starts. Do not reload on every widget
open; loading resets categories to project defaults before applying the
saved values.

## 6. Verify the widget

- Every required widget name matches the table exactly.
- Each slider has a `0`–`1` range.
- The parent class is `UA_AudioOptionsWidget` if you want its built-in sample
  controls.
- `CueLibrary` contains the exact cue names typed into the widget.
- `SFX`, `UI`, and `Dialogue` have their `SoundClass` assignments.
- Values persist after saving and restarting the game.
