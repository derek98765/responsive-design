# Responsive patterns — full reference

Detailed how-to for each pattern indexed in [SKILL.md](../SKILL.md). Section
numbers (§N) match the index there.

## Reading the samples

Samples are **project-agnostic**. Translate every placeholder through the
project's `.claude/responsive-design.config.md` before writing code.

- **Spacing** is written as arbitrary pixel values (`gap-[16px]`, `px-[20px]`)
  so it is unambiguous in any project. Replace each with the project's spacing
  utility for that pixel value — and check the scale: in default Tailwind
  `gap-4` is 16px, but some projects define 1px-literal steps where `gap-16`
  is 16px.
- **Colour, radius and shadow** use placeholder tokens:

  | Placeholder | Role |
  |---|---|
  | `--surface` | card / panel / sheet fill (usually white) |
  | `--surface-muted` | tinted panel fill (table rows, stat cards) |
  | `--line` | default hairline — card outline, row divider |
  | `--line-subtle` | lightest hairline — a rule inside an already-bordered container |
  | `--text-title` / `--text-body` / `--text-label` | heading / body copy / the small word that introduces a value |
  | `--radius-md` / `--radius-lg` | ~12px / ~20px corners |
  | `--shadow-tint` | the RGB the project's drop shadows are tinted with |
  | `--header-offset` | height a sticky element must clear below a fixed header |

- **Type** uses placeholder classes `type-heading`, `type-value`, `type-body`,
  `type-label` (uppercase micro-label). Map to the project's type tokens.
- **Components** (`<Pill>`, `<Dialog>`, `<Button>`) mean "the project's own
  component for this" — find it, don't hand-roll one.

## §1 Progressively reduce, don't just shrink

At each breakpoint going down, reduce:
- **Padding** — ~50–60% of desktop. `px-[80px]` → `md:px-[48px]` → `px-[20px]`; `py-[48px]` → `md:py-[32px]` → `py-[20px]`.
- **Gaps** — tighten vertical rhythm. `gap-[48px]` → `md:gap-[32px]` → `gap-[24px]` (between sections); `gap-[24px]` → `md:gap-[20px]` → `gap-[16px]` (inside sections).
- **Font sizes** — drop one tier per breakpoint. Free if the type tokens auto-scale: use the token and add nothing. Body-size type (and smaller) usually sits at a legibility floor (~13–14px) and should hold its size — check which tiers the project scales before shrinking anything.
- **Fixed widths** — replace with `w-full` or `max-w-*`.

## §2 Reshape, don't squeeze

- **Multi-column grids** → step down one column at a time (4→3→2→1, 3→2→1, 2→1). Don't jump straight to 1.
- **Side-by-side rows** → `flex-col md:flex-row`.
- **Wide tables** → card-per-row on mobile (§6).
- **Inline labels + values** → stack label above value when space runs out.

### Column reduction — Tailwind patterns
```jsx
className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-[16px] md:gap-[24px]"   // 3-col
className="grid grid-cols-1 md:grid-cols-2 gap-[16px] md:gap-[24px]"                   // 2-col
```
Min readable cell width: ~260–320px for content cards, ~160px for compact stats. Drop a column when `viewport / columns < min`.

### Awkward near-full-width containers → stretch to full width

If a `w-fit` / `inline-flex` container ends up ≳ 70% of viewport on mobile (wide enough to dominate, narrow enough to leave an ugly gutter), **stretch it to `w-full`** at that breakpoint. Also redistribute inner columns equally:

```jsx
<div className="w-full md:w-fit flex items-stretch md:items-center gap-[20px] md:gap-[48px] ...">
  <div className="flex-1 md:flex-initial flex flex-col gap-[8px] md:gap-[12px]">…</div>
  <div className="w-px self-stretch bg-[var(--line)] shrink-0" aria-hidden />
  <div className="flex-1 md:flex-initial flex flex-col gap-[8px] md:gap-[12px]">…</div>
</div>
```

Don't apply this to individual buttons or pills — only to container/group elements (stats bars, tab strips, filter rows).

## §3 Sticky side panels → sticky bottom section on mobile

A desktop sticky right rail (CTA, key stats, eligibility panel) must stay reachable on mobile. Two approaches:

**Option A — `position: sticky` (preferred when panel is tall or has scrollable content):** The panel stays in document flow and pins only when it reaches the bottom of the viewport. Cancel parent padding with negative margins to go edge-to-edge.

