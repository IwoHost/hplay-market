# Hplay Market

The marketplace website for [Hplay](https://github.com/iwohost/hplay) — the "Get More" screen in the
app reads `manifest.json` from this site to list downloadable apps, background themes, and decals.
The app itself never needs updating for any of this; add content here and it shows up next time
someone opens Get More.

Point Hplay's Settings → Get More → Marketplace URL at this site's published root, e.g.
`https://iwohost.github.io/hplay-market`.

## Layout

```
index.html      human-facing store homepage (lists what manifest.json has)
manifest.json   the file Hplay actually fetches
apps/           standalone HTML mini-apps referenced from manifest.json
decals/         images referenced from manifest.json's decals list
```

## manifest.json schema

```jsonc
{
  "apps": [
    { "id": "tuner", "name": "Tuner", "desc": "Guitar tuner", "file": "apps/tuner.html" }
  ],
  "themes": [
    {
      "id": "sunset", "name": "Sunset",
      "b": ["#hex1", "#hex2", "#hex3", "#hex4"],
      "w": ["#hex1", "#hex2", "#hex3", "#hex4"],
      "c": ["#hex1", "#hex2", "#hex3"],
      "lab": "#hex", "labsh": "rgba(0,0,0,.6)"
    }
  ],
  "decals": [
    { "id": "star", "name": "Star", "file": "decals/star.svg", "x": 80, "y": 45, "w": 12, "rot": -10 }
  ]
}
```

- `file` paths are relative to this site's root (or use a full `https://` URL to host elsewhere).
- **apps**: `file` is a complete, standalone HTML file. It runs inside a sandboxed iframe in Hplay
  (`sandbox="allow-scripts allow-forms allow-pointer-lock"`, no `allow-same-origin`) — it can't touch
  the user's library, storage, or the rest of the app directly, except through the read-only bridge
  below.
- **themes**: `b` = body gradient stops (4 colors), `w`/`c` = click-wheel gradient stops (4 + 3
  colors). Optional `lab`/`labsh` style the wheel's text labels; both default sensibly if omitted.
- **decals**: `file` is a plain PNG/SVG image (no code, so no sandboxing needed). `x`/`y` are position
  as a percent of the body (0–100, clamped), `w` is width as a percent of screen width (clamped
  4–40), `rot` is rotation in degrees (clamped ±180). Decals render behind the screen and click wheel,
  so they can never cover a control regardless of position.

## Reading the library & stats from an app

Apps can request a read-only snapshot of the song list and usage stats via `postMessage` — there's
no write path back, so this can't touch playback, the library, or settings:

```js
parent.postMessage({channel:'hplay', type:'getLibrary'}, '*');
parent.postMessage({channel:'hplay', type:'getStats'}, '*');

window.addEventListener('message', (e) => {
  if (!e.data || e.data.channel !== 'hplay') return;
  if (e.data.type === 'library') { /* e.data.data: [{title, artist, album}, ...] */ }
  if (e.data.type === 'stats')   { /* e.data.data: {opens, playCount, playMs,
    quizAnswered, quizCorrect, quizBest, songPlays, songCount} */ }
});
```

See `apps/library-stats.html` for a working example.

## Click wheel input

Hplay is a click-wheel device — users navigate everything else in the app with the wheel and center
button, not touch. An installed app should honor that instead of forcing the user to switch to
tapping the screen. Hplay forwards wheel turns and center-button presses into the app's iframe via
`postMessage`:

```js
window.addEventListener('message', (e) => {
  if (!e.data || e.data.channel !== 'hplay') return;
  if (e.data.type === 'wheel')  { /* e.data.dir: -1 or 1, e.data.mult: usually 1, higher on a fast spin */ }
  if (e.data.type === 'select') { /* center button pressed */ }
});
```

There's no single right way to use this — it depends on what the app does:

- **One main action** (e.g. a dice roller): make `select` trigger it. See `apps/dice.html`.
- **A scrollable read-only view**: make `wheel` scroll the page. See `apps/library-stats.html`.
- **Multiple buttons/rows to choose between**: track a focused index yourself, move it on `wheel`,
  highlight it (e.g. an outline), and call `.click()` on the focused element on `select`. See
  `apps/mashup.html` (button + list navigation) or `apps/metronome.html` (a value dialed in by
  `wheel`, started/stopped by `select`).

The hardware Menu/back button always exits the app back to Hplay's Get More screen — it's never
forwarded into the iframe, so there's no need to handle it.

Touch still works underneath this — keep your tap handlers too, both for testing the app directly in
a browser and so it isn't wheel-only.

## Adding content

1. Drop the file in `apps/` or `decals/` (or add a theme entry directly to `manifest.json`).
2. Add an entry to `manifest.json` pointing at it.
3. Commit and push (or edit directly on GitHub) — no app update needed.

Note: there's intentionally no music category here — distributing audio files needs rights to the
music, which this site doesn't have.
