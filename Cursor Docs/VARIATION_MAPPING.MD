# VARIATION_MAPPING.MD

## Purpose

This document is the **single reference for how variation identifiers in the ID registry (Plattar 3D) align with option structure and choice IDs in the model pricing config**. It defines naming conventions, key structure, and mapping rules so an agent or implementation can resolve one system from the other. **Live data** for each model is in that model’s **model content file** (e.g. Model Content Data/Staggered Harmony/MODEL_CONTENT_STAGGERED_HARMONY.json): `pricing` (sidebar options), `plattar.idRegistry` (scene/variation IDs), and `plattar.exteriorVariationMap` (choice id → variation UUID for 3D).

---

## 1. Document Roles

| Source (in model’s MODEL_CONTENT_&lt;NAME&gt;.json) | Role |
|--------------------------------|------|
| **pricing** | Single source of truth for all configurable options in the sidebar: categories, choice `id`s, labels, display type, prices. State and UI are driven from this. |
| **plattar.idRegistry** | Scene and variation identifiers used only for the **3D exterior viewer**. Not all sidebar options have a corresponding 3D variation. |
| **plattar.exteriorVariationMap** | Map from pricing choice id to Plattar variation UUID for Exterior Cladding; used directly by the app for 3D updates. |

---

## 2. Key Structures (Summary)

### 2.1 Pricing (MODEL_CONTENT.pricing)

- **Path:** `pricing.options[]` → each element has `category`, `type`, `display`, `choices[]`.
- **Choice identifier:** `choices[].id` — string, snake_case (e.g. `corrugated_iron_grey_friars`). This is the **option key** used in central state and for resolving behaviour.
- **Categories** define accordion sections; **choices** define selectable options. No UUIDs; only semantic IDs and labels.

### 2.2 ID Registry (MODEL_CONTENT.plattar.idRegistry)

- **Scenes:** `scenes[]` — combined scene(s) for the configurator. One scene may contain multiple **sceneproducts** (e.g. Interior, Exterior).
- **Products:** `products[]` — product-level view; each has `id`, `sceneId`, `variations[]`.
- **Variation identifier:** `variations[].id` — UUID used by Plattar API (e.g. `129b77b7-c712-45af-a0a3-311736dc8924`).
- **Variation label in registry:** `variations[].title` — human-readable name (e.g. "Grey Friars", "Ebony"). Used for documentation and for deriving or validating mappings; the API uses the UUID.

---

## 3. Which Options Have 3D Variations?

Only **Exterior Cladding** choices in Pricing correspond to **3D exterior variations** in the ID Registry.

- **ID Registry – Exterior product:** Under the "Staggered Harmony Exterior" product, there are **four** variations: Grey Friars, Ebony, Corten, Sandstone (each with a UUID).
- **Pricing – Exterior Cladding:** Four choices with `id`s: `corrugated_iron_grey_friars`, `corrugated_iron_ebony`, `corrugated_iron_sandstone_grey`, `seemlok_printed_corten_steel_corrugated_iron_ebony`.

All other Pricing options (Interior Lining, Flooring, Fireplace, Heatpump, Water Heating, Laundry Joinery, Bedroom Wardrobe, Solar) **do not have 3D objects** in the ID Registry. They drive pricing, interior images (where applicable), and state only.

---

## 4. Mapping: Pricing Option ID → Plattar Variation ID (Exterior Only)

To update the 3D viewer when the user selects an exterior cladding option:

1. Read the selected **option id** from state for the category that corresponds to exterior (Exterior Cladding).
2. Resolve that option id to a **Plattar variation UUID** using the mapping below.
3. Call the Plattar API with that variation id (and the correct scene/product context from the ID Registry).

**Reference mapping (Pricing choice `id` → ID Registry variation `title` / usage):**

| Pricing `options[].choices[].id` (Exterior Cladding) | ID Registry variation `title` | Notes |
|------------------------------------------------------|-------------------------------|--------|
| `corrugated_iron_grey_friars`                         | Grey Friars                   | 1:1   |
| `corrugated_iron_ebony`                               | Ebony                         | 1:1   |
| `corrugated_iron_sandstone_grey`                      | Sandstone                     | Label in Pricing is "Sandstone Grey" |
| `seemlok_printed_corten_steel_corrugated_iron_ebony`  | Corten                        | Label in Pricing references Corten + Ebony |

