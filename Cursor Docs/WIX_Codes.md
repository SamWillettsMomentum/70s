# Wix element IDs — canonical reference

After any layout change (duplicate, wrap in Box/Stack, etc.), open each element in the editor and confirm **Properties → ID** matches this file, then update the pasted Velo constants in one go.

---

## Configurator pages (each of the 5 model pages)

| Role | Velo ID | Code |
|------|---------|------|
| HTML Embed (configurator) | `html1` | `$w('#html1')` |
| Get Quote button | `button10` | `$w('#button10')` |

---

## Get Quote lightbox (shared)

| Role | Velo ID | Code |
|------|---------|------|
| Lightbox component (editor reference only) | `lightbox1` | Not used in `openLightbox()` |
| Pop-up name | — | `openLightbox('Get Quote', …)` |
| Form display name | Get Quote Form | — |
| Form element | `form1` | `$w('#form1').setFieldValues({ … })` |
| HTML summary embed (recommended) | `htmlSummary` | `$w('#htmlSummary')` |

**Recommended:** Set the lightbox summary HTML embed’s ID to `htmlSummary` so it is never confused with the configurator embed `html1` on pages. If your site still uses `html1` for the summary embed inside the lightbox only, keep that ID and set `SUMMARY_EMBED_ID` to `'html1'` in the lightbox Velo ([WIX_VELO_QUICK_SETUP.MD](WIX_VELO_QUICK_SETUP.MD) Step 5).

### Layout-only containers (no Velo reference required)

These wrap the form and summary embed in the current design; Velo does **not** need to select them unless you add custom behaviour:

- `#box155` — parent container (holds the stack)
- `#box154` — stack (contains the form and HTML embed)

---

## Hidden form fields (field keys, not `$w` IDs)

Wix **Form** blocks do not expose inner inputs as `$w('#…')`. Use **`$w('#form1').setFieldValues({ … })`** with these **field keys** (set in the form field settings for each hidden input):

| Purpose | Field key |
|---------|-----------|
| Product name | `hidden_product` |
| Price (formatted string from page Velo) | `hidden_price` |
| Selections (multi-line text) | `hidden_selections` |

**Verify after form edits:** In the form builder, open each hidden field → confirm its **field key** is exactly as above. If any key differs, update the constants `FIELD_PRODUCT`, `FIELD_PRICE`, `FIELD_SELECTIONS` in the lightbox script and this table together.
