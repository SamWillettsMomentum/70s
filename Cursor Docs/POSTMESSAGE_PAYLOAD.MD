# postMessage Payload — Configurator

The configurator sends the current price and selected options to the Wix parent page via `window.parent.postMessage()` so a Velo listener can populate hidden form fields before form submission. The message `type` is set from **MODEL_CONTENT.metadata.postMessageType** (e.g. `"configurator-config"`); the Wix listener must filter on that same value.

**Lightbox form population:** Wix **Form** blocks only expose the parent **form** element to Velo. Use **`$w('#form1').setFieldValues({ … })`** with each hidden field’s **field key** from the form builder — not `$w('#hiddenPrice').value` on inner inputs (those elements are not individually `$w`-selectable). Canonical IDs, field keys, and the summary-embed handshake live in [WIX_VELO_QUICK_SETUP.MD](WIX_VELO_QUICK_SETUP.MD) and [WIX_Codes.md](WIX_Codes.md). If the form and summary embed sit inside a Box/Stack, re-verify **Properties → ID** after layout changes (see that guide’s **Lightbox layout / Stack** note).

## Multi-page architecture

There are **5 configurator pages** on Wix — one per home model. Each page has its own HTML Embed containing that model's configurator code. All 5 pages share a **single Lightbox** (site-level) with a native form inside it.

This means:

- **Page code** (the Velo listener + "Get Quote" button handler) is written **per page** (5 copies). The code is identical across pages — the only value that may differ is the embed element ID (e.g. `#html1`).
- **Lightbox code** is written **once**. The lightbox receives data via `wixWindow.lightbox.getContext()` and doesn't know or care which model the data came from. The `product` field in the payload identifies the model (e.g. `"Staggered Harmony"`, `"Compact Elegance"`).
- All 5 configurators use the same `postMessageType` (`"configurator-config"`), which keeps the page code identical across pages.

| # | Model | Product name in payload | Page has own embed | Shares lightbox |
|---|-------|------------------------|--------------------|-----------------|
| 1 | Staggered Harmony | `"Staggered Harmony"` | Yes | Yes |
| 2 | Compact Harmony | `"Compact Harmony"` | Yes | Yes |
| 3 | Straight Elegance | `"Straight Elegance"` | Yes | Yes |
| 4 | Simplified Elegance | `"Simplified Elegance"` | Yes | Yes |
| 5 | Compact Elegance | `"Compact Elegance"` | Yes | Yes |

---

## When messages are sent

- Once after initial load (after state and pricing are initialized, in the `init()` function).
- After every option change (`handleOptionSelect` calls `sendConfigToWix()` at the end) so the parent always has the latest configuration.

## Payload shape

Each message is a single object:

| Field       | Type   | Description |
|------------|--------|--------------|
| `type`     | string | From MODEL_CONTENT.metadata.postMessageType. For Staggered Harmony this is `"configurator-config"`. Use this to filter messages from this configurator. |
| `product`  | string | Product name from pricing config (e.g. `"Staggered Harmony"`). |
| `price`    | number | Current total price (integer, from base price + selected option modifiers). |
| `currency` | string | Currency code (e.g. `"NZD"`). |
| `selections` | object | Map of category name → selected choice id. Keys are the category names from the pricing config. Values are the selected choice `id` string. |

### Categories in the selections object

The `selections` keys are the category names from that model's pricing config. All 5 models share the same set of categories, though default choice IDs and available choices differ per model. The table below shows Staggered Harmony as an example — the Wix/Lightbox code does not need to know the categories in advance because the formatter iterates over whatever keys the payload contains.

| Category (Staggered Harmony example) | Default choice id | Type |
|---------------------------------------|------------------|------|
| Exterior Cladding | `corrugated_iron_grey_friars` | single_select |
| Interior Lining | `white_gib_redline_ply_lounge_ceiling` | single_select |
| Flooring | `smooth_oak_high_noon` | single_select |
| Cabinets | `cabinets_bullet` | single_select |
| Fireplace | `none_fireplace` | single_select |
| Heatpump | `none_heatpump` | single_select |
| Water Heating | `no_water_heating` | single_select |
| Laundry Joinery | `none_laundry` | single_select |
| Bedroom Wardrobe | `none_bedroom_wardrobe` | single_select |
| Solar | `none_solar` | single_select |

### Example payload (Staggered Harmony defaults)

```json
{
  "type": "configurator-config",
  "product": "Staggered Harmony",
  "price": 183900,
  "currency": "NZD",
  "selections": {
    "Exterior Cladding": "corrugated_iron_grey_friars",
    "Interior Lining": "white_gib_redline_ply_lounge_ceiling",
    "Flooring": "smooth_oak_high_noon",
    "Cabinets": "cabinets_bullet",
    "Fireplace": "none_fireplace",
    "Heatpump": "none_heatpump",
    "Water Heating": "no_water_heating",
    "Laundry Joinery": "none_laundry",
    "Bedroom Wardrobe": "none_bedroom_wardrobe",
    "Solar": "none_solar"
  }
}
```

