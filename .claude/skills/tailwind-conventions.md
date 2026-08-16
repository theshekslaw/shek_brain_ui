# Skill: Tailwind CSS Conventions

**When to use:** Writing or reviewing any component that ships styling. Applies across `shek_ui_common_utility` and `shek_brain_ui`.

## Design tokens live in the preset

`shek_ui_common_utility` ships a Tailwind preset (`src/theme/preset.ts`). Consumers extend it:

```ts
// shek_brain_ui/tailwind.config.ts
import { uiCommonPreset } from '@shek/ui-common/theme';
export default { presets: [uiCommonPreset], content: [...] };
```

Never hardcode a hex color or a raw `px` spacing in a component — use `bg-surface-1`, `p-4`, etc. If the token doesn't exist, add it to the preset.

## Class-ordering

- Layout → spacing → sizing → typography → color → state → responsive.
- Example: `flex items-center gap-2 p-4 w-full text-sm text-fg-1 hover:bg-surface-2 md:p-6`.
- Enforced by `prettier-plugin-tailwindcss` — install it and let the formatter sort.

## Conditional classes

Use `clsx` or `cn()` (from `shek_ui_common_utility/lib/cn.ts`):

```tsx
<button className={cn(
  "px-3 py-1.5 rounded-md",
  variant === "primary" && "bg-brand-1 text-white",
  variant === "ghost"   && "bg-transparent text-fg-1 hover:bg-surface-2",
  disabled && "opacity-50 pointer-events-none",
)}>
```

No ternaries or string interpolation for classes — they break the tailwind class detector.

## Avoid

- **`@apply` in a global CSS file** — you lose per-component locality; hurts tree-shaking.
- **Arbitrary values (`w-[137px]`)** — sign the design isn't tokenized. Fix the token.
- **Component-level dark-mode overrides** — dark mode is a theme concern, handled in the preset via CSS variables.
- **Inline `style={...}`** — only for values that genuinely can't live in tokens (dynamic transform).

## The Obsidian graph look (for `shek_brain_ui /graph`)

- Muted background: `bg-surface-0` with subtle grid via a repeating linear-gradient in the preset.
- Node color = category (concept, paper, author) — tokens `graph-node-*`.
- Edge = `stroke-graph-edge/60`; hover highlights the neighborhood, dimming the rest.
- Text labels appear on zoom > 1.5 (handled in `GraphView`, not styling).
