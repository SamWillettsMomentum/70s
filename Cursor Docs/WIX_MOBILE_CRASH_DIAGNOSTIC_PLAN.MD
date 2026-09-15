# Wix configurator — mobile reload / “Can’t open this page” diagnostic plan

Use this document **in order** with a real phone (or simulator + remote debugging). Check off each section as you complete it. Goal: decide whether the failure is **Wix/page**, **embed size or Velo**, **Plattar / WebGL**, or **memory / GPU crash**, then apply the matching fix.

---

## What you are seeing (symptom summary)

| Phase | Behaviour |
|--------|-----------|
| 1 | Page loads; “configurator loading” appears (expected). |
| 2 | ~1 s later the **whole page** seems to **reload**; phase 1 repeats. |
| 3 | Chrome: **“Can’t open this page”** (generic tab failure). Safari: **“A problem repeatedly occurred on [URL]”** (WebKit / renderer crash signature). |
| Desktop | Works. |
| Wix publish | Vague error (no detail in UI). |

**Important distinction:** This pattern often is **not** your code calling `location.reload()`. It can be the **browser killing the tab** (out-of-memory, GPU/WebGL process crash, or repeated renderer crashes) and the user perceiving a reload—or Chrome offering to restore a crashed tab. Safari’s message is a strong hint of **renderer / WebKit instability**, not a normal JS exception.

Your repo’s model `index_*.html` files do **not** use `location.reload()` / `parent.location` for navigation; `postMessage` is used for Wix integration only. So treat **crash / platform / third-party** as first-class hypotheses alongside **accidental full-page navigation**.

---

## Likely cause categories (short)

1. **WebGL / GPU / memory (Plattar 3D inside Wix iframe)**  
   Mobile browsers allocate **less RAM** to tabs and are **stricter** with WebGL in **nested iframes**. Initializing the 3D viewer can spike memory and GPU use; a crash looks like a blank reload or “can’t open page.”

2. **Very large single embed (HTML + inline JS + JSON)**  
   Wix HTML embeds are one document; huge inline `MODEL_CONTENT` and scripts increase parse/compile cost and baseline memory—**before** WebGL. This can contribute on low-RAM devices.

3. **Resize / layout feedback loop**  
   Less common, but code that writes layout based on `window.innerHeight` / iframe height and triggers parent reflow can cause thrashing. Usually shows jank before crash, not always.

4. **Velo / parent page script**  
   Page-level code that runs on mobile only, or that reacts to `postMessage` / `wixWindow` in a way that refreshes or breaks the page, is worth ruling out even if desktop works.

5. **Network / blocked third-party on mobile**  
   Different DNS, ad blockers, or corporate filters on mobile data can cause **partial** loads; rarely causes the exact Safari “repeatedly occurred” message, but easy to verify.

6. **Wix publish / site health**  
   The generic publish error may be **unrelated** (another page, app, consent banner) or **related** (embed flagged). Capture **exact** wording and **which asset** Wix highlights if possible.

---

## Phase A — Confirm it is a crash vs a real navigation

**A1. Remote debugging (best signal)**  
- **Android + Chrome:** USB debugging → `chrome://inspect` → inspect the **Wix tab** and the **embed iframe** if listed. Watch **Console** when the failure happens.  
- **iOS + Mac:** Safari → Develop → [your device] → pick the Wix page. Watch console / network when it dies.

**What to look for**

- Messages about **WebGL**, **context lost**, **GPU**, **out of memory**, or **terminated**.  
- **No** new navigation URL in the Network tab right before death → supports **crash**, not `location` change.

**A2. Does the address bar URL change?**  
If the URL stays the same and the tab goes white / error, **crash** is more likely than your code navigating away.

**Check when done:** ☐ Crash-like (renderer dies) ☐ Real navigation (URL or full document reload with new request)

---

## Phase B — Isolate the HTML embed (no Velo)

**B1. Duplicate the page in Wix (draft)**  
Create a **test page** with **only** the HTML embed (same code as production). **Remove** all Velo page code temporarily (or use a fresh page with no Dev Mode code).

**B2. Test on mobile**  
- If **still fails:** problem is **embed + Wix shell + browser**, not Velo listeners.  
- If **works:** reintroduce Velo **line by line** or restore full page code and retest—suspect **page script** or **element interactions**.

**Check when done:** ☐ Fails with embed-only ☐ Works embed-only (Velo suspected)

---

## Phase C — Strip the embed (binary search inside the iframe)

Work from a **copy** of your embed HTML in a test Wix page. Goal: find **which layer** triggers the failure.

**Suggested order (fastest to narrow):**

