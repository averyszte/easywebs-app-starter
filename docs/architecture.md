# Architecture Standards

This document defines how every project is structured. It is the source of truth when reviewing code, onboarding contributors, or instructing AI assistants.

## Philosophy

Software gets worse over time unless you actively prevent it. The biggest sources of decay:

1. **Files that grow without limits** — "just for now" becomes a 2000-line monster.
2. **Dumping-ground modules** — `utils.ts`, `helpers.ts`, `common.ts` accumulate unrelated logic.
3. **Mixed concerns** — business logic in JSX, DB calls in components, types redefined everywhere.
4. **Inconsistent shape** — every project organized differently means slow context-switching and predictable bugs.

Every rule below exists to prevent one of those failure modes.

## Standard folder shape
src/
├── types/ # shared TypeScript types, one domain per file
├── data/ # repository pattern — only place that talks to external services
├── hooks/ # React hooks, one hook per file
├── lib/ # pure utility functions, named by purpose
├── components/ # UI components, one component per file
│ ├── ui/ # primitives (shadcn, etc.) — do not modify
│ └── [feature]/ # feature-specific components grouped in subfolders
└── pages/ # route components, one per route

Variations require justification.
## The layer rule
Dependencies flow one direction only.
pages (routes)
↓
hooks (React state + side effects)
↓
data (repository — abstracts external services)
↓
external services (database, payment, email, storage)

- **components** and **lib** are shared across layers
- **lib** is pure functions (no React, no side effects) — usable from any layer
- **components** can be used by pages and other components; rarely by hooks
A component must never call an external service directly. It calls a hook, which calls the repository, which calls the service. This makes backend swaps trivial — replace the repository, nothing else changes.
## File size rules
- **No file over 300 lines** (pure data files are exempt)
- **No function over 80 lines**
When you're about to grow past 250 lines, stop and ask:
- Does this file do more than one thing? → Split by concern.
- Are there logical sections inside it? → Each becomes its own file.
- Is half data and half logic? → Data goes to `lib/foo-data.ts`, logic stays.
## When to refactor (signals)
- You copy-pasted code → extract a function or component
- A function takes 5+ arguments → wants a config object or splitting
- A component has 10+ props → it's doing too much
- A useState becomes complex → consider useReducer or a custom hook
- Same data is computed in multiple places → memoize once in a hook
- A file imports >15 things → probably too coupled
## Decision boundaries (what lives where)
| Concern | Lives in |
|---|---|
| Types shared across components/hooks | `types/` |
| External service calls | `data/` only |
| React state and effects | `hooks/` |
| Pure data transforms (date format, price format, slug builder) | `lib/` |
| One-off helpers used by a single component | inline in that component file |
| Component rendering UI | `components/` |
| Route component (page) | `pages/` |
| Constants used across the app | `lib/constants.ts` or per-domain |
| URL state (filters, pagination) | the URL itself, via the router |
| Server data cache | React Query, never local state |
## Enforcement
1. **CLAUDE.md** — AI assistants read this on every session
2. **ESLint** — fails the build on violations
3. **Prettier** — uniform formatting
4. **Husky + lint-staged** — blocks bad commits
5. **CI** — runs typecheck and lint on every PR
6. **Code review** — catches what tooling misses
Bypassing any layer is a process violation.
## Breaking a rule
Rules are not laws. If a rule prevents a genuinely better solution:
1. Document the case in the PR description
2. Get sign-off
3. If it becomes a pattern, update this document — don't let exceptions become shadow conventions
File 2 → skills/coding-rules.md
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
