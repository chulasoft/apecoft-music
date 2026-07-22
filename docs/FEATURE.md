# FEATURE.md

Read [`/CONTEXT.md`](../CONTEXT.md) and [`SKILL.md`](SKILL.md) first. Status legend: **Shipped** (working, in active use) · **Locked** (theme-specific — only exists/maintained in one of the two HTML files) · **Stub/Gap** (missing, partial, or a known limitation worth knowing before you touch it).

## Shipped — core flow (`index.html`, mirrored conceptually in `index-noir.html`)

| Feature | Files / functions | Notes |
|---|---|---|
| Title / tap-to-enter splash | `TitleScreen` | Also kicks off background intro music on first tap (`startBgMusic`). |
| Artist roster select | `ArtistSelect` | Keyboard: ↑↓ to move, Enter to select, ← to go back. |
| Album select | `AlbumSelect` | Same keyboard pattern as roster. |
| Track/song select ("Song Select") | `SongSelect` | Keyboard nav + **Play All** (`onPlayAlbum` → `playAlbum()` in `App()`, queues the rest of the album via `queue.current`). |
| Story page | `StoryPage` | Liner-notes/story paragraphs + quote for the selected song; prev/next song via `onChangeSong`; triggers `ShareCard`. |
| Persistent mini player | `MiniPlayer` | Shown on every screen except `title` and `story`; shows now-playing cover/title/artist + play/pause, or a BGM toggle when nothing is loaded. |
| Screen transitions ("wipes") | `Wipe`, `TRANSITIONS`/`BACK_TRANSITIONS`/`DEPTH` maps in `App()` | Direction-aware: forward vs. back transitions differ per destination screen. |
| Audio playback + queue | `useAudio()`, `App()`'s `queue`/`playAlbum`/`changeSong` | Single shared `<audio>` element; `onEnded` advances the queue. |
| Background intro music | `App()` (`bgMusic` ref, two `useEffect`s, `startBgMusic`/`toggleBg`) | Loops quietly, auto-pauses when a real track plays or the user mutes it. |
| Hold-to-mute gesture | `useHoldKey`, `onMuteHold`/`muteActive` in `App()` | Tap = normal action, hold ~2s = mute whichever audio source is currently audible. |
| Shareable poster card | `ShareCard` | Renders a 1080×1350 canvas poster (cover art + accent duotone + wrapped text); offers `navigator.share` where supported, else image download. |
| CSV-driven content | `parseCSV`, `buildRealm`, `loadRealmData` | Fetches `data/*.csv` at runtime; see [DATABASE.md](DATABASE.md) for schema. |
| Offline/fetch-failure fallback | `DEMO_REALM`, `mkSong` | Used automatically when the CSV `fetch()`s fail (e.g. `file://`). Same shape as CSV-derived data — every screen is agnostic to which source it came from. |
| Missing-asset fallback | `FALLBACK_IMG` | Blank `image`/`cover` in the CSVs (or in `DEMO_REALM`) resolves to this embedded placeholder — nothing breaks visually. |
| Mobile-responsive layout | `useIsMobile()` | Single breakpoint at `window.innerWidth < 820`; every screen branches its layout on this flag. |

## Locked to one theme

| Feature | File | Notes |
|---|---|---|
| Red/black poster visual theme | `index.html` | Jagged cutout shapes (`JAG1`/`JAG2`/`JAG3`), halftone dots, ransom-note lettering (`RansomText`), star-shaped wipes. This is the **active** theme per `README.md`. |
| Dark editorial visual theme | `index-noir.html` | Warm cream/gold on near-black, `Bloom` (radial-gradient glow) component, `Particles` (small floating dust dots via a `float` keyframe), fan-flip song cards. **Not** kept in sync with `index.html` — it has its own independent copy of every component. |
| Spinning sunburst removal | `index.html` only | The rotating conic-gradient `Burst` effect was removed from `SongSelect` and `StoryPage` in `index.html` in a prior change. `index-noir.html`'s own ambient effects (`Bloom`, `Particles`) were **not** touched or evaluated against the same complaint — flag this explicitly if asked to "polish" the noir theme too. |

## Stub / known gaps

| Gap | Notes |
|---|---|
| No automated tests | No test runner, no test files, anywhere in the repo. |
| No lint/format config | No ESLint/Prettier config — see [STYLE_GUIDE.md](STYLE_GUIDE.md) for why this is intentional, and don't add one without discussing it. |
| No CI | No `.github/workflows` or equivalent. |
| No formal accessibility audit | Keyboard nav exists (arrows/Enter/hold) but there's no verified `aria-*` labeling or focus-visible styling pass. |
| Single-breakpoint responsiveness | Only a mobile/desktop split at 820px — no dedicated tablet tuning. |
| CDN dependency | Both HTML files require `cdnjs.cloudflare.com` reachability on load (React/ReactDOM/Babel). No offline/self-hosted fallback for those scripts exists today — see [TODO.md](TODO.md). |
