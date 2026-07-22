# DATABASE.md

Read [`/CONTEXT.md`](../CONTEXT.md) and [`SKILL.md`](SKILL.md) first. There is no real database — **the three CSV files in `data/` are the database.** This doc is the canonical schema reference; `README.md` has the same column list for content editors, keep both in sync if you change anything here.

## Source tables (`data/*.csv`)

All three are fetched at runtime (`fetch("data/....csv")`) and parsed by the hand-rolled `parseCSV()` — standard CSV quoting rules apply: wrap a field in `"double quotes"` if it contains a comma or line break, escape an inner quote as `""`.

### `artists.csv`

| Column | Type | Notes |
|---|---|---|
| `id` | string | Referenced by `albums.csv.artist_id`. Must be unique. |
| `name` | string | Display name. |
| `role` | string | e.g. `"Vocalist · Songwriter"`. |
| `tagline` | string | Short one-liner shown on the roster. |
| `bio` | string | Longer paragraph. |
| `accent` | hex color | Bright per-artist theme color. |
| `bloom` | hex color | Slightly deeper glow/companion color. |
| `dark` | hex color | Very dark shade of the same hue, for backgrounds. |
| `tags` | `\|`-separated string | e.g. `Warm\|Nostalgic\|Storyteller`. |
| `image` | path or blank | Relative to the HTML file, e.g. `covers/apcr.png`. Blank → `FALLBACK_IMG`. |
| `debut` | string | Debut year, shown in stats. |

### `albums.csv`

| Column | Type | Notes |
|---|---|---|
| `id` | string | Referenced by `songs.csv.album_id`. Must be unique. |
| `artist_id` | string | Must match an `artists.csv.id`; unmatched rows are silently dropped by `buildRealm`. |
| `title` | string | |
| `year` | string | |
| `issue` | string | Roman-numeral-style catalog marker (e.g. `"I"`, `"II"`) shown as a badge. |
| `desc` | string | |
| `accent` | hex color | |
| `bloom1`, `bloom2` | hex color | Two-stop glow gradient. |
| `tags` | `\|`-separated string | |
| `cover` | path or blank | Blank → `FALLBACK_IMG`. |

### `songs.csv`

| Column | Type | Notes |
|---|---|---|
| `id` | string | Unique. |
| `album_id` | string | Must match an `albums.csv.id`; unmatched rows are silently dropped. |
| `n` | string | Track number, e.g. `"01"`. |
| `title` | string | |
| `dur` | string | `"m:ss"` display string, not computed from the audio file. |
| `genre` | string | |
| `accent`, `g1`, `g2` | hex color | `accent` is the primary theme color; `g1`/`g2` are gradient stops used in track-list/story decoration. |
| `tags` | `\|`-separated string | |
| `desc` | string | Short blurb shown in the track list. |
| `audio` | path or blank | Relative to the HTML file, e.g. `audio/01.mp3`. **Blank means no playback** — the play button becomes a UI-only toggle (progress bar never moves). Filename must match exactly, case-sensitive on some hosts. |
| `cover` | path or blank | Optional. Blank → falls back to the **album's** cover automatically (not `FALLBACK_IMG` directly — see `buildRealm`). |
| `story_title` | string or blank | Blank → falls back to the song's `title`. |
| `story_paragraphs` | `\|\|`-separated string | Each `\|\|`-delimited chunk becomes one paragraph on the story page. |
| `story_quote` | string | Pull-quote shown on the story page. |

## Derived in-memory shape (what components actually consume)

`buildRealm(A, AL, S)` (called from `loadRealmData()`) joins the three flat row arrays into a nested tree and is what every screen component's props actually look like:

```
realm: Artist[]

Artist = {
  id, name, role, tagline, bio, accent, bloom, dark,
  tags: string[],           // split from "tags" on "|"
  image: string,             // resolved, never blank (FALLBACK_IMG if missing)
  stats: { debut, albums: number, songs: number },  // albums/songs counts computed after the join
  albums: Album[],
}

Album = {
  id, artistId, title, year, issue, desc, accent, bloom1, bloom2,
  tags: string[],
  cover: string,              // resolved, never blank
  songs: Song[],
}

Song = {
  id, n, title, dur, genre, accent, g1, g2,
  tags: string[],
  desc,
  audioSrc: string | null,    // null means "no playback", not an error state
  cover: string,               // song's own cover if set, else the parent album's cover
  story: { title, paragraphs: string[], quote },
}
```

Two important join behaviors to preserve if you touch `buildRealm`:

- **Orphan filtering**: an album with no matching `artist_id`, or a song with no matching `album_id`, is silently dropped (not an error). An artist with zero albums after the join is filtered out of the final `realm` array entirely (`artists.filter(a => a.albums.length > 0)`).
- **Stats are computed, not stored**: `stats.albums`/`stats.songs` are counted after assembly, not read from any CSV column.

## Fallback data (`DEMO_REALM`)

When any of the three `fetch()` calls fail or return non-OK (most commonly: opened via `file://`, where `fetch` on local files is blocked by the browser), `loadRealmData()` catches the error and returns `DEMO_REALM` instead — a hardcoded array built with the `mkSong(id,n,title,dur,genre,accent,g2,tags,desc,audio,paras,quote)` helper, matching the exact shape above (`g1` is set equal to `accent` inside `mkSong`). Every screen component is written against the shape, not the source, so it never needs to know whether data came from CSV or the fallback.

## Asset resolution rules

- `image` (artists), `cover` (albums/songs) — all paths are relative to the HTML file itself (e.g. `covers/minta.png`), not to `data/`.
- `audio` — same, relative to the HTML file (e.g. `audio/01.mp3`).
- Any of the three left blank in the CSV degrades gracefully: missing image/cover → `FALLBACK_IMG`; missing song cover → parent album's cover; missing audio → playback UI becomes inert but nothing crashes.
