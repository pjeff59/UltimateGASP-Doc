# UltimateLandscape — Worked Terrain and Blueprint Examples

This companion guide adds step-by-step recipes to the [user guide](README.md). It focuses on tasks a level designer can perform in the UltimateLandscape editor panel and on Blueprint graphs that run in the editor.

> **Editor-only workflow:** generation and the Blueprint nodes in this guide are for editor tools. They do not generate terrain during packaged gameplay. The nodes are synchronous; use buttons or deliberate editor actions, not Tick.

## A repeatable work loop

For each terrain recipe:

1. Open **UltimateLandscape** from the Level Editor toolbar.
2. Select a preset and change only a few settings at a time.
3. Use **Preview** at a smaller resolution while shaping.
4. Inspect the 3D preview and the 2D guides/regions/zones where relevant.
5. Click **Final Generation** after the last edit.
6. Use **Save Recipe** before exporting or creating a map.
7. Create the map, review it in the level, then save the level and any changed assets.

**Auto Preview Edits** can refresh the visual preview after an edit, but it does not generate a final result or apply changes to a level. The automatic edit preview is capped at 505 samples per axis. The **3D Mesh Budget** affects the displayed preview mesh, not the output heightmap.

Values below are reproducible starting points, not guaranteed artistic outcomes. The same seed and settings reproduce the same terrain; changing resolution changes the amount of represented detail.

## Terrain recipes

### 1. Rolling hills with a level building site

**Goal:** make a broad, gentle landscape and reserve a flat area for a small settlement.

1. Select **Rolling Hills**.
2. In **01 Terrain**, set **Width Meters** and **Length Meters** to `2016`, **Resolution** to `505 x 505`, **Seed** to `4242`, and leave **Base Elevation Meters** at `50`.
3. In **02 Shapes**, start with **Relief Meters** at `250` and **Feature Size Meters** at `700`. These are tuning values; adjust them after the preview.
4. In **06 Gameplay**, add a protected buildable zone:
   - **Center:** `0.50, 0.50`
   - **Radius Meters:** `120`
   - **Blend Meters:** `80`
   - **Elevation Meters:** `60`
5. Click **Preview**. Move the zone in **Edit Zones (2D)** if it does not sit on a useful part of the landscape.
6. Click **Final Generation**, save the recipe and create a map.

The zone constrains the generated heightmap. It does not place buildings, roads, or gameplay actors.

### 2. Compose a plateau beside a mountain range

**Goal:** combine two distinct landforms in one terrain.

1. Select **Composed Terrain** and set the footprint to `2016 m x 2016 m`. Use **Resolution** `505 x 505` while arranging the regions.
2. Add a region named `WestPlateau`:
   - **Preset:** `Plateau`
   - **Center:** `0.28, 0.50`
   - **Radius Meters:** `420, 560`
   - **Blend Meters:** `180`
   - **Strength:** `0.9`
   - **Seed:** `1101`
3. Add a region named `EastMountains`:
   - **Preset:** `Mountains`
   - **Center:** `0.72, 0.52`
   - **Radius Meters:** `520, 620`
   - **Blend Meters:** `240`
   - **Strength:** `0.85`
   - **Seed:** `2202`
4. Turn on **Edit Regions (2D)** and adjust the centers and radii until the transition looks deliberate.
5. Click **Preview**, then **Final Generation** and create a map.

Region positions are normalized from `0` to `1` on each axis. Regions are applied in order; a later region takes priority where regions overlap. Keep the blend areas clear of one another when the intended transition is uncertain.

### 3. Shape a graded pass through a mountain valley

**Goal:** lower and smooth terrain along a chosen route without claiming it is a tested road.

1. Start with **Mountains** or **Valley**, then click the **Corridor** guide button.
2. In **Guides / Water (2D)**, place three ordered points along the intended pass, for example near `(0.15, 0.52)`, `(0.50, 0.48)`, and `(0.85, 0.50)`.
3. Set each point's **Elevation Meters** to a gentle profile, for example `80`, `65`, and `85`.
4. Start with **Half Width Meters** `100`, **Blend Meters** `150`, and **Strength** `1.0`.
5. Preview and widen the guide if the surrounding terrain remains too steep. Then run **Final Generation**.
6. Use **Analyze Traversability** with **Max Slope (deg)** at `25`, then compare with `35`. Inspect the largest connected passable region and the map view.

