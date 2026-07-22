# ARCHITECTURE.md

Read [`/CONTEXT.md`](../CONTEXT.md) and [`SKILL.md`](SKILL.md) first. This doc explains how the pieces fit together: directory layout, data flow, module contracts, and deployment.

## Directory layout

```
apecoft-music/
├── index.html          ← active app: HTML shell + CDN <script> tags + one big <script type="text/babel"> block
├── index-noir.html     ← alternate app: same shape, independent copy, different visual theme
├── README.md            ← human/editor docs (content editing workflow, asset specs, troubleshooting)
├── CONTEXT.md
├── docs/
│   ├── SKILL.md
│   ├── ARCHITECTURE.md  ← this file
│   ├── STYLE_GUIDE.md
│   ├── FEATURE.md
│   ├── DATABASE.md
│   └── TODO.md
├── data/
│   ├── artists.csv
│   ├── albums.csv
│   └── songs.csv
├── audio/
│   ├── bgintro.mp3      ← quiet looping intro/idle music
│   └── *.mp3 / *.wav    ← per-song tracks, referenced by songs.csv's `audio` column
└── covers/
    └── *.png / *.jpg     ← artist portraits + album/song cover art
```

There is no `src/`, no `dist/`, no `node_modules/` — each `index*.html` file is the entire application: markup, styles (inline + a small `<style>` block of `@keyframes`), and logic.

## No build step, by design

Both HTML files load, in `<head>`:

```html
<script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.2/babel.min.js"></script>
```

...then a single `<script type="text/babel">…</script>` block containing all app code as JSX. Babel Standalone transpiles it **in the browser at load time** — there's no pre-built bundle to go stale, but it does mean:

- The app needs internet access on first paint (to fetch those three CDN scripts). If that's blocked (as it is in some sandboxed agent environments), the app won't render — that's an environment limitation, not a code defect.
- JSX syntax errors only surface at runtime in the browser console, not at "compile time." When you can't open a real browser, a lightweight sanity check is running the extracted script through `tsc --noEmit --jsx react --allowJs --checkJs false --skipLibCheck` and grepping for `TS1xxx` (syntax) errors — type errors (`TS2xxx`/`TS7xxx`) are expected noise since the code is untyped JS.

## Data flow

```
data/*.csv  --fetch()-->  parseCSV()  --rows-->  buildRealm(A, AL, S)  --nested tree-->  React state (App's `realm`)
                                                        │
                                          (on fetch failure, e.g. file://)
                                                        ▼
                                                  DEMO_REALM (hardcoded, same shape)
```

- **`parseCSV(text)`** — hand-rolled CSV parser (handles quoted fields, `""` escaped quotes, CRLF/LF). No library.
- **`buildRealm(A, AL, S)`** — joins the three flat row arrays into a nested tree: `artists[] → albums[] → songs[]`, splitting `tags` on `|` and `story_paragraphs` on `||`, and resolving missing `image`/`cover` to `FALLBACK_IMG` (an embedded base64 placeholder — this is the giant single-line `const FALLBACK_IMG = "data:image/png;base64,…"` near the top of the script).
- **`DEMO_REALM`** — a hardcoded tree built with the `mkSong(...)` helper, used only when the three `fetch()` calls fail (e.g. opened via `file://`, or CSVs missing). It must stay structurally identical to what `buildRealm` produces, since every screen component consumes `realm` without checking which source it came from.
- Full column-level schema lives in **[DATABASE.md](DATABASE.md)**.

## Screen / state model

`App()` is a hand-rolled state machine — there is no router library:

- `screen` state: `"title" | "artists" | "albums" | "songs" | "story"`.
- `goTo(next, payload)` sets a `Wipe` transition variant (from `TRANSITIONS`/`BACK_TRANSITIONS`, chosen by comparing `DEPTH[next]` vs `DEPTH[screen]` to detect "going back"), stages `payload` (artist/album/song) in a ref, plays the wipe-in, then swaps `screen` + commits the payload mid-transition, then wipes out.
- Each screen is a top-level function component taking the relevant slice of data plus callbacks (`onSelect`, `onBack`, `audio`, `onMuteHold`, …) — see the table below.
- A persistent **`MiniPlayer`** renders on every screen except `title` and `story` (the story page has its own inline player controls).
- A **`Wipe`** overlay (fixed, z-index 999) renders the poster-style transition animation between every screen change.

| Screen (function) | Shows | Key props in |
|---|---|---|
| `TitleScreen` | Splash / tap-to-enter | `onEnter` |
| `ArtistSelect` | Roster of artists | `artists`, `onSelect`, `onBack`, `audio`, `onMuteHold` |
| `AlbumSelect` | One artist's albums | `artist`, `onSelect`, `onBack`, `audio`, `onMuteHold` |
| `SongSelect` | One album's tracklist, "Play All" | `artist`, `album`, `onSelectSong`, `onPlayAlbum`, `onBack`, `audio`, `onMuteHold` |
| `StoryPage` | One song's liner notes/story, share card, prev/next | `artist`, `album`, `song`, `songIdx`, `onChangeSong`, `onBack`, `audio`, `onMuteHold` |
| `LoadingScreen` | Shown while `realm` is still `null` | — |

## Shared building blocks

- **`useAudio()`** — wraps a single `<audio>` element (created once in a `useEffect`) with `playing`/`progress`/`curSrc` state, `playSrc`/`toggle`/`seek`, and an `onEnded` callback slot used by `App()` to advance a play-queue (`queue.current`, filled by `playAlbum()`).
- **Background intro music** — a second, independent `Audio("audio/bgintro.mp3")` managed directly in `App()` (looped, low volume), auto-paused whenever a real track plays or the user mutes it, resumed otherwise.
- **`useHoldKey(targetKey, {onTap, onHold, ms})`** — generic press-vs-hold keyboard handler; used for the hold-to-mute gesture (`onMuteHold`/`muteActive` in `App()`).
- **`useIsMobile()`** — single breakpoint (`window.innerWidth < 820`) driving the mobile/desktop layout branches sprinkled through each screen.
- **`ShareCard`** — draws a 1080×1350 shareable poster to an off-screen `<canvas>` (cover art + accent-tinted duotone + wrapped title/quote text), then offers `navigator.share` (if available) or a direct image download.
- **Visual primitives** — `Halftone`, `Stripes`, `Star`, `OutlineText`, `SplitLabel`, `Callout`, `RansomText`, `Tag`, `P5Button`, `BackBtn`, `KeyHints`: small, reusable, purely presentational components threaded with each screen's `accent` color for per-artist/album/song theming.

## Deployment model

Per `README.md`, three supported ways to run it, all with zero build step:

1. **Live Server / any local static server** (recommended) — CSV `fetch()` works, full content loads.
2. **Double-click the HTML file** — opens over `file://`; CSV `fetch()` is blocked by the browser, so it silently falls back to `DEMO_REALM`. Audio still plays (relative paths still resolve for `<audio>`/`<img>` under `file://` in most browsers).
3. **Any static host** (e.g. Walrus, GitHub Pages, S3) — upload the folder as-is; everything works including CSV loading.

Switching the active theme is a file rename (swap `index.html` ↔ `index-noir.html`), not a config flag — reinforcing that the two are independent, complete apps rather than one app with a theme switch.
