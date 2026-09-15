# Third-Party Reference Summary (Euca Example)

High-level orientation to the Euca cabin configurator example. Use this instead of re-reading the full source file.

---

## Overview

- Single HTML file: configurator markup, inline scripts, and external script refs.
- Built for WordPress with Gravity Forms; Euca-specific (multi-size cabin product).
- **Use:** Structural and pattern inspiration only—not a source for direct copy-paste.

---

## Structure

**Layout**
- `.ec-configurator` → price container (sticky “From $X incl. GST”) → main section: AR buttons, preview slider (3 slides), then sidebar `.ec-options` with accordions and CTAs.
- Drawer for “site assessment”; modals for location, share, and CTA.

**3D**
- `<plattar-embed>` with `scene-id`, `embed-type="viewer"`; camera prev/next buttons.
- Scene ID set from model buttons (`data-scene-id`).

**Options**
- Accordion sections (Models, Exterior, Interior, Utilities).
- Option groups use buttons (e.g. `.layout-option`) with `.active`, `data-name`, `data-price`, `data-scene-id`.
- Many option containers are placeholders filled by JS (colors, exterior, interior, utilities).

---

## Architectural Patterns

**State**
- Not a single central object: `window.selectedOptions` (variation IDs) plus an inline closure with `currentSize`, `currentLayout`, `currentModel` and a `pricingData` matrix for “Models” pricing.
- Form fields (Configuration Link, Configured Options) updated by `updateFormField()`.

**Variation flow**
- Option click → update `selectedOptions` or size/layout/model → call `applyVariations()`, `updatePriceDisplay()`, `updateFormField()` (implementations live in external scripts).

**Pricing**
- Nested object keyed by size → layout → model; `updatePrices()` reads current selections and updates main price and per-option “From $X” labels.

**UI**
- Accordion open/close; Slick for preview tabs (no arrows, fade, no swipe); tab switches on accordion open (e.g. interior accordion → gallery slide); drawer/modal via class + `display`.

---

## Reusable Concepts (Useful Inspiration)

- Accordion grouping of options (Models, Exterior, Interior, Utilities).
- Tabbed media area: 3D | Interiors | Floorplan with pill-style nav.
- Sticky price display and “From $X” per option where relevant.
- Model selection driving scene via `data-scene-id` on buttons.
- Pattern: user choice → update state → apply variations → update price → update form.
- Drawer for final step (request proposal) instead of navigating away.

---

## Irrelevant / Do Not Copy

- WordPress, Gravity Forms, Euca URLs, theme paths.
- Multiple product sizes (Studio / 1 bed / 2 bed) and Euca-specific pricing matrix.
- Location modal, Google Places, AR/QR button implementation.
- Any form submission or backend flow; our integration is Wix + postMessage.
