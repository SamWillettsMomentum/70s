# SPEC 1: General UI Architecture & Layout

## 1. Purpose

Define the overall UI system, layout structure, responsiveness rules, and interaction architecture for the configurator. This spec governs layout, responsiveness, sidebar behaviour, and the media area framework.

General rule:
- 3D renders are exterior only.
- Interior visuals are static images only.

---

## 2. High-Level Layout Structure

The UI consists of two primary areas:

1. Sidebar (configuration controls)
2. Media Area (visual display)

### Desktop Layout

- Two-column layout.
- Right: Accordion-based sidebar. Sidebar width approximately 20% of container (or similar proportion); use a reasonable fixed or max width so Media Area has adequate space.
- Left: Media Area (flexible width).
- Both sidebar and Media Area remain fixed vertically.
- Entire UI must scale to 100% width and 100% height of the Wix embed container.

### Tablet Layout (768px–1023px)

- Single-column layout (same as mobile).
- Media Area fixed at top; sidebar below scrolls inside its own container (page does not scroll; only sidebar content scrolls).
- Accordion-based sidebar for compact usability.
- Floating tab navigation and pricing display remain inside the Media Area as overlays.

### Mobile Layout

- Single-column layout.
- Media Area fixed at top; sidebar below scrolls inside its own container (page does not scroll; only sidebar content scrolls).
- Accordion-based sidebar for compact usability.
- Floating tab navigation and pricing display remain inside the Media Area as overlays.

---

## 3. Responsive Rules

- UI must be fully responsive within a Wix HTML Embed container.
- Root container (e.g. with class `.configurator-root`) must use 100% width and height and wrap all configurator markup and styles for scoping.
- Layout should use flexbox or grid (no fixed pixel positioning for major sections).
- No viewport height assumptions (avoid fixed 100vh where it could conflict with Wix container sizing).
- Use scalable units (%, rem, flex).

Breakpoints:
- Desktop: ≥1024px
- Tablet: 768px–1023px
- Mobile: ≤767px

---

## 4. Sidebar Architecture

### Structure

- Vertical navigation panel.
- **Single source of truth for sidebar options:** All configurable options shown in the sidebar accordion (categories, option lists, labels, display type, and per-option prices) are defined in that model’s **model content file** (e.g. Model Content Data/Staggered Harmony/MODEL_CONTENT_STAGGERED_HARMONY.json) under `pricing.options`. Each model’s index file (e.g. index_staggered_harmony.html) embeds this; the sidebar must be built from this configuration only; no other source may define which options appear or how they are grouped.
- Contains grouped configuration categories as defined by `pricing.sidebarGroups` in the model content file. Each group becomes one accordion section; categories within a group are rendered as sub-blocks with subheadings inside the accordion panel.
- Each accordion section contains:
  - An accordion header (group title + expand/collapse icon).
  - A collapsible content panel with one or more category sub-blocks.
- Each sub-block contains selectable variations (choices from the pricing configuration).
- Selectable variations support box UI types as specified per option in the configuration (e.g. box with image, box without image). Add-on options use two selectable boxes: "None" and the option (e.g. "Option included"); no tick box.
- **Fireplace:** The "Add Fireplace" option box displays "Requires consent" below the price within the box (medium grey, secondary note).
- **Flooring (Owner Supplied):** The choice with id `none_flooring` is labelled **Owner Supplied** (not "None"). Below the price it shows a tertiary note in the same style as the fireplace line (`.option-box-note`): *Renders use 'Premium Hush – Smooth Oak High Noon' flooring*.
- **Laundry Joinery** has two choices (No / Yes), rendered as box without image. Always shown regardless of model bedroom count. Selecting **Yes** opens a full-screen modal above the configurator: full-viewport overlay at **85% opacity** (`rgba(0,0,0,0.85)`); a centred **solid black** (`#000`) panel with **no border-radius**, **no inner padding** (the image fills the panel edge-to-edge). The close control is a **white X** in a square chip **`rgba(0,0,0,0.9)`** background, **positioned over the top-right of the image** with margin from the corner. Dismiss via **X**, **backdrop click**, or **Escape**; focus returns to the prior control. **Yes** opens the modal **every time** it is chosen, including **re-clicking Yes** while it is already selected; the modal does **not** open on initial page load. Selecting **No** closes the modal if it is open. The elevation image URL is a **per-model constant** in each `index_<model>.html` (Wix `static.wixstatic.com` asset).
- **Bedroom Wardrobe** choices are filtered by `IS_2_BEDROOM`: when true, show None / 1x / 2x; when false, show None / 1x only (omit the 2x choice).
- Sidebar has padding-bottom (e.g. 125px) below the accordion for clear spacing.

