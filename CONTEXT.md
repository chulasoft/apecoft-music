# CONTEXT.md — Read Me First

> **Mandatory entry point.** Read this file before touching anything else in this repo — code, data, or docs. It's short on purpose. Once you're done here, go to [`docs/SKILL.md`](docs/SKILL.md) for the detailed map and rules.

## What this project is

**APECOFT MUSIC** is an interactive music portfolio — a single-page app styled like a menu-driven game UI: `roster → artist → albums → tracks → story page`, with poster-style wipe transitions between screens. It ships as **plain static HTML files with no build step**: React, ReactDOM, and Babel are loaded from a CDN, JSX is transpiled in the browser, and content comes from CSV files fetched at runtime. Open `index.html` in a browser (ideally via a local server) and it runs.

There are two interchangeable themes, each a **complete, self-contained HTML file**:

| File | Status | Look |
|---|---|---|
| `index.html` | **active** | Red/black/white poster-collage — jagged cutouts, halftone dots, ransom-note lettering, star wipes |
| `index-noir.html` | alternate | Dark editorial — cream/gold on near-black, soft bloom glow, fan-flip cards |

They are **not** two views over shared code — each file duplicates its own copy of every component. See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for why, and the hard rule below about keeping edits scoped.

## Rough structure

```
apecoft-music/
├── index.html        ← active app (self-contained: HTML + CSS + JSX in one file)
├── index-noir.html   ← alternate app (separate self-contained copy)
├── README.md          ← human/editor-facing docs (content editing, asset specs, troubleshooting)
├── CONTEXT.md          ← you are here
├── docs/               ← agent/contributor-facing coordination docs (read next)
├── data/               ← artists.csv, albums.csv, songs.csv — this IS the database
├── audio/              ← .mp3/.wav track files + bgintro.mp3
└── covers/             ← artist/album/song cover images
```

## Hard rules

1. **No build step, ever.** Don't introduce a bundler, package.json, or npm dependency to make this "buildable." The whole point is that it stays double-clickable / drag-and-drop static-hostable. CDN `<script>` tags (React/ReactDOM/Babel) and inline `<script type="text/babel">` are the architecture, not a shortcut.
2. **`index.html` and `index-noir.html` are independent copies.** A fix or change in one does **not** automatically apply to the other. Decide explicitly (or ask the user) whether a change should be ported to both, and never claim "fixed" for one when only the other was touched.
3. **Content lives in `data/*.csv`, never hardcoded into components.** The only hardcoded content allowed is `DEMO_REALM` (the offline/`file://` fallback) and `FALLBACK_IMG` (missing-asset placeholder) — both intentionally mirror the CSV shape.
4. **The CSV schema is a contract.** Columns, the `|` tag separator, and `||` story-paragraph separator are documented in [docs/DATABASE.md](docs/DATABASE.md) and `README.md`. If you change columns, update `buildRealm()`/`mkSong()` in the HTML **and** both docs in the same change.
5. **Match the existing code style** (dense inline `style={{...}}` objects, PascalCase components, `useX` hooks, ALL_CAPS constants, `// ═══ SECTION ═══` banner comments) — see [docs/STYLE_GUIDE.md](docs/STYLE_GUIDE.md). Don't run a formatter over the file or introduce CSS classes/modules.
6. **Scope visual/animation changes to the screen(s) actually named.** Don't blanket-change shared primitives without checking every screen that uses them.
7. **This environment cannot always reach `cdnjs.cloudflare.com`** (proxy policy). If the app fails to render here, that's a sandbox network limitation, not a code bug — say so explicitly rather than "fixing" it by vendoring the CDN scripts without asking.

## Where to go next

→ **[docs/SKILL.md](docs/SKILL.md)** — the detailed map + full rule set, written in portable skill format.
Then, as needed: [ARCHITECTURE.md](docs/ARCHITECTURE.md) · [STYLE_GUIDE.md](docs/STYLE_GUIDE.md) · [FEATURE.md](docs/FEATURE.md) · [DATABASE.md](docs/DATABASE.md) · [TODO.md](docs/TODO.md)
