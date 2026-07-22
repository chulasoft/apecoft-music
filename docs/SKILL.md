---
name: apecoft-music-guide
description: Map and hard rules for working in the apecoft-music repo (a no-build-step, CDN-React/Babel, CSV-driven interactive music portfolio). Load this before editing index.html, index-noir.html, or data/*.csv, or when asked about the project's architecture, data schema, feature status, or style conventions.
---

# apecoft-music — Agent Skill

Read [`/CONTEXT.md`](../CONTEXT.md) first if you haven't. This file is the detailed map: what each doc covers, when to read it, and the full rule set (CONTEXT.md's rules are the short version of the same rules).

## Reading order

1. **[/CONTEXT.md](../CONTEXT.md)** — mandatory, always first. What the project is, rough structure, hard rules.
2. **This file** — the map + detailed rules.
3. From here, jump directly to whichever doc matches your task (table below) — no need to read all of them front-to-back.

## Doc map

| File | Location | Job | Read when |
|---|---|---|---|
| `CONTEXT.md` | root | Mandatory kickoff — what the project is, rough structure, hard rules, points here | Before any other file, every task |
| `docs/SKILL.md` | docs/ | This file — detailed map + full rules, portable skill format | Right after CONTEXT.md |
| `docs/ARCHITECTURE.md` | docs/ | How the pieces fit together — directory layout, data flow, screen/module contracts, deployment | Before editing structure or adding a screen/module |
| `docs/STYLE_GUIDE.md` | docs/ | Naming, formatting, framework quirks specific to this repo | Before writing code |
| `docs/FEATURE.md` | docs/ | Every feature's status (shipped / partial / known-limitation) mapped to the files/functions behind it | When you need to know what already exists and where |
| `docs/DATABASE.md` | docs/ | The CSV schema (the actual "database") plus the derived in-memory shape components consume | Before touching `data/*.csv`, `parseCSV`, `buildRealm`, or `mkSong` |
| `docs/TODO.md` | docs/ | Remaining/open work, priority-ordered, linking back to FEATURE/ARCHITECTURE | When picking up the next task |

## Hard rules (full version)

1. **No build step.** This repo intentionally has no `package.json`, no bundler, no transpile step outside the browser. `index.html`/`index-noir.html` each load React 18 UMD + ReactDOM UMD + Babel Standalone from `cdnjs.cloudflare.com` via `<script src>`, then contain one `<script type="text/babel">` block with all app JSX. Do not "fix" this by adding tooling — that would break the "download the folder, open it, it works" deployment model documented in `README.md`.
2. **Two themes, zero shared code.** `index.html` (active) and `index-noir.html` (alternate) are complete, independent files — every component (`Wipe`, `useAudio`, `ShareCard`, etc.) is duplicated between them with theme-specific styling. A change to one is invisible to the other. When a task names one file explicitly (as most do), touch only that file. When a task is about a bug/behavior rather than a look, ask whether it should be ported to both before doing so.
3. **CSV is the database.** `data/artists.csv`, `data/albums.csv`, `data/songs.csv` are fetched at runtime and are the single source of truth for content. `DEMO_REALM` (hardcoded in each HTML file) is the *only* sanctioned hardcoded content — it's the `file://`/fetch-failure fallback and must stay shape-compatible with the CSV-derived data (see DATABASE.md). Never hardcode real artist/album/song content into a component.
4. **Schema changes are cross-cutting.** Adding/renaming/removing a CSV column requires updating, together: the CSV header + all data rows, `parseCSV`/`buildRealm`/`mkSong` in **both** HTML files (if the column is theme-relevant), `README.md`'s column tables, and `docs/DATABASE.md`. A schema change that only touches the code is incomplete.
5. **Match house style, don't reformat.** Dense inline `style={{...}}`, no CSS files/classes, PascalCase components, `useX` hooks, `ALL_CAPS` constants for colors/shapes, `// ═══ SECTION ═══` banner comments. Full detail in `docs/STYLE_GUIDE.md`. Do not run Prettier/ESLint autofix across the file — it will blow up the diff and fight the intentional density.
6. **Scope effects to the named screen.** Shared primitives (`Stripes`, `Halftone`, `Burst`-style effects, `Wipe` variants) are reused across screens. When asked to change how something looks on "the story page" or "the song list," find and edit only the JSX for those screen components (`StoryPage`, `SongSelect`, etc.) — don't touch a shared primitive's definition unless the change is meant to apply everywhere it's used.
7. **Sandbox network limits are not app bugs.** Some execution environments block `cdnjs.cloudflare.com` (proxy policy). If the app fails to render there, say so and verify via static analysis (e.g. `tsc --noEmit --jsx react` on the extracted script for a syntax check) instead of concluding the change is broken.
8. **Don't commit without being asked**, and don't push destructive git operations — standard repo hygiene applies on top of everything above.

## Quick orientation for common tasks

- **"Change how X looks on screen Y"** → find `function Y(...)` in the target HTML file, edit the JSX there. Check `docs/FEATURE.md` for which file/function owns the screen.
- **"Add/edit an artist, album, or song"** → edit `data/*.csv` per `docs/DATABASE.md` and `README.md`, not the HTML.
- **"Something about audio/playback"** → `useAudio()` hook + `MiniPlayer` component + `App()`'s background-music effects (both HTML files, independently).
- **"Something about transitions between screens"** → `Wipe` component + `TRANSITIONS`/`BACK_TRANSITIONS`/`DEPTH` maps inside `App()`.
- **"What's left to do"** → `docs/TODO.md`.