The sidebar must be fully responsive and adapt between desktop (two-column layout context) and mobile (stacked layout under Media Area).

---

### Behaviour Rules

- Only one accordion section may be open at a time.
- All accordion sections default to collapsed (desktop and mobile).
- Opening one section automatically closes any other open section.
- Selecting an option:
  - Updates the central state object.
  - Triggers pricing recalculation.
  - Triggers relevant media update logic.
  - Triggers media tab switching (if applicable).

Single-select groups:
- Selecting one option deselects others in the same group.

Multi-select groups (tick box style):
- Multiple variations may be active simultaneously.

All visual selected states must reflect the central state object immediately.

---

### State Management

- All configuration selections must be stored in a single central JS state object.
- No UI element may act as the authoritative source of truth.
- Media updates, slider variations logic, and pricing engine must read from this state.
- UI must re-render based on state, not DOM assumptions.

---

## 5. Accordion Component Specification

### Header Layout

Each accordion header must:

- Use a horizontal flex layout (space-between).
- Contain:
  - Left-aligned category heading text.
  - Right-aligned "+" icon.
- Entire header row must be clickable (not just the icon).

---

### Icon Behaviour

- Default state: "+" at 0° rotation.
- Open state: rotate 45° to visually resemble an "×".
- Rotation must use CSS transform.
- Smooth transition (150–250ms ease).
- Rotation state must be controlled via an "open" class on the parent accordion item.

---

### Expand / Collapse Behaviour

- Expanding a section:
  - Adds open state class.
  - Reveals content panel.
  - Rotates "+" icon 45°.
- Collapsing:
  - Removes open state class.
  - Hides content panel.
  - Resets icon rotation to 0°.
- Only one section may be open at a time.
- Toggling an already open section closes it (result: all collapsed allowed).

---

### Accessibility Requirements

- Accordion headers must be keyboard accessible.
- Support Enter and Space to toggle.
- Must include visible focus styling.
- Content panels must remain in DOM (hidden via CSS, not removed).

---

## 6. Accordion UI

###  Selectable Option Styles

All selectable variations must:

- Be fully clickable (entire container).
- Have a clear hover state.
- Have a clearly distinguishable selected/active state.
- Trigger state updates on click.
- Support keyboard focus styling.

---

## Option Type 1: Box with Image

### Structure

Clickable container containing:

- Left: Small square image.
- Right: Text container (vertical stack):
  - Option name (top line).
  - Price (bottom line) formatted as: "+$XXX".

### Layout Rules

- Horizontal flex layout.
- Image fixed square aspect ratio.
- Text left-aligned.
- Price always shown beneath the name.
- Price must include "+" prefix where applicable. For options that do not incur additional cost, display "+$0".

### Visual States

Unselected:
- Subtle border.
- Neutral background.
- Price in secondary text colour (medium grey).

Hover:
- Slight border emphasis.

Selected:
- Stronger border OR highlighted background.
- Clearly differentiated from hover.
- Must not rely on colour alone (include border or weight change).

---

## Option Type 2: Box without Image

### Structure

Identical to “Box with Image” except:

- No image element.
- Text aligns left.
- Content spans full width.

### Layout Rules

- Same padding and height rhythm as image box variant.
- Maintain consistent visual alignment between both box types.
- Option name on top.
- Price below in "+$XXX" format; use "+$0" for options that do not incur additional cost.

### States

- Same hover and selected logic as image box variant.
- Visual treatment must remain consistent for UI cohesion.

---

## Option Type 3: Custom Tick Box

### Structure

- Square tick element.
- Label text to the right.
- Entire row clickable.

### Visual Design

Unselected:
- Square with thin border.
- No fill.
- Transparent interior.

Selected:
- Solid filled square.
- Optional check indicator (if included, must contrast clearly).
- Fill colour must be distinct from border-only state.

### Behaviour

- Toggle selection state on click.
- Updates central state object.
- Triggers pricing recalculation and relevant media updates.

---

## 3. Interaction Rules for All Option Types

- Clicking an option updates central state.
- If single-select group: selecting one must deselect others.
- If multi-select group (tick boxes): allow multiple active.
- All selections must visually reflect state immediately.
- No duplicated state logic in DOM attributes.

---

## 4. Accessibility Requirements

