# AR Button Rollout Guide

Per-model guide for adding the "View in AR" button to each configurator index file.

---

## Prerequisites

1. **Plattar SDK** — The model's index file must already load `plattar-plugin.min.js` from the Plattar CDN.
2. **Scene ID** — Found at `plattar.sceneId` in the model's `MODEL_CONTENT_<NAME>.json`.
3. **AR enabled** — The Plattar scene must support AR (verify in Plattar CMS that the scene has an AR-compatible model).
4. **Embed element** — The `plattar-embed` element must be created with `id="plattar-viewer"` (this is the existing convention).

---

## Information to gather per model

| Item | Where to find it | Example (Staggered Harmony) |
|---|---|---|
| Scene ID | `MODEL_CONTENT_<NAME>.json` → `plattar.sceneId` | `79e51489-718b-49f2-876e-e85473685b8d` |
| Embed element ID | `createEmbedAndPoll()` in index file | `plattar-viewer` |
| AR icon URL | Provided by design team (custom SVG per model, or shared) | `https://static.wixstatic.com/shapes/15e926_3b31988db3e84945b80e28e69130536e.svg` |
| Close icon URL | Provided by design team (custom SVG, typically shared) | `https://static.wixstatic.com/shapes/15e926_28611c7621114ff1834d9b0fd588a0af.svg` |

---

## Steps to add AR to a new model

### Step 1: Add the CSS (if not already present)

Add the `.ar-button` CSS block to the embedded `<style>` in the model's index file. Place it after the media-overlay responsive rules and before the PHASE 5 block. This includes:

- Base `.ar-button` rule (absolute positioning top-right, 44x44px, circular, solid `#333333` background, AR SVG icon via `--ar-icon-url` CSS custom property). **Use `z-index: 100`** so the button stays above the Plattar launcher overlay.
- `.ar-button.launcher-active` rule — swaps `background-image` to `var(--ar-close-icon-url)` (close/X icon shown when AR launch page is open)
- Hover state (`#444444`) and focus-visible states
- `@media (max-width: 767px)` responsive override (38x38px, tighter positioning)

Two CSS custom properties are used: `--ar-icon-url` (the AR icon) and `--ar-close-icon-url` (the close icon). If all models share the same icons, the CSS is identical. If not, only the icon URLs in JS differ.

### Step 2: Add the HTML button

Inside the `.media-area` div, after the three `media-panel` divs and before the `.media-overlay` div, add:

```html
<button id="ar-button" class="ar-button" style="display:none;" aria-label="View in AR" title="View in AR"></button>
```

Place it as a direct child of `.media-area` (not inside `#panel-3d`) because `initPlattar()` clears panel-3d's innerHTML.

### Step 3: Add the JS functions

Add the following JS block after the `updateAddOnVariations()` function and before PHASE 6 (Interior Slider):

1. **`AR_ICON_URL`** variable — set to the AR SVG icon URL for this model
2. **`AR_CLOSE_ICON_URL`** variable — set to the close SVG icon URL
3. **IIFE to set CSS custom properties** — sets both `--ar-icon-url` and `--ar-close-icon-url` on `document.documentElement`
4. **`showARButton()` / `hideARButton()`** — toggle display on `#ar-button`
5. **`setARButtonIcon(isLauncher)`** — adds/removes `launcher-active` class on `#ar-button` to swap between the AR icon and close icon
6. **`toggleLaunchPage(forceLauncher)`** — switches `embed-type` between `"launcher"` and `"viewer"` on the embed element; calls `setARButtonIcon()` to update the icon
7. **`launchAR()`** — first checks if already in launcher mode (`embed-type === 'launcher'`) and returns to viewer (resetting icon) if so; otherwise checks `PlattarARAdapter.Util.canAugment()`, calls `embed.startAR()` if supported, falls back to `toggleLaunchPage(true)` otherwise. This makes the single AR button act as both open and close.
8. **Event listener** — `document.getElementById('ar-button').addEventListener('click', launchAR)`

### Step 4: Wire visibility

Two integration points in existing code:

1. **In `switchTab(tabId)`** — After updating tab/panel classes:
   - If `tabId === '3d'` and `plattarReady` is true: call `showARButton()`, reset embed-type to `"viewer"` if it was `"launcher"`, and call `setARButtonIcon(false)` to restore the AR icon
   - Otherwise: call `hideARButton()`

2. **In the `setLoading` callback** (inside `createEmbedAndPoll()`), after `plattarReady = true`:
   - Add: `if (state.activeTab === '3d') showARButton();`

### Step 5: Export (optional)

Add `launchAR` and `toggleLaunchPage` to the `window.__configurator` object for debugging access.

---

## Testing checklist

- [ ] **Desktop — QR code fallback:** Clicking the AR button switches the 3D viewer to the Plattar Launch Page (QR code). Clicking again returns to the 3D viewer.
- [ ] **Mobile — AR-capable device:** Clicking the AR button opens the AR experience directly. If it fails, it falls back to the Launch Page.
- [ ] **Mobile — non-AR device:** Clicking the AR button shows the Launch Page.
- [ ] **Tab switching:** AR button is visible only when on the 3D Exterior tab and Plattar has finished loading. Hidden on Interiors and Floorplan tabs.
- [ ] **Tab return:** Switching away from the 3D tab and back resets the embed from launcher mode to viewer mode.
- [ ] **Plattar not loaded:** If Plattar fails to load or times out, the AR button does not appear.
- [ ] **Responsive:** Button is 44x44px on desktop, 38x38px on mobile, positioned in the top-right corner of the media area.
- [ ] **Icon swap:** AR icon shows by default; when launcher/QR is open the button swaps to the close icon; closing returns to the AR icon.
- [ ] **Icon:** The correct AR icon SVG is displayed.
