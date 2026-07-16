# apps/

Drop standalone HTML mini-apps here, then reference them from `manifest.json`'s `apps` list
(`file` should be `apps/yourfile.html`). Each one runs sandboxed inside Hplay — treat it as a
self-contained page with its own inline CSS/JS, since it can't reach anything outside its iframe.