The guide button is **Corridor**; its mode is shown as **Elevation Corridor** in the guide settings. It shapes elevation only. The traversability analysis checks the encoded height grid and its triangles; it does not validate collision, character movement, or navigation.

### 4. Prepare an island with a protected landing area

**Goal:** make an island silhouette and reserve a gently sloped area near one coast.

1. Select **Island** and set the footprint to `2016 m x 2016 m`.
2. Keep **Sea Level Meters** at `0` for the first pass. Set **Coast Blend** to `0.35` and **Coast Irregularity** to `0.45`.
3. In **06 Gameplay**, add a protected zone at `(0.68, 0.50)` with **Radius Meters** `90`, **Blend Meters** `100`, and **Elevation Meters** `30`.
4. Enable **Show Context** and **Edit Zones (2D)**. Move the zone toward the coast that best fits the intended layout.
5. Preview the coast before adding more detail. Run **Final Generation** and inspect the created map.

The protected zone is a height constraint, not a shoreline, harbor, or structure. Check the surrounding slopes in the actual level.

### 5. Create a dune field with a controlled repeatable pattern

**Goal:** compare dune spacing and height without changing the map between iterations.

1. Select **Dunes** and set **Seed** to `8128`.
2. Use **Resolution** `505 x 505` and the default `2016 m x 2016 m` footprint while tuning.
3. In **02 Families**, start with **Dune Spacing Meters** `160` and **Dune Height Meters** `18`.
4. Click **Preview**. Change one value at a time and compare the result.
5. Once the pattern is suitable, choose the final output resolution, run **Final Generation**, save the recipe, and create the map.

These settings shape terrain. They do not create vegetation, sand materials, or wind effects.

### 6. Add restrained hydraulic erosion to a mountain recipe

**Goal:** introduce surface incision as an artistic variation.

1. Select **Eroded Mountains** and start at **Resolution** `505 x 505`.
2. Keep **Hydraulic Iterations** at `0` for the baseline preview.
3. Set **Hydraulic Iterations** to `8`, then preview again. If useful, try `16` as a separate comparison.
4. Keep the same **Seed** while comparing iteration counts. Adjust **Hydraulic Rainfall Meters** and **Hydraulic Max Depth Meters** only after seeing the first result.
5. When satisfied, set the intended final resolution and click **Final Generation**.

Hydraulic erosion is an artistic heightmap process, not a real-time rainfall or water simulation. Higher iteration counts can increase generation cost. Keep a saved recipe for the un-eroded baseline if you need a reliable comparison.

### 7. Use a heightmap file as the terrain source

**Goal:** preview and generate a landscape from a supported input file.

1. Create or open a recipe and choose **Import** in the panel, or use the Blueprint node described in example 10.
2. Select a square, supported source: headerless 16-bit little-endian `.raw`/`.r16`, or a 16-bit grayscale `.png`. Supported sides are `127`, `253`, `505`, `1009`, `2017`, or `4033` samples.
3. Set **Width Meters**, **Length Meters**, **Min Altitude Meters**, and **Max Altitude Meters** to describe the intended physical extent and height range.
4. Preview and inspect the terrain. Without a valid export manifest, the current recipe's dimensions and altitude range interpret the height values.
5. Run **Final Generation** before map creation or export.

Color images, 8-bit PNGs, non-square images, and tiled image sets are not accepted as a single heightmap.

### 8. Add a lake request to a recipe

**Goal:** prepare a lake surface and its guide for construction with supported native Water features.

1. In **Visual Setup, Materials & Native Water**, click **Lake**. This adds a closed guide enclosing the water and a lake request.
2. In **Guides / Water (2D)**, move the contour points so the loop encloses the intended lake. A lake contour must be closed and have at least three points.
3. In **07 Water Prototype**, set the request's **Surface Elevation (m)** and **Depth (m)**. Check its **Guide Index**; guide indices start at zero, so `0` means Guide 1.
4. Keep the lake away from protected zones for the first test. New requests have **Affects Landscape (Carve)** enabled; turn it off if the request should form a surface without carving.
5. Run **Check Water Plan**, review the preview, and then use **Final Generation** before creating a map.

