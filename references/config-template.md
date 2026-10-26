# Responsive-design config template

Copy to `<project>/.claude/responsive-design.config.md` and fill in. Keep it to
facts about THIS project — the general patterns stay in the skill. Anything
left blank, the skill discovers itself.

```markdown
# Responsive-design config — <Project name>

Project-specific values for the global `responsive-design` skill. Where this
file and the skill disagree, this file wins.

## Design system
- Tokens file(s): <path> — <how tokens are applied, e.g. `bg-[var(--x)]` or `bg-surface`>
- Design doc: <path to DESIGN.md or none>
- Spacing scale: <e.g. "default Tailwind 4px scale" or "1px-literal steps: 1,2,4,8…">
- Breakpoints: <Tailwind defaults, or custom>

## Placeholder → project token
| Skill placeholder | This project |
|---|---|
| `--surface` | |
| `--surface-muted` | |
| `--line` | |
| `--line-subtle` | |
| `--text-title` / `--text-body` / `--text-label` | |
| `--radius-md` / `--radius-lg` | |
| `--shadow-tint` | |
| `--header-offset` | |
| `type-heading` / `type-value` / `type-body` / `type-label` | |

## Typography scaling
<Which type tokens auto-scale at which widths; which deliberately don't.>

## Components to reuse
<Pill/badge, button, card, dialog (+ body/footer parts), inputs, search bar — with paths.>

## Layout facts
<Page wrapper padding per breakpoint, max width, fixed header height, sticky offsets.>

## Already handled
<e.g. viewport-fit=cover present; dialog already caps height + safe-area; touch-target utility.>

## Project-specific rules and gotchas
<Things the general skill can't know, and the mistakes made here before.>
```
