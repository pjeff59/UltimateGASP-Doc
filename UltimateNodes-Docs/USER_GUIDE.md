# Ultimate Nodes — User Guide

[Français](../FR/GUIDE_UTILISATEUR.md) · [Plugin overview](../../README.md)

Ultimate Nodes is an Unreal Editor tool for organizing graphs and customizing their appearance. Use it while editing Blueprints and Materials; it does not change gameplay logic or add runtime behavior to a packaged game. The automatic layout can change node positions, comments, and, when explicitly enabled, reroute nodes. Review the result before saving.

## 1. Before you start

- Use an Unreal Engine project with the **Ultimate Nodes** plugin available. This version has been built and tested with Unreal Engine 5.8; other engine versions are not certified by this guide.
- The plugin declares **Ultimate GASP** as a required dependency. It must be available with your distribution; allow Unreal to enable required dependencies if prompted. If it is missing, obtain a complete distribution before proceeding.
- The plugin is an editor module. Its settings and graph tools are for the editor, not for shipping gameplay.
- Defaults below describe a fresh configuration. Existing preferences, profiles, or presets may have changed them.

### Installation

1. Close Unreal Editor.
2. Place the supplied `UltimateNodes` folder in your project's `Plugins` folder, keeping the supplied dependency available as well. If the plugin is already installed for your engine, do not install a second copy.
3. Open the project, choose **Edit → Plugins**, find **Ultimate Nodes**, and enable it. Accept any required dependency prompt.
4. Restart the editor if requested. If the editor reports a missing or incompatible module, use a distribution built for your engine version; do not assume the plugin is active.
5. Open **Edit → Editor Preferences → UltimateGASP → Ultimate Nodes**. Automatic-layout preferences are separate: **Edit → Editor Preferences → Plugins → Ultimate Nodes Auto Layout**.
6. In the main Level Editor toolbar, open **UN** for quick controls. The Blueprint editor also has an **Auto Layout** toolbar button.

No separate gameplay Blueprint asset or setup node is required. The examples below use standard Blueprint nodes so you can practice without changing your game.

## 2. First five minutes: organize a Blueprint

1. Open a Blueprint and its **Event Graph**.
2. For a small practice graph, connect **Event BeginPlay** → **Print String**. Enter a recognizable message in the string input. Move the two nodes apart manually.
3. Click inside the graph, then press **Ctrl+Shift+L**, or right-click empty graph space and choose **Auto Layout Graph**.
4. Inspect the result. The nodes should be arranged for reading; their existing connection should remain.
5. Press **Ctrl+Z** if the result is not suitable. Otherwise, save the Blueprint.

To arrange only part of a graph, select its nodes first, then press **Ctrl+Shift+K** or choose **Auto Layout Selected** from the Blueprint toolbar dropdown. Unselected nodes are not rearranged. The selection action is available only when nodes are selected.

### Available layout actions

In an editable Blueprint graph, the **Auto Layout** toolbar dropdown and the empty-space context menu offer:

| Action | Use |
|---|---|
| **Auto Layout Graph** | Arrange the graph. The configured **Wire handling** option controls the standard action. |
| **Auto Layout Selected** | Arrange selected nodes near their current area without moving unselected nodes. |
| **Auto Layout && Group Graph** | Arrange the graph and create named comment groups for eligible event or function flows. |
| **Auto Layout (Route Wires)** | Arrange and add reroute nodes where routing is needed. |
| **Auto Layout, Group && Route** | Combine grouping and wire routing. |

Grouping requires at least two detected roots and skips groups containing only one node. Grouping creates new comments: repeated use can create additional boxes. The main toolbar button repeats the last chosen whole-graph action; selecting **Auto Layout Selected** does not change that default. The two shortcuts are listed under **Editor Preferences → Keyboard Shortcuts → Ultimate Nodes Auto Layout** and can be reassigned.

### Layout preferences

Open **Editor Preferences → Plugins → Ultimate Nodes Auto Layout**:

| Setting | Fresh default | Effect |
|---|---:|---|
| **Use layered (Sugiyama) engine** | On | Arrange connected nodes in columns. |
| **Knot-route long edges** | Off | Optionally add reroute nodes on long Blueprint connections when the layered engine is enabled and wire handling is not off. |
| **Wire handling** | **Straighten & move nodes (default)** | Standard layout tries to improve wire alignment by moving nodes. **Reroute with knots** may add reroutes; **Off (don't touch wires)** disables these wire-handling passes. Explicit route actions can still add reroutes. |
| **Pin-align tolerance** | 120 | Maximum vertical adjustment used for pin alignment; 0 disables that adjustment. |
| **Comment color mode** | **Keyword (semantic)** | Choose semantic colors or **Cycling palette** for new groups. |
| **Horizontal spacing** / **Vertical spacing** | 110 / 44 | Control the gaps between columns and stacked nodes. |
| **Branch / lane spacing** / **Event spacing** | 90 / 200 | Additional spacing used when the layered engine is off. |

These are editor preferences, not settings on an individual Blueprint. Existing reroutes and comments should be checked after layout. When a route option creates reroutes, inspect the connections and use **Ctrl+Z** to undo the operation if needed.

### Example: an event graph with two flows

1. In a practice Blueprint, create an **Event BeginPlay** → **Print String** flow and a second independent event flow.
2. Choose **Auto Layout && Group Graph** from the Blueprint toolbar dropdown.
3. Inspect the named comment boxes. A single detected root or a lone event node does not produce a group.
4. Rename or resize comments as needed. Avoid running grouping repeatedly unless you want more comment boxes.

### Example: straighten a selected section

1. Select the nodes of one long chain, leaving neighboring nodes unselected.
2. Choose **Auto Layout Selected**.
3. Verify that only the selected part moved and that wires to the rest of the graph still lead to the intended nodes.
4. Undo if the new position makes the boundary wires harder to read.

## 3. Material and Material Function graphs

Automatic layout also works on editable Material and Material Function graphs:

1. Open the Material or Material Function.
2. Click the graph and press **Ctrl+Shift+L** for the whole graph. Alternatively, right-click empty space and choose **Auto Layout Graph**.
3. To arrange selected expressions only, select them and press **Ctrl+Shift+K**, or use **Auto Layout Selected**. On a Material node's context menu, **Auto Layout Material Graph** arranges the whole graph.
4. Inspect the connections and comments, then apply or save the asset using the editor's normal controls. Use **Ctrl+Z** to undo.

Material layout uses the layered engine and preserves existing connections and existing reroutes. The Material output node stays in its original position when it participates. Blueprint grouping and automatic reroute creation are not applied to Materials, even if related Blueprint options are enabled.

**Example:** connect a **Constant** to one input of an **Add** node, connect the result to an appropriate Material input, spread the expressions apart, and run **Auto Layout Graph**. Confirm that the same input connections remain before applying the Material.

Automatic layout is limited to editable Blueprint and Material graphs. The availability of a wire style in another graph type does not imply automatic layout support there.

## 4. Theme and wire appearance

Open **UN** in the main editor toolbar for quick controls, or use **Editor Preferences → UltimateGASP → Ultimate Nodes** for the complete settings.

### Fresh defaults

| Setting | Default | Meaning |
|---|---|---|
| **Master Activate**, **Activate Theme**, **Activate Wires** | On | Enable appearance overrides, theme, and wire styling. |
| **Theme Preset** | **Cool Dark** | Default theme preset; **Custom** enables manual theme values. |
| **Node Radius** | **Round (8px)** | Rounded node corners. |
| **Wire Style** | **Diagonal (45°)** | Styled wire route. Other choices are **Default (Spline)** and **Orthogonal (90°)**. |
| **Wire Thickness** | 1.0 | Base wire thickness multiplier. |
| **Wire Focus Mode** | **None** | No additional focus filter. |
| **Enable Bubbles**, **Glow Effect**, **Dash Data Wires**, **Auto Performance Governor** | Off | Optional effects; enable them deliberately. |

The **UN** panel has **Active**, theme **Preset**, **Luminosity**, **Saturation**, **Hue**, wire **Style**, **Thickness**, **Focus**, **Depth**, and **Governor** controls. Moving a theme slider selects **Custom**. The theme is an editor-wide appearance setting, not a property of the current Blueprint.

**Reset to UE Default** in the settings turns off the theme override while leaving wire styling active. To restore the theme, enable **Activate Theme** again. To disable wire styling separately, turn off **Activate Wires**; **Active** in the **UN** panel controls the master appearance switch.

### Example: readable wires in a busy Blueprint

1. Open **UN** → **WIRES** and choose **Orthogonal (90°)** under **Style**.
2. Increase **Thickness** slightly if the wires are hard to see.
3. Set **Focus** to **Neighborhood**, then choose a **Depth** from 1 to 4.
4. Select a node in your Blueprint. Wires outside the selected neighborhood are faded; choose **None** to remove the filter.

The preferences also offer **Selected nodes only** and **Exec neighborhood** focus modes. Focus changes the display only; it does not disconnect wires.

### Optional appearance controls

- **Theme → Nodes** adjusts headers, node shape, selection color, and optional pin styling. **Theme → Grid** controls the grid and optional background color. **Theme → Fonts** includes a font-size setting and an optional font file.
- **Wire Style → Exec Override** can give execution wires a separate style. **Ribbon** separates overlapping wires. **Bubbles** adds animated markers; bubble shape, speed, size, spacing, zoom threshold, and selection behavior can be adjusted.
- **Glow**, **Dash**, **Focus**, and **Performance** provide additional readability controls. **Wire Style → Visual FX** includes gradient, rainbow, wire-age highlight, hover comet, compile-failure flash, selection pulse, wire labels, direction arrows, end caps, and outline. These effects are off in the fresh configuration.
- **Pin Type Color Overrides** can assign colors to pin categories, such as `exec` or `bool`. An empty map uses the default pin colors.

Animations and highlights are visual aids, not proof that execution occurred. The compile-failure flash is an optional Blueprint editing indicator; use the normal compile messages to diagnose errors.

| Visual FX option | What to expect when enabled |
|---|---|
| **Wire Gradient** | Interpolates between the colors at the wire ends. |
| **Rainbow Mode** | Cycles wire colors over time. |
| **Wire Age Highlight** | Temporarily highlights recently detected wires. |
| **Hover Comet** | Animates a marker along a hovered wire. |
| **Failure Flash** | Pulses wires when the owning Blueprint has compile errors. |
| **Selection Pulse** | Pulses wires connected to selected nodes. |
| **Wire Labels** | Shows the source pin name on data wires; by default labels are restricted to the selection and sufficiently close zoom. |
| **Wire Direction Arrow** | Adds a direction marker; it can be restricted to execution wires. |
| **Wire End Caps** | Adds a dot or ring near the ends of wires. |
| **Wire Outline** | Adds a contrasting outline or shadow. |

### Example: make selected data easier to follow

1. In **Editor Preferences → UltimateGASP → Ultimate Nodes**, enable **Wire Labels** under **Wire Style → Visual FX → Wire Labels**.
2. Select a node with connected data wires and zoom in. The labels show the source pin names.
3. If you also need labels on execution wires, enable **Wire Labels On Exec**. Adjust **Wire Labels Selection Only** if you want to show labels without selecting a node.
4. Disable **Wire Labels** when you no longer need them.

### Visual FX presets

The **UN** panel provides quick **Minimal**, **Production**, **Studio**, and **Deep Focus** buttons. The **Visual FX Presets** settings section also offers **Apply Showcase Preset**, **Apply Production Preset**, **Apply Cinematic Preset**, **Apply Minimal Preset**, **Apply Debug Preset**, **Apply Teaching Preset**, **Apply Retro Synthwave Preset**, **Apply Color Flow Preset**, **Apply Studio Preset**, **Apply Deep Focus Preset**, **Apply Accessibility Preset**, **Apply Compact Big Graph Preset**, and **Apply Full Party Preset**.

Presets change several settings at once. Save a profile first if you want to restore your current look. **Reset All Visual FX** disables the effects while retaining the main activation and theme switches. For a large graph, try **Apply Compact Big Graph Preset** or enable the governor.

### Example: reduce visual load

1. Open **UN** and enable **Governor → Auto**.
2. Adjust **Node cap** if needed. The fresh preference values are 250 nodes and a low-zoom threshold of 0.45.
3. Zoom out or open a larger graph. The governor may temporarily suppress selected effects; it does not delete wires or nodes.
4. Disable **Auto** if you want effects to remain enabled regardless of these conditions.

## 5. Profiles

Profiles save editor appearance settings so you can switch between configurations. The fresh **Profile Storage Location** is **In Plugin (shareable with plugin)**. **In Project (per-project, survives plugin updates)** stores profiles under the project's `Saved/UltimateNodes/Profiles` folder. Choose the location before saving or looking for a profile; changing it changes which list is shown.

### Example: create and restore a profile

1. Choose a theme, wire style, and any effects you want.
2. Open **UN** → **PROFILES**, enter `Readable Graphs` in **Profile name...**, and click **Save**.
3. Close and reopen **UN** to refresh the profile list. Find the profile card under **ALL PROFILES** or **FAVORITES**.
4. Change the appearance, then click **Apply** on the saved card to restore it. The miniature preview lets you inspect a profile without applying it.
5. Click **Add Fav** to pin it, or **Set Default** to make it available through **Quick Load**.

Saving under an existing name replaces that profile. A name must retain at least one letter, digit, hyphen, or underscore after unsupported characters are removed; a name that becomes empty is rejected. Two different typed names can become the same saved name, so verify the notification before reusing a name.

### Share, import, and manage profiles

- **Export...** exports the previously saved profile named in **Profile name...**. **Import...** imports one profile file using its filename as the profile name.
- **Export Favorites...** and **Export All Profiles...** create multi-profile packs; **Import Profiles...** imports a pack. Reopen **UN** to refresh the list.
- Imports can replace profiles with the same resulting name. Back up or rename existing profiles before importing. An imported pack can contain both accepted and rejected entries; check the resulting list.
- **Install Accessibility** creates the built-in high-contrast and color-vision profiles and adds them to favorites. Existing profiles with those names can be replaced.
- In **Editor Preferences → UltimateGASP → Ultimate Nodes → Profiles**, **Load** applies a listed profile and **Delete** permanently removes it. Check the name before deleting.
- **Quick compare** displays previews of two selected profiles without applying them.

### Example: different profiles for Blueprint and Material editors

1. Save two profiles, for example `Blueprint Reading` and `Material Reading`.
2. Open **UN** → **AUTO PROFILE** and enable **By editor → Enabled**.
3. Select the corresponding saved profiles in the **Blueprint** and **Material** rows.
4. Switch between those editors and allow the polling interval to elapse. The matching profile is loaded when the active editor changes.
5. Disable **By editor → Enabled** to stop automatic switching.

Automatic switching is off in a fresh configuration. The default polling interval is 0.75 seconds. The theme is shared by the editor: switching a profile changes the current appearance globally, rather than maintaining independent simultaneous themes. A separate **Theme → Auto-Switch** option can load day/night profiles based on the system clock; it is also off by default. Make sure the named profiles exist before enabling either feature.

## 6. Other Blueprint graph tools

Right-click a Blueprint graph and look for the **Ultimate Nodes** section:

| Menu entry | Result |
|---|---|
| **Align selection to grid** | Snaps selected nodes to a 16-unit grid. The default shortcut is **Ctrl+Shift+A**. |
| **Insert preset comment** | Creates a colored **TODO (amber)**, **FIXME (red)**, **REVIEW (blue)**, or **HACK (magenta)** comment. |
| **Export graph to PNG** | Captures the visible graph widget to the project's `Saved/Screenshots/UltimateNodes` folder. |

These tools operate on the graph open in the editor. The PNG is a capture of the visible graph area, not an automatic image of the entire graph. Frame and zoom the graph before exporting. Use **Ctrl+Z** for an unwanted alignment or inserted comment.

### Example: mark an unfinished section

1. Select the nodes you want to annotate.
2. Right-click and choose **Insert preset comment → TODO (amber)**.
3. Edit the new comment text to describe the remaining work, then save the Blueprint.
4. For a note without a selection, insert a comment at the graph's paste location and move it as needed.

### Example: share a graph view

1. Open the Blueprint graph and zoom until the relevant nodes and wires are legible.
2. Right-click and choose **Export graph to PNG**.
3. Open the output path shown in the notification. If content is cut off, reframe the graph and export again.

## 7. Troubleshooting and limits

| Symptom | What to check |
|---|---|
| **UN** or layout actions are missing | Confirm the plugin and its required dependency are enabled, restart if requested, and open an editable supported graph. The **UN** panel is in the main Level Editor toolbar. |
| A shortcut does nothing | Click the graph, check the selection for **Auto Layout Selected**, and inspect the binding under **Editor Preferences → Keyboard Shortcuts**. Use the context menu if needed. |
| Layout reports no nodes or cannot edit the graph | Check that the graph contains eligible nodes and is editable. Some graph types do not support automatic layout. |
| A group was not created | Grouping needs at least two detected roots and does not wrap a lone node. It is not available for Materials. |
| A result is difficult to read | Undo, try the selection action, change spacing, or choose another wire-handling mode. Complex graphs may still need manual adjustments. |
| The theme or wires did not change | Check **Active**, **Activate Theme**, **Activate Wires**, and the relevant graph target. **Reset to UE Default** turns off the theme override only. |
| Bubbles or other effects disappear | Check that the effect is enabled, the zoom/selection settings allow it, and the optional governor is not suppressing it. |
| A profile is missing | Check **Profile Storage Location**, reopen **UN**, and verify the profile name. Export requires a previously saved profile. |
| Imported settings replaced an existing profile | Import and save may overwrite names. Restore a backup if available; use unique names before the next import. |
| PNG export fails or is cropped | Open a graph, wait until it is visible, frame the desired area, and confirm the project can write to its `Saved` folder. |

The plugin's visual and layout features are editor conveniences. No custom gameplay node or gameplay event is required for the workflows in this guide.