- All selectable variations must:
  - Be keyboard accessible.
  - Have visible focus state.
  - Use semantic roles where appropriate.
- Accordion headers must support keyboard toggling.

---

## 6. Media Area Architecture

The Media Area contains:

1. 3D Exterior Viewer
2. Static Interior Slider
3. Floor Plan Viewer

These are controlled via a tab system.

---

## 7. Media Tab System and Pricing Display Overlay

### Tabs

Three tabs:
- 3D Exterior
- Interiors
- Floorplan

### Tab Navigation UI

- Pill-shaped tab system (pill-style buttons).
- Floating navigation element positioned at the bottom-centre of the Media Area.
- Tabs and the **pricing display box** stack vertically with a small gap: **tabs on top**, then gap, then **pricing display box** below. Both overlay the Media Area content; exact same horizontal positioning (bottom-centre), stacked.
- Horizontally aligned pill-style tab buttons.
- Active tab visually highlighted.
- Must overlay the content area without affecting layout flow.

### Pricing Display Box (overlay)

- The **pricing display** (the live-updating total, e.g. “From $[PRICE] incl. GST”) is a separate element from the per-option prices shown in the sidebar accordion.
- Includes a "Go to Kit Guide" link (e.g. below the price) that opens in a new tab; link target is configurable.
- Positioned bottom-centre of the Media Area, directly below the tab pills with a small gap.
- Rendered inside a div with a solid background so it remains readable over the media content.
- Floating overlay; does not affect layout flow of the Media Area.

### Tab Behaviour

- Clicking a tab:
  - Hides other media types.
  - Shows selected media panel.
  - Updates active state styling.

### Default and Automatic Tab Switching

- **Default tab on load:** 3D Exterior. This tab must be visible when the page first loads.
- **Automatic tab switching:** When a user selects an option in the sidebar:
  - If the option affects exterior → switch to 3D Exterior tab.
  - If the option affects interiors → switch to Interiors tab.
  - No option affects the floor plan image; the Floorplan tab is never auto-switched to. The user may open it only by clicking the Floorplan tab.

Tab switching must be handled via JS, not anchor links.

---

## 8. Media Panel Visibility Rules

Each media type exists in DOM simultaneously but:

- Only one is visible at a time.
- Others are hidden using CSS class toggling (not removed from DOM).
- Slick slider (interiors) must not be reinitialized on every tab switch — only initialized once.

---

## 9. Interaction Architecture

All interactions follow this pattern:

1. User selects option in sidebar.
2. Central state object updates.
3. Pricing recalculates.
4. Relevant media logic runs.
5. Media tab auto-switch (if required).
6. UI re-renders dependent elements.

No direct DOM-driven logic without referencing state.

---

## 10. Wix Integration Constraints

This UI will:

- Be built as static HTML, CSS, and JS.
- Be pasted directly into a Wix HTML Embed element.
- **No hosting of the configurator’s own code;** third-party scripts (Slick, Plattar) are loaded from their official CDNs only.
- All configurator markup and styles MUST be wrapped in a single root container (e.g. `.configurator-root`) so all selectors are scoped and do not conflict with Wix page CSS.

Requirements:

- All CSS must be internal to the embed (style tag).
- All JS must be internal to the embed (script tag).
- Slick slider must be loaded via CDN.
- **Parent–iframe communication (MVP):** The configurator must pass the final calculated price and all selected variation options from the HTML embed (iframe) to the native Wix form on the page. Implementation: `window.parent.postMessage()` from inside the embed, with a Wix Velo page-level listener that receives the message and populates hidden form fields programmatically before form submission.
- There is no known Wix embed size limit documented; code should be kept as short as possible (minimize redundancy, consider minification if needed) in case limits apply.
- Implementation MUST comply with Wix embed CSP and script policies; Slick and Plattar CDN URLs must be permitted by Wix.

---

## 11. Performance Requirements

- Avoid unnecessary DOM reflows.
- Initialize slider only once.
- No polling loops.
- Use event listeners rather than timeouts where possible.
- Keep JS modular and clearly structured.

---

## 12. Accessibility & UX Requirements

- Tabs must be keyboard accessible.
- Sidebar variations must have visible active states.
- Images must support alt attributes.
- Buttons must use semantic elements where possible.

---

## 13. Future-Proofing

Architecture must allow:

- Adding new sidebar categories.
- Adding new media types.
- Adding new pricing rules.
- Swapping 3D viewer provider if needed.

State-driven architecture is mandatory to support this.