The 2D contour is a planning shape, not a 3D water surface. If the editor reports that native Water construction is unavailable or the request conflicts with protected terrain, resolve the report or leave the request disabled.

### 9. Review and accept a drainage proposal

**Goal:** inspect generated catchments and tributaries before adding them to the recipe.

1. Generate a current **Final** result for the recipe.
2. In **09 Drainage**, start with **Analysis Resolution** `513`, **Minimum Catchment (km²)** `0.05`, **Channel Depth (m)** `3`, and **River Half Width (m)** `8`.
3. Click **Preview Drainage**. Review the proposed paths in the 2D view and the status summary.
4. If the proposal is useful, click **Apply Drainage to B**. This accepts the proposal into the working recipe only.
5. Generate **Final Generation** again. Native Water actors are a separate step; the drainage preview does not create actors or change a level.

Use the preview before applying. It is an analysis proposal, not a guarantee of hydrologically correct or playable water.

## Editor Blueprint examples

### 10. Create a one-click classic map tool

**Goal:** create a new classic Landscape map from a saved recipe and report failures.

1. Create an **Editor Utility Widget** with a **Create Map** button and a status `TextBlock`.
2. Add an editable recipe asset reference to the widget and assign a saved recipe before running it.
3. In the button's `OnClicked` graph, call **Create Landscape Map**.
4. Connect the recipe reference to **Recipe**. Set **Map Package Name** to a new package such as `/Game/Generated/Terrain/Practice_Valley`.
5. Connect **Return Value** to a `Branch`.
6. On success, update the status text with a success message and review/save the newly created level. On failure, display **Message** verbatim.

Use a unique package name. Run the widget in the editor, outside Play In Editor. Do not put this graph in a runtime gameplay widget.

### 11. Create a World Partition map from a button

**Goal:** create a partitioned level from the same kind of recipe.

Use the widget pattern from example 10, but call **Create World Partition Landscape Map**:

1. Set **Map Package Name** to a new package, for example `/Game/Generated/Terrain/Practice_Valley_WP`.
2. Set **Grid Size In Components** to `2` for the documented initial example.
3. Leave **Loading Range** at `80000` cm (800 m) unless the intended level needs a different range.
4. Branch on **Return Value** and show **Message** for either result.
5. Review the created level and save the packages requested by the editor.

The loading range applies to a newly created World Partition map. It does not guarantee that all cells in a very large world remain loaded.

### 12. Export a reusable recipe from an editor button

**Goal:** let a level designer export the current saved recipe without opening the panel's export controls.

1. Add an editable recipe asset reference to an **Editor Utility Widget**.
2. Add text fields or editable variables for **Directory** and **Base Name**. Choose a new destination for each export.
3. On the export button's `OnClicked`, call **Export Recipe** with the recipe, directory, and base name.
4. Branch on **Return Value**. Display the output **Message** in a status text block.
5. Confirm the new folder contains `Height.r16`, `Height.png`, `Slope.png`, and `Manifest.json`.

The export does not overwrite an existing destination. Choose a new folder/name rather than relying on replacement.

### 13. Assign a heightmap file from an editor widget

**Goal:** select a source file for a recipe, then preview it in the panel.

1. Add an editable recipe reference and a text field for the full input-file path to an **Editor Utility Widget**.
2. On a **Set Heightmap Source** button's `OnClicked`, call **Configure Heightmap Source** with the recipe and path.
3. Branch on **Return Value** and show **Message**.
4. On success, open the recipe in **UltimateLandscape**, confirm the source and dimensions, and click **Preview**.
5. If the source path or file contents change, configure/import the source again explicitly before generating.

The node assigns a supported source; it does not create the final Landscape or choose its physical scale for you.

### 14. Preview and apply drainage from Blueprint

**Goal:** expose drainage analysis as two explicit editor actions.

Create an **Editor Utility Widget** with an editable recipe reference, a stored proposal reference, a stored token string, and a status text block.

For the **Preview Drainage** button:

1. Call **Preview Drainage** with the recipe.
2. Store the returned **Proposal** and **Token** in the widget.
3. Branch on **Return Value** and display **Message**.
4. Enable the separate **Apply Proposal** button only if the preview succeeded.