| Step | Change | If crash stops → investigate |
|------|--------|------------------------------|
| C1 | Replace embed body with **static HTML only** (no scripts, no Plattar, no Slick): e.g. `<p>Embed OK</p>` | Wix iframe / page OK; issue is script or size. |
| C2 | Add back **only** your CSS + layout shell (no Plattar script tags, no Slick) | Layout/CSS. |
| C3 | Add **Slick + images** but **no** Plattar | Slider / image memory. |
| C4 | Add **Plattar** but delay init (see Phase D) or use Plattar’s minimal test scene | WebGL / Plattar. |

**Check when done:** ☐ Minimal static OK ☐ Fails at step: ___________

---

## Phase D — Plattar / WebGL–specific checks

**D1. Plattar docs / support**  
Confirm **mobile** and **iframe** recommendations for your embed type (some viewers need specific attributes or mobile fallbacks).

**D2. Lazy-init the 3D viewer**  
If you initialize Plattar on `DOMContentLoaded` or immediately, try **initializing only when** the user switches to the **3D Exterior** tab (or after first user gesture). This reduces simultaneous load with Slick + many images and can avoid early GPU spikes.

**D3. `webglcontextlost`**  
In dev tools (or temporary logging in embed), listen for `webglcontextlost` on the canvas / viewer container. If it fires before crash, plan for **graceful degradation** (static image fallback, “3D unavailable on this device” message).

**D4. Test outside Wix**  
Host the **same** HTML (or a trimmed copy) on a simple **HTTPS** static host or local tunnel, **not** in an iframe. If mobile works there but fails **only** inside Wix iframe → **iframe + memory / GPU** interaction is confirmed.

**Check when done:** ☐ Lazy-init tested ☐ Outside-iframe mobile test: ☐ OK ☐ Fail

---

## Phase E — Embed weight and mobile performance

**E1. Measure size**  
Approximate **characters / KB** of the pasted embed (especially inline JSON). If it is **very large**, consider splitting strategies Wix allows (e.g. multiple embeds, or reducing duplicated data—only if product allows).

**E2. Image payload**  
On mobile, **decode** of many large interior images + WebGL can OOM. Check **image dimensions / format** (WebP where supported, reasonable max width for mobile).

**E3. `postMessage` frequency**  
Ensure you are **not** sending large payloads on **every** `resize` or **every** pricing tick. Throttle to **meaningful** state changes (your spec already aims for structured updates—verify on the live pasted code).

**Check when done:** ☐ Embed size noted: ___________ ☐ postMessage throttling verified

---

## Phase F — Wix editor / publish error

**F1. Exact text**  
Screenshot or copy the **full** publish error. Check Wix **Site Issues** / **SEO** / **Accessibility** panels for **page-specific** flags.

**F2. Correlation**  
Does publish fail **only** when this page changed, or always? If always, the mobile crash might still be runtime-only while publish error is **separate** (e.g. broken link on another page).

**Check when done:** ☐ Publish error captured ☐ Correlates with configurator page: ☐ Yes ☐ No

---

## Phase G — Decide fix path (after Phases A–F)

Use the table below once you know where it breaks.

| Finding | Direction |
|---------|-----------|
| Crash with **static-only** embed | Wix page settings, other global scripts, or device-specific Wix bug — contact Wix support with repro. |
| Crash only when **Plattar** loads | Lazy init, static fallback for mobile / low memory, reduce concurrent media, confirm Plattar iframe/mobile guidance. |
| Crash when **Slick + many images** without Plattar | Reduce image sizes, lazy-load off-screen slides, fewer simultaneous decoded images. |
| **Works** embed-only, **breaks** with Velo | Audit page code: `wixWindow`, `postMessage` handlers, anything that runs `reload` or opens URLs; test on mobile with logging. |
| **Works** outside iframe, **fails** in Wix only | Emphasize iframe memory limits + WebGL; combine lazy Plattar + lighter embed + Wix support. |

---

## Artifacts to collect (for your notes or support tickets)

1. Device model, OS version, browser + version.  
2. Wi‑Fi vs cellular (same behaviour?).  
3. Remote-debug **console** output (last 20 lines before death).  
4. Whether **Preview** vs **Published** behaves the same.  
5. Smallest embed variant that **still** crashes (from Phase C).  
6. Wix publish error **full text** and any **page URL** Wix highlights.

---

## References in this repo

- Product context: [AI_BRIEF.MD](AI_BRIEF.MD)  
- Wix + Velo wiring: [WIX_VELO_QUICK_SETUP.MD](WIX_VELO_QUICK_SETUP.MD)  
- postMessage payload: [POSTMESSAGE_PAYLOAD.MD](POSTMESSAGE_PAYLOAD.MD)  
- Plattar integration: [Plattar_Architecture.MD](Plattar_Architecture.MD), `Plattar Docs/`

---

*Last updated: working doc for paired diagnosis — adjust checkboxes and notes as you run through it.*
