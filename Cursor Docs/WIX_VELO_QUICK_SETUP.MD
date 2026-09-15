# Wix Velo quick setup — 5 configurators + 1 shared lightbox

Use this file in two passes: read **Part 1** for what to do in the Wix UI, then jump to **Part 2** for the matching **Code for Step N** block. Step numbers line up (Step 2 ↔ Code for Step 2).

**Canonical Wix IDs for this site** are listed in [WIX_Codes.md](WIX_Codes.md). The embed and Get Quote button use the **same** element IDs on every configurator page; the code below is already filled in to match. If you rename anything in Wix, update `WIX_Codes.md` and the constants in Part 2 together.

Payload shape and architecture details live in [POSTMESSAGE_PAYLOAD.MD](POSTMESSAGE_PAYLOAD.MD).

---

# Part 1 — What to do in Wix (instructions only)

## Step 1 — Turn on Dev Mode (IDs already fixed for this site)

1. Open your site in **Wix Studio** (or the editor you use with Velo).
2. Turn on **Dev Mode**: click the **`{ }`** icon in the top bar. The code panel opens (usually along the bottom).

Your **element IDs are already chosen** and recorded in [WIX_Codes.md](WIX_Codes.md):

- Every configurator page uses the same **HTML Embed** id (`html1`) and the same **Get Quote** button id (`button10`).
- The lightbox opens by **popup name** `'Get Quote'` (not by the lightbox component id `lightbox1` in code — that id is for reference in the editor only).
- The quote **form** is **Get Quote Form**; in Velo you only select the **form block** (e.g. `#form1`). Hidden inputs inside it are **not** individually addressable with `$w` — you fill them with **`$w('#form1').setFieldValues({ … })`** using each field’s **field key** (`hidden_product`, `hidden_price`, `hidden_selections`).

**Tip:** If something in Wix stops matching the doc (after a duplicate or redesign), select the form → **Properties → ID** for the form element, and confirm field keys in the form field settings. Update [WIX_Codes.md](WIX_Codes.md) plus the constants in **Part 2** in one go.

### Lightbox layout / Stack (troubleshooting)

If the **Get Quote** form and the **summary HTML embed** sit inside a **Box** or **Stack** (e.g. `#box155` wrapping `#box154`), Velo does **not** need to reference those containers for the standard flow. If the summary stuck on “Loading configuration…” or hidden fields stay empty after a redesign, check in order:

1. **Element IDs** — Moving or rebuilding the layout often changes the form or embed ID (`form2`, `html2`, etc.). Re-read **Properties → ID** on the lightbox form and summary embed; update `QUOTE_FORM_ID`, `SUMMARY_EMBED_ID`, and [WIX_Codes.md](WIX_Codes.md).
2. **Race to the summary iframe** — `$w.onReady` may run `postMessage` before the embed’s `message` listener exists. The **Step 5** script uses **staggered retries** (including longer delays for real mobile devices), a short **repeat `postMessage` interval** for the first few seconds, and a **`quote-summary-ready` handshake** from the embed (immediate + on **`window` `load`**) so the parent can resend after the iframe is listening.
3. **Field keys** — If the form was recreated, confirm `hidden_product`, `hidden_price`, `hidden_selections` still match the form builder (see [WIX_Codes.md](WIX_Codes.md)).

---

## Step 2 — Page code on each of the five configurator pages

1. In the editor, open **one** configurator page (e.g. Staggered Harmony).
2. Make sure **Dev Mode** is on and the **code panel** is visible.
3. In the code panel, ensure you’re editing **this page’s** script (header shows the page name — not Global, not the lightbox).
4. **Replace** everything already in that page’s code file with the **entire** block from **Part 2 → Code for Step 2** (do not nest it inside Wix’s default `$w.onReady`). The file must start with `import wixWindow …` at the **top** (imports cannot go inside `onReady`). Our snippet already includes one `$w.onReady(function () { … })` at the end for the listeners.
5. Save.
6. **Repeat** for the other four pages. Use the **identical** script each time — embed `#html1` and button `#button10` are the same on every page.

**Tip:** You can keep one copy of the page code in a text file and paste the same block onto all five pages to avoid drift.

---

## Step 3 — Lightbox code (once, on the lightbox only)