```jsx
<aside className="
  sticky bottom-0 z-30 bg-[var(--surface)]
  rounded-t-[var(--radius-lg)] shadow-[0_-8px_24px_rgba(var(--shadow-tint),0.08)]
  -mx-[20px] md:-mx-[48px]                  /* = parent's padding → edge-to-edge */
  max-h-[80dvh] overflow-y-auto scrollbar-none
  lg:sticky lg:bottom-auto lg:top-[var(--header-offset)]
  lg:rounded-[var(--radius-lg)] lg:shadow-none lg:bg-transparent lg:mx-0
  lg:max-h-[calc(100dvh-var(--header-offset))]
">
```

- Match the negative margin to the **actual** parent padding at every breakpoint.
- The mobile sheet casts its shadow **upward**. Most elevation tokens only cast downward — if the project has no upward one, reuse its shadow tint flipped upward rather than a neutral black drop, and propose a token if the pattern repeats.

**Option B — `position: fixed`:** Use only for small, always-visible bars (price + CTA). Add `pb-[calc(barHeight+env(safe-area-inset-bottom))]` on the page so content isn't hidden behind it.

### Collapsible header bar

When the panel contains substantial content, add an expand/collapse header on mobile. Use a 3-column grid so the title stays visually centered with the chevron on the right:

```jsx
<button
  className="lg:hidden grid w-full grid-cols-[20px_1fr_20px] items-center gap-[8px] px-[20px] py-[20px] border-b border-[var(--line-subtle)] ..."
  onClick={() => setExpanded(v => !v)}
>
  <span aria-hidden />                          {/* spacer — same width as chevron */}
  <span className="flex items-center justify-center gap-[8px]">
    {/* icon + title */}
  </span>
  <ChevronDown className={expanded ? "rotate-180" : ""} aria-hidden />
</button>
<div className={`${expanded ? "block" : "hidden"} lg:block`} id="panel-content">
  {/* panel body */}
</div>
```

Default state: **expanded**. No drag handle (this is not a draggable sheet modal). Use the **lightest** hairline (`--line-subtle`) for the header divider — not the default card-outline line.

**Don't pin** decorative or low-priority rails (related content, ads). Only convert when the panel holds a primary action or critical context.

## §4 Full-bleed images inside padded containers

When a hero/banner image should go edge-to-edge on mobile but sits inside a padded page container, break it out with negative margins:

```jsx
<div className="-mx-[20px] w-[calc(100%+40px)] md:mx-0 md:w-full overflow-hidden md:rounded-[var(--radius-lg)]">
  <img className="block w-full h-[240px] md:h-auto object-cover object-center md:object-contain" ... />
</div>
```

- The negative margin and the `calc()` width must equal the parent's padding on each side (`-mx-[20px]` / `+40px` for a `px-[20px]` parent).
- Remove border radius on mobile (fully bleed), add it back on `md:`.
- Set a fixed height on mobile (`h-[240px]`) so the image reads as a designed banner, not a stretched photo.
- Use `object-cover object-center` to crop from the center of the source.

## §5 Keep tags, badges, and buttons on a single line

Always add `whitespace-nowrap` to pills/badges/chips. Pair with `shrink-0` when inside a flex row.

```jsx
<span className="inline-flex ... whitespace-nowrap shrink-0">No Tier</span>
```

Prefer the project's pill/badge component — check it already carries both; if not, add them to the **component's** base className so every usage is fixed.

If the label still overflows: reduce `px`, drop font size, shorten copy, or reshape the parent. Never let a pill wrap — it always looks broken.

## §6 Wide tables → card-per-row on mobile

Never horizontally scroll a data table. Transform it: each row becomes a card, column headers move inline above each value.

```jsx
{/* Desktop */}
<div className="hidden md:flex flex-col gap-[4px]">
  <div className="grid grid-cols-[140px_96px_1fr] px-[24px] py-[8px]"> {/* headers */} </div>
  <div className="flex flex-col gap-[4px] rounded-[var(--radius-md)] bg-[var(--surface-muted)] overflow-hidden">
    {rows.map(row => (
      <div key={row.id} className="grid grid-cols-[140px_96px_1fr] items-center px-[24px] py-[24px]"> {/* cells */} </div>
    ))}
  </div>
</div>

{/* Mobile card stack */}
<div className="flex md:hidden flex-col gap-[12px]">
  {rows.map(row => (
    <div key={row.id} className="flex flex-col gap-[16px] rounded-[var(--radius-md)] bg-[var(--surface-muted)] p-[20px]">
      <div className="grid grid-cols-2 gap-[20px]">           {/* equal-width for comparable cols */}
        <div className="flex flex-col gap-[4px]">
          <p className="m-0 type-label text-[var(--text-label)]">Tier</p>
          <Pill>{row.tier}</Pill>
        </div>
        <div className="flex flex-col gap-[4px]">
          <p className="m-0 type-label text-[var(--text-label)]">Rewards</p>
          <p className="m-0 type-value text-[var(--text-title)]">{row.reward}</p>
        </div>
      </div>
      <div className="flex flex-col gap-[4px]">
        <p className="m-0 type-label text-[var(--text-label)]">Who Qualifies</p>
        <p className="m-0 type-body text-[var(--text-body)]">{row.who}</p>
      </div>
    </div>
  ))}
</div>
```

