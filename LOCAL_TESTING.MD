# Local testing (multiple index files)

The 3D viewer **does not work** when you open an HTML file directly (`file://`). You need a local web server.

**Quick start:**

1. From the project root, run:
   ```bash
   npx serve .
   ```
   (Or `python3 -m http.server 8000` if you prefer.)

2. In the browser go to:
   - **Staggered Harmony:** `http://localhost:3000/index_staggered_harmony.html`
   - **Compact Harmony:** `http://localhost:3000/index_compact_harmony.html`

Port may differ (e.g. 3000 for `serve`, 8000 for Python). Check the terminal output.

**Note:** Each index file has its own embedded `MODEL_CONTENT`. After editing a model's JSON file (e.g. `Model Content Data/Staggered Harmony/MODEL_CONTENT_STAGGERED_HARMONY.json`), copy its contents into the matching index’s `var MODEL_CONTENT = { ... };` block so the configurator uses the latest data.
