# Skill: React Component Patterns

**When to use:** Building or refactoring a React component in `shek_ui_common_utility` or `shek_brain_ui`.

## Rules

### Composition over configuration
Don't build a component with 15 boolean props. Split into subcomponents.

**Bad:**
```tsx
<Card title="X" showFooter footerActions={...} isCollapsible onCollapse={...} />
```

**Good:**
```tsx
<Card>
  <Card.Header>X</Card.Header>
  <Card.Body>...</Card.Body>
  <Card.Footer><Button>Save</Button></Card.Footer>
</Card>
```

### Headless-first
Primitives in `shek_ui_common_utility` are unstyled or minimally styled. Consumers layer Tailwind classes via `className` (or a `cn()` merge util) on top. This makes the library reusable across visual themes.

### One component per file
`Button.tsx` exports one `Button`. Related tiny helpers can co-locate; separate components move to their own file.

### Props first, state second
Prefer a controlled component. Add optional uncontrolled fallback (`defaultValue`) only when the DX pain justifies it. Never both simultaneously.

### `useEffect` is a last resort
If you can derive from props/state or handle in an event, do that. Effects create implicit ordering, subtle bugs, and testing pain.

## File layout

```
src/components/Button/
  Button.tsx          # component
  Button.stories.tsx  # storybook (if enabled)
  Button.test.tsx     # vitest + testing-library
  index.ts            # re-export
```

`index.ts` at every folder — makes `import { Button } from '@shek/ui-common'` clean.

## Anti-patterns

- **Prop drilling more than 2 levels** — use context or move state up.
- **Global CSS overrides** — kills the encapsulation the library exists for.
- **`useEffect` with an empty dep array to fetch data** — use TanStack Query.
- **`any` in props** — the whole point of TypeScript is fighting exactly this.
- **Inline object/array literals in JSX props** — new reference every render, breaks memoization.
