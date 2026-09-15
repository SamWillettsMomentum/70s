# Phase D4 — Test the configurator outside Wix (detailed steps)

This guide supports **[WIX_MOBILE_CRASH_DIAGNOSTIC_PLAN.MD](WIX_MOBILE_CRASH_DIAGNOSTIC_PLAN.MD) Phase D4**: run the **same** HTML (especially `index_staggered_harmony_test.html`) in a normal browser tab over **HTTPS** (or HTTP on LAN), **without** the Wix parent page or Velo. Use it to separate **“Plattar / WebGL / our embed”** problems from **“Wix iframe + site shell”** problems.

**Official Plattar SDK documentation** (mobile, embed, and API behaviour): [Plattar AR Adapter SDK — Documentation](https://docs.plattar.com/#/).

---

## Why do this

| Result | Interpretation |
|--------|------------------|
| **Stable on phone** when opened as a **top-level** page (same file, HTTPS), but **crashes inside Wix** | Strong signal for **iframe memory limits**, **nested browsing context**, or **Wix-specific** constraints — combine lazy Plattar init (test file) with Wix/support follow-up. |
| **Crashes the same way** top-level on mobile | Problem is more likely **Plattar / WebGL / device**, very large embed, or **network** — focus on viewer options, scene weight, image load, and Plattar guidance. |
| **Works on desktop** top-level but **fails on mobile** top-level | Mobile **GPU / RAM / WebGL** — align with Plattar docs, reduce concurrent work, lazy init, fallbacks. |

---

## Requirements

1. **Do not open the HTML file via `file://`**  
   Plattar and many embed paths expect a real origin. The test index already shows an error for `file:`; use a local or hosted server.

2. **Prefer HTTPS for parity with Wix**  
   Wix is always HTTPS. Some APIs behave differently on plain HTTP. For a quick check, HTTP on a LAN IP is still useful.

3. **Use the test build when comparing lazy Plattar**  
   Repo file: **`index_staggered_harmony_test.html`** — starts on **Interiors**; creates `<plattar-embed>` only after the user (or auto-switch) goes to **3D Exterior**.

---

## Option A — Local static server (fastest)

From the **project root** (the folder that contains `index_staggered_harmony_test.html`):

### A1. Node (`npx serve`)

```bash
cd "/path/to/plattar-test-backup-1"
npx --yes serve -l 3000
```

- On your computer, open: `http://localhost:3000/index_staggered_harmony_test.html`  
- On your phone (same Wi‑Fi), open: `http://<your-computer-LAN-IP>:3000/index_staggered_harmony_test.html`  
  (Find the IP in macOS **System Preferences → Network**, or run `ipconfig getifaddr en0` in Terminal.)

### A2. Python 3

```bash
cd "/path/to/plattar-test-backup-1"
python3 -m http.server 8080
```

Then open `http://localhost:8080/index_staggered_harmony_test.html` (and the LAN URL on the phone).

**Firewall:** If the phone cannot connect, allow incoming connections on that port for your dev machine.

---

## Option B — HTTPS tunnel (closest to Wix)

Plain HTTP on a LAN IP is enough for many checks; **HTTPS** avoids mixed-content surprises if you later compare subresources.

1. Start a local static server (Option A).  
2. Expose it with a tunnel tool (e.g. **ngrok**, **Cloudflare Tunnel**, **localtunnel**):  
   - Example pattern: `ngrok http 3000`  
3. Open the **https://…** URL the tool prints on your **phone** (cellular or Wi‑Fi).

**Tip:** Keep the tunnel URL private; it exposes your dev server to the internet while the tunnel runs.

---

## Option C — Hosted static file (no terminal on phone)

1. Zip or upload **`index_staggered_harmony_test.html`** to a static host that serves over HTTPS (e.g. Netlify Drop, Cloudflare Pages, S3 + CloudFront, GitHub Pages).  
2. Ensure the file is the **default document** or use the full path in the URL.  
3. Open that URL on **mobile Safari and Chrome**.

**Caveat:** You must upload **one self-contained file** as your repo does (all JS/CSS/embed data inline). No extra build step required.

---

## What to do on the phone (consistent test)

1. Open the test URL in a **private/incognito** window once, to reduce extension interference.  
2. **Stay on Interiors** for ~10–15 seconds — confirm the page **does not** crash (validates lazy Plattar: no viewer yet).  
3. Tap **3D Exterior** — note time-to-spinner and whether the tab **ever** becomes stable.  
4. Repeat once with **Wi‑Fi** and once with **cellular** if behaviour differs.  
5. Optionally change an **Exterior Cladding** option (should auto-switch to 3D and trigger the same init path).

Record: **device model, OS version, browser, Wi‑Fi vs cellular**, and whether the crash happens **before** or **after** opening 3D.

---

## Optional: iframe vs top-level

Wix wraps your markup in an **iframe**. To mimic that **without** Wix:

1. Create a minimal **`iframe-host.html`** on the same server:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Iframe host (Plattar test)</title>
  <style>
    html, body { margin: 0; height: 100%; }
    iframe { width: 100%; height: 100%; border: 0; display: block; }
  </style>
</head>
<body>
  <iframe src="index_staggered_harmony_test.html" title="Configurator"></iframe>
</body>
</html>
```

2. Open `iframe-host.html` on the phone.  
3. If **top-level works** but **iframe fails**, that narrows the issue to **nested iframe + GPU/memory** (still not proof of Wix-only, but very similar class of bug).

---

## Remote debugging (capture evidence)

- **iPhone + Mac:** Safari → **Develop** → *[device]* → select the page → **Console** / **Network**.  
- **Android + Chrome:** USB debugging → desktop Chrome → **`chrome://inspect`**.

Save **screenshots or the last console lines** before a crash (WebGL context lost, terminated process, etc.).

---

## After you have results

Update your notes in [WIX_MOBILE_CRASH_DIAGNOSTIC_PLAN.MD](WIX_MOBILE_CRASH_DIAGNOSTIC_PLAN.MD) **Phase G** table with:

- **Top-level mobile:** stable / crash  
- **Iframe host mobile:** stable / crash / not tested  
- **Lazy 3D (test file):** crash while still on Interiors yes/no  

That decides whether the next step is **more Plattar/scene tuning**, **promoting lazy init to production index**, or **Wix/support** with a minimal repro.

---

*Companion file: lazy-init test HTML is `index_staggered_harmony_test.html` in the project root.*
