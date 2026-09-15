# Model content — Schema and usage (multiple models)

**Purpose:** Each configurator model has its own **model content file** and **index file**. The model content file is the single source of truth for that model; edit it, then paste its contents into the `MODEL_CONTENT` JSON block in **that model’s index file**. No dynamic loading.

**Location and naming:** Each model's JSON lives in `Model Content Data/<Model Name>/MODEL_CONTENT_<NAME>.json` (e.g. `Model Content Data/Staggered Harmony/MODEL_CONTENT_STAGGERED_HARMONY.json`, `Model Content Data/Compact Harmony/MODEL_CONTENT_COMPACT_HARMONY.json`). Paired with `index_<model_slug>.html` in the project root (e.g. index_staggered_harmony.html, index_compact_harmony.html). Adding a new model: create a folder under Model Content Data, add the JSON file, duplicate an index, and update the content.

---

## Top-level structure

| Section | Description |
|--------|-------------|
| `metadata` | Model name and postMessage type for the embed. |
| `pricing` | Product (name, basePrice, currency) and sidebar options (categories, choices, prices, display type). |
| `plattar` | 3D viewer: scene ID, full ID registry (scenes/sceneproducts/variations), and **exteriorVariationMap** (pricing choiceId → Plattar variation UUID). |
| `images` | Interior slider images (per slide type), variation option thumbnails, and floor plan URL. |

---

## metadata

| Field | Type | Description |
|-------|------|-------------|
| `modelName` | string | Display name (e.g. used in page title and postMessage payload). |
| `postMessageType` | string | Message type sent to parent (e.g. `"configurator-config"`). Wix Velo filters on this. |

---

## pricing

- **product**: `{ "name", "basePrice", "currency" }` — Same shape as before; drives sidebar and pricing display.
- **sidebarGroups**: Array of accordion section groups that define how options are grouped in the sidebar UI. Each entry has:
  - **id**: string identifier for the group (e.g. `"exterior"`).
  - **title**: display title for the accordion header (e.g. `"Exterior"`).
  - **categories**: array of category strings that belong to this group, in display order (e.g. `["Exterior Cladding"]`). Every category in `options` must appear in exactly one group.
- **options**: Array of option groups. Each has:
  - **category**: string (e.g. `"Exterior Cladding"`).
  - **type**: `"single_select"`.
  - **display**: `"box_with_image"` or `"box"` (sidebar rendering).
  - **choices**: Array of `{ "id", "label", "price" }` (price may be `null` for some options).

Choice `id` values must match:
- Keys in `images.variationOptions` for options with `display: "box_with_image"`.
- Keys in `plattar.exteriorVariationMap` for the category that drives 3D exterior (e.g. Exterior Cladding).

**Bedroom Wardrobe visibility:** The `IS_2_BEDROOM` flag (set per model in the index file) controls whether the **2x wardrobe choice** is shown. When `IS_2_BEDROOM` is false, the `bedroom_wardrobe_2x` choice is omitted from the rendered list. All other categories (including Laundry Joinery) are always shown regardless of this flag.

**Initial selection (index `initState`):** The first entry in each option’s `choices` array becomes the default on load. For **Flooring**, keep `none_flooring` ($0) first and include it in `images.variationOptions`. For **Water Heating**, keep `no_water_heating` ($0) first so the 3D add-on maps to `hotwater.none`.

---

## plattar

| Field | Description |
|-------|-------------|
| `sceneId` | UUID of the Plattar scene loaded by the 3D viewer. |
| `idRegistry` | Full registry: `scenes[]` (each with `id`, `sceneproducts[]`, product titles, `variations[]` with `id` and `title`), and optionally `products[]`. |
| `exteriorVariationMap` | Object: **pricing choiceId** → **Plattar variation UUID**. Used to switch 3D exterior when the user picks an option. No hardcoded product title in the app. |

---

## images

| Field | Description |
|-------|-------------|
| **interior** | Keys are slide types: `living`, `bathroom`. Each value is an object with a **base** URL (fallback) and variation keys (e.g. `Floor-X_Walls-Y_Ceiling-Z_Cabinets-W`) mapping to image URLs. Variation keys must align with the app’s `buildVariationKey()` output. |
| **variationOptions** | Object: choice **id** → image URL. Used for sidebar option boxes when `display` is `"box_with_image"`. |
| **floorPlan** | Single URL for the Floorplan tab. |

---

## Workflow (update a model’s content)

1. Edit the model’s JSON file (e.g. `Model Content Data/Staggered Harmony/MODEL_CONTENT_STAGGERED_HARMONY.json`).
2. Copy the entire JSON.
3. Open that model’s index file (e.g. `index_staggered_harmony.html`). Find the `var MODEL_CONTENT = { ... };` block and replace the **object** (the `{ ... }` part) with the pasted JSON. Leave the rest of the script unchanged.
4. Save. The app derives `PRICING_CONFIG`, `ID_REGISTRY`, `INTERIOR_IMAGE_REGISTRY`, `VARIATION_OPTION_IMAGE_REGISTRY`, `FLOORPLAN_IMAGE_URL`, `EXTERIOR_VARIATION_MAP`, and `SCENE_ID` from `MODEL_CONTENT` at load time.

No build step; each model runs from its own index file. To verify, open that model’s index file in a browser (over HTTP/HTTPS for 3D; file protocol will show a message that 3D requires a server).