1. Open the **lightbox** in the editor (not a normal site page).
2. Turn on **Dev Mode** if it isn’t already.
3. In the code panel, switch to **this lightbox’s** code file (it should not say a configurator page name).
4. **Replace** the lightbox file’s contents with the script from **Part 2 → Code for Step 3** (pick the **basic** block unless you’re doing Step 5). Same rule as page code: `import` first, then `$w.onReady` — do not wrap the paste inside an extra `onReady` shell.
5. The script calls **`setFieldValues`** on the **form** element (`#form1` by default). Keys in the object must match your hidden fields’ **field keys** in the form builder ([WIX_Codes.md](WIX_Codes.md)). If your form’s Velo id is not `form1`, change `QUOTE_FORM_ID` in the script.
6. Save.

**Tip:** Lightbox code runs **once** for the whole site. You do **not** paste this on the five model pages.

---

## Step 4 — Form: hidden fields actually submit

1. In the lightbox, select the **Get Quote Form** (see [WIX_Codes.md](WIX_Codes.md)).
2. Confirm each hidden input is **part of the form** and **connected** so submissions include them (Wix may require connecting inputs to the form dataset / submission settings — follow your form’s setup).
3. Publish or use **Preview**, open a configurator, change options, click **Get Quote**, submit the form, and confirm the submission contains product, price, and selections text.

**Tip:** If hidden values are empty on submit, the lightbox may not have received context (`openLightbox` name / button), or **`setFieldValues` keys** may not match the form field keys in Wix (they must match exactly, not the inner elements’ Velo ids).

**Field keys checklist:** In the form builder, open each hidden field and confirm **field key** is exactly `hidden_product`, `hidden_price`, and `hidden_selections` ([WIX_Codes.md](WIX_Codes.md)). If any key was renamed when the form was recreated, update the constants in the lightbox script to match.

---

## Step 5 — (Optional) Summary HTML embed inside the lightbox

1. In the lightbox, add a second **HTML Embed** for the on-screen summary (if you want a styled recap next to the form).
2. Set its Velo ID to **`htmlSummary`** (recommended; avoids confusion with the configurator **`html1`** on model pages). Document it in [WIX_Codes.md](WIX_Codes.md).
3. Use **Part 2 → Code for Step 5**: replace the lightbox script with the **extended** version and paste the **HTML** into that embed. Set `SUMMARY_EMBED_ID` to match your embed’s ID.

---

# Part 2 — Code and edits (reference)

## Code for Step 1 — ID checklist (nothing to paste)

Source of truth: [WIX_Codes.md](WIX_Codes.md).

| Role | Value | Used in code as |
|------|--------|-----------------|
| Configurator HTML Embed (all pages) | `html1` | `$w('#html1')` |
| Get Quote button (all pages) | `button10` | `$w('#button10')` |
| Open lightbox by name | `Get Quote` | `openLightbox('Get Quote', …)` |
| Lightbox component (editor ref.) | `lightbox1` | Not used in `openLightbox`; name above is what Velo needs |
| Form display name | Get Quote Form | For finding the form in the editor |
| Form element (Velo) | `form1` | `$w('#form1').setFieldValues({ … })` |
| Hidden field keys (inside form) | `hidden_product`, `hidden_price`, `hidden_selections` | Keys in the object passed to `setFieldValues` — not `$w`-selectable individually |
| (Optional) Summary embed (lightbox only) | `htmlSummary` (recommended) | `$w('#htmlSummary')` — not the same element as page `html1` |

---

## Code for Step 2 — Full page Velo (paste on each of the 5 pages)

**Same file on every page.** Constants match [WIX_Codes.md](WIX_Codes.md). Only change them if you rename elements in Wix.

