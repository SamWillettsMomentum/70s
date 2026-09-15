# Master Implementation Plan: Staggered Harmony Configurator

**Primary entry point:** [AI_BRIEF.MD](AI_BRIEF.MD)  
**Authority:** SPEC.MD (architecture, layout, sidebar, tabs), Features_SPEC.MD (features, pricing, images), Pricing_STAGGERED_HARMONY.json (options), ID_REGISTRY_STAGGERED_HARMONY.json (3D IDs), VARIATION_MAPPING.MD (ID ↔ pricing alignment).

This plan breaks the project into phases and tasks in dependency order. Each task references the spec sections that define it. No implementation code is included; the plan is documentation only.

---

## 0. Embed Shell and Configuration Loading

**Goal:** Single HTML document that runs inside a Wix HTML Embed, loads configs, and exposes a scoped root for all markup.

| Task | Description | Spec / config reference |
|------|-------------|-------------------------|
| 0.1 | Create single HTML file with one root container (e.g. `.configurator-root`) wrapping all markup. Use internal `<style>` and `<script>`; no external hosting of app code. | SPEC §10 (Wix constraints, root container); AI_BRIEF §5 |
| 0.2 | Load Slick and Plattar from official CDNs via `<script>` tags. Handle Slick load failure: show fallback message so rest of app remains usable. | Features_SPEC §2 (Slick CDN, fallback); SPEC §10 |
| 0.3 | Embed or inline Pricing_STAGGERED_HARMONY.json so the app can read `product`, `options[]`, and choice structure. No duplicate definition of options. | AI_BRIEF §8 (Pricing as source of truth); Features_SPEC §7 |
| 0.4 | Embed or inline ID_REGISTRY_STAGGERED_HARMONY.json (or the subset needed for 3D) for scene/variation IDs. | AI_BRIEF §8; VARIATION_MAPPING.MD |
| 0.5 | Define INTERIOR_IMAGE_REGISTRY structure (living/base, living[variation_key], bathroom/base, bathroom[variation_key]) and populate from Images.json / Wix CDN URLs. Variation key format: Floor → Walls → Ceiling → Cabinets, segment `Category-Slug`, separator `_`. | Features_SPEC §15, §15.2; Images.json |
| 0.6 | Define VARIATION_OPTION_IMAGE_REGISTRY: map each choice id for options with `display` `box_with_image` (Exterior Cladding, Interior Lining, Flooring) to an image URL. Populate from Images.json `variationOptions`. Used by the sidebar’s image box in the variation selection component. | Images.json `variationOptions`; Pricing `display` |

**Exit criteria:** Embed runs in Wix; root is scoped; configs and registries (pricing, IDs, interior images, variation option images) are available in JS; CDN scripts load (or fail gracefully for Slick).

---

## 1. State and Core Layout

**Goal:** Single central state object and responsive two-area layout (Sidebar + Media Area) with correct breakpoints.

| Task | Description | Spec / config reference |
|------|-------------|-------------------------|
| 1.1 | Define central state object. State must hold: base price, selected option per category (keyed by Pricing choice `id` or category), and any fields needed for pricing and variation key. No feature keeps duplicate state. | AI_BRIEF §3, §9; Features_SPEC §11; SPEC §4 (State Management) |
| 1.2 | Initialize state from Pricing config: default selection per category (e.g. first choice per option) and base price from `product.basePrice`. | Features_SPEC §8, §12; Pricing_STAGGERED_HARMONY.json |
| 1.3 | Implement responsive layout: Desktop (≥1024px) two-column — Media Area left (flex), Sidebar right (~20%); Tablet/Mobile (≤1023px) single-column — Media Area top, Sidebar below. Use flexbox/grid, %, rem, flex; no 100vh; 100% width/height of embed container. | SPEC §2, §3 |
| 1.4 | Ensure root container uses 100% width/height and wraps all configurator markup for CSS scoping. | SPEC §3, §10 |

**Exit criteria:** State exists and is initialised from Pricing config; layout matches SPEC for desktop, tablet, mobile; no viewport assumptions.

---

## 2. Sidebar and Accordion

**Goal:** Sidebar built entirely from Pricing_STAGGERED_HARMONY.json; one accordion open at a time; three option types with correct behaviour.

