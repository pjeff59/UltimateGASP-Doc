# UltimateHUD: Blueprint user guide

[Documentation home](README.md) · [Français](../FR/GUIDE_UTILISATEUR.md)

## 1. What you can build

The **Ultimate HUD Manager Component** manages HUD elements on a PlayerController. Your Widget Blueprints provide the actual visuals. The manager accepts health, stamina, ammunition, weapon information, damage and time updates; it can also dispatch pickup, achievement and toast notification events.

This package does not contain ready-made Widget Blueprint artwork. A type listed in the element picker is not necessarily a finished feature: for example, a minimap, compass or objective list requires your own widget and gameplay logic. The manager does not create notification visuals for you.

## 2. Install and prepare

1. Copy the supplied `UltimateHUD` folder into your project's `Plugins` folder if it is not already installed.
2. Open the project in Unreal Editor. In **Edit > Plugins**, find **Ultimate HUD**, enable it and restart the editor when prompted.
3. If the editor reports a missing or incompatible required dependency, use a complete compatible distribution of the plugin and its required dependencies. Do not assume the plugin can start without them.
4. Create or open a **PlayerController** Blueprint, for example `BP_PlayerController`. In **Components**, use **Add** and search for `Ultimate HUD Manager`. Add the component.
5. In your GameMode Blueprint, set **Player Controller Class** to `BP_PlayerController`. Assign that GameMode to the level in **World Settings**, or set it as the project default in **Project Settings > Maps & Modes**. Compile and save.

The manager must be attached to a PlayerController. Adding it to a character or another actor does not initialize its HUD. The GameMode actually used by the level determines which controller is spawned.

## 3. First working example: health

### Create the visual

1. In the **Content Browser**, choose **Add > User Interface > Widget Blueprint**. Choose the parent class `HUDHealthBarWidget`; if needed, search the full class name in **All Classes**. Name the asset `WBP_Health`.
2. Open it in **Designer**. Add a **Progress Bar** named exactly `HealthBar`. Optional children are a **Text** named `HealthText` and an **Image** named `HealthIcon`. Capitalization matters.
3. Arrange the controls in a root layout, give the bar a visible size and style, then **Compile** and **Save**.

If you started with an ordinary Widget Blueprint, use **File > Reparent Blueprint** to select `HUDHealthBarWidget` before adding the required controls. A child of a specialized class needs the required widget name and type to compile correctly.

### Connect it to the manager

1. Open `BP_PlayerController`, select its HUD manager and find **Style Widget Classes** in **Details**.
2. Expand the **Theme 1** entry and its **Widget Classes** map. Set the **Health Bar** entry to `WBP_Health`. Keep the desired other entries; do not assume an entry is created for every possible element.
3. In **Elements To Enable At Start**, add **Health Bar**. Leave **Default Style** at **Theme 1**.
4. In the controller's Event Graph, drag the manager component into the graph as a reference. From it call **Update Health** on **Event BeginPlay**, with **Current Health** `75` and **Max Health** `100`.
5. Compile, save and press **Play**. The bar should move toward 75%, and the optional `HealthText` displays `75 / 100`.

For a real game, call **Update Health** with your own current and maximum values at initialization and whenever health changes. Use a positive maximum. The manager displays values; it does not apply damage or store your game's health.

**Important:** entering a type in **Elements To Enable At Start** does not create its widget. Its class must exist in the active theme's map, or be one of the limited default fallback classes described below. An unassigned default class is not a designed visual.

## 4. Create other elements

Create each visual as a Widget Blueprint derived from the indicated parent. Add the required named controls in **Designer**; optional controls may be omitted. Add its element type and class to **Style Widget Classes > Theme 1 > Widget Classes**, then add that type to **Elements To Enable At Start** if it should be shown initially.

| Element picker | Parent class to search | Required control | Optional controls |
|---|---|---|---|
| **Health Bar** | `HUDHealthBarWidget` | Progress Bar `HealthBar` | Text `HealthText`; Image `HealthIcon` |
| **Stamina Bar** | `StaminaBarWidget` | Progress Bar `StaminaBar` | Text `StaminaText`; Image `StaminaIcon` |
| **Ammo Display** | `AmmoDisplayWidget` | Text `CurrentAmmoText` | Text `ReserveAmmoText`, `SeparatorText`; Image `AmmoIcon` |
| **Day/Time Display** | `DayTimeDisplayWidget` | Text `TimeText` | Text `PhaseText`; Image `PhaseIcon` |
| **Damage Indicator** | `DamageIndicatorWidget` | Canvas Panel `IndicatorContainer` | A separate indicator Widget Blueprint, assigned as **Indicator Widget Class** |
| **Interaction Prompt** | `HUDInteractionPromptWidget` | Text `InteractionText` | Text `InputKeyText`; Image `InputKeyIcon`; Progress Bar `ProgressBar` |

