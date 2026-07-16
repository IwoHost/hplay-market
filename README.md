# Hplay Market

The marketplace website for [Hplay](https://github.com/iwohost/hplay) — the "Get More" screen in the
app reads `manifest.json` from this site to list downloadable apps, music, and background themes.
The app itself never needs updating for any of this; add content here and it shows up next time
someone opens Get More.

Point Hplay's Settings → Get More → Marketplace URL at this site's published root, e.g.
`https://iwohost.github.io/hplay-market`.

## Layout

```
index.html      human-facing store homepage (lists what manifest.json has)
manifest.json   the file Hplay actually fetches
apps/           standalone HTML mini-apps referenced from manifest.json
music/          audio files referenced from manifest.json
```

## manifest.json schema

```jsonc
{
  "apps": [
    { "id": "tuner", "name": "Tuner", "desc": "Guitar tuner", "file": "apps/tuner.html" }
  ],
  "music": [
    { "title": "Song Name", "artist": "Artist", "album": "Album", "file": "music/song.mp3" }
  ],
  "themes": [
    {
      "id": "sunset", "name": "Sunset",
      "b": ["#hex1", "#hex2", "#hex3", "#hex4"],
      "w": ["#hex1", "#hex2", "#hex3", "#hex4"],
      "c": ["#hex1", "#hex2", "#hex3"],
      "lab": "#hex", "labsh": "rgba(0,0,0,.6)"
    }
  ]
}
```

- `file` paths are relative to this site's root (or use a full `https://` URL to host elsewhere).
- **apps**: `file` is a complete, standalone HTML file. It runs inside a sandboxed iframe in Hplay
  (`sandbox="allow-scripts allow-forms allow-pointer-lock"`, no `allow-same-origin`) — it can't touch
  the user's library, storage, or the rest of the app.
- **music**: downloaded straight into the user's library, same as adding a local file.
- **themes**: `b` = body gradient stops (4 colors), `w`/`c` = click-wheel gradient stops (4 + 3
  colors). Optional `lab`/`labsh` style the wheel's text labels; both default sensibly if omitted.

## Adding content

1. Drop the file in `apps/` or `music/`.
2. Add an entry to `manifest.json` pointing at it.
3. Commit and push (or edit directly on GitHub) — no app update needed.