### Example payload (with add-ons selected)

```json
{
  "type": "configurator-config",
  "product": "Staggered Harmony",
  "price": 200700,
  "currency": "NZD",
  "selections": {
    "Exterior Cladding": "seemlok_printed_corten_steel_corrugated_iron_ebony",
    "Interior Lining": "redline_ply_everywhere",
    "Flooring": "turin_oak_slacked_oyster",
    "Cabinets": "cabinets_green_slate",
    "Fireplace": "fireplace_added",
    "Heatpump": "heatpump_added",
    "Water Heating": "electric_cylinder",
    "Laundry Joinery": "1_laundry_joinery",
    "Bedroom Wardrobe": "bedroom_wardrobe_1x",
    "Solar": "none_solar"
  }
}
```

---

## Wix Velo implementation (receiving the payload)

This section covers the Velo (JavaScript) code you need to write on the Wix page that contains the HTML embed. All code goes in the **page code** panel in Wix Studio (the code editor at the bottom of the page editor).

### Prerequisites

- The HTML Embed element containing the configurator code must be on the page. Note its element ID (e.g. `#html1`). You can find/change this in the Properties panel when the embed is selected.
- The shared Lightbox must exist with a native form inside it, containing hidden input fields for the configurator data (see Per-model production installation § 4).
- Each configurator page needs a "Get Quote" button that opens the Lightbox.
- Enable Velo: In Wix Studio, toggle on Dev Mode (the `{ }` icon in the top bar) to access the code editor.

### Step 1: Listen for postMessage from the embed

The HTML Embed element in Wix fires an `onMessage` event when the iframe sends a `postMessage`. You attach a listener to the embed element itself — not `window`.

In your **page code**:

```javascript
$w.onReady(function () {

  // Listen for messages from the configurator embed
  $w('#html1').onMessage((event) => {
    const data = event.data;

    // Filter: only handle messages from our configurator
    if (!data || data.type !== 'configurator-config') return;

    handleConfiguratorMessage(data);
  });

});
```

Replace `#html1` with whatever your embed element's ID is.

### Step 2: Store the latest config data

Keep a module-scoped variable so the data is available when the user opens the quote form:

```javascript
let latestConfig = null;

function handleConfiguratorMessage(data) {
  latestConfig = {
    product: data.product || '',
    price: data.price || 0,
    currency: data.currency || 'NZD',
    selections: data.selections || {}
  };
}
```

### Step 3: Format selections for the form field

The `selections` object needs to be turned into a human-readable string for the form submission (since hidden fields are plain text). Create a formatter:

```javascript
const SELECTION_LABEL_OVERRIDES = {
  none_fireplace: 'No Fireplace',
  none_flooring: 'Owner Supplied Flooring',
  none_heatpump: 'No Heatpump',
  none_laundry: 'No Laundry Joinery',
  none_bedroom_wardrobe: 'No Bedroom Wardrobe',
  none_solar: 'No Solar'
};

function formatSelectionsForForm(selections) {
  if (!selections) return '';
  return Object.keys(selections).map((category) => {
    const value = selections[category];
    // Prefer explicit customer-facing labels for known ids.
    const label = value
      ? (SELECTION_LABEL_OVERRIDES[value] || value.replace(/_/g, ' ').replace(/\b\w/g, (c) => c.toUpperCase()))
      : 'None';
    return category + ': ' + label;
  }).join('\n');
}

function formatPriceForForm(price, currency) {
  return 'From $' + Math.ceil(price || 0).toLocaleString() + ' incl. GST (' + (currency || 'NZD') + ')';
}
```

### Step 4: Pass data to the shared Lightbox on "Get Quote"

When the user clicks "Get Quote", open the shared Lightbox and pass the config data as context. The Lightbox receives data via `wixWindow.lightbox.getContext()` — it doesn't need to know which model page it was opened from because the `product` field identifies the model.

**Page code** (same on every configurator page):

```javascript
import wixWindow from 'wix-window';

$w('#button10').onClick(() => {
  const formData = {
    product: latestConfig ? latestConfig.product : '',
    price: latestConfig ? formatPriceForForm(latestConfig.price, latestConfig.currency) : '',
    selections: latestConfig ? formatSelectionsForForm(latestConfig.selections) : ''
  };

  wixWindow.openLightbox('Get Quote', formData);
});
```

**Lightbox code** (written once — the Lightbox is a site-level element shared by all 5 pages). Use the **form element** id (e.g. `form1`) and **field keys** from the form builder — see [WIX_Codes.md](WIX_Codes.md).

