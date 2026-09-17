# SPEC 2: Feature Specifications (3D Viewer, Interior Slider, Floorplan, Pricing)

## 1. 3D Exterior Viewer

### Purpose

Displays interactive 3D exterior model only.

Interiors are not rendered in 3D.

### Requirements

- Embedded Plattar viewer. Scene ID and other Plattar IDs (variation IDs, etc.) will be provided in a later stage with full documentation; the current implementation need only provide the basic scaffolding (embed container, loading state, and a placeholder for future API integration).
- No custom arrows or slider controls.
- Viewer occupies full size of Media Area container.
- Responsive scaling inside container.

### Behaviour

- Loads on page initialization. Until the Plattar viewer is ready, show a loading state in the 3D panel (e.g. a simple loader); do not show a blank area.
- Does not reload when switching tabs.
- When exterior-related option is selected:
  - State updates.
  - Media tab auto-switches to 3D Exterior.
  - If applicable, 3D configuration updates via API messaging. Throttle or debounce 3D API calls (e.g. variation updates) to avoid excessive requests and ensure smooth UX.

---

## 2. Static Interior Photo Slider

### Library Requirement

Must use Slick Slider, loaded via CDN. Document in implementation that Slick is loaded from CDN; if the Slick script fails to load, show a fallback message (e.g. that the interior gallery is unavailable) so the rest of the app (tabs, pricing, 3D, sidebar) continues to work. No custom-built slider logic.

---

## 3. Slider Structure

### Core Rules

- Exactly 4 slides.
- Each slide represents a different camera angle or room grouping.
- Each slide contains multiple images stacked in the same position.
- Only one image per slide visible at any time.

Each image variation corresponds to configuration selections (e.g. colour, finish).

---

## 4. Image Variation Logic

### Behaviour Model

Each slide contains:
- Base image
- Multiple variant images (for each interior option combination)

**Initial load:** The first interior slide image must be visible on page load without user interaction. Slider images are initialised with the correct variation URL (from the current variation key) when the slider is built; lazy loading must not prevent the first slide from displaying.

When a user selects an interior option:

1. State updates.
2. Active interior variation key recalculates.
3. Each slide:
   - Hides all images.
   - Shows a loading state until the target variation image has loaded, then shows only the image matching the current variation key (or base image if not yet loaded).
4. Slick remains on current slide index.

No slider reinitialization allowed.

---

## 5. Slider Interaction Rules

### Desktop

- Arrows enabled.
- Dots disabled.
- Swipe enabled.

### Mobile

- Arrows hidden.
- Swipe enabled.
- Dots disabled.

All of the above must use native Slick configuration variations.

---

## 6. Floor Plan Viewer

### Structure

- Single static image. Exactly one floor plan image for the model; no variation options affect it. The image URL is defined in that model’s **model content file** (e.g. Model Content Data/&lt;Model Name&gt;/MODEL_CONTENT_&lt;NAME&gt;.json) under `images.floorPlan` and exposed in the app as `FLOORPLAN_IMAGE_URL`.
- No slider.
- Fully responsive.
- Maintains aspect ratio.
- Centred within container.

No dynamic image switching. No user action in the configurator automatically switches to the Floorplan tab; the user opens it only by clicking the Floorplan tab.

---

## 7. Pricing Calculator

### Architecture

- That model’s **model content file** (e.g. Model Content Data/&lt;Model Name&gt;/MODEL_CONTENT_&lt;NAME&gt;.json) `pricing` is the **single source of truth** for all configurable options shown in the sidebar accordion. Categories, option IDs, labels, display type (box with image, box without image), choice lists, and per-option prices are defined there. Add-on options use two selectable boxes (None / Option included) rather than tick boxes. The sidebar must be rendered from this configuration; no duplicate or alternate definition of options is permitted.
- Fully JS-based.
- Driven by central state object.
- No hardcoded UI pricing logic.
- Pricing rules stored in the same structured configuration (MODEL_CONTENT.pricing).
- **Pricing recalculation is triggered only when the user changes one of the add-on options that affect price.** Interior and exterior colour/finish options do not change the price; only the designated add-on options (e.g. the six add-on options defined in the pricing configuration) should invoke the pricing calculation function. Other option changes (e.g. exterior cladding, interior lining) update state and media but do not trigger pricing recalculation.

### Sidebar option behaviour (ConfiguratorUIUpdates)