The label colour is for the fixed word that introduces a value; the value itself takes body/title colour. If the project's label type token already sets uppercase/bold/tracking, don't re-add those utilities.

Use `grid grid-cols-2` inside cards when two columns are comparable (e.g. tier + reward) to guarantee a 50/50 split. For a narrow column inside a card, equal widths may give too much room — stack vertically instead.

## §7 Touch targets

Interactive elements need ≥ 44×44 CSS px — on the element, not just the icon.

- **Deliberately compact controls** (30–36px icon buttons, dense rows that are the better composition on a pointer device): grow the **hit area only**, not the painted box — an `::after` overlay (`absolute`, centred, `min-width/min-height: 44px`, transparent) under `@media (pointer: coarse)`. Check whether the project already ships a utility for this before writing one.
- **Everything else**: padding or `min-h-[44px]` / `min-w-[44px]`.

## §8 Text-over-image overlays

For **left text + right/background image** layouts, apply this 3-part pattern:

1. **Cap text column to ~70% on mobile** — reserve right 30% as image-only reveal. `w-[70%] md:w-full md:max-w-[720px]`.

2. **Left-anchored gradient overlay** — between `<img>` and text, `absolute inset-y-0 left-0`, `aria-hidden pointer-events-none`. Width slightly wider than text column (`w-[80%] md:w-[85%] xl:w-[70%]`). Match solid color to image's left-edge color.
   ```jsx
   style={{ background: "linear-gradient(to right, #dfe6f1 0%, #dfe6f1 35%, rgba(223,230,241,0.88) 65%, transparent 100%)" }}
   ```
   The hex here is **sampled from the image's left edge**, not a palette colour — that's the one place a literal is correct, because the gradient has to dissolve into that specific artwork. If the edge colour happens to match a project surface token, use the token instead.

3. **`object-position` per breakpoint** — for wide hero art with focal content on the right of the source: `object-[75%_center] md:object-right`. Higher % moves the crop window right, pulling right-side artwork *left* into the frame. If user says "move image right," raise %; if "character bleeding under text," lower %. Adjust in 5–10% increments.

## §9 Dev tooling

Check that any dev panel/FAB doesn't collide with mobile content (especially a sticky bottom panel). Move to a corner with `safe-area-inset` support.

## §10 Safe areas — notches, home indicators, rounded corners

Any `fixed`/`sticky` element pinned to a screen edge (bottom nav, sticky CTA bar, `position: fixed` panels from §3 Option B) needs `env(safe-area-inset-*)` padding, or it sits under the home indicator / notch on modern phones.

```jsx
// Bottom-pinned bar
<div className="fixed bottom-0 inset-x-0 pb-[max(16px,env(safe-area-inset-bottom))]">
  {/* bar content */}
</div>

// Page content behind a fixed bar needs matching bottom padding so nothing is hidden:
<main className="pb-[calc(80px+env(safe-area-inset-bottom))]">
```

Requires the viewport meta tag to opt in:
```html
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
```

Without `viewport-fit=cover`, `env()` values resolve to `0` and this is a no-op — check the meta tag exists before assuming the padding is working. Use `max(Npx, env(...))` (not `env()` alone) so there's still padding on devices/browsers that don't report an inset.

## §11 Pointer and hover detection — don't assume viewport width means touch

Screen size doesn't tell you input method — a laptop can have a touchscreen, a tablet can have a trackpad. Don't gate touch-friendly sizing behind `md:`/`lg:` breakpoints alone if the real distinction is input type.

```css
/* Coarse pointer (touch, stylus) → larger hit targets */
@media (pointer: coarse) {
  .button { padding: 12px 20px; }
}

/* Device has no hover capability → don't rely on :hover for critical info/actions */
@media (hover: none) {
  .card-actions { opacity: 1; } /* always show, don't wait for hover to reveal */
}
```

In Tailwind, reach for `pointer-coarse:`/`pointer-fine:` and `hover:`/(config-added) `no-hover:` variants where available, or arbitrary variants (`[@media(hover:hover)]:`) if not configured. Check for an existing project utility (§7) before writing a new query.

