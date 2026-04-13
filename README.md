# GaggiMate Pump Flow Calibration

One-click pump flow calibration for [GaggiMate](https://github.com/jniebuhr/gaggimate). Runs the calibration shot, analyzes the shot history, and updates `pumpModelCoeffs` on the machine — without leaving the GaggiMate web UI.

### 👉 [Open the installer page](https://ggazzo.github.io/gaggimate-pump-flow-calibration/)

[![Open](https://img.shields.io/badge/try_it-ggazzo.github.io-c98a3d?style=for-the-badge)](https://ggazzo.github.io/gaggimate-pump-flow-calibration/)

## How it works

Open the [installer page](https://ggazzo.github.io/gaggimate-pump-flow-calibration/) and drag the **🔧 Calibrate GaggiMate** button to your bookmarks bar. Then:

1. Navigate to your GaggiMate web UI (e.g. `http://gaggimate.local`).
2. Place a scale with a cup under the steam wand.
3. Click the bookmarklet — a modal appears inside the GaggiMate UI.
4. Click **Start calibration**. The tool will:
   - Read the current `pumpModelCoeffs` via `GET /api/settings`.
   - Save and select the calibration profile via the WebSocket API.
   - Switch to BREW mode and activate the process.
   - Wait for the shot to finish (watching `evt:status.process.a`).
   - Fetch the latest `.slog` from `/api/history/`, with retry/backoff while the firmware flushes to flash.
   - Parse the binary shot, extract samples from the `Measure 1 bar` and `Measure 9 bar` phases.
   - Compute new coefficients using the same math as the [original `analyze.js`](https://discord.com/channels/951416527721230336/1410049551791951943).
   - Offer to write them back via `POST /api/settings`.
5. During the shot, adjust the steam valve so pressure reaches 1 bar during the first Measure phase and 9 bar during the second. Keep it stable for the 10 seconds of each Measure phase.

## Why a bookmarklet?

The GaggiMate firmware's HTTP server does not emit CORS headers. A standalone page hosted on a different origin (or opened via `file://`) cannot call `/api/settings` — the browser blocks it. Injecting the tool into the GaggiMate UI itself makes all calls same-origin, so nothing is blocked and nothing leaves your network.

## Fallback: manual analysis

If you prefer to run the calibration profile yourself and analyze the downloaded `.slog` manually, `index.html` also has a file-upload mode that computes the new coefficients from a local file plus the current coefficients.

## Files

- **`index.html`** — landing page with the installable bookmarklet and the manual fallback. Uses Tailwind v4 + DaisyUI v5 via CDN, same stack as the GaggiMate web UI.
- **`calibrate.js`** — the injected tool itself. Embedded into `index.html` at build time so the bookmarklet is self-contained (no external JS needed once installed).

## Updating

Edit `calibrate.js`, then re-embed it into `index.html`:

```bash
python3 -c "
src = open('calibrate.js').read()
html = open('index.html').read()
import re
new = re.sub(r'(<script type=\"text/plain\" id=\"calibrate-src\">).*?(</script>)', lambda m: m.group(1) + src + m.group(2), html, count=1, flags=re.DOTALL)
open('index.html', 'w').write(new)
"
```

## License

MIT.
