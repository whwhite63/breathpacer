---
name: verify
description: How to run and verify the breath pacer PWA locally
---

# Verifying the breath pacer

Static PWA — one `index.html` (two `<script>` blocks: minified NoSleep.js first, app code second), `service-worker.js`, `manifest.json`.

## Run

```bash
python3 -m http.server 8888   # serve repo root, open http://localhost:8888
```

## Headless verification recipe

Copy the app files to a scratch dir and generate a harness page that pre-seeds
settings and auto-starts (there is no URL API):

- Insert before the app scripts: `<script>localStorage.setItem('bp-settings', JSON.stringify({...}))</script>`
- Append before `</body>`: a script that calls `pauseBtn.click()` after load (the app opens with the panel visible and one Start/Pause button; there is no separate start-session button) and `console.log`s state.
- Top-level `const`/`function` bindings in the app script (`periods`, `cue`, `panel`, `cvs`, …) are reachable from the harness script; `cue` can be wrapped to count firings.

Screenshot a mid-session frame:

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" --headless=new \
  --disable-gpu --user-data-dir=<fresh-tmp> --autoplay-policy=no-user-gesture-required \
  --enable-logging=stderr --window-size=390,700 --virtual-time-budget=6000 \
  --screenshot=out.png http://localhost:PORT/harness.html
```

## Gotchas

- **`--virtual-time-budget` starves rAF**: timers fast-forward but the animation
  loop barely runs, so anything timing-dependent (cue firing, HUD elapsed) must be
  probed in a *real-time* run (drop the budget flag, poll the log ~7s). Virtual time
  is fine for screenshots (frame rendered at budget end is correct).
- Chrome sometimes doesn't exit cleanly; run it backgrounded, poll for the
  screenshot/log marker, then kill it.
- `index.html` historically contains odd Unicode (U+2011 hyphens, U+202F/U+00A0
  spaces) — if an Edit fails to match, hexdump the line.
- The panel opens by tapping the canvas (any time); harness can dispatch
  `new PointerEvent('pointerdown',{bubbles:true})` on `cvs`.
