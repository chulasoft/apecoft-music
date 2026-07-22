# TODO.md

Read [`/CONTEXT.md`](../CONTEXT.md) and [`SKILL.md`](SKILL.md) first. Ordered by priority (highest first). Each item links back to the doc with the relevant background — check there before starting.

## High priority

1. **Decide whether `index-noir.html` should get the sunburst-removal treatment.** A prior change removed the spinning `Burst` effect from `SongSelect`/`StoryPage` in `index.html` only. `index-noir.html` has its own ambient effects (`Bloom` glow, `Particles` drift) that were never evaluated against the same "floating light I don't like" feedback. See [FEATURE.md](FEATURE.md#locked-to-one-theme) and [ARCHITECTURE.md](ARCHITECTURE.md). Needs a decision from whoever owns the noir theme, not an assumption.
2. **No verification path for CDN-dependent rendering in restricted environments.** This repo's only runtime dependency (`cdnjs.cloudflare.com` for React/ReactDOM/Babel) is unreachable in some sandboxed agent environments (confirmed firsthand — proxy policy blocks it here). There's currently no offline fallback and no lightweight way to visually verify a change beyond a syntax check. Worth deciding: vendor a local copy for dev/agent use (opt-in, not default — see hard rule #1 in [SKILL.md](SKILL.md)), or accept the limitation and document the syntax-check workaround more prominently.

## Medium priority

3. **No automated tests or CI.** Every change today is verified manually (open in a browser, click through). At minimum, a smoke check (does the script parse, do the three CSVs load and join without orphaned rows) would catch schema-shape regressions early. See [FEATURE.md](FEATURE.md#stub--known-gaps).
4. **Accessibility pass.** Keyboard nav (arrows/Enter/hold-to-mute) works, but there's no confirmed `aria-label`/focus-visible audit across screens. Screen-reader users get essentially no context beyond what's visually implied.
5. **Duplication risk between the two theme files.** Every fix made to `index.html` needs a manual judgment call about whether it also applies to `index-noir.html` — there's no shared module to fix once (see [ARCHITECTURE.md](ARCHITECTURE.md#no-build-step-by-design)). This is an accepted tradeoff for the "single portable file" deployment model, not necessarily something to "solve," but it's the source of most double-work on this repo — worth keeping in mind when scoping estimates.

## Low priority / nice-to-have

6. **Tablet-specific layout tuning.** Only one breakpoint (`useIsMobile()` at 820px) exists; there's no dedicated in-between layout for tablet-width screens.
7. **Content-authoring ergonomics.** Editing CSVs by hand (with `|`/`||` separators and CSV quoting rules) works but is error-prone for non-technical contributors — README documents the rules clearly, but there's no validation step that catches a malformed row before it silently drops (see [DATABASE.md](DATABASE.md#derived-in-memory-shape-what-components-actually-consume) — orphan rows fail silently by design, which is convenient for robustness but unhelpful for catching a typo'd `artist_id`).
