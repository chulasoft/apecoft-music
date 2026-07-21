# APECOFT MUSIC — Music Realm

**Presented by Apecoft & Evilelf**

An interactive music portfolio, styled like a menu-driven game UI: roster → artist → albums → tracks → story pages, with wipe transitions between screens.

## 🎨 Themes

| File | Look |
|---|---|
| `index.html` **(active)** | Red/black/white poster-collage style — jagged cutout shapes, halftone dots, slanted menus, duotone portraits, ransom-note lettering, star-shaped screen wipes |
| `index-noir.html` (alternate) | Dark editorial style — warm cream/gold on near-black, soft bloom glow, fan-flip song cards |

Both read the same CSV files, so switching is just a file rename (e.g. rename `index.html` → `index-alt.html`, then rename `index-noir.html` → `index.html`).

## 📁 Files

```
apecoft-music-realm/
├── index.html          ← main app (red/black poster theme)
├── index-noir.html     ← alternate app (dark editorial theme)
├── README.md
├── data/
│   ├── artists.csv     ← edit these to change content
│   ├── albums.csv
│   └── songs.csv
├── audio/               ← put .wav/.mp3 files here (song tracks + bgintro.mp3)
└── covers/               ← put cover images here
```

## 🚀 Running it

- **Live Server (recommended)** — open the folder in VS Code, right-click `index.html` → "Open with Live Server". CSVs, audio, and covers all load.
- **Double-click** — opens fine, but CSVs can't be fetched over `file://`, so it falls back to built-in demo content. Audio still plays.
- **Static host (Walrus, etc.)** — upload the folder as-is; everything works including CSV loading.

## ✏️ Editing content

Edit the three CSVs in `data/` with any spreadsheet app or text editor. Save as **CSV (UTF-8)**.

**`artists.csv`** — `id, name, role, tagline, bio, accent, bloom, dark, tags, image, debut`
**`albums.csv`** — `id, artist_id, title, year, issue, desc, accent, bloom1, bloom2, tags, cover`
**`songs.csv`** — `id, album_id, n, title, dur, genre, accent, g1, g2, tags, desc, audio, story_title, story_paragraphs, story_quote`

Rules:
- Wrap a field in `"double quotes"` if it has commas or line breaks; escape an inner quote as `""`
- Tags separate with `|`, story paragraphs separate with `||`
- Paths (`image`, `cover`, `audio`) are relative to `index.html`, e.g. `covers/minta.png`
- Leave `image`/`cover`/`audio` blank and the app falls back to a built-in placeholder — nothing breaks

## 🎨 Color pairs (accent → bloom → dark)

Pick a bright accent, a slightly deeper "bloom" glow color, and a very dark shade of the same hue for backgrounds:

| Tone | accent | bloom | dark |
|---|---|---|---|
| Red | `#e60012` | `#ff2438` | `#6b0410` |
| Blue | `#00b4e6` | `#28c8ff` | `#043048` |
| Purple | `#c838ff` | `#d868ff` | `#38084a` |
| Orange | `#ff6a00` | `#ff8c28` | `#5a2400` |
| Teal | `#00e0c0` | `#28f0d8` | `#043a32` |

The site's own black/white/red brand frame stays fixed — these palettes are just for per-artist/album/song accent variety.

## 🖼️ Asset specs

| Asset | Size | Notes |
|---|---|---|
| Artist portrait | 900×1200+ | Portrait orientation, runs edge-to-edge on desktop |
| Album/song cover | 1000×1000 (1:1) | Used on album cards, song cards, and story pages |
| Audio | .wav or .mp3 | mp3 for smaller files |

## 🕹️ Controls

| Screen | Mouse/touch | Keyboard |
|---|---|---|
| Title | Tap anywhere | — |
| Artist select | Click a name in the list | ↑↓ arrows, Enter |
| Album select | Click an album | ↑↓ arrows, Enter |
| Track select | Click a track, ▶ Play All | ↑↓ arrows, Enter |
| Story page | Scroll | player bar controls below |

## 🔧 Common tweaks (in `index.html`)

| What | Where |
|---|---|
| Title screen text / credit line | `TitleScreen` function |
| Brand colors | `const RED`, `BLACK`, `WHITE` near the top |
| Page title | `<title>` tag in `<head>` |
| Transitions between screens | `TRANSITIONS` object inside `App()` |
| Jagged cutout shapes | `JAG1`, `JAG2`, `JAG3` constants |
| Letter-by-letter name styling | `RansomText` function |
| Mobile breakpoint | `useIsMobile()` |

## ⚠️ Troubleshooting

- **CSV edits don't show up** → you're opening via double-click; use Live Server instead.
- **Audio won't play** → check the filename in `songs.csv` matches the file in `audio/` exactly (case-sensitive on some hosts). Browsers also block autoplay until the first click.
- **Thai/unicode text looks broken** → re-save the CSV as UTF-8.
- **Progress bar doesn't move during playback** → that song has no `audio` path set; the button just toggles the UI.
- **No background music after entering the site** → make sure `audio/bgintro.mp3` exists with that exact name; it plays quietly on a loop, auto-pauses when a real track starts, and resumes when nothing else is playing.
