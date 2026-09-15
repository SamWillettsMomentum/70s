# AI_BRIEF.MD

## 1. What this project is

A self-contained, embeddable house configurator UI built for Wix.  

It combines:
- A Plattar-powered 3D exterior viewer
- A Plattar-powered "View in AR" feature (AR launch / QR code fallback)
- A dynamic interior image slider (Slick Slider) driven by composite variation keys (Floor / Walls / Ceiling / Cabinets)
- A floor plan viewer
- A JS-driven pricing calculator
- A state-driven sidebar configuration system

All HTML, CSS, and JS are embedded directly inside a Wix HTML Embed element (no external hosting). The system must be modular, state-driven, and extensible.

---

## 2. Primary Objective

Build a robust, responsive configurator interface that:

- Allows users to configure a specific home model (one model per configurator instance)
- Updates visuals (3D or images) based on selections
- Recalculates pricing in real time
- Auto-switches media views contextually
- Maintains a single authoritative state object

The system must be deterministic, structured, and cleanly architected.

---

## 3. Core System Shape

The UI consists of:

- Sidebar (accordion-based configuration controls; each option may show a price, e.g. "+$0" or "+$XXX"). Options display as `box_with_image` (with variation swatch) or plain `box`.
- Media Area (tabbed: 3D Exterior, Interiors, Floorplan), with floating pill tabs and a **pricing display box** (live total, e.g. "From $[PRICE] incl. GST") as overlays at bottom-centre.
- **AR button** — floating top-right of the media area, visible only on the 3D Exterior tab when Plattar is ready. Launches AR directly on capable devices or shows the Plattar Launch Page (QR code) on desktop. Toggles between AR icon and close icon.

All logic flows through a central JS state object.

Interaction pattern:

1. User selects an option.
2. State updates.
3. Pricing recalculates.
4. Media updates.
5. Tab auto-switches if required.
6. UI re-renders dependent elements.

No feature may maintain duplicate state outside the central state object.

---

## 4. Product Boundaries

This project:

- Provides **one configurator per home model**. Each model has its own **index file** and **model content file** (no shared runtime; each pair is self-contained).
- Uses Plattar for 3D exterior rendering and AR (View in AR / QR code).
- Uses dynamic static images for interiors, selected via composite variation keys built from Floor, Interior Lining (Walls + Ceiling), and Cabinet selections.
- Uses a static floorplan image.
- Uses Slick Slider for interior slides.
- Runs entirely inside a Wix HTML embed.

It does not:
- Use a single shared index for all models (each model has its own index and content).
- Include user accounts or persistence.
- Include backend storage.
- Manage scene authoring inside Plattar.
- Include CMS integration.

**Security assumptions:** State is not persisted; config and registry data are trusted; no user input is executed or injected into DOM/script; image URLs are from a controlled source (e.g. Wix CDN).

---

## 5. Technical Constraints

- No hosting of the configurator’s own code; third-party scripts (Slick, Plattar) are loaded from their official CDNs only. All HTML, CSS, and JS for the configurator are embedded directly in the Wix HTML embed.
- All CSS and JS embedded directly in Wix (inside the single embed code block).
- Slick Slider and Plattar loaded via their official CDNs.
- Fully responsive inside Wix container.
- **Parent–iframe communication (MVP):** The configurator must pass the final calculated price and all selected variation options to the Wix page using `window.parent.postMessage()` from inside the embed. The Wix page uses a Velo listener to receive the message and populate hidden form fields before form submission.
- All state-driven logic in vanilla JS.

---

## 6. Success Criteria

The system is correct when:

- The UI is responsive inside Wix (including tablet single-column and overlay layout for tabs and pricing display). On mobile/tablet, media area is fixed at top and only the sidebar scrolls inside its container.
- Only one accordion section is open at a time.
- Option selections visually reflect state. Add-on options use two boxes (None / Option). Fireplace "Add Fireplace" option box shows "Requires consent" below the price; Laundry Joinery (No / Yes) is always shown. Bedroom Wardrobe shows None / 1x / 2x for 2-bedroom models, or None / 1x when `IS_2_BEDROOM` is false.
- Loading states are shown for 3D and for variation images where appropriate.
- Interior images switch correctly within slides based on composite variation key (Floor + Walls + Ceiling + Cabinets), with loading state until image has loaded. Fallback to base image when a key has no match (e.g. "None" flooring).
- 3D exterior updates when exterior options change (with throttling/debounce and loading until viewer ready).
- Pricing recalculates correctly and only when add-on options change; display rounds up to whole numbers.
- Media tab auto-switching works reliably; tabs and pricing display box (with "Go to Kit Guide" link) stack at bottom-centre of Media Area.
- First interior slide image is visible on load without user interaction. Sidebar has bottom padding (125px) below the accordion, visible at all breakpoints.
- Slider arrows use custom SVG icons, are visible and functional at all breakpoints (including mobile where swipe also works).
- AR button is visible only on the 3D Exterior tab when Plattar is ready. Clicking opens AR (direct or launch page fallback). Button icon swaps to a close icon while AR/launcher is active. Clicking again returns to the 3D viewer.
- Form submission is supported via postMessage to Wix (price and selected options passed to page).
- No feature conflicts or duplicated logic exist.

---

## 7. Decision Authority

If conflicts arise:

- SPEC.MD defines authoritative general behaviour and architecture.
- Features_SPEC.MD defines feature-level behaviour.
- **Per model:** That model’s **MODEL_CONTENT_&lt;NAME&gt;.json** (and the embedded block in that model’s index file) is the authoritative source for that model’s content: pricing, 3D variation IDs, images, and metadata.
- Plattar documentation files define integration rules.

Implementation must not override these sources.

---

## 8. Reference File Directory

### Core Specs

- **SPEC.MD** [Cursor Docs/SPEC.MD]
  Defines global UI architecture, layout rules, responsiveness, sidebar structure, accordion behaviour, state management, and media tab system.

- **Features_SPEC.MD** [Cursor Docs/Features_SPEC.MD]
  Defines detailed feature requirements for:
  - 3D Viewer
  - Interior Slick Slider
  - Floorplan Viewer
  - Pricing Calculator
  - Cross-feature interaction rules

- **IMPLEMENTATION_PLAN.MD** [Cursor Docs/IMPLEMENTATION_PLAN.MD]
  Master implementation plan. Phased tasks and dependencies; uses this brief as the primary entry point.

- **POSTMESSAGE_PAYLOAD.MD** [Cursor Docs/POSTMESSAGE_PAYLOAD.MD]
  Documents the payload shape sent via `window.parent.postMessage()` from the embed. Use when implementing the Wix Velo listener to populate hidden form fields (price and selections) before form submission.

- **Plattar_Architecture.MD** [Cursor Docs/Plattar_Architecture.MD]
  Describes the Plattar integration architecture: embed creation, SDK readiness polling, variation updates, and add-on wiring.

- **AR_ROLLOUT_GUIDE.md** [Cursor Docs/AR_ROLLOUT_GUIDE.md]
  Step-by-step guide for adding the "View in AR" button to each model. Includes prerequisites, per-model info to gather, implementation steps, and a testing checklist.

---

### Project structure (multiple models)

Each model has a **pair** of core files, plus supporting reference files:

- **Index:** `index_<model_slug>.html` in the project root (e.g. `index_staggered_harmony.html`, `index_compact_harmony.html`). One HTML file per model; contains the full configurator UI and an embedded `MODEL_CONTENT` block.
- **Model content:** `Model Content Data/<Model Name>/MODEL_CONTENT_<NAME>.json` (e.g. `Model Content Data/Staggered Harmony/MODEL_CONTENT_STAGGERED_HARMONY.json`, `Model Content Data/Compact Harmony/MODEL_CONTENT_COMPACT_HARMONY.json`). Authoritative source for that model’s metadata, pricing, Plattar IDs, and images. Workflow: edit the JSON file, copy, paste into the `MODEL_CONTENT` block in that model’s index file. See **MODEL_CONTENT_README.MD** [Cursor Docs/MODEL_CONTENT_README.MD] for schema and workflow.
- **Supporting files per model** (in `Model Content Data/<Model Name>/`): variation pricing JSON, Plattar IDs JSON, images JSON. These are reference/source files; the model content JSON is the authoritative source for the index file.
- **Images_Directory.json** [Model Content Data/Images_Directory.json] — Centralised registry of variation option image URLs (e.g. flooring swatches, cabinet swatches). Source of truth for `variationOptions` entries in model content files.

**Current models (5 total):**
- **Staggered Harmony** — fully implemented (`index_staggered_harmony.html`, `Model Content Data/Staggered Harmony/`)
- **Compact Harmony** — partially implemented (`index_compact_harmony.html`, `Model Content Data/Compact Harmony/`)
- **Straight Elegance** — reference data only (`Model Content Data/Straight Elegance/`)
- **Simplified Elegance** — reference data only (`Model Content Data/Simplified Elegance/`)
- **Compact Elegance** — reference data only (`Model Content Data/Compact Elegance/`)

More models can be added by duplicating an existing index + model content pair and updating the content.

---

### Variation Mapping

- **VARIATION_MAPPING.MD** [Cursor Docs/VARIATION_MAPPING.MD]
  Single reference for ID ↔ pricing alignment. Explains how variation identifiers in the ID registry map to option structure and choice IDs in the model pricing config (MODEL_CONTENT.pricing); defines naming conventions and key structure. Use when linking sidebar selections to 3D variation IDs. Live data is in each model’s model content file (e.g. Model Content Data/&lt;Model Name&gt;/MODEL_CONTENT_&lt;NAME&gt;.json).

---

### External Reference & Implementation Guides

**THIRD_PARTY_REFERENCE_SUMMARY.MD** [Cursor Docs/THIRD_PARTY_REFERENCE_SUMMARY.MD] is the orientation guide for the Euca example code. It summarises that example’s structure and patterns for context only; it is not a source for direct copy-paste implementation. Our implementation follows SPEC.MD, Features_SPEC.MD, and each model’s model content file.

- **Example_Implementation_Euca** [Example Code/Example_Implementation_Euca] 
  Source code from another company using Plattar.  
  See **THIRD_PARTY_REFERENCE_SUMMARY.MD** for an orientation summary.  
  Used as a structural reference for:
  - 3D messaging patterns
  - Variation handling patterns
  - Integration structure

- **Installation_Docs.MD** [Plattar Docs/Installation_Docs.MD]
  Plattar installation instructions.  
  Defines how to embed and initialize the 3D viewer.

- **Loading_a_Scene_Docs.MD** [Plattar Docs/Loading_a_Scene_Docs.MD]
  Plattar documentation on loading scenes programmatically.

- **Working_with_Variations_Docs.MD** [Plattar Docs/Working_with_Variations_Docs.MD]
  Plattar documentation on activating and switching variations/options via API.

- **ar_Docs.md** [Plattar Docs/ar_Docs.md]
  Plattar documentation on AR features: QR code toggle, Launch Page, direct AR launch (`startAR`), and dual-embed (viewer + QR) patterns.

---

## 9. Shared Invariants

- Single central state object.
- No duplicated pricing logic in UI.
- No duplicated variation mapping in UI.
- Media components do not own state.
- Tabs do not destroy/reinitialize components.
- Slick slider initializes once only.
- All visual state reflects the central state.

---

## 10. Implementation Philosophy

- Data-driven, not hardcoded.
- Modular logic separation.
- Clear separation between:
  - UI rendering
  - State updates
  - Pricing engine
  - 3D variation mapping
- No hidden side effects.
- No implicit DOM-derived logic.

The coding agent must treat this as a structured system, not a collection of ad-hoc UI behaviours.