| Task | Description | Spec / config reference |
|------|-------------|-------------------------|
| 2.1 | Render sidebar from `options[]`: one accordion section per category. Each section has clickable header (category label + “+” icon), collapsible content panel. Only one section open at a time; all default collapsed. “+” rotates 45° when open. Content panels stay in DOM (hidden via CSS). | SPEC §4, §5; Pricing_STAGGERED_HARMONY.json |
| 2.2 | Implement accordion behaviour: click header toggles section; opening one closes others. Keyboard: Enter/Space to toggle; visible focus styling. | SPEC §5 (Accessibility, Expand/Collapse) |
| 2.3 | For each category, render choices from `options[].choices[]` using `display`: **box_with_image** (image + name + price “+$XXX” or “+$0”; image from VARIATION_OPTION_IMAGE_REGISTRY[choice.id]), **box** (no image, name + price), **tick_box** (checkbox + label). Entire option clickable; hover and selected states; selected reflects state. Single-select per category unless type is boolean/tick. | SPEC §4, §6; Features_SPEC §7; Pricing `display`, `choices[].id`, `choices[].label`, `choices[].price`; VARIATION_OPTION_IMAGE_REGISTRY (task 0.6) |
| 2.4 | On option click: update central state for that category/choice, then trigger (a) pricing recalculation only if option is an add-on that affects price, (b) media update logic, (c) tab auto-switch if applicable. Re-render dependent UI from state. | SPEC §4 (Behaviour Rules); Features_SPEC §10, §7 (add-on only triggers pricing) |
| 2.5 | Identify add-on options from Pricing config (options with non-zero or non-null price that should trigger calculation). Only those invoke the pricing engine on change. | Features_SPEC §7, §8 |

**Exit criteria:** Sidebar and accordion match SPEC; options and prices come only from Pricing JSON; state updates and triggers are correct; no duplicate state.

---

## 3. Media Area Shell, Tabs, and Pricing Overlay

**Goal:** Media Area with three panels (3D, Interiors, Floorplan), pill tabs and pricing display overlay at bottom-centre, only one panel visible.

| Task | Description | Spec / config reference |
|------|-------------|-------------------------|
| 3.1 | Create Media Area container. Inside it, three panels (3D Exterior, Interiors, Floorplan). Only one visible at a time; switch via CSS class toggling (no DOM remove). Default visible: 3D Exterior. | SPEC §6, §8; Features_SPEC §12 |
| 3.2 | Implement pill-style tab bar: bottom-centre of Media Area, floating. Three tabs: 3D Exterior, Interiors, Floorplan. Click shows corresponding panel and updates active style. No anchor links; JS-only. Keyboard accessible. | SPEC §7 (Tab Navigation UI, Tab Behaviour) |
| 3.3 | Add pricing display box: bottom-centre, below tab bar with small gap, same horizontal alignment. Solid background div; content “From $[PRICE] incl. GST” (price updated by pricing engine) and “Go to Kit Guide” link. Distinct from per-option prices in sidebar. | SPEC §7 (Pricing Display Box); Features_SPEC §9 |
| 3.4 | Implement tab auto-switch: when user selects an option that affects exterior → show 3D Exterior tab; when option affects interiors → show Interiors tab; never auto-switch to Floorplan. | SPEC §7 (Default and Automatic Tab Switching); Features_SPEC §10 |

**Exit criteria:** Three panels and tab bar work; pricing overlay in place; default tab 3D Exterior; auto-switch rules by option type.

---

## 4. Pricing Engine and Display

**Goal:** Pricing calculated from state and config only; displayed in overlay; only add-on options trigger recalculation; whole numbers, round up.

| Task | Description | Spec / config reference |
|------|-------------|-------------------------|
| 4.1 | Implement pricing engine: input = central state + Pricing config. Start with base price; for each selected option that carries a price modifier (add-ons), add that modifier; apply any conditional logic from config; return total. Round up to whole numbers. Treat missing modifier as 0. | Features_SPEC §7, §8, §13 |
| 4.2 | Invoke pricing engine only when an add-on option changes (not on exterior/interior colour-only changes). On init, run once with default state. | Features_SPEC §7, §10, §12 |
| 4.3 | Update pricing display box with result: format “From $[PRICE] incl. GST”, no decimals; update only the price text so “Go to Kit Guide” link remains. No page reload. | Features_SPEC §9; SPEC §7 |

**Exit criteria:** Total matches config and state; only add-ons trigger recalc; display shows whole-number “From $X incl. GST”.

---

## 5. 3D Exterior Viewer (Scaffolding)

**Goal:** 3D panel with Plattar embed container and loading/error states; variation updates when Exterior Cladding selection changes (with throttling/debounce). Scene ID and variation IDs from ID Registry via VARIATION_MAPPING.

| Task | Description | Spec / config reference |
|------|-------------|-------------------------|
| 5.1 | Add 3D panel markup: container that fills Media Area, placeholder for Plattar embed (e.g. `<plattar-embed>` or div). Use scene ID from ID_REGISTRY (which scene to use is defined in registry or later spec). Show loading state until viewer is ready. | Features_SPEC §1, §12; ID_REGISTRY; Plattar Docs |
| 5.2 | On Plattar load failure: show user-friendly message in centre of Media Area in a solid-background box; do not break rest of UI. | Features_SPEC §13 |
| 5.3 | Build option-id → Plattar variation-id lookup for Exterior Cladding only (from ID_REGISTRY + VARIATION_MAPPING.MD §4). When state’s Exterior Cladding choice changes, resolve to variation UUID and call Plattar API to set variation. Throttle or debounce API calls. | VARIATION_MAPPING.MD §4, §7; Features_SPEC §1 (throttle/debounce) |
| 5.4 | Ensure 3D panel does not reload when user switches tabs; viewer stays in DOM. | Features_SPEC §1; SPEC §8 |

**Exit criteria:** 3D panel has embed and loading/error states; exterior choice drives variation via mapping; throttling in place; tab switch does not reinit 3D.

---

## 6. Interior Slider (Slick and Variation Images)

**Goal:** Two slides (Living Area, Bathroom), Slick initialised once; variation key from state; image from INTERIOR_IMAGE_REGISTRY; loading state until image loaded; no reinit on tab switch.

| Task | Description | Spec / config reference |
|------|-------------|-------------------------|
| 6.1 | Implement variation key builder: from central state, build string in order Floor → Walls → Ceiling → Cabinets, format `Category-Slug`, separator `_`. Slugs must align with Pricing choice ids (convention or mapping per Features_SPEC §15.2 and Images.json). | Features_SPEC §15.2, §15.3; Images.json; Pricing (Flooring, Interior Lining, etc.) |
| 6.2 | In Interiors panel, add two slides (living, bathroom). Each slide: one base image + one visible variation image (or loading state). DOM: multiple images per slide, visibility toggled or src swapped; no slider rebuild. | Features_SPEC §3, §15.1, Image Resolution Logic |
| 6.3 | Initialise Slick once (after DOM ready, not on tab switch). Config: arrows on desktop, hidden on mobile; dots disabled; swipe enabled. | Features_SPEC §2, §5, §12 |
| 6.4 | On interior-related option change: recompute variation key; for each slide lookup `INTERIOR_IMAGE_REGISTRY[slide_type][variation_key]`; if missing use `base`. Show loading state until target image has loaded, then show image (src or visibility). Do not change slide index; do not reinit Slick. | Features_SPEC §4, §15, Performance Rules (on-demand load) |
| 6.5 | Preload base images for both slides. Load variation images on demand only. | Features_SPEC §15 (Performance Rules and Image-Loading Strategy) |

**Exit criteria:** Two slides, Slick once, variation key from state, registry lookup O(1), fallback to base, loading state, no reinit.

---

## 7. Floorplan Viewer

**Goal:** Single static floorplan image in Floorplan panel; responsive, centred, aspect ratio preserved; no dynamic switching; tab only via user click.

| Task | Description | Spec / config reference |
|------|-------------|-------------------------|
| 7.1 | Add Floorplan panel with one image (URL from Images.json `floorPlan` / FLOORPLAN_IMAGE_URL). Responsive, centred, maintain aspect ratio. No slider, no variation logic. | Features_SPEC §6; SPEC §7 (no auto-switch to Floorplan); Images.json |

**Exit criteria:** Floorplan panel shows single image; no option drives floorplan content or auto-switch.

---

## 8. Cross-Feature Wiring and Option Classification

**Goal:** Correct tab auto-switch and pricing triggers per option category; which options are “exterior”, “interior”, or “add-on” derived from Pricing config.