```javascript
import wixWindow from 'wix-window';

// --- Matches WIX_Codes.md (all 5 pages use the same embed + button ids) ---
const CONFIGURATOR_EMBED_ID = 'html1';
const GET_QUOTE_BUTTON_ID = 'button10';
const LIGHTBOX_NAME = 'Get Quote';
// ----------------------------------------------------------------------------

const configuratorEmbed = '#' + CONFIGURATOR_EMBED_ID;
const getQuoteButton = '#' + GET_QUOTE_BUTTON_ID;

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

  $w(configuratorEmbed).onMessage((event) => {
    const data = event.data;
    if (!data || data.type !== 'configurator-config') return;
    handleConfiguratorMessage(data);
  });

  $w(getQuoteButton).onClick(() => {
    const formData = {
      product: latestConfig ? latestConfig.product : '',
      price: latestConfig ? formatPriceForForm(latestConfig.price, latestConfig.currency) : '',
      selections: latestConfig ? formatSelectionsForForm(latestConfig.selections) : ''
    };
    wixWindow.openLightbox(LIGHTBOX_NAME, formData);
  });

});
```

**Note:** The configurator iframe must send `type: 'configurator-config'`. All current model index files use that value from `MODEL_CONTENT.metadata.postMessageType`.

---

## Code for Step 3 — Lightbox Velo (paste once on the lightbox)

Wix **Form** blocks only expose the parent form to Velo. Use **`setFieldValues`** on the form; object keys are the hidden fields’ **field keys** from the form settings ([WIX_Codes.md](WIX_Codes.md)). If your form’s **Properties → ID** is not `form1`, change `QUOTE_FORM_ID`.

```javascript
import wixWindow from 'wix-window';

// --- Matches WIX_Codes.md: form element id in the lightbox ---
const QUOTE_FORM_ID = 'form1';
// Field keys must match the form builder (not inner element ids)
const FIELD_PRODUCT = 'hidden_product';
const FIELD_PRICE = 'hidden_price';
const FIELD_SELECTIONS = 'hidden_selections';
// -------------------------------------------------------------

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

---

## Code for Step 4 — No script

Nothing to paste. Complete the form connections and test a submission (see Part 1 — Step 4).

---

## Code for Step 5 — Optional summary embed

**A) Replace** the lightbox `$w.onReady` block **or** the whole Step 3 script with this. Form + field keys match [WIX_Codes.md](WIX_Codes.md). Set `SUMMARY_EMBED_ID` to your summary HTML embed’s Velo id (`htmlSummary` recommended).

Payloads sent **into** the summary iframe use `type: 'quote-summary'` so the embed can ignore unrelated `message` events.

**Mobile / slow iframes:** Part **B** spreads **`postMessage` retries** over several seconds and runs a brief **`setInterval`** so delivery still happens if the embed mounts late (common on real phones vs desktop). Part **C** registers the **`message` listener first**, then sends **`quote-summary-ready` immediately and again on `load`** so Velo can respond after the iframe is fully up.

```javascript
import wixWindow from 'wix-window';

const QUOTE_FORM_ID = 'form1';
const FIELD_PRODUCT = 'hidden_product';
const FIELD_PRICE = 'hidden_price';
const FIELD_SELECTIONS = 'hidden_selections';
const SUMMARY_EMBED_ID = 'htmlSummary';

// One-off delays (ms) then interval below — tune if needed for very slow devices
const SUMMARY_RETRY_DELAYS_MS = [0, 50, 150, 400, 800, 1500, 2500, 4000];
const SUMMARY_INTERVAL_MS = 600;
const SUMMARY_INTERVAL_TICKS = 9; // ~5.4s of extra pushes after lightbox opens

let summaryPayload = null;
let summaryDeliveryIntervalId = null;

function buildSummaryMessage(ctx) {
  return {
    type: 'quote-summary',
    product: ctx.product || '',
    price: ctx.price || '',
    selections: ctx.selections || ''
  };
}

function pushSummaryToEmbed() {
  if (!summaryPayload) return;
  $w('#' + SUMMARY_EMBED_ID).postMessage(summaryPayload);
}

function scheduleSummaryDelivery() {
  if (summaryDeliveryIntervalId) {
    clearInterval(summaryDeliveryIntervalId);
    summaryDeliveryIntervalId = null;
  }
  SUMMARY_RETRY_DELAYS_MS.forEach(function (ms) {
    setTimeout(pushSummaryToEmbed, ms);
  });
  var n = 0;
  summaryDeliveryIntervalId = setInterval(function () {
    pushSummaryToEmbed();
    n += 1;
    if (n >= SUMMARY_INTERVAL_TICKS) {
      clearInterval(summaryDeliveryIntervalId);
      summaryDeliveryIntervalId = null;
    }
  }, SUMMARY_INTERVAL_MS);
}

