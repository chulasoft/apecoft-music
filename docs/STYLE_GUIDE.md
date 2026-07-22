# STYLE_GUIDE.md

Read [`/CONTEXT.md`](../CONTEXT.md) and [`SKILL.md`](SKILL.md) first. Conventions here are descriptive of the existing code, not aspirational — match them, don't "improve" them wholesale.

## No reformat rule

There is no Prettier/ESLint config in this repo, on purpose — the density is intentional (a single portable file, easy to skim in sections via the banner comments below). **Never run a formatter across `index.html`/`index-noir.html`.** Edit in place, matching the surrounding lines byte-for-byte in style.

## Naming

- **Components**: `PascalCase` function declarations — `function StoryPage(...)`, `function ArtistSelect(...)`. Small purely-presentational ones are `const PascalCase = (props) => (...)` arrow functions — e.g. `const Halftone = ({opacity=.5,size=9}) => (...)`.
- **Hooks**: `useX` — `useAudio()`, `useIsMobile()`, `useHoldKey(...)`.
- **Constants**: `ALL_CAPS` for fixed values — colors (`RED`, `BLACK`, `WHITE`), shape data (`JAG1`, `JAG2`, `JAG3` clip-path polygons), config maps (`TRANSITIONS`, `BACK_TRANSITIONS`, `DEPTH`), the `LABEL` string, `STRIPE_BG`, `FALLBACK_IMG`, `DEMO_REALM`.
- **Local variables**: short, terse `camelCase` — `al` (album), `a` (artist), `s`/`song`, `cur`, `g` (canvas 2D context), `p` (payload/particle/paragraph, contextual). Favor brevity over spelled-out names in tight scopes (loops, small callbacks); use full words for anything that outlives a few lines (props, state).

## Formatting

- Double quotes for strings (`"absolute"`, not `'absolute'`); template literals with backticks for interpolation (`` `${a.accent}0d` ``).
- No space after `:` or `,` inside inline style objects and tight literals: `{position:"absolute",top:0,left:0}`. This is consistent house style, not a mistake — keep it when adding to an existing object.
- Semicolons always, including after single-statement arrow bodies.
- 2-space indentation.
- JSX conditionals use `&&` for presence checks and ternaries for either/or — no early-return-heavy JSX; the `return (...)` is usually one JSX tree with inline `{cond && <X/>}` blocks.
- Multi-branch visual logic (e.g. `Wipe`'s per-variant rendering) uses a sequence of `if (variant==="x") return <.../>;` blocks inside one component rather than splitting into many components — keep new variants in the same shape.

## Section banners

Large logical sections are marked with a banner comment, used for navigating the ~2000-line script without a symbol index:

```js
// ═══ AUDIO (with queue support) ═══
// ═══ THEME PRIMITIVES ═══
// ═══ RESPONSIVE ═══
// ═══ KEYBOARD HINTS (bottom-left) — same chip style as the track list ═══
```

Add a new banner when introducing a genuinely new section; don't add one for a single small helper.

## Comment style

Comments are sparse and explain **why**, not what — a hidden constraint, an ordering dependency, or a non-obvious interaction:

```js
// slow drift for background cutout shapes — float only, never scale (clip-path edges would tear)
// → tap: play the highlighted song (toggles if it's already the one loaded) · hold 2s: mute
```

Don't add comments that restate the JSX/CSS in words. Don't reference task/ticket context ("added for X's request") — that belongs in the commit message, not the file.

## Framework-specific quirks

- **React 18 UMD global**, not an ES module import — `const {useState,useRef,useEffect,useCallback} = React;` at the top of the script, and `React.Fragment`/`React.useMemo` used via the full `React.` prefix elsewhere (mixed with the destructured hooks — both are fine, follow whichever the surrounding function already uses).
- **Babel Standalone** transpiles `<script type="text/babel">` at runtime — write normal JSX, but remember there is no type checking and no build-time lint; a typo only shows up in the browser console at runtime.
- **Inline styles only** — every visual property is a `style={{...}}` object; there are no CSS classes or CSS Modules to add rules to. A small `<style>` block in `<head>` holds only `@keyframes` and a couple of global resets (`button{...}`) — add new keyframes there, not new selectors.
- **Accent color threading** — nearly every screen component takes (directly or via `song`/`album`/`artist`) an `accent` (and sometimes `bloom`/`g1`/`g2`) hex string and threads it into `style` props so the UI retints per artist/album/song. When adding new decorative elements to a screen, thread the existing accent through rather than hardcoding a color.
- **`pointerEvents:"none"` on decorative layers** — purely visual absolutely-positioned divs (stripes, halftone, cutout shapes) consistently set `pointerEvents:"none"` so they never intercept clicks/taps. Keep this on any new decorative element.