| Task | Description | Spec / config reference |
|------|-------------|-------------------------|
| 8.1 | Classify options by category for behaviour: Exterior Cladding → state + 3D update + tab to 3D (pricing only if add-on). Interior Lining, Flooring → state + variation key + tab to Interiors (pricing only if add-on). Fireplace, Heatpump, Water Heating, Laundry Joinery, Bedroom Wardrobe, Solar → state + pricing if add-on; no tab auto-switch to Floorplan. | Features_SPEC §10; Pricing_STAGGERED_HARMONY.json |
| 8.2 | Ensure no option auto-switches to Floorplan tab. Floorplan only when user clicks Floorplan tab. | SPEC §7; Features_SPEC §10 |

**Exit criteria:** Tab and pricing behaviour match Features_SPEC §10 for each option type; Floorplan never auto-switched.

---

## 9. Wix Integration and Polish

**Goal:** postMessage to parent with price and selected options; loading/error UX; accessibility and code hygiene.

| Task | Description | Spec / config reference |
|------|-------------|-------------------------|
| 9.1 | Implement `window.parent.postMessage()` from embed: send final calculated price and all selected variation options (e.g. as structured payload). Document expected payload shape for Velo listener. Velo side: listener receives message and populates hidden form fields before submit. | SPEC §10; AI_BRIEF §5 (Parent–iframe communication) |
| 9.2 | Ensure loading states: 3D until viewer ready; interior slide until variation image loaded. Error messages: centre of Media Area, solid background box, friendly copy. | Features_SPEC §12, §13; SPEC (error placement) |
| 9.3 | Accessibility: keyboard for tabs and accordion; focus styles; semantic roles where appropriate; alt on images. | SPEC §5, §12; Features_SPEC (accessibility) |
| 9.4 | Keep embed code minimal (no known size limit; avoid redundancy, consider minification if needed). Comply with Wix CSP; use only permitted CDN URLs for Slick and Plattar. | SPEC §10; AI_BRIEF §5 |

**Exit criteria:** postMessage implemented and documented; loading/error UX per spec; a11y basics in place; code ready for paste into Wix embed.

---

## 10. Dependency Overview

```
0 (Shell, configs)
  → 1 (State, layout)
       → 2 (Sidebar, accordion)
       → 3 (Media Area, tabs, pricing overlay)
  → 4 (Pricing engine)  ← 2, 3
  → 5 (3D scaffolding)  ← 1, 3, VARIATION_MAPPING
  → 6 (Interior slider) ← 1, 3, 0.5 (registry), 2 (state)
  → 7 (Floorplan)      ← 3
  → 8 (Cross-feature)  ← 2, 3, 4, 5, 6, 7
  → 9 (Wix, polish)   ← all
```

---

## 11. Reference Checklist

Before implementation, ensure these are to hand:

- [ ] **AI_BRIEF.MD** — scope, constraints, success criteria, references
- [ ] **SPEC.MD** — layout, breakpoints, sidebar, accordion, tabs, pricing overlay, Wix, root container
- [ ] **Features_SPEC.MD** — 3D, Slick, floorplan, pricing engine, init order, variation key, INTERIOR_IMAGE_REGISTRY, VARIATION_OPTION_IMAGE_REGISTRY, FLOORPLAN_IMAGE_URL, error/loading
- [ ] **Pricing_STAGGERED_HARMONY.json** — options, categories, choices, prices, display types
- [ ] **ID_REGISTRY_STAGGERED_HARMONY.json** — scene ID(s), exterior product, variation UUIDs
- [ ] **VARIATION_MAPPING.MD** — option id → Plattar variation id (Exterior Cladding only)
- [ ] **Images.json** — variation key format, slide keys (living, bathroom); variationOptions (Exterior Cladding, Interior Lining, Flooring choice ids → image URLs); floorPlan (single URL for Floorplan tab)
- [ ] **Plattar Docs** — embed, scene, variation API (Installation, Loading_a_Scene, Working_with_Variations)
- [ ] **THIRD_PARTY_REFERENCE_SUMMARY.MD** / Example_Implementation_Euca — structural reference only, not copy-paste

Implementation must not override Decision Authority (AI_BRIEF §7). Resolve ambiguities via SPEC.MD and Features_SPEC.MD first.