Use the exact control names, not merely matching text displayed on screen. Configure visuals, layout and sizes yourself. The time phase icon does not automatically change texture with the phase.

The **Theme 1** default map contains base classes for **Health Bar**, **Ammo Display**, **Day/Time Display** and **Damage Indicator**. Other themes without a populated map fall back to those four base classes. Those classes are not a substitute for designed Widget Blueprints. Once a theme has a nonempty map, only its nonempty class entries are created; adding one custom entry does **not** fill the other missing types automatically. In particular, stamina and interaction prompt must be assigned explicitly.

The manager creates the configured elements at startup, not when **Enable Element** is called. Configure the map before play. For a custom element, use a Widget Blueprint based on `UltimateHUDElementBase`, assign it to an appropriate element type, and supply its own content and update logic. There is only one managed instance per type in the active theme.

## 5. Values and useful events

Get the manager component from your PlayerController and call these nodes at the relevant gameplay events:

| Node | Example | Behavior |
|---|---|---|
| **Update Health** | `75`, `100` | Updates the assigned health widget and sends **On HUD Value Changed** with a clamped 0–1 fraction. |
| **Update Stamina** | `40`, `100` | Sends the clamped fraction and writes it to a control named `StaminaBar`. See the stamina example below. |
| **Update Ammo** | `12`, `48` | Updates the assigned ammo widget; **On HUD Value Changed** carries the current ammunition count, not a fraction. |
| **Update Weapon Info** | `Tool`, `Single` | Writes text to controls named `WeaponNameText` and `FireModeText` in the assigned **Weapon Info** element, if present. |
| **Update Day Time** | `14`, `30`, `Day` | Updates an assigned time widget. Hours are clamped to 0–23 and minutes to 0–59. |
| **Show Hit Marker** | **Kill Confirm** false, **Headshot** true | Writes `HEADSHOT` to a Text control named `MarkerText` in an assigned **Hit Marker** element. Kill confirmation takes precedence. It does not schedule hiding the text. |
| **Show Damage Indicator** | Direction `90`, Intensity `0.8` | Requests an indicator on the assigned damage widget when that element is enabled and the HUD is visible. |

The manager emits **On HUD Value Changed** for health, stamina and ammunition updates, including when a visual is missing. Bind it when you need the type and value; do not interpret every value as a percentage.

### Example: stamina with its own text and event

The manager's **Update Stamina** changes the `StaminaBar` control directly; it does not call the specialized widget's **Set Stamina**, update `StaminaText`, or trigger **On Stamina Changed**. The specialized widget can overwrite that direct bar update with its own stored value on its next update. For the complete specialized widget behavior:

1. Configure and enable a **Stamina Bar** based on `StaminaBarWidget` as in section 4.
2. On your stamina-changed event, get the manager, call **Get Element** with **Stamina Bar**, then cast the result to `StaminaBarWidget`.
3. From the cast result call **Set Stamina**, passing the current and maximum stamina. For example, `40` and `100` show `40/100` if `StaminaText` exists. Use **Set Stamina Percent** for a 0–1 fraction instead.
4. In the stamina widget, implement **On Stamina Changed** for additional feedback. **Set Show Numeric Values** controls its optional text.

The cast can fail if the entry is missing or uses another parent. Refresh the element reference after a theme change.

### Example: ammunition and health warnings

On a shot or reload, call **Update Ammo** with the new magazine and reserve counts. In `WBP_Ammo`, implement **On Ammo Changed**, **On Low Ammo** or **On Empty Ammo** to add your own feedback. By default the low-ammo threshold is 10, warning is enabled, reserve text is shown, and the initial values are 30 and 120. **Set Show Reserve Ammo** and **Set Low Ammo Warning** change the corresponding options on the widget.

The health widget initially uses 100/100, shows numeric values, and enables the low-health warning at a fraction of 0.25. Its bar interpolates at speed 5 by default. Implement **On Health Changed** or **On Low Health** in `WBP_Health`; use **Set Show Numeric Values** or **Set Low Health Warning** on the widget to change those options. **Set Health Percent** changes the target fraction but does not update the stored numeric health values; use **Update Health** or **Set Health** when the numbers must change too.

### Example: a clock

Create and assign the time widget, then call **Update Day Time** with `14`, `30`, `Day`. `TimeText` shows `14:30`; `PhaseText`, if present and enabled, shows `Day`. From **Get Element** for **Day/Time Display**, cast to `DayTimeDisplayWidget` and call **Set Use 24 Hour Format** with false to display `2:30 PM`. Defaults are 12:00, `Day`, 24-hour format and phase shown. Seconds are not displayed by this widget. Automatic time updates, when active in the supplied environment, can overwrite manually supplied values; disable **Auto Bind Plugins** if you want to supply time exclusively yourself.