- **Fireplace:** The "Add Fireplace" variation option box displays "Requires consent" (medium grey) below the price text within the box.
- **Laundry Joinery:** Three choices (None, 1 Laundry Joinery, 2 Laundry Joinery), box without image. Visible only when the model is 2-bedroom (single source of truth in app; e.g. constant or future selector).

---

## 8. Pricing Logic Model

State object contains:
- Base price.
- Selected exterior variations.
- Selected interior variations.
- Selected structural/layout variations.

Pricing engine:

1. Start with base price.
2. Loop through selected variations (only those that carry price modifiers / add-ons).
3. Add price modifiers.
4. Apply conditional logic if required.
5. Return total (rounded up to whole numbers).

---

## 9. Pricing UI Requirements

- Live updating total (pricing display box overlay; see SPEC for placement).
- Format as currency: round up to whole numbers only; no decimal places.
- Include "From $[PRICE] incl. GST".
- Include a "Go to Kit Guide" link (e.g. below the price) that opens in a new tab; link target is configurable.
- No page reloads.

---

## 10. Cross-Feature Interaction Rules

When user selects:

### Exterior Option
- Update state.
- Recalculate pricing only if the option is one of the add-on options that affect price (not for colour/finish-only options).
- Switch to 3D Exterior tab.

### Interior Option
- Update state.
- Recalculate pricing only if the option is one of the add-on options that affect price (not for colour/finish-only options).
- Switch to Interiors tab.
- Update visible images inside each slide (with loading state until variation image has loaded).

### Layout Option
- Update state.
- Recalculate pricing only if the option is one of the add-on options that affect price.
- Do not switch tab. (No option affects the floor plan; the Floorplan tab is only shown when the user manually selects it.)

---

## 11. State Synchronisation Rules

Single source of truth:
- One global state object.

All features subscribe to it:
- 3D viewer.
- Interior slider.
- Floorplan display.
- Pricing calculator.

No feature stores duplicate logic.

---

## 12. Initialization Order

On page load:

1. Initialize state with defaults.
2. Show loading state for any content that may take time (3D viewer, static assets). Initialize Plattar 3D viewer (embed + API; may run async); keep 3D panel in loading state until viewer is ready.
3. Initialize Slick slider (if Slick fails to load, show fallback message and leave rest of app functional).
4. Initialize tab system.
5. Bind sidebar event listeners.
6. Run first pricing calculation (using add-on options only).
7. Ensure default media tab is visible (3D Exterior); 3D panel shows loading until viewer ready.

---

## 13. Error Handling

- If variation image not found:
  - Fallback to base image.
- If pricing modifier missing:
  - Default to zero.
- If 3D API fails:
  - Do not break UI. Show a clear, user-friendly error message (e.g. that the 3D view is temporarily unavailable and they can continue configuring). Use friendly, non-technical language. **Error message placement:** Display in the centre of the Media Area, inside a box with a solid background so it is readable.
- General loading and error messages (e.g. for 3D or slider) must use a very basic loader or message box as appropriate; place error messages in the centre of the Media Area in a box with a solid colour background.

---

## 14. Extensibility Requirements

System must support:

- Adding new image variations without rewriting slider logic.
- Adding new pricing modifiers without rewriting pricing engine.
- Adding new slides (up to 4 max).
- Adding new configuration categories.

All mappings must be data-driven, not hardcoded per option.

---

## 15. Images Architecture – Detailed

This update replaces previous assumptions about slide count and partial coverage.

The interior image system is now locked to:

- Exactly 4 slides
  1. Living Area
  2. Bathroom
  3. Laundry
  4. Wardrobe
- 36 variation images per slide
- Full permutation coverage of all supported interior option combinations

All combinations are supported. No partial matching logic is required.

---

### 15.1 Slide Structure (Final)

The Slick slider must contain exactly 4 slides:

### Slide 1 — Living Area  
Contains:
- 1 base fallback image
- 36 variation images (one per interior permutation)

### Slide 2 — Bathroom  
Contains:
- 1 base fallback image
- 36 variation images (one per interior permutation)

### Slide 3 — Laundry  
Contains:
- 1 base fallback image
- 36 variation images (one per interior permutation)

### Slide 4 — Wardrobe  
Contains:
- 1 base fallback image
- 36 variation images (one per interior permutation)

Slides are fixed.  
They must not be dynamically created or removed.

Slick initializes once and never reinitializes.

---

### Interior Image Registry (Authoritative Mapping)

All image URLs must live inside:

`INTERIOR_IMAGE_REGISTRY`

Structure conceptually:

- Top level keys:
  - `living`
  - `bathroom`
  - `laundry`
  - `wardrobe`
- Each contains:
  - `base` (fallback URL)
  - Variation key → Wix CDN URL mapping

There must be exactly 36 variation keys under each slide type.

No image URLs may exist outside this registry.

Implementation notes:
1. Everything Must Live Inside the Same Script Scope
2. Avoid Global Namespace Pollution

---

### 15.2 Variation Key Specification

The variation key is derived deterministically from the central state and used only for image lookup (rendering). Canonical keys and option categories can be derived from the image list in that model’s **model content file** under `images.interior`, where keys encode categories and variation styles.

**Format:**

- **Category order (fixed):** Floor → Walls → Ceiling → Cabinets.
- **Segment format:** `Category-Slug` (e.g. `Floor-SmoothOakHighNoon`, `Walls-Birch`).
- **Separator between segments:** underscore `_`.
- **Example key:** `Floor-SmoothOakHighNoon_Walls-Birch_Ceiling-Birch_Cabinets-White`.

The INTERIOR_IMAGE_REGISTRY must use this same key format. State must store selections per category so the key can be built in this order with these slugs. When adding or updating images, generate the key from this convention and ensure the registry entry matches.

### 15.3 Variation Key Architecture (Rendering Scope Only)

The variation key:

- Is derived deterministically from the central state using the format above.
- Is used only for visual resolution (image lookup).

The variation key is:

- Not a pricing engine.
- Not a pricing lookup table.
- Not a 3D ID abstraction.
- Not the system backbone.

It is a rendering-layer utility.

---

### Relationship to Pricing Engine

Pricing must remain:

- Option-based
- Additive or rule-driven
- Category-normalized

The pricing engine must never:

- Read from the image registry
- Depend on variation key lookups
- Store prices keyed by variation string

Both pricing and variation key derive independently from the same central state object.

---

### Image Resolution Logic (Final)

When an interior-related option changes:

1. Central state updates.
2. Variation key is regenerated.
3. For each slide:
   - Lookup:
     `INTERIOR_IMAGE_REGISTRY[slide_type][variation_key]`
4. Because all permutations exist:
   - A match is expected.
5. If no match is found:
   - Fallback to `base` image.
   - Do not throw error.
   - Do not break slider.

Slider index must not change.

Slick must not reinitialize.

Image swap must occur via:
- src replacement OR
- visibility class toggling

No DOM rebuild loops.

---

### Fallback Behaviour (Safety Layer)

Even though all permutations exist:

If:

- Registry entry missing
- URL undefined
- Key malformed
- Image fails to load

Then:

- Display slide `base` image.
- Continue functioning.
- Do not interrupt pricing.
- Do not interrupt tab system.

Fallback must be silent and non-blocking.

---

### Performance Rules and Image-Loading Strategy

- Exactly 144 variation images total (36 per slide). Performance must be managed carefully.
- **Single image-loading strategy (authoritative):** Load only what is needed for the current view. Preload the base image for each slide (4 images) so fallbacks are instant. For variation images: load the image matching the current variation key for the visible slide when it is needed; when the user changes options or switches slides, load the newly required variation image(s) on demand. Do not preload all 144 variation images on page load. This keeps initial load minimal and is the standard approach for many images on one page.
- Avoid destroying/creating image elements on variation change (prefer src update or visibility toggling).
- O(1) registry lookup required.
- No conditional chains allowed in resolution logic.

Resolution flow must always be:

State → Variation Key → Registry Lookup → Render.

---

### Authoring Workflow (Locked Process)

When adding or updating images:

1. Upload image to Wix.
2. Copy CDN URL.
3. Generate correct variation key.
4. Insert under:
   - `living[variation_key]`
   - `bathroom[variation_key]`
   - `laundry[variation_key]`
   - `wardrobe[variation_key]`
5. Verify deterministic key generation matches registry entry.

No UI logic changes permitted during this process.

---

### Non-Negotiable Invariants (Final)

- Exactly 4 slides.
- 36 variation keys per slide.
- Full permutation coverage.
- All image URLs centralized in registry.
- Variation key used only for rendering.
- Pricing logic fully independent.
- Slick initializes once.
- Image switching never rebuilds slider.
- Registry lookup is the sole image resolution mechanism.

This final structure ensures clarity, predictability, and long-term maintainability.