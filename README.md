# APECOFT MUSIC — Music Realm

**Presented by Apecoft & Evilelf**

An interactive, game-inspired music portfolio. Roster → artist profile → albums → tracks → story pages, styled after Persona 5 Royal's UI: jagged cutout shapes, halftone dots, slanted menus, ransom-note lettering, and star-wipe transitions.

**Two themes included:**
- `index.html` — **Persona 5 Royal style** *(default/active theme)*: red/black/white, jagged cutouts, halftone, slanted confidant-style menus, duotone artist portraits, per-screen wipe transitions
- `index-noir.html` — **Dark editorial style** *(alternate)*: warm cream/gold on near-black, bloom glow, fan-flip cards

Both themes read the same CSV files. To switch the default, just rename the files (e.g. rename `index.html` → `index-p5r.html` and `index-noir.html` → `index.html`).

---

## 📁 Folder Structure

```
apecoft-music-realm/
├── index.html          ← main app — Persona 5 Royal theme (single file, React via CDN)
├── index-noir.html     ← alternate app — dark editorial theme
├── README.md           ← this file
│
├── data/               ← ✏️ EDIT THESE to add your content
│   ├── artists.csv
│   ├── albums.csv
│   └── songs.csv
│
├── audio/              ← put your music files here (.wav / .mp3)
│   └── README.txt
│
└── covers/             ← put your cover images here (.png / .jpg)
    └── README.txt
```

---

## 🚀 How to Run

### Option A — Live Server (recommended)
CSV loading requires a local server (browsers block `fetch()` on `file://`).