### Example: a directional damage indicator

1. Create a small ordinary Widget Blueprint, for example `WBP_DamageArrow`, with a visible arrow pointing upwards.
2. Create `WBP_Damage`, parent `DamageIndicatorWidget`, with a Canvas Panel named `IndicatorContainer`. In its class defaults set **Indicator Widget Class** to `WBP_DamageArrow`. Make the container large enough for the indicator positions.
3. Assign `WBP_Damage` to **Damage Indicator** in the active theme and enable that element.
4. When your game detects a hit, calculate the direction relative to the player and call **Show Damage Indicator**. `0` is forward, `90` is right, `180` is behind and `270` is left; intensity is clamped to 0–1.

The default maximum is eight simultaneous indicators, at distance 200, with a two-second duration and a fade during the last second. The oldest indicator is removed at the limit. The manager node does not calculate the direction from the attacker. **On Damage Shown** is exposed on the widget but is not emitted by this display path; do not depend on it.

### Example: interaction prompt

Create and assign `WBP_Prompt` using `HUDInteractionPromptWidget`, with `InteractionText` and, optionally, `InputKeyText` and `ProgressBar`. Get **Interaction Prompt** through **Get Element**, cast to `HUDInteractionPromptWidget` and:

1. On entering your own interaction range, call **Show Prompt** with **In Interaction Text** `Open`, **Input Key** `E` and **Show Progress** true.
2. While your own hold action progresses, call **Update Progress** with a fraction from `0` to `1`.
3. When the action ends or the target leaves range, call **Hide Prompt**.

These calls only update the prompt; your Blueprint must detect the target, input and completion. The widget exposes **On Prompt Shown**, **On Prompt Hidden** and **On Progress Updated**. It starts hidden, although startup element settings can subsequently show it.

### Example: simple weapon information and hit marker

Create a Widget Blueprint based on `UltimateHUDElementBase` with Text controls named `WeaponNameText` and `FireModeText`, and assign it to **Weapon Info**. Call **Update Weapon Info** when your selected item changes. For a marker, make another base-derived widget with a Text control named `MarkerText`, assign it to **Hit Marker**, and call **Show Hit Marker** after your own hit detection. Use your own Blueprint timer or event to hide the text afterward.

## 6. Visibility, themes and layout

The manager starts with **Default Style = Theme 1**, **Auto Bind Plugins** enabled, **Elements To Enable At Start** empty, HUD visibility true, context **Exploration**, aspect ratio 16:9 and scale 1. Auto binding is an attempt, not a guarantee that your game values are connected; manually connect the update nodes where needed. For predictable manual examples, turn **Auto Bind Plugins** off.

| Node | Use |
|---|---|
| **Enable Element**, **Disable Element**, **Toggle Element** | Operate on an already-created type. **Immediate** bypasses the configured transition. **Is Element Enabled** tests enabled state, not necessarily visible pixels. |
| **Get Element** | Returns the current instance or an invalid reference if that type was not created. |
| **Show HUD**, **Hide HUD**, **Toggle HUD**, **Is HUD Visible** | Manage the global visibility state. Showing the HUD shows enabled elements, which can include elements not listed at startup. |
| **Set Global Opacity** | Applies opacity clamped to 0–1 to existing elements. |
| **Set Global Scale** | Applies a scale clamped to 0.5–2 to existing elements. |
| **Set HUD Style**, **Get Current Style** | Switch or inspect the current theme. Switching rebuilds the widgets: reacquire references and resend game values. Configure each theme you intend to use. |
| **Set Context Mode**, **Get Context Mode** | Select **Exploration**, **Combat**, **Stealth**, **Cinematic**, **Menu** or **Critical**. |
| **Set Responsive Layout** | Supply an aspect ratio and UI scale; it adjusts render scale, not a full automatic rearrangement. UI scale is clamped to 0.5–2. |

The four theme labels in the editor are **Theme 1**, **Theme 2**, **Theme 3** and **Theme 4**. Their default opacity/scale profiles are respectively `1.0/1.0`, `0.9/1.05`, `0.85/0.95` and `1.0/1.15`. They do not provide four complete sets of designed widgets. Context sets opacity to `0.55` for **Stealth**, `0.35` for **Menu** and **Cinematic**, and the theme opacity for the other modes. Showing an element, changing global opacity or a low-health pulse can subsequently change opacity; these settings are not a permanent visibility policy.