**Critical**: never make an action *only* reachable via `:hover` (revealed icons, hover-only menus) — touch users have no hover state at all. Any hover-revealed control needs a tap-visible fallback (always-visible, or revealed on tap/focus).

## §12 Responsive images — serve the right file, not just the right crop

§4 covers *cropping/treatment* per breakpoint (`object-cover`, fixed heights). Separately, for any non-trivial image (hero, banner, large photo), serve a *smaller file* on mobile — don't ship the same 1920px asset to a 390px viewport.

```html
<img
  src="hero-800.webp"
  srcset="hero-400.webp 400w, hero-800.webp 800w, hero-1200.webp 1200w"
  sizes="(max-width: 768px) 100vw, 50vw"
  alt="..."
>
```

For art direction (different crop/composition per breakpoint, not just resolution — e.g. a tall mobile crop vs. a wide desktop crop of the same scene), use `<picture>`:

```html
<picture>
  <source media="(min-width: 768px)" srcset="wide.webp">
  <source media="(max-width: 767px)" srcset="tall.webp">
  <img src="fallback.webp" alt="...">
</picture>
```

If the project's image pipeline (bundler/CDN) already generates responsive variants automatically, use that mechanism instead of hand-writing `srcset` — check for one before doing this manually.

## §13 Tall modals — scrollable body, sticky CTA

When a modal/dialog's content is taller than the mobile viewport, don't let the whole modal scroll (the CTA scrolls out of view) and don't let it overflow the screen. Cap the modal to the viewport height, make only the body scroll, and pin the footer/CTA so it's always visible:

```jsx
<Dialog>
  <DialogContent className="flex flex-col max-h-[90dvh] p-0 overflow-hidden">
    <div className="shrink-0 px-[24px] py-[20px]">
      {/* title, close button */}
    </div>

    <div className="flex-1 min-h-0 overflow-y-auto px-[24px] py-[16px]">
      {/* body content — this is the only part that scrolls */}
    </div>

    <div className="shrink-0 px-[24px] py-[20px] border-t border-[var(--line)] pb-[max(20px,env(safe-area-inset-bottom))]">
      <Button className="w-full">Confirm</Button>
    </div>
  </DialogContent>
</Dialog>
```

**Check the project's dialog component first** — it may already provide the cap, the scrolling body region and the safe-area padding. Don't re-declare what it does; supply only what's missing.

- `flex flex-col` + a `dvh` max height on the outer content — caps total modal height, never lets it exceed the viewport. Use `dvh`, not `vh`: on iOS Safari `vh` ignores the collapsing toolbar, so a `90vh` modal's pinned CTA can sit under it. Fixed heights should be in `dvh` too.
- `min-h-0` on the scrolling body is required — without it, a flex child won't shrink below its content size and `overflow-y-auto` does nothing.
- Header and footer get `shrink-0` so they hold their size while the body flexes.
- Footer gets the safe-area bottom padding from §10 (a modal footer is edge-adjacent on mobile the same way a fixed bar is) and a top border to separate it from the scrolling content once it's scrolled.
- Watch for library footer components (e.g. shadcn's `DialogFooter`) that ship negative margins or a fill — they can fight the project's surface; a plain `shrink-0` div is often the safer pinned footer.
- This is the same "cap height, scroll the middle, pin the ends" shape as §3's sticky panel — apply it to any modal/sheet/drawer, not just the page-level sticky rail.

## §14 Text inputs must never render below 16px on mobile (iOS auto-zoom)

Any text input (search bars, email/text fields, textareas) rendered below 16px causes iOS Safari to auto-zoom the page on focus — a jarring, broken-feeling interaction. **This must always be true, with no exceptions**, for every text input on every screen ≤ the `md` breakpoint.

**Check every time you touch a text-input component** (the project's input, textarea, search bar, or any new one) or add a new usage of one:
1. Resolve what font size it actually renders at on mobile (below `md`) — trace through the component's default className, not just what a call site passes.
2. If it's below 16px on mobile → **fix the shared component itself**, not the call site. Set the mobile font size at the component's default (e.g. `text-[16px] md:text-[14px]` if the design wants 14px on desktop), so every current and future usage is correct automatically. Never patch around it with a one-off `className` override at a single call site — that leaves the underlying component wrong for the next usage.
3. Don't set an input's size with a body/small type token that renders below 16px on mobile — that triggers the zoom just the same.
4. Never hardcode a value smaller than 16px via inline `style={{ fontSize: ... }}` on mobile under any circumstance — this is a hard floor, not a default to be overridden. (Inline styles also beat any responsive class.)

If the project's base input component(s) already encode this pattern, treat any new or sibling text-input component the same way: build the 16px-mobile floor into the component's own default styling, don't rely on every caller remembering to add it.
