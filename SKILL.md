---
name: responsive-design
description: Adapt desktop-first designs to tablet and mobile by progressively reducing paddings, font sizes, gaps, and restructuring layouts (e.g. columns stacking, side-by-side rows becoming vertical) so each screen size looks designed for it, not just scaled down. Use when the user mentions responsive design, mobile layout, mobile view, "looks broken on mobile", image cropping at small widths, banner/hero needs responsive treatment, or wants a desktop-only design to work on smaller devices.
version: 2.0.0
user-invocable: true
argument-hint: "[target component or page]"
---

Adapt desktop-first interfaces to smaller viewports. The goal: each breakpoint looks *intentional*, not a shrunken desktop layout.

This skill is **project-agnostic**. It holds the general patterns; each project's
token names, components and gotchas live in that project's
`.claude/responsive-design.config.md`.

## Preflight — Read Before Editing

1. **Project config** — read `.claude/responsive-design.config.md` in the current project. It maps this skill's placeholder tokens (`--surface`, `--line`, `--radius-lg`…) to the project's real ones, names the components to reuse, and lists project-specific rules. **Where the config and this skill disagree, the config wins.**
   - **No config?** Discover the same facts yourself (design-system doc such as `DESIGN.md`, tokens/theme files, `tailwind.config` or a Tailwind v4 `@theme` block, the component library folder), then offer to save them as a config using [references/config-template.md](references/config-template.md).
2. **Design tokens** — if typography tokens auto-scale at breakpoints, don't add per-breakpoint `text-[Xpx]` overrides — that defeats the system. Note which type steps scale and which deliberately don't.
3. **Breakpoints** — confirm the scale (Tailwind defaults: `sm 640 / md 768 / lg 1024 / xl 1280 / 2xl 1536`) and whether the type tokens step at the same widths.
4. **Component library** — find the project's shared components. Extend, don't duplicate.
5. **CLAUDE.md** — respect rules about hardcoding colors, spacing, etc.

## Pattern index

Full how-to (with code) for every pattern lives in
[references/patterns.md](references/patterns.md) — **read that file before
applying any pattern below** (for a full-page pass, read it in full; for a
targeted fix, read at least the matching §). Its samples use placeholder
tokens — translate them through the project config.

| § | Pattern | Apply when |
|---|---|---|
| 1 | Progressively reduce (padding/gaps/fonts ~50–60% per breakpoint) | every screen |
| 2 | Reshape, don't squeeze (grids step down 4→3→2→1; rows → `flex-col`; stretch awkward `w-fit` containers) | multi-column layouts |
| 3 | Sticky side panel → sticky bottom section (+ collapsible header) | desktop right rail with a primary action |
| 4 | Full-bleed images via negative margins | hero/banner inside a padded container |
| 5 | Pills/badges: `whitespace-nowrap shrink-0`, never wrap | any tag/badge/chip |
| 6 | Wide tables → card-per-row stack | data tables that would h-scroll |
| 7 | Touch targets ≥ 44×44px — grow the hit area, not the painted box | all interactive elements |
| 8 | Text-over-image: cap text ~70%, left gradient, `object-position` per breakpoint | left text + right/bg image |
| 9 | Dev panels/FABs must not collide with mobile content | screens with dev tooling |
| 10 | Safe areas: `env(safe-area-inset-*)` on edge-pinned elements (needs `viewport-fit=cover`) | fixed/sticky bars |
| 11 | `pointer: coarse` / `hover: none` — input type ≠ viewport width; no hover-only actions | touch sizing, hover-revealed controls |
| 12 | Responsive image files (`srcset` / `<picture>`), not just crops | heroes, banners, large photos |
| 13 | Tall modals: cap height in `dvh`, scroll only the body (`min-h-0`!), pin the CTA | modal content taller than viewport |
| 14 | Text inputs ≥ 16px on mobile — fix the **shared component**, never the call site | any screen touching a text input |

## Workflow

1. Read the target component in full.
2. Run preflight (project config, tokens, breakpoints, component library, CLAUDE.md).
3. Read [references/patterns.md](references/patterns.md) and apply the relevant patterns, translating placeholders through the project config; prefer tokens over hardcoded values.
4. For text-over-image: apply §8's full 3-part pattern.
5. **If the target touches or renders a text input** (search bar, email/text field, textarea): check its mobile font size (§14) every time, even if the input itself isn't the reason you're in this file. Fix it in the shared component if it's under 16px — don't skip this because the input wasn't the primary target.
6. Verify at ~390px (mobile), ~768px (tablet), ≥1280px (desktop) — in the browser, or by the project's own verification rule if its config/CLAUDE.md sets one.

## Quick Reference

| Aspect | Desktop | Tablet (md) | Mobile |
|---|---|---|---|
| Section padding X | 80px | 48px | 20px |
| Section padding Y | 48px | 32px | 20px |
| Card padding | 24–32px | 24px | 16–20px |
| Gap between sections | 48px | 32px | 24px |
| Gap inside section | 24px | 20px | 16px |
| Hero heading | largest display tier | one tier down | two tiers down (free if tokens auto-scale) |
| Body copy | body | body | body — never below the legibility floor (~13–14px) |
| Grid columns | 3–4 | 2 | 1 |

Snap every value to the project's spacing scale. If pages already sit inside a padded page wrapper, don't stack section padding on top of it — the section-padding rows are for full-width sections.

## Anti-patterns

- Hardcoding `text-[Xpx]` when tokens auto-scale, or shrinking body-size type below its floor on mobile.
- Reaching past a semantic token to a raw palette step or a hex.
- Same padding across all breakpoints.
- `hidden md:block` to "fix" overflow — reshape instead.
- Touch targets under 44px — or "fixing" a deliberately compact control by enlarging its painted box instead of its hit area.
- Text over a busy image with no gradient treatment.
- `fixed bottom-0` when `sticky bottom-0` would flow better with the document.
- `inline-flex w-full` — `inline-flex` ignores explicit widths; use `flex` instead.
- Fixed/sticky edge-pinned elements with no `env(safe-area-inset-*)` padding.
- Actions only reachable via `:hover` with no touch/tap fallback.
- Serving the same full-resolution image file to mobile and desktop.
- Gating touch-target sizing behind a viewport breakpoint when the real signal is input type (`pointer: coarse`).
- A tall modal that scrolls as a whole (CTA scrolls out of view) instead of capping height with only the body scrollable — or capping it in `vh` instead of `dvh`.
- Any text input rendering below 16px on mobile, or patched at the call site instead of fixed in the shared component.
