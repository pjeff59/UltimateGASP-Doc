# UltimateLandscape — User Guide

This guide covers the user-facing workflow in Unreal Editor and editor Blueprints. Interface labels are kept in English so they can be found as shown in the editor.

## Quick start

1. Install the complete compatible UltimateLandscape distribution in the project's `Plugins` folder. In **Edit > Plugins**, enable **UltimateLandscape** and restart the editor when prompted. Keep required dependencies from the supplied distribution enabled.
2. Click **UltimateLandscape** on the Level Editor toolbar. The editor console command `UltimateLandscape.Open` opens the same panel.
3. The panel starts with the **Valley** preset. The initial terrain is 2016 m by 2016 m, with a 1009 x 1009 height grid, a seed of 1337 and an altitude range from -300 m to 1500 m.
4. Choose a preset, adjust the settings and click **Preview**. Preview quality is selectable: **Auto - 2 m**, **Fast - 8 m**, **Fine - 1 m** or **Final Resolution**. The separate **3D Mesh Budget** controls preview mesh detail; it does not change the output heightmap.
5. When the result is ready, click **Final Generation**. This computes at the selected output resolution and is required before creating a map or exporting.
6. Use **Save Recipe** to keep the settings as a reusable recipe asset. Create a classic or **World Partition** map from the generated result, then review and save the level in the editor.

The tool's automatic visual update after an edit is controlled by **Auto Preview Edits**, on by default. These quick updates are capped at 505 samples per axis. They refresh the preview only; they do not apply terrain changes to a level. **Show Context** is also on by default in the 2D preparation view.

## Installation and opening

UltimateLandscape is an Unreal Editor plugin prepared for Unreal Engine 5.8. Install it from the supplied, compatible plugin distribution; do not copy only a partial plugin set when required dependencies are missing. Enable **UltimateLandscape** in **Edit > Plugins**, restart the editor if requested, then open the panel from the Level Editor toolbar or the `UltimateLandscape.Open` console command.

The plugin has no separately supplied content pack to add to a level. Its generator runs in the editor; the resulting Landscape is a standard level asset. The plugin's editor functions are not available as runtime terrain generation in a packaged game.

## Presets, dimensions and output

The panel provides these presets: **Valley**, **Mountains**, **Rolling Hills**, **Plateau**, **Island**, **Alpine**, **Canyon**, **Eroded Mountains**, **Composed Terrain**, **Guided Terrain**, **Stratified Cliffs**, **Archipelago**, **Dunes**, **Volcano** and **Glacial Valley**.

The default terrain settings are:

| Setting | Initial value | What it controls |
|---|---:|---|
| Width Meters / Length Meters | 2016 m / 2016 m | Physical footprint of the terrain |
| Resolution | 1009 x 1009 | Height samples along each axis |
| Seed | 1337 | Repeatable procedural variation |
| Min Altitude Meters / Max Altitude Meters | -300 m / 1500 m | Heightmap encoding range |
| Base Elevation Meters | 50 m | Starting elevation for the terrain profile |
| Nanite | Off | Whether to build Nanite for the generated Landscape |
| Hydraulic Iterations | 0 | Off until hydraulic erosion is explicitly configured |

Available square resolutions are **127**, **253**, **505**, **1009**, **2017** and **4033** samples per axis. At the default size, 1009 samples span 2016 m at 2 m per sample. Larger grids need more memory and generation time. Start with a smaller grid while shaping the terrain, then choose the output resolution deliberately.

**Preview Quality** controls how the preview is sampled. **3D Mesh Budget** controls how many preview vertices are rendered and is not a substitute for choosing the output resolution. Use **Final Generation** after changing the recipe and before creating a map or exporting.

## Shaping terrain

The settings are grouped by their visible categories:

- **01 Terrain**: seed, resolution, physical dimensions, altitude range and optional **Nanite**.
- **02 Shapes**: mountain, hill, plateau and valley contributions, relief, feature size and related shape controls.
- **03 Details**: smaller-scale surface variation.
- **04 Coast**: island silhouette, sea level and coast blending.
- **04 Composition**: regions and guides. Up to 32 regions and 64 guides can be used. Regions have their own preset, seed, footprint and blend. Ordinary guides have 2–32 ordered points and use **Ridge**, **Valley** or **Corridor** modes.
- **05 Erosion**: thermal erosion is available; hydraulic erosion is an optional artistic heightmap process and is disabled by default because **Hydraulic Iterations** starts at 0.
- **06 Gameplay**: protected buildable zones define a target elevation with a radius and a blend.
- **07 Water Prototype**: optional Lake, River and Ocean requests. Requests start disabled. **Affects Landscape (Carve)** is selected for newly added requests; turn it off for a surface that should not deform the terrain.
- **08 Biomes**: optional permanent biome and soil masks. Biomes are disabled by default, and **Paint On Creation** is off. Painting requires an existing compatible Landscape material and the six existing weight-blended Layer Info assets (Grass, Dirt, Sand, Rock, Snow and Mud).

The selected **Landscape Material** is an existing asset. The generator assigns it when requested but does not create or rewrite the material or its textures. Biome and soil diagnostics do not replace material setup.

### Editing regions, guides and zones in 2D

Enable **Edit Regions (2D)**, **Guides / Water (2D)** or **Edit Zones (2D)** to work on the top-down map beside the 3D preview. Drag a region center, its X/Y radii or its rotation; drag guide points and handles; or move and resize a protected zone. The 2D Water shapes are planning overlays, not a 3D rendering of Water actors.

For guides, **Shift+click** inserts a point, **Ctrl+click** appends to an open path, **Delete** removes a point and **Esc** cancels the current edit. **Ctrl+Z** and **Ctrl+Y** undo and redo. Edits change the recipe only; they do not directly alter a level. Keep **Auto Preview Edits** enabled for automatic preview refresh, or run **Preview** manually.

A **Corridor** guide shapes terrain; it does not create a road, actor, navigation path or guaranteed vehicle route. Verify slopes and gameplay movement in the actual level.

## Create and update a map

Use **Create Map** after **Final Generation**. The panel supports a classic Landscape map and **World Partition**; World Partition is selected by default. The initial **Components/Proxy** value is 2. **Loading Range (cm)** starts at 80000 cm (800 m) and applies only to newly created World Partition maps. A larger range can load more terrain and actors; it does not guarantee that all of a very large world is loaded.

The map-creation dialog can ask to save the level currently open before switching to the new map. A new map receives the default environment actors used by the supplied compatible distribution; it does not copy every actor or environment setting from the level being left.

**Nanite** is off by default. Enable it in **01 Terrain** before creation if needed. When enabled, the generated Landscape and its loaded proxies receive Nanite after terrain composition. It can increase build time and does not enable displacement in the selected material.

**Update Selected** is deliberately restricted. It updates the generated base of a recognized UltimateLandscape terrain, not arbitrary or legacy Landscapes. Keep the same grid and scale, load every World Partition cell, and keep manual edits separate from the generated base. If the terrain's source identity or base has been changed directly, the update can be refused rather than overwrite those changes. Save the level and modified packages after reviewing the result.

## Recipes and heightmap files

Use **Save Recipe** to preserve settings and reopen them later with the recipe-loading controls. Recipes make the seed, dimensions, resolution and composition reusable; they do not automatically modify an existing map when loaded.

**Import** accepts:

- Headerless `.raw` or `.r16` data in 16-bit little-endian order.
- A square 16-bit grayscale `.png`.
- A valid `Manifest.json` produced by an UltimateLandscape export. The manifest references `Height.r16` and checks its recorded source hash.

The supported sample counts per side are 127, 253, 505, 1009, 2017 and 4033. Color images, 8-bit PNGs, non-square images and tiled sets are not supported. Without a manifest, the current recipe's physical dimensions and altitude range are used to interpret the height values.

The export contains `Height.r16`, `Height.png` (16-bit grayscale), `Slope.png` (8-bit grayscale) and `Manifest.json`. Exports use a new destination; an existing destination is not overwritten. Choose a new folder/name if the requested destination already exists.

## Blueprint workflows

Blueprint nodes are editor-only. Use them in an editor utility graph, not in gameplay Blueprints or runtime event loops. Search for the displayed node names in the Blueprint action menu; the nodes are synchronous, so do not call generation every frame.