```javascript
import wixWindow from 'wix-window';

const QUOTE_FORM_ID = 'form1';
const FIELD_PRODUCT = 'hidden_product';
const FIELD_PRICE = 'hidden_price';
const FIELD_SELECTIONS = 'hidden_selections';

$w.onReady(function () {
  const data = wixWindow.lightbox.getContext();

  if (data) {
    $w('#' + QUOTE_FORM_ID).setFieldValues({
      [FIELD_PRODUCT]: data.product || '',
      [FIELD_PRICE]: data.price || '',
      [FIELD_SELECTIONS]: data.selections || ''
    });
  }
});
```

### Step 5: Ensure hidden fields submit with the form

In the Wix Form settings:
- Each hidden field must be a **Text Input** (or equivalent) with **hidden** styling as needed. Inner inputs are **not** set with `$w('#someInnerId').value` in Velo for native Form blocks — use **`setFieldValues`** on the form with the correct **field keys** (`hidden_product`, `hidden_price`, `hidden_selections` unless you rename them in the builder).
- Confirm each field key in the form field settings matches the constants in your lightbox script ([WIX_Codes.md](WIX_Codes.md)).
- The field must have a **field connection** to the form's dataset/collection, or it will not be included in the submission. Connect it via the CMS (Content Manager) or ensure the form's `onWixFormSubmit` event captures these values.

### Complete page code example (identical on all 5 pages)

```javascript
import wixWindow from 'wix-window';

let latestConfig = null;

function handleConfiguratorMessage(data) {
  latestConfig = {
    product: data.product || '',
    price: data.price || 0,
    currency: data.currency || 'NZD',
    selections: data.selections || {}
  };
}

const SELECTION_LABEL_OVERRIDES = {
  none_fireplace: 'No Fireplace',
  none_flooring: 'Owner Supplied Flooring',
  none_heatpump: 'No Heatpump',
  none_laundry: 'No Laundry Joinery',
  none_bedroom_wardrobe: 'No Bedroom Wardrobe',
  none_solar: 'No Solar'
};

function formatSelectionsForForm(selections) {
  if (!selections) return '';
  return Object.keys(selections).map((category) => {
    const value = selections[category];
    const label = value
      ? (SELECTION_LABEL_OVERRIDES[value] || value.replace(/_/g, ' ').replace(/\b\w/g, (c) => c.toUpperCase()))
      : 'None';
    return category + ': ' + label;
  }).join('\n');
}

function formatPriceForForm(price, currency) {
  return 'From $' + Math.ceil(price || 0).toLocaleString() + ' incl. GST (' + (currency || 'NZD') + ')';
}

$w.onReady(function () {

  // Listen for configurator postMessage
  $w('#html1').onMessage((event) => {
    const data = event.data;
    if (!data || data.type !== 'configurator-config') return;
    handleConfiguratorMessage(data);
  });

  // "Get Quote" button opens lightbox with config data
  $w('#button10').onClick(() => {
    const formData = {
      product: latestConfig ? latestConfig.product : '',
      price: latestConfig ? formatPriceForForm(latestConfig.price, latestConfig.currency) : '',
      selections: latestConfig ? formatSelectionsForForm(latestConfig.selections) : ''
    };
    wixWindow.openLightbox('Get Quote', formData);
  });

});
```

---

## Per-model production installation

