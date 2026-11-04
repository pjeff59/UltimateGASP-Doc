# Widgets, themes, and speakers

## Supplied widget

The shipped `Default Widget Class` is `WBP_Dialogue_Ready`. Other variants available in the Content Browser are:

- `WBP_Dialogue_Cinematic`
- `WBP_Dialogue_RPGChoices`
- `WBP_Dialogue_Minimal`
- `WBP_Dialogue_Chronicle`

Select a variant in `Default Widget Class`, or duplicate one into your project to customize it. The supplied widget displays a speaker name, line, optional portrait, and available choices.

## Theme

Create a `DialogueTheme` asset and assign it to `Default Theme` in `Project Settings > Ultimate Dialogue`. Widgets derived from `DialogueWidgetDefault` can also set their own `Theme Override`, which takes precedence over the project theme.

| Theme property | Effect | Initial value |
|---|---|---|
| `Background Color` | Background tint and opacity | 50% black |
| `Speaker Color` | Speaker name color | White |
| `Line Color` | Dialogue line color | White |
| `Choice Color` | Choice text color | Off-white |
| `Choice Highlight Color` | Highlight for the focused or hovered choice | Yellow |
| `Global Scale` | Overall widget scale | `1.0` |
| `Text Scale` | Speaker, line, and choice text scale | `1.0` |

Scale values range from `0.5` to `3.0`. A value of `1.0` keeps the base scale.

## Speakers, names, and portraits

You can set `Speaker Display Name` and `Portrait` directly on each `Dialogue: Line` task. Values on the line take precedence.

To centralize speaker data:

1. Create a `SpeakerDatabase` asset.
2. Add an entry for each speaker with `Speaker Id`, `Display Name`, `Portrait`, and any available emotions.
3. Assign the asset to `Speaker Database` in the `Ultimate Dialogue` project settings.
4. Use the same `Speaker Id` in `Dialogue: Line` tasks.

If a line does not provide a name or portrait, the supplied widget uses the database values. If no name is resolved, it displays the `Speaker Id`; if no portrait is resolved, it hides the image.

## Customize a widget

### Quick visual changes

Duplicate `WBP_Dialogue_Ready` into your project, edit its layout in the Designer, and assign the copy to `Default Widget Class`. You can also start from another supplied variant.

For a layout derived from `DialogueWidgetDefault`, keep the UMG elements bound by name. In particular, `ChoicesBox` is needed to display choices; line and portrait widgets use `SpeakerText`, `LineText`, and `PortraitImage`.

### Custom Widget Blueprint

1. Create a Widget Blueprint derived from `DialogueWidget`.
2. In the Blueprint that starts the conversation, create the widget, add it to the viewport, and call `Bind To Dialogue` with the instance returned by `Start Dialogue`.
3. Implement `On Prompt Received` to update the UI and `On Dialogue Finished` to close it.
4. Keep a reference to the instance. Build buttons from `Prompt.Choices`; when the visible choice `N` is clicked, call `Advance(N)`.
5. After binding, call `Reemit Current Prompt` on the instance to receive a prompt that may have been emitted before the widget was created.

If you create the widget manually, also handle its focus, input mode, and controls. `Show Default Dialogue Widget` performs these steps for the configured widget.

## Built-in controls

The supplied widget accepts keyboard, mouse, and gamepad input. `E`, `Space Bar`, and `Enter` advance or confirm; number keys `1` to `9` select the first nine visible choices; up/down arrows, D-pad, or stick move the selection. The gamepad bottom face button confirms. `Escape` and the gamepad right face button may play `SFX Back`, but do not close the dialogue.