The **exact UUID** for each is in that model’s model content file under `plattar.exteriorVariationMap` (choice id → UUID). The app uses this map directly; no runtime lookup in the registry is required for 3D variation updates.

**Implementation:** The app uses `plattar.exteriorVariationMap` from MODEL_CONTENT (choice id → UUID) for Exterior Cladding. The table above documents the semantic alignment for reference.

---

## 5. Interior and Other Categories: No 3D Mapping

- **Interior Lining:** Pricing has four choices. The ID Registry has a single "Staggered Harmony Interior" product with one variation ("Interior"). The configurator does **not** switch 3D interior variations; interior is shown via **static images** (INTERIOR_IMAGE_REGISTRY / variation key). No mapping from Pricing interior choice ids to ID Registry variation ids is required for 3D.
- **Flooring, Fireplace, Heatpump, Water Heating, Laundry Joinery, Bedroom Wardrobe, Solar:** No entries in the ID Registry for these. They exist only in Pricing and in state; they drive pricing (add-ons) and, where applicable, interior image variation key and tab behaviour. No Plattar variation IDs to resolve.

---

## 6. Naming Conventions That Connect the Two Systems

- **Pricing:** `category` = accordion section; `choices[].id` = stable option key (snake_case). Used everywhere in state and UI.
- **ID Registry:** `variations[].title` = short display name; `variations[].id` = UUID for API. No snake_case in the registry.
- **Link:** For **Exterior Cladding only**, the link is semantic: option id and variation title refer to the same finish (e.g. grey friars, ebony, sandstone, corten). The mapping table in §4 defines that link; UUIDs must always come from the registry.

---

## 7. Resolving One from the Other (Reference Pattern)

**From state (Pricing choice id) to Plattar variation id (for 3D):**

1. If the option is not in the Exterior Cladding category → do not call Plattar; no 3D variation to set.
2. If the option is Exterior Cladding → read the Plattar variation UUID from `plattar.exteriorVariationMap[choiceId]` in MODEL_CONTENT.

**From ID Registry to Pricing (for documentation or tooling):**

1. Under the Exterior product, each `variations[].title` corresponds to one Exterior Cladding choice in Pricing (see §4).
2. Other registry entries (Interior product, other scenes) do not map to Pricing choice ids in a 1:1 way; Interior is a single 3D placeholder; the rest of Pricing has no 3D representation.

---

## 8. Inconsistencies and Gaps

- **Exterior count match:** Pricing has 4 Exterior Cladding choices; ID Registry has 4 exterior variations. Counts align. Naming alignment is semantic (Grey Friars, Ebony, Sandstone, Corten) with one Pricing label differing ("Sandstone Grey" vs "Sandstone", and the long **UniCote LUX - Printed Corten Red/Corrugated Iron - Ebony** label for the fourth exterior option). **Flag:** If more exterior options are added to Pricing, corresponding variations must be added to the ID Registry (and this mapping updated); otherwise 3D cannot be shown for the new option.
- **Interior:** ID Registry has one interior variation; Pricing has four Interior Lining choices. By design, interior 3D is not used; interior is driven by static images and variation key. No inconsistency, but worth stating: **no mapping from Interior Lining choice id to Plattar variation id is used.**
- **Scene ID:** The ID Registry contains multiple scene/product identifiers (e.g. combined scene vs product-specific `sceneId`). Which scene ID the embed must use for the 3D viewer is defined elsewhere (e.g. implementation or later spec). This document does not prescribe a single scene ID; it only links **variation** identity between Pricing and the registry.

---

## 9. Summary

- **MODEL_CONTENT.pricing** defines all sidebar options and their structure; **MODEL_CONTENT.plattar** holds scene ID, idRegistry, and **exteriorVariationMap** (choice id → variation UUID for 3D).
- Only **Exterior Cladding** options map to 3D variation IDs; the app uses `exteriorVariationMap` directly.
- Use this document to understand ID ↔ pricing alignment and which options have 3D mapping. Live data is in each model’s model content file (e.g. Model Content Data/Staggered Harmony/ or Model Content Data/Compact Harmony/); the table in §4 documents semantic alignment for reference.
