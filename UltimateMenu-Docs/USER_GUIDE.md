# UltimateMenu user guide

[Français](../FR/GUIDE_UTILISATEUR.md) · [Plugin overview](../../README.md)

This guide describes the current Blueprint workflow. UltimateMenu supplies menu behavior, not finished visual assets: you create the Widget Blueprints, buttons, text and animations yourself. Create only the optional screens your menu actually uses.

## Contents

1. [Install and prepare](#1-install-and-prepare)
2. [Create a minimal main menu](#2-create-a-minimal-main-menu)
3. [Configure and start the menu](#3-configure-and-start-the-menu)
4. [Navigation and main-menu actions](#4-navigation-and-main-menu-actions)
5. [Pause](#5-pause)
6. [Levels and progression](#6-levels-and-progression)
7. [Options and persistence](#7-options-and-persistence)
8. [Intro and credits](#8-intro-and-credits)
9. [Confirmation dialog](#9-confirmation-dialog)
10. [Transitions and loading screen](#10-transitions-and-loading-screen)
11. [Events and state](#11-events-and-state)
12. [Troubleshooting and test checklist](#12-troubleshooting-and-test-checklist)

## 1. Install and prepare

1. Put the `UltimateMenu` folder in your project's `Plugins` folder. Open the project in a compatible Unreal Engine installation.
2. In **Edit → Plugins**, enable **Ultimate Menu**. The plugin declares **CommonUI** and **Enhanced Input** as dependencies; verify they are enabled too. Restart the editor if requested.
3. Create and save a menu map and at least one playable map. Add maps to your project's packaging settings and verify the packaged build can open them. Examples below use `Menu` and `Level_01` as map names; replace them with your own.
4. In **Edit → Project Settings**, search for **Game Viewport Client Class**. For a project without an existing custom viewport client, select **CommonGameViewportClient** and restart if requested. If the project already uses a custom client, check its compatibility before replacing it. Configure CommonUI input for your project's keyboard, mouse or gamepad setup. Create focusable buttons and test focus and back input on every intended device. The plugin does not create your input actions or widget styles.

**Minimum:** a layout, a main-menu widget, a configuration asset, a valid player controller and an initialization call. Intro, options, pause, level selection, credits, confirmations and loading screens are optional and need their own widgets and configuration.

## 2. Create a minimal main menu

### Layout

1. In the **Content Browser**, create a **Widget Blueprint**, choose **UltimateMenuLayout** as parent and name it `WBP_MenuLayout`.
2. In **Designer**, add a **Canvas Panel** and, inside it, a **Common Activatable Widget Stack** named exactly `MenuStack`. Stretch the stack across the canvas and leave it visible.
3. **Compile** and **Save**.

The name `MenuStack` is required. Do not manually add individual menu screens to the viewport: the menu system uses this stack.

### Main screen

1. Create a Widget Blueprint with parent **UltimateMenuMainScreen**; name it `WBP_MainMenu`.
2. Add a background and a vertical list of buttons. Use any labels appropriate to your game; the plugin does not create or connect buttons.
3. Select each button, add its **On Clicked** event in the graph and connect it to the matching action on **self**:

| Example button | Action node |
|---|---|
| New game | `On New Game Clicked` |
| Continue | `On Continue Clicked` |
| Level selection | `On Level Select Clicked` |
| Options | `On Options Clicked` |
| Credits | `On Credits Clicked` |
| Quit | `On Quit Clicked` |

4. Add the **Refresh Button States** event. Use its `Continue Available` value with **Set Is Enabled** on your Continue button, or use it to control visibility. It does not update the button for you.
5. Optionally add a focusable widget named exactly `DefaultFocusWidget` for initial focus. Compile and save.

Only wire buttons for screens you will configure. The default focus target is optional; verify device navigation rather than assuming it is automatic for every layout.

## 3. Configure and start the menu

1. In the **Content Browser**, create **Miscellaneous → Data Asset**, select **UltimateMenuConfig**, and name it `DA_MenuConfig`.
2. Assign **Menu Layout Class** = `WBP_MenuLayout` and **Main Menu Widget Class** = `WBP_MainMenu`.
3. Set **Default New Game Map Name** to `Level_01` for a direct start. Set **Main Menu Map Name** to `Menu` if you will return from pause. These are map names, not widget names.
4. For each optional feature, create a widget with the matching parent below and assign its class in the asset. Leave unused classes empty.

| Asset field | Widget Blueprint parent |
|---|---|
| Intro Widget Class | `UltimateMenuIntroScreen` |
| Level Select Widget Class | `UltimateMenuLevelSelectScreen` |
| Options Widget Class | `UltimateMenuOptionsScreen` |
| Options Graphics Widget Class | `UltimateMenuOptionsGraphicsScreen` |
| Options Audio Widget Class | `UltimateMenuOptionsAudioScreen` |
| Options Controls Widget Class | `UltimateMenuOptionsControlsScreen` |
| Credits Widget Class | `UltimateMenuCreditsScreen` |
| Pause Menu Widget Class | `UltimateMenuPauseScreen` |
| Confirmation Dialog Widget Class | `UltimateMenuConfirmationDialog` |
| Loading Screen Widget Class | `Ultimate Menu Loading Screen` |

5. In the menu map's **Level Blueprint**, use this sequence on **Event BeginPlay**:

```text
Event BeginPlay
  → Get Ultimate Menu Subsystem → Apply Config (Config = DA_MenuConfig)
  → Initialize Menu System (In Owning Player = Get Player Controller, Player Index = 0)
```

Connect both calls to the subsystem returned by **Get Ultimate Menu Subsystem**. Use **Get Player Controller** as the input value, not as the execution target. Initialize once for the current player and map: repeated calls create additional layouts. The first screen is selected on the next tick.

If an intro widget is configured, it runs when the saved intro has not been seen, or whenever **Always Play Intro** is enabled. Otherwise the main screen opens. **Always Play Intro** is off by default. Configuration is not discovered automatically by this Blueprint example: connect `DA_MenuConfig` explicitly.

**Example:** launch the menu map, click New game, and confirm `Level_01` opens. A nonempty **Default New Game Map Name** takes priority even when **Available Levels** has entries. With no default name, one entry starts directly; multiple entries open level selection; no entries means no start. For the latter case configure the level-selection widget.

## 4. Navigation and main-menu actions

Within a screen derived from `UltimateMenuScreenBase`, **Navigate To** pushes the configured screen and **Navigate Back** returns through the menu stack. For example, on an options root widget:

| Button | Action |
|---|---|
| Graphics | `On Graphics Tab Clicked` |
| Audio | `On Audio Tab Clicked` |
| Controls | `On Controls Tab Clicked` |
| Back | `Navigate Back` |

Create `WBP_Options` with parent **UltimateMenuOptionsScreen**, wire these buttons and assign the root and the three desired sub-screen classes. A destination without a configured widget does not open. In **Navigate To**, the screen choices include the labels `Menu Principal`, `Sélection de Niveau`, `Options`, `Options Graphiques`, `Options Audio`, `Options Contrôles`, `Crédits`, `Menu Pause` and `Confirmation`. Those labels are defined by the plugin and may remain French in an English editor. `Nouvelle Partie`, `Continuer`, `Options (Pause)` and `Chargement` are not mapped to navigable screen widgets; use the corresponding action or transition instead.

**Continue is not a game-save system.** The menu remembers a map and slot, or can fall back to the last unlocked level. `Can Continue` checks those menu records, not whether your gameplay save exists. `Continue Last Game` requests a map load; your Blueprint must load and validate actual gameplay data. `On Request Start Game` fires before travel with the map and slot for both new and continue; distinguish your intended action in your own Blueprint if needed. Starting records the map and slot in menu settings in memory; call **Save Settings** after that event if the record must survive a restart. A negative requested slot becomes slot `0`.

## 5. Pause

1. Create `WBP_Pause` with parent **UltimateMenuPauseScreen**, then assign **Pause Menu Widget Class**.
2. Wire Resume → `On Resume Clicked`, Options → `On Options Clicked`, Return to menu → `On Return To Main Menu Clicked`, and optionally Quit → `On Quit Clicked`.
3. Configure **Main Menu Map Name** on `DA_MenuConfig`. The pause widget's own **Main Menu Map Name**, if nonempty, takes priority over the asset value.
4. In your gameplay setup obtain **Get Ultimate Menu Subsystem**, call **Set Game Active** with `Active = true`, and connect your own pause input to **Open Pause Menu**. Resume uses **Close Pause Menu**. Set up the input action to be usable while paused if you also want a key to resume.

**Important:** `Set Game Active` does not create the layout. `Open Pause Menu` does nothing when the game is not active and pauses the game before attempting navigation. A valid layout, stack, player controller and pause class must therefore exist first. **Initialize Menu System** always creates a layout and opens the intro or main menu; it has no pause-only initialization option. Level travel and input modes may invalidate the previous layout or controller, so a gameplay-specific Blueprint setup must manage initialization, the initial screen and input focus; do not assume pause works automatically after travel. Test the complete flow in your maps. Returning to the menu unpauses and loads the configured menu map; initialize that map's menu again.

While navigating to Options from Pause, the game remains paused; return to Pause and resume explicitly. Avoid opening Pause repeatedly.

## 6. Levels and progression

In `DA_MenuConfig → Available Levels`, add entries:

| Field | Example / behavior |
|---|---|
| Display Name / Description | `Forest` / `A quiet area` |
| Map Name | `Level_01` |
| Preview Image | Optional texture |
| Is Unlocked | On by default |
| Sort Order | Integer data; sort the displayed list yourself if needed |

For a second entry, use e.g. `Level_02` with **Is Unlocked** off. Create and assign `WBP_LevelSelect` derived from **UltimateMenuLevelSelectScreen**. On **On Levels Loaded**, clear your list, iterate `Levels`, create a button or row for each entry, and display its name and preview. Store the complete entry in that row. On click, pass the stored entry to **On Level Selected** on the level-select screen. Locked entries are rejected; set a disabled appearance yourself. **Show Only Unlocked Levels** defaults to off; enable it to filter the event's list.

**Example unlock:** when an objective completes, call **Get Ultimate Menu Subsystem → Unlock Level** with `Map Name = Level_02`. This stores the unlock and saves menu settings. Reopen level selection to refresh the list. An entry originally marked locked can still contain `Is Unlocked = false` after a saved unlock is restored; for your row's appearance also check **Get Settings Save Game → Is Level Unlocked** with its map name. The selection action performs that additional check.

**On Level Hovered** is available for a custom preview, but is not fired automatically by the provided selection action; implement hover behavior in your own row Blueprint. The menu does not verify map availability or unlock conditions on your behalf.

## 7. Options and persistence

Create the options root and desired child widgets, assign each class, then add your own sliders, selectors, buttons and labels. For each child:

1. On **On Settings Loaded**, populate the controls from `Settings`.
2. Keep a working copy of the relevant settings structure. Update it when controls change.
3. On Apply, pass the working copy to that screen's **Apply and Save**. Add a **Reset To Defaults** button if desired. Changes from subsystem-level **Apply ... Settings** calls require a separate **Save Settings** to persist.
4. Add **Navigate Back**. Test both opening from the main menu and from pause.

### Graphics

The graphics screen also exposes **Apply Preset** and **Get Available Resolutions**. Presets use the plugin's displayed values `Faible`, `Moyen`, `Élevé`, `Ultra` and `Personnalisé`. To apply individual texture, shadow, anti-aliasing, post-process, foliage and shading quality values, set **Overall Quality** to `Personnalisé`; otherwise the overall preset is applied instead. `Apply Preset` saves immediately.

New settings default to 1920 × 1080, `Plein écran`, `Élevé`, quality values 2, frame limit 0, VSync off and resolution scale 100. The graphics structure also stores Motion Blur and Depth of Field flags, default on, but the menu's apply action does not apply those two flags. Provide your own Blueprint handling if you expose them. Test resolution and scale on target devices; supported resolutions can be empty or device-dependent.

### Audio

Defaults are Master 1.0, Music 0.8, SFX 1.0, Voice 1.0, Ambient 0.7, Audio Enabled on and Subtitles Enabled off. The plugin stores these values; it does **not** change sound classes, mute audio or display subtitles automatically.

**Example:** create your project's sound classes and sound mix. In the audio widget, handle **On Apply Audio Settings** and use **Set Sound Mix Class Override** with the corresponding values, then activate the mix according to your audio setup. Implement mute and subtitle behavior in your own Blueprints. On startup, read **Get Audio Settings** and apply your audio setup again: loading the saved values does not fire the audio widget's event.

### Controls

Defaults: mouse and gamepad sensitivity 1.0, both Y inversions off, rumble on, and binding maps empty. These values are stored, not automatically applied to gameplay input or vibration.

**Example rebind:** in `WBP_OptionsControls`, connect a button to **Start Key Rebinding** with `Action Name = MoveForward` and `Is Gamepad = false`. On **On Rebinding Started**, show a prompt. On **On Key Captured**, update the displayed key and apply it to your own input mappings. The next key event is stored under the requested name and saved; it is not automatically installed in your input system. The requested device chooses which map stores the key; validate device, reserved keys and conflicts in your own UI. Make sure the widget can receive keyboard focus.

To restore settings after a restart, read **Get Controls Settings** during gameplay initialization and apply sensitivity, inversion, rumble and saved bindings yourself. Capturing a key does not broadcast **On Options Saved**.

### Storage

Menu settings, unlocked map names, intro-seen flag and last menu map/slot use the `UltimateMenuSettings` slot, user index 0 by default. They are separate from gameplay saves. **Reset All Settings To Default** resets settings and selected language, not unlocked levels or the intro-seen flag. The selected language is stored data; switching the application's language is your responsibility. Treat **On Options Saved** as a notification that a save was requested, not proof that disk writing succeeded.

## 8. Intro and credits

### Intro example

1. Create `WBP_Intro` with parent **UltimateMenuIntroScreen**. Add an **Image** named exactly `LogoImage` and assign the widget to **Intro Widget Class**.
2. Add an entry to **Intro Logos**: texture, **Display Duration** 3 seconds, **Fade In Duration** 0.5 and **Fade Out Duration** 0.5. An optional **Logo Sound** plays with the logo. Default initial delay is 0.5 seconds and **Allow Skip** is on.
3. The image and opacity fade are handled by the screen. Optionally implement **On Logo Fade In**, **On Logo Fade Out** and **On Intro Finished** for additional visuals. Do not add a competing opacity animation unless you intend to override the appearance.
4. An empty local **Logo Sequence** uses the configured **Intro Logos**. No logos means the intro finishes immediately. On completion or skip, the intro-seen flag is saved and navigation to the main menu is requested automatically; do not open the menu map again in **On Intro Finished**.

**Background Color** is data on each entry; set your own background in Blueprint if desired. To replay for testing, enable **Always Play Intro**, then disable it after testing if appropriate.

### Credits example

Add **Credit Sections** in the configuration: a `Section Title` such as `Team` and `Entries` such as `Design` → `Alex`. Create and assign a widget derived from **UltimateMenuCreditsScreen**. In **On Credits Loaded**, build rows from the supplied sections and entries; connect Back to **On Back Clicked**. Implement scrolling yourself: **Auto Scroll Speed** defaults to 60 but does not itself scroll the widget.

## 9. Confirmation dialog

1. Create a widget derived from **UltimateMenuConfirmationDialog** and assign **Confirmation Dialog Widget Class**. Add title and message text, plus Confirm and Cancel buttons.
2. In **On Show Confirmation**, set the text from **Dialog Title**, **Dialog Message**, **Confirm Button Label**, and **Cancel Button Label**. Wire buttons to **On Confirm Clicked** and **On Cancel Clicked**.
3. From an initialized menu screen, call the subsystem's **Show Confirmation Dialog** with title, message and both labels. Check **Is Valid** on the returned dialog, then **Bind Event to On Result**. Branch on `Confirmé` / `Annulé` to perform or cancel your action. Supply labels explicitly; empty labels fall back to French text.

**Example:** ask “Return to menu?”; only on `Confirmé` call **Return To Main Menu**. Test cancel and subsequent Back navigation: the result buttons deactivate the dialog but do not themselves update the subsystem's navigation history. Do not assume the reported current screen is synchronized after a dialog closes.

## 10. Transitions and loading screen

**Quick Load Level** uses **Default Transition Config** from the configuration. **Load Level With Fade** accepts a level name, total fade duration and color. **Load Level With Transition** accepts a `Menu Transition Config`; helper nodes **Make Fade Config** and **Make Loading Screen Config** produce configurations. If search offers both subsystem and library nodes, use the context-appropriate one; they invoke the same menu transition behavior.

Default transition: `Fade to Black`, minimum duration 1.0, fade-in 0.5, fade-out 0.5, black, show tip on. Alternatives are `Fade to Color`, `Instant` and `Loading Screen`. A request during an existing transition is ignored. A map name must be nonempty and valid.

**Loading-screen example:**

1. Create a Widget Blueprint with parent **Ultimate Menu Loading Screen**, add a title, tip and progress bar, and assign **Loading Screen Widget Class**.
2. Implement **On Loading Started** using **Current Level Name** and **Current Tip**; **On Progress Updated** uses **Current Progress** and **Current Status Text**; **On Loading Completed** can start an exit animation. **Get Progress Percent** and **Get Smoothed Progress** are optional helpers.
3. Set the default transition type to `Loading Screen`, or connect **Make Loading Screen Config** to **Load Level With Transition** and supply `Level Name`. Optional **Loading Screen Widget Override** replaces the default for that call.
4. Add **Loading Tips** to the config. If left empty, built-in tips are used; provide your own text for your desired language. For manual work use **Show Loading Screen**, **Update Loading Progress (Manual)** and **Hide Loading Screen**; update from a staged process that lets the UI render.

The displayed progress is **simulated**, reaches 100% before the map is opened and is not actual asset-loading progress. The loading widget is removed before travel; it is not a guaranteed screen during blocking map loading. **On Transition Completed** indicates that the open request was issued, not that gameplay assets are ready. `Minimum Duration` applies to the loading-screen path; the fade path waits for fade-in before opening the map and does not automatically perform a destination fade-out. Configure destination input, layout and any fade-in effect yourself and test actual travel.

## 11. Events and state

Obtain **Get Ultimate Menu Subsystem** and bind only events your Blueprint needs:

| Event | Meaning |
|---|---|
| `On Menu Screen Changed` | Reported previous and new screen |
| `On Intro Sequence Finished` | Intro completed or skipped |
| `On Options Saved` | Menu settings save requested |
| `On Request Start Game` | Map and slot before new/continue travel |
| `On Request Quit Game` | Before the quit request |
| `On Transition Started` / `On Transition Progress` / `On Transition Completed` | Transition notifications; progress is simulated for loading screens |

Screen events **On Screen Activated** and **On Screen Deactivated** are useful for visuals. **Get Current Screen**, **Get Session State**, **Get Navigation History**, **Get Current Flow State** and **Is Game Active** are available for inspection. Flow state is not a complete automatic state machine for all navigation; **Set Flow State** changes the reported state, not the current widget. Use **Get Available Levels**, **Add Loading Tips** and **Clear Loading Tips** for optional runtime data changes.

## 12. Troubleshooting and test checklist

| Symptom | Check |
|---|---|
| No menu appears | Correct player controller, asset connected to **Apply Config**, layout and main screen assigned, `MenuStack` named correctly, widget compiled and visible. Inspect **Window → Developer Tools → Output Log**. |
| Button does nothing | Its click event is connected, destination class is assigned, and the screen is active and focusable. |
| New game does not start | Nonempty valid map name, correct default-vs-level-list precedence, level included in the build. |
| Continue is enabled but no game data loads | Menu metadata is not a gameplay save; validate and load your own save. |
| Pause does not appear or game stays paused | Set game active; ensure layout, controller and pause widget are valid before opening. Provide a usable resume input and test after travel. |
| Audio, subtitles or controls do not change | Apply stored values in your own Blueprint systems, including after startup. |
| Intro is skipped | It was already marked seen, the class is absent or the logo sequence is empty. Enable **Always Play Intro** for testing. |
| Loading progress is misleading | It is simulated and ends before map travel; do not use it as real load progress. |
| A level looks locked after unlock | Check the saved unlock as well as the entry's `Is Unlocked` field when drawing rows. |

Test a fresh run and a restart; New Game and Continue; locked and unlocked levels; every configured screen and Back; pause → options → resume and return to menu; audio and controls after restart; both confirmation answers; and transitions in a packaged build. Check keyboard, mouse and any supported gamepad, different resolutions and focus behavior.