$w.onReady(function () {
  summaryPayload = null;
  if (summaryDeliveryIntervalId) {
    clearInterval(summaryDeliveryIntervalId);
    summaryDeliveryIntervalId = null;
  }

  const data = wixWindow.lightbox.getContext();

  $w('#' + SUMMARY_EMBED_ID).onMessage((event) => {
    const msg = event.data;
    if (msg && msg.type === 'quote-summary-ready') {
      pushSummaryToEmbed();
    }
  });

  if (data) {
    summaryPayload = buildSummaryMessage(data);

    $w('#' + QUOTE_FORM_ID).setFieldValues({
      [FIELD_PRODUCT]: data.product || '',
      [FIELD_PRICE]: data.price || '',
      [FIELD_SELECTIONS]: data.selections || ''
    });

    scheduleSummaryDelivery();
  }
});
```

**B) Paste** into the **HTML Embed** used for the summary (Code mode / HTML):

```html
<html>
<head>
<style>
  * { margin: 0; padding: 0; box-sizing: border-box; }
  body {
    font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
    color: #2a2a2a;
    padding: 20px 24px;
    line-height: 1.5;
  }
  .summary-title { font-size: 15px; font-weight: 600; margin-bottom: 4px; }
  .summary-price { font-size: 14px; font-weight: 600; color: #1a1a1a; margin-bottom: 16px; }
  .summary-list { list-style: none; padding: 0; }
  .summary-list li {
    font-size: 13px;
    padding: 4px 0;
    border-bottom: 1px solid #e8e8e8;
    display: flex;
    justify-content: space-between;
    gap: 12px;
  }
  .summary-list li:last-child { border-bottom: none; }
  .cat-name { color: #666; flex-shrink: 0; }
  .cat-value { font-weight: 500; text-align: right; }
  .placeholder { color: #999; font-size: 13px; }
</style>
</head>
<body>
  <div id="content"><span class="placeholder">Loading configuration…</span></div>
  <script>
    (function () {
      function renderSummary(data) {
        if (!data || data.type !== 'quote-summary' || data.selections === undefined) return;
        var html = '';
        html += '<div class="summary-title">' + (data.product || '') + '</div>';
        html += '<div class="summary-price">' + (data.price || '') + '</div>';
        html += '<ul class="summary-list">';
        var lines = (data.selections || '').split('\n');
        lines.forEach(function (line) {
          if (!line.trim()) return;
          var parts = line.split(': ');
          var cat = parts[0] || '';
          var val = parts.slice(1).join(': ') || '';
          html += '<li><span class="cat-name">' + cat + '</span><span class="cat-value">' + val + '</span></li>';
        });
        html += '</ul>';
        document.getElementById('content').innerHTML = html;
      }

      // Register before any ping so parent retries cannot arrive before the listener exists
      window.addEventListener('message', function (event) {
        renderSummary(event.data);
      });

      function pingSummaryReady() {
        if (window.parent && window.parent !== window) {
          window.parent.postMessage({ type: 'quote-summary-ready' }, '*');
        }
      }

      pingSummaryReady();
      if (document.readyState === 'complete') {
        pingSummaryReady();
      } else {
        window.addEventListener('load', pingSummaryReady);
      }
    })();
  </script>
</body>
</html>
```

**Edit:** `SUMMARY_EMBED_ID` must match your summary embed’s **Properties → ID** (document it in [WIX_Codes.md](WIX_Codes.md)). If your summary embed is still `html1` inside the lightbox only, set `SUMMARY_EMBED_ID` to `'html1'`.

---

## Configurator HTML (not Velo) — which file per page

When you update the **contents** of each page’s HTML Embed from this repo, the **Wix wrapper** stays `#html1` on every page ([WIX_Codes.md](WIX_Codes.md)); only the pasted HTML differs per model.

| Page / model | Paste from project root |
|--------------|-------------------------|
| Staggered Harmony | `index_staggered_harmony.html` |
| Compact Harmony | `index_compact_harmony.html` |
| Straight Elegance | `index_straight_elegance.html` |
| Simplified Elegance | `index_simplified_elegance.html` |
| Compact Elegance | `index_compact_elegance.html` |