For **Apply Proposal**:

1. Call **Apply Drainage** with the original recipe, the stored proposal, and the exact stored token.
2. Branch on **Return Value** and display **Message**.
3. If the recipe changes after preview, preview again instead of reusing the old token.
4. Save the recipe asset after reviewing the accepted proposal.

The preview creates a temporary proposal; applying it changes the recipe, not an existing Landscape or native Water actor.

### 15. Synchronize a recipe with an existing Landscape cautiously

**Goal:** review proposed Water actor changes before applying them to a map.

Use this only for the matching generated Landscape and recipe:

1. In the widget, provide references to the recipe and target Landscape.
2. Call **Associate Landscape Recipe** and stop if its **Return Value** is false; show **Message**.
3. Call **Preview Water Synchronization**. Store the returned **Token** and display **Message** for review.
4. After the user reviews the plan, call **Synchronize Water** with the same recipe, Landscape, and exact preview token.
5. Start with **Replace Edited** and **Allow Removal** set to false. Do not enable either automatically.
6. If the source changed, preview again. If the operation asks to replace edited actors or remove actors, inspect and authorize that choice deliberately.
7. Review the level and save it explicitly after a successful synchronization.

Preview and synchronization are separate on purpose. The preview token becomes stale when the recipe or relevant level state changes.

### 16. Paint generated biome layers on a new Landscape

**Goal:** assign permanent terrain masks to an existing compatible Landscape material.

1. In **08 Biomes**, enable **Biomes** and choose a **Background** biome such as `Temperate` or `Alpine`.
2. For a mountain example, start with **Rock Start Slope Degrees** `25`, **Rock Full Slope Degrees** `50`, **Snow Start Altitude Meters** `800`, and **Snow Full Altitude Meters** `1200`.
3. Preview the masks. Preview and export do not require Layer Info assets.
4. For actual paint, assign a compatible **Landscape Material** and bind the six existing weight-blended Layer Info assets for Grass, Dirt, Sand, Rock, Snow, and Mud.
5. On a new map, optionally enable **Paint On Creation**. For an existing fully loaded generated Landscape, call **Apply Generated Biome Layers** from an editor widget or use **Apply Base Biomes** in the panel.
6. Review the paint, undo if needed, and save the level explicitly.

This paints permanent layers. It does not create or modify the material or its textures. **Paint On Creation** is off by default.

### 17. Recompute soils from a composed Landscape

**Goal:** include a loaded native Water composition when analyzing shoreline and wetness masks.

1. Enable **Water Aware Soils** in the recipe's biome settings.
2. Select the associated, fully loaded Landscape outside Play In Editor and ensure the recipe has a current **Final** generation.
3. Click **Analyze Composed Soils**. This captures the native composition and computes a preview; it does not paint.
4. Inspect **Show composed snapshot** and the shore/wetness/submerged diagnostics.
5. If the result is intended, click **Apply Composed Soils**, or call **Apply Water-Aware Biome Layers** from an editor widget.
6. Review the affected owned biome paint, then save the level explicitly.

This workflow samples the loaded Landscape and managed Water geometry. It is not a groundwater simulation. Do not apply a stale analysis after changing the recipe, Landscape, or Water actors.

### 18. Check and refresh cached Landscape grass

**Goal:** diagnose grass placement after a supported generated-base update.

1. In an editor widget, provide a reference to the fully loaded target Landscape.
2. Call **Check Landscape Grass Heights**.
3. Display **Heights Match**, **Max Error Centimeters**, **Sample Count**, and **Message** for review.
4. If the cached placement needs rebuilding, call **Refresh Landscape Grass** and branch on **Return Value**.
5. Review the Landscape and save the level if the editor marks it dirty.

These nodes inspect or invalidate existing cached grass on loaded proxies. They do not create grass, alter the terrain, or guarantee that foliage assets are configured.

## Final checks before handing off a map

- Run **Final Generation** after the last recipe change.
- Save the recipe separately from the map.
- For World Partition, load all cells before updating or applying map operations that require the complete Landscape.
- Review every Blueprint **Return Value** and display **Message** when a call fails.
- Treat preview, drainage, slope, and soil analyses as aids for iteration; verify the final Landscape, collision, and movement in the level.