For example, call **Hide HUD** before a cinematic and **Show HUD** afterward. To dim instead, call **Set Context Mode** with **Cinematic**, then **Exploration**. To test a wide screen, call **Set Responsive Layout** with aspect ratio `2.333` and UI scale `1.0`; also test in the actual viewport.

Element configuration exposes animation and layout fields, but do not assume all listed animation types produce full visual animations or that the manager automatically repositions widgets for every aspect ratio. Design and test transitions and placement in your widgets. The manager's **On HUD Animation Completed** dispatcher is exposed but is not emitted by the current manager. Element-level **On Animation Completed**, **On Element Initialized**, **On Element Shown** and **On Element Hidden** are available; test timing before relying on them.

## 7. Notifications: build the presentation

**Show Pickup Notification**, **Show Achievement Notification** and **Show Toast Notification** dispatch events; they do not create a visible popup, queue or timer. Build a normal Widget Blueprint such as `WBP_Notification` with your own text, image and presentation. Create it from the PlayerController with **Create Widget**, provide the owning player and **Add to Viewport**. Keep the widget reference.

In the PlayerController Event Graph, after creating the widget, drag the manager reference and use **Bind Event to On Pickup Notification**. Connect a custom event with the matching signature. From its `Notification Data`, use **Break Pickup Notification Data**, then update the retained widget's text/image through your own widget function or variables. Use **Set Visibility** and a timer to show and hide it. Bind **On Achievement Notification** and **On HUD Toast Notification** similarly if needed. Bind before sending a notification; a past event is not replayed.

### Example: item pickup

After your game confirms a pickup, call **Show Pickup Notification** with **Make Pickup Notification Data**:

| Field | Example / default |
|---|---|
| **Display Name** | `Medical supplies` |
| **Description** | Optional text |
| **Icon** | Optional texture |
| **Quantity** | `3`; default `1` |
| **Display Duration** | `2.5` seconds; default `0` |

Display the name and quantity in your listener. If duration is `0`, have your own widget choose a duration; no automatic default timer is supplied. Decide how overlapping notifications should behave in your widget.

### Example: achievement

Call **Show Achievement Notification** with **Make Achievement Notification Data**: **Title** `First discovery`, **Description** `Find a hidden room`, optional **Icon**, **Rarity** `Rare`, **Reward Points** `50`, and **Display Duration** `4`. Rarities are **Common**, **Uncommon**, **Rare**, **Epic** and **Legendary**; the default is **Common**, points default to `0` and duration defaults to `0`. Your listener decides whether and how to display rarity, points, icon and duration. This notification does not unlock or save an achievement.

### Example: toast

Call **Show Toast Notification** with **Title** `Saved`, **Message** `Progress recorded`, a chosen **Accent Color** and **Duration** `3`. The listener receives **On HUD Toast Notification** and can break its notification data to update the widget. Duration defaults to three seconds and is clamped to at least one; the emitted **Stackable** flag is true. Stacking and timed removal still require your own Blueprint logic.

## 8. Troubleshooting and limitations

| Symptom | Check |
|---|---|
| Nothing appears | The active GameMode uses your PlayerController; its manager is on that controller; the active theme has the correct Widget Blueprint class and the element is enabled. Confirm the widget contains visible controls with the exact required names. |
| A type is in the startup list but missing | The startup list enables existing instances only. Add its class to the active theme's map before play. A partially populated map does not add other missing types. |
| Wrong value or no update | Call the appropriate node after the game value changes; check **Get Element** and casts. For stamina text and event use **Set Stamina** on the specialized widget rather than relying on **Update Stamina** alone. |
| References or displayed values disappear after changing theme | Theme changes rebuild the elements. Call **Get Element** again, validate the result and send current values again. |
| HUD returns with unexpected elements | **Show HUD** shows enabled elements, including ones not listed at startup. Explicitly disable unwanted elements. |
| Notification call has no visible result | Create a presentation widget and bind its event before calling the notification node. Implement your own duration and overlap handling. |
| Damage indicator is absent | Verify the element is enabled, HUD is visible, `IndicatorContainer` exists, **Indicator Widget Class** is assigned and the indicator has visible content. |
| Prompt or hit marker stays visible | Explicitly call **Hide Prompt**, or hide the marker text using your own event/timer. |
| Opacity or scale changes unexpectedly | Context, show/hide, warnings, theme changes and responsive layout can override earlier settings. Apply your desired setting at the appropriate time and test it. |
| Time changes after a manual update | Automatic binding may also update time. Turn **Auto Bind Plugins** off for a purely manual clock. |

Existing older documents are retained separately; their examples may refer to unavailable resources or outdated names. Use this guide for the current workflow.