Each of the 5 model pages follows the same setup. The only thing that differs per page is the HTML Embed contents (each model's index file) and potentially the embed's element ID.

### 1. HTML Embed setup (per page)

On each model's Wix page, add an **HTML Embed** element. Paste the contents of that model's index file into it.

| Model | Index file to paste into embed |
|---|---|
| Staggered Harmony | `index_staggered_harmony.html` |
| Compact Harmony | `index_compact_harmony.html` |
| Straight Elegance | `index_straight_elegance.html` |
| Simplified Elegance | `index_simplified_elegance.html` |
| Compact Elegance | `index_compact_elegance.html` |

Note each embed's element ID (e.g. `#html1`) — you reference it in that page's Velo code.

### 2. postMessage type (all models)

All 5 configurators use the same type:

```
postMessageType: "configurator-config"
```

This is set in `MODEL_CONTENT.metadata.postMessageType` inside each embed. Because each page has only one embed, there is no risk of message collision — the same type string keeps the page code identical across all 5 pages.

### 3. Page Velo code (per page)

Copy the complete page code example (see above) onto each of the 5 pages. The only value you may need to change per page is the embed element ID (`#html1` etc.) to match the actual ID Wix assigns to that page's embed.

### 4. Hidden form fields (in the shared Lightbox — set up once)

The Lightbox form needs these hidden text input fields. Create them once — they serve all 5 models. The `product` field value identifies which model the submission came from.

Velo fills them via **`$w('#form1').setFieldValues({ … })`** using **field keys** (set in each field’s form settings), not Velo ids on inner inputs:

| Field purpose | Field key (form builder) | Example content |
|---|---|---|
| Product name | `hidden_product` | `"Staggered Harmony"` / `"Compact Elegance"` / etc. |
| Calculated price | `hidden_price` | `"From $183,900 incl. GST (NZD)"` |
| Selected options | `hidden_selections` | Multi-line text of all selections |

### 5. Lightbox Velo code (written once)

The Lightbox is a site-level element in Wix. Its code is written **once** and applies every time the Lightbox is opened, regardless of which page opened it. It uses `wixWindow.lightbox.getContext()` to receive the data passed by the calling page, then **`setFieldValues`** on the form. Copy-ready snippets (basic lightbox and optional summary + retries + handshake) are in [WIX_VELO_QUICK_SETUP.MD](WIX_VELO_QUICK_SETUP.MD) — **Code for Step 3** and **Code for Step 5**.

### 6. Element ID checklist

**Per page** — confirm these match your actual elements:

| Element | Code reference | What it is |
|---|---|---|
| HTML Embed | `$w('#html1')` | The embed containing this model's configurator (ID may differ per page) |
| Get Quote button | `$w('#button10')` | Button that opens the Lightbox |

**In the shared Lightbox** — confirm once:

| Element | Code reference | What it is |
|---|---|---|
| Lightbox name | `'Get Quote'` | The name of the Lightbox (exact string match in `wixWindow.openLightbox()`) |
| Form block | `$w('#form1')` | Parent form; use `.setFieldValues` with field keys above |
| Hidden fields | *(via form)* | Set by keys `hidden_product`, `hidden_price`, `hidden_selections` — not individual `$w` selectors |
| Summary embed (optional) | `$w('#htmlSummary')` recommended | HTML Embed in the Lightbox for the visual summary ([WIX_Codes.md](WIX_Codes.md)) |

---

## Displaying selected options in the Lightbox (HTML Embed)

An HTML Embed block placed alongside the form in the shared Lightbox can show the user a plain-text summary of their configuration and price. Since the Lightbox is site-level, this summary automatically works for all 5 models — the `product` field shows which model was configured, and the `selections` field lists whatever categories that model has.

The Lightbox page code sends the data into the embed via **`$w('#htmlSummary').postMessage(...)`** (or your summary embed id); the embed listens, formats it, and renders it. Because Wix HTML embeds are iframes, they cannot inherit page styles — all styling lives inside the embed itself.

### Complete Lightbox code (with summary embed)

Use **`setFieldValues`** on the form, then push a typed payload into the summary iframe. For **retries** and the **`quote-summary-ready`** handshake (recommended when the embed sits inside a Stack or loads slowly), use the full **Code for Step 5** block in [WIX_VELO_QUICK_SETUP.MD](WIX_VELO_QUICK_SETUP.MD) — do not duplicate it here.

Minimal pattern (summary only; embed HTML must expect `type: 'quote-summary'` if you use this shape):

```javascript
import wixWindow from 'wix-window';

const QUOTE_FORM_ID = 'form1';
const FIELD_PRODUCT = 'hidden_product';
const FIELD_PRICE = 'hidden_price';
const FIELD_SELECTIONS = 'hidden_selections';
const SUMMARY_EMBED_ID = 'htmlSummary';

$w.onReady(function () {
  const data = wixWindow.lightbox.getContext();

  if (data) {
    $w('#' + QUOTE_FORM_ID).setFieldValues({
      [FIELD_PRODUCT]: data.product || '',
      [FIELD_PRICE]: data.price || '',
      [FIELD_SELECTIONS]: data.selections || ''
    });

    $w('#' + SUMMARY_EMBED_ID).postMessage({
      type: 'quote-summary',
      product: data.product || '',
      price: data.price || '',
      selections: data.selections || ''
    });
  }
});
```

Replace `SUMMARY_EMBED_ID` with your summary embed’s **Properties → ID**.

### HTML Embed code (paste into the embed)

The maintained snippet (including `quote-summary` handling, `addEventListener('message', …)`, and `quote-summary-ready`) is **Part 2 → Code for Step 5 → B** in [WIX_VELO_QUICK_SETUP.MD](WIX_VELO_QUICK_SETUP.MD). It renders a two-column list (category / value) plus product and price; adjust the `<style>` block to match your site.

---

## Reference

- **SPEC.MD** §10 — Parent–iframe communication.
- **AI_BRIEF.MD** §5 — Technical constraints, postMessage requirement.
- **AR_ROLLOUT_GUIDE.md** — For AR button rollout per model (does not affect postMessage).
