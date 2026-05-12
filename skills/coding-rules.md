# Coding Rules

The non-negotiable practices for writing code in this organization.

## Hard rules

- NO file over 300 lines. Split first.
- NO function over 80 lines. Extract helpers.
- NO files named `utils.ts`, `helpers.ts`, `misc.ts`, `common.ts`. Name by purpose.
- NO direct external-service calls outside `src/data/`. Components use hooks; hooks use the repository.
- NO inline business logic in JSX. Extract to a hook or pure function.
- NO relative imports past one level. Use `@/` path aliases.
- NO new wrapper around a single primitive (shadcn, etc.) that adds no behavior.
- NO `any` type. Use `unknown` and narrow, or model properly.
- NO `// @ts-ignore` or `// @ts-expect-error`. Fix the type.
- NO `console.log` in committed code.
- NO magic numbers / strings in business logic. Extract named constants.
- NO mutating props, state, or function arguments. Treat them as readonly.

## Naming conventions

| Kind | Convention | Example |
|---|---|---|
| File | kebab-case | `product-card.tsx`, `format-price.ts` |
| Component | PascalCase | `ProductCard` |
| Hook (file) | use-kebab.ts | `use-products.ts` |
| Hook (export) | useCamelCase | `useProducts` |
| Function | camelCase | `formatPrice` |
| Constant | SCREAMING_SNAKE_CASE | `MAX_PRICE_CENTS` |
| Type / Interface | PascalCase | `Product`, `ProductFilters` |
| Boolean | is / has / should / can | `isLoading`, `hasFilter`, `shouldRetry` |

Pick once. Never deviate. Inconsistency is grep-hostile.

## Component rules

- **One component per file.**
- **No business logic in JSX.** Extract to a hook or pure function above the return.
- **Props use TypeScript types** (`type FooProps = {...}`), not `interface` unless extending.
- **Default props in destructuring**, not React's `defaultProps`.
- **Children by composition** — `<Card><CardHeader/></Card>` over `<Card header="..." />`.

## Hook rules

- **One hook per file.** `use-foo.ts`, exports `useFoo`.
- **Follow the rules of hooks.** Never conditional.
- **Side effects belong in `useEffect`** or returned callbacks. Never during render.

## Repository rules

- **All external-service calls happen in `data/*-repository.ts` files.**
- **Read and write functions both go here**, even if writes are stubbed.
- **Function signatures are stable.** Swapping backends changes bodies, not signatures.
- **Mock data is imported by the repository only.**

## Imports

- **Use `@/` path aliases** beyond one folder up.
- **Group imports**: external → internal aliases → relative, blank line between groups.
- **No barrel files** (`index.ts` re-exporting everything) except `components/ui/`.

## Comments

- **Default: no comments.** Names should explain what code does.
- **Comments are for WHY, not WHAT.**
- **No commented-out code.** Delete it; git remembers.
- **No `TODO` without an issue number.**

## Error handling

- **Validate at boundaries** (user input, API responses). Trust internal calls.
- **Don't catch errors you can't handle.**
- **Don't add defensive checks for impossible states.** Trust the type system.

## Async

- **`async/await` over `.then`.**
- **Always handle errors.**
- **Don't fire-and-forget promises** without `void` or proper handling.

## State

- **URL is the source of truth for shareable state.**
- **Server cache is the source of truth for server data** — React Query, not `useState`.
- **Local component state is for ephemeral UI** (modal open, hover, input draft).
- **Avoid global state managers** unless there's a real cross-cutting concern.

## Styling

- **Tailwind utility classes**, not custom CSS.
- **No inline `style` prop** for static styles.
- **Long className strings** (>100 chars) → extract via `cva` or a wrapper.
- **No `!important`.**

## Performance defaults

- Images use `loading="lazy"` and `decoding="async"` below the fold.
- LCP images get `fetchpriority="high"` and a `preload` link.
- Routes code-split via `React.lazy` for non-critical pages.
- Lists virtualized above ~100 items.
- Network calls cached via React Query.

## Before finishing any task

1. `pnpm run typecheck` — must pass
2. `pnpm run lint` — must pass with zero warnings
3. If any file you created or grew crosses 250 lines, propose how to split it
4. If you added a dependency, justify it