1. Open the folder in **VS Code**
2. Install the **Live Server** extension (if you haven't)
3. Right-click `index.html` → **"Open with Live Server"**
4. Your CSV data, audio, and covers all load correctly ✓

### Option B — Double-click (demo mode)
Opening `index.html` directly also works, but:
- CSV files **cannot** be loaded → the app falls back to **built-in demo data**
- Audio files still play (they use relative paths from `<audio>`, not fetch)

### Option C — Deploy (Walrus / any static host)
Upload the whole folder as-is. Everything works, including CSV loading.

---

## ✏️ Adding Your Content (CSV Guide)

All content lives in three CSV files inside `data/`. Edit them with any
spreadsheet app (Excel, Google Sheets, Numbers) or a text editor.
**Save as CSV (UTF-8).**

### `data/artists.csv`

| Column    | Description                                    | Example                     |
|-----------|------------------------------------------------|-----------------------------|
| `id`      | Unique artist ID (used by albums.csv)          | `1`                         |
| `name`    | Artist name (displayed large)                  | `MINTA`                     |
| `role`    | Role line under the name                       | `Vocalist · Songwriter`     |
| `tagline` | Short one-liner on the select card             | `The warmest voice…`        |
| `bio`     | Longer bio (quote if it contains commas)       | `"A singer with a warm…"`   |
| `accent`  | Main theme color (hex)                         | `#e60012`                   |
| `bloom`   | Glow color (hex)                               | `#ff2438`                   |
| `dark`    | Dark base color for gradients (hex)            | `#1a0508`                   |
| `tags`    | Tags separated by `\|` (pipe)                  | `Warm\|Nostalgic`           |
| `image`   | Portrait image path (blank = built-in image)   | `covers/minta.png`          |
| `debut`   | Debut year shown in stats                      | `2025`                      |

### `data/albums.csv`

| Column      | Description                                  | Example                  |
|-------------|----------------------------------------------|---------------------------|
| `id`        | Unique album ID (used by songs.csv)          | `11`                      |
| `artist_id` | Which artist owns this album (artists.csv id)| `1`                       |
| `title`     | Album title                                  | `Love Passion`            |
| `year`      | Release year                                 | `2025`                    |
| `issue`     | Volume label shown on the card               | `I`                       |
| `desc`      | Album description (quote if commas inside)   | `"The debut album…"`      |
| `accent`    | Album theme color                            | `#e60012`                 |
| `bloom1`    | Outer glow color                             | `#ff2438`                 |
| `bloom2`    | Inner glow color (darker)                    | `#6b0410`                 |
| `tags`      | Tags separated by `\|`                       | `Love\|Passion\|J-Pop`    |
| `cover`     | Cover image path (blank = built-in image)    | `covers/love-passion.png` |

### `data/songs.csv`

| Column             | Description                                        | Example                          |
|--------------------|-----------------------------------------------------|-----------------------------------|
| `id`               | Unique song ID                                       | `111`                             |
| `album_id`         | Which album this song belongs to                     | `11`                              |
| `n`                | Track number label                                   | `01`                              |
| `title`            | Song title                                           | `Off the Record`                  |
| `dur`              | Duration shown (mm:ss)                               | `2:45`                            |
| `genre`            | Genre shown on the card                              | `J-Pop`                           |
| `accent`           | Song theme color                                     | `#e60012`                         |
| `g1`               | Gradient tint start color                            | `#e60012`                         |
| `g2`               | Gradient tint end color (darker)                     | `#40040a`                         |
| `tags`             | Tags separated by `\|`                               | `Upbeat\|Opener`                  |
| `desc`             | One-line description under the card                  | `"Feelings never spoken…"`        |
| `audio`            | Audio file path (blank = no audio, UI still works)   | `audio/Off_the_Record.wav`        |
| `story_title`      | Title of the story page                              | `Off the Record`                  |
| `story_paragraphs` | Story text — separate paragraphs with `\|\|`         | `"Para one.\|\|Para two."`        |
| `story_quote`      | Pull-quote at the end of the story                    | `"Sometimes the truest story…"`   |

**CSV rules to remember:**
- Wrap a field in `"double quotes"` if it contains commas, quotes, or line breaks
- Inside a quoted field, escape a quote by doubling it: `""like this""`
- Tags use `|` (pipe) as separator, story paragraphs use `||` (double pipe)
- Paths are relative to `index.html` (e.g. `audio/song.wav`, `covers/art.png`)

---

## 🎨 Color Palette Cheatsheet

| Artist tone (used in demo) | accent    | bloom / bloom1 | bloom2 / dark |
|-----------------------------|-----------|----------------|---------------|
| Red (MINTA)                 | `#e60012` | `#ff2438`      | `#6b0410`     |
| Blue (WAVE)                 | `#00b4e6` | `#28c8ff`      | `#043048`     |
| Purple (SORA)                | `#c838ff` | `#d868ff`      | `#38084a`     |
| Orange (alt album)           | `#ff6a00` | `#ff8c28`      | `#5a2400`     |
| Teal (alt album)             | `#00e0c0` | `#28f0d8`      | `#043a32`     |
| Pink (alt album)             | `#ff3d9a` | `#ff68b8`      | `#4a0828`     |

Rule of thumb: `accent` = bright, `bloom1` = slightly deeper, `bloom2`/`dark` = very dark version of the same hue. The site's fixed brand colors (red/black/white) stay constant — these palette rows are for per-artist/per-album accent variety only.

---

## 🖼️ Asset Specs

| Asset          | Recommended size | Notes                                              |
|----------------|-------------------|-----------------------------------------------------|
| Artist portrait| 900×1200 or larger| Portrait orientation; runs large/edge-bleeding on desktop, so higher resolution looks sharper |
| Album cover    | 1000×1000 (1:1)   | Square; also used on song cards and the story page   |
| Audio          | .wav or .mp3       | .mp3 recommended for smaller file size               |

If an image path is blank or missing, the app uses a built-in placeholder image, so nothing breaks.

---

## 🕹️ Controls (index.html — P5R theme)

| Screen        | Mouse / Touch                          | Keyboard              |
|---------------|-----------------------------------------|------------------------|
| Title         | Tap anywhere                            | —                      |
| Artist select | Click a name in the roster list         | ↑ ↓ arrows, Enter      |
| Album select  | Click an album in the list              | ↑ ↓ arrows, Enter      |
| Track select  | Click a track bar, ▶ Play All button    | ↑ ↓ arrows, Enter      |
| Story page    | Scroll to read, player bar controls below | —                    |

---

## 🔧 Common Tweaks (index.html — P5R theme)

| What                          | Where in `index.html`                                  |
|-------------------------------|-----------------------------------------------------------|
| Brand name / title screen text| `TitleScreen` function — "APECOFT" / "MUSIC" / credit line |
| Brand colors (red/black/white)| `const RED`, `BLACK`, `WHITE` near the top of the script    |
| Page `<title>`                | `<title>APECOFT MUSIC — Music Realm</title>` in `<head>`    |
| Transition per screen         | `TRANSITIONS` object inside `App()`                          |
| Wipe transition styles        | `function Wipe({phase,variant,color})`                       |
| Jagged cutout shapes          | `JAG1`, `JAG2`, `JAG3` (clip-path constants)                  |
| Ransom-note name lettering    | `function RansomText`                                          |
| Mobile breakpoint              | `useIsMobile()` → `window.innerWidth<820`                    |

---

## ⚠️ Troubleshooting

**"My CSV changes don't show up"**
→ You're probably opening via `file://` (double-click). Use Live Server.

**"Audio doesn't play"**
→ Check the path in `songs.csv` matches the actual filename in `audio/` exactly (case-sensitive on some servers). Browsers may also block autoplay until you interact with the page once.

**"Thai/unicode text looks broken"**
→ Make sure the CSV is saved as **UTF-8** (in Excel: Save As → CSV UTF-8).

**"A song plays but the progress bar doesn't move"**
→ That song has no `audio` path — the play button just toggles the UI state in that case.