Useful nodes include:

- **Export Recipe** — takes a saved recipe, a directory and a base name. Branch on **Return Value** and report **Message** when it fails.
- **Create Landscape Map** — creates a classic map from a saved recipe.
- **Create World Partition Landscape Map** — takes a recipe, map package name and **Grid Size In Components**; **Loading Range** defaults to 80000 cm.
- **Update Generated Landscape** — updates a recognized generated base on a target Landscape; it does not turn an arbitrary Landscape into a generated one.
- **Configure Heightmap Source** — explicitly assigns a supported source file to the recipe. A failed call leaves the recipe unchanged.
- **Preview Drainage** and **Apply Drainage** — preview returns a proposal and token; apply the exact proposal with its token. Preview alone changes neither the recipe nor a level.
- **Preview Water Synchronization** and **Synchronize Water** — preview first, then pass the exact returned token. Replacing edited actors and allowing removals require separate explicit authorizations.

### Example: create a World Partition map from a button

In an editor utility widget, create a button called **Create Terrain**. In its click event:

1. Connect the saved recipe asset to **Create World Partition Landscape Map**.
2. Set **Map Package Name** to a new path such as `/Game/Generated/Terrain/Valley_WP`, **Grid Size In Components** to `2`, and leave **Loading Range** at `80000` for the default 800 m.
3. Connect **Return Value** to a Branch. On failure, show **Message** so the reason is visible; on success, verify the newly created level and save any requested level changes.

Use a unique package name. Run the button from the editor, outside Play In Editor.

## Worked examples

### Mountain valley with a broad travel corridor

1. Start with **Valley** or **Mountains** and keep the default 2016 m footprint for the first preview.
2. In **04 Composition**, add a guide with an open path, select **Corridor**, and place points along the intended lowland. Adjust its width and blend gradually rather than making it narrower than the terrain grid can represent.
3. Use **Preview** while editing, then **Final Generation**. Inspect the slope and the terrain in the created level; the guide is not a navigation or movement test.
4. Save the recipe before creating a map so the same seed and guide can be revised later.

### Archipelago with a protected landing area

1. Choose **Archipelago** or **Island** and preview the coastline before adding details.
2. Adjust the coast and sea-level settings, then add a protected zone in **06 Gameplay** where a landing area should remain buildable. Set its elevation, radius and blend to fit the surrounding terrain.
3. Preview again, then run **Final Generation** and create a classic or World Partition map. Keep Nanite off until you have a reason to build it and have checked the cost on the target terrain.

### Reuse an exported heightmap

1. Export a completed recipe to a new folder.
2. In another recipe, choose **Import** and select that export's `Manifest.json`. Keep `Height.r16` beside the manifest so its recorded hash can be checked.
3. Review width, length and altitude settings, generate a preview, then run **Final Generation**. If the source file changed after export, create a fresh export or select the intended heightmap directly and verify its scale.

## Troubleshooting and limits

| Symptom | Likely cause and action |
|---|---|
| **Create Map** or export is unavailable or uses an old result | Run **Final Generation** again after the last recipe edit. |
| Heightmap import is rejected | Check square dimensions, one of the six supported resolutions, 16-bit grayscale PNG or little-endian RAW, and the manifest/hash when using a manifest. |
| A map update is refused | Select a recognized generated Landscape, keep its grid and scale, and load all partition cells. Do not merge manual edits into the generated base. |
| Biome/soil preview is gray or no soil is painted | Enable **Biomes** and regenerate. Painting also needs a compatible existing material and all six Layer Info bindings; **Paint On Creation** is optional and off by default. |
| Water requests are refused or conflict | Keep requests disabled if native Water support is unavailable. Use the explicit preview/synchronization workflow and resolve conflicts rather than forcing an overwrite. |
| The preview is smooth or slow | Check both **Preview Quality** and **3D Mesh Budget**. The preview mesh budget does not increase output heightmap resolution. |
| Nanite is absent | **Nanite** is off by default. Enable it and create or update the generated Landscape again. |

The result is a static Landscape authored in the editor. A guide is terrain shaping, hydraulic erosion is an artistic heightmap effect, and a preview or slope diagnostic does not guarantee character traversal, collision quality or navigation behavior.
