# AI Pairing Skill

How to work effectively with Cursor, Claude Code, and similar AI coding assistants without sliding into spaghetti.

## The failure mode

Left to default behavior, AI assistants:

- Dump new code into the nearest open file regardless of fit
- Reinvent abstractions instead of using existing ones
- Skip lint/typecheck steps
- Add features beyond what was asked
- Use `any` or `@ts-ignore` when types are inconvenient
- Don't split files that have grown too large
- Hallucinate file paths and function names that don't exist
- Generate impressive-looking code that doesn't actually work

These failures compound. Three weeks of unsupervised AI coding can produce a codebase no one wants to maintain.

## Defense in depth

Five layers. Each catches what the others miss.

### 1. CLAUDE.md / project rules file

Every project has a `CLAUDE.md` at the root that the AI reads on every session. It specifies:

- Stack
- Folder structure
- Hard rules (file size limits, naming, banned patterns)
- Before-finish checklist

Non-negotiable. Without it, you're starting from scratch every session.

### 2. ESLint + pre-commit hooks

Talk is cheap. Automate enforcement.

Critical ESLint rules:

- `max-lines: 300`
- `max-lines-per-function: 80`
- `@typescript-eslint/no-explicit-any: error`
- `@typescript-eslint/ban-ts-comment: error`
- `import/no-cycle: error`
- `import/no-relative-parent-imports: error`
- `no-console: warn`
- `@typescript-eslint/naming-convention` matching `coding-rules.md`

Husky + lint-staged: bad code physically can't commit.

### 3. Specific prompts

Vague prompts produce vague code. Specific prompts produce structured code.

**Bad:** "Add a product detail page."

**Good:**
> Add product detail page at `src/pages/product-detail.tsx`. Use a `useProduct(slug)` hook (create at `src/hooks/use-product.ts` if missing). Render: image gallery, name, brand, price, short description, add-to-cart button. Match the existing dark theme. Run `pnpm run typecheck` after.

### 4. Periodic audits

Every few features, prompt the AI:

> Audit `src/` for these issues, in priority order. Do not implement fixes yet:
>
> 1. Any file over 250 lines
> 2. Any function over 50 lines
> 3. Duplicated logic across files (3+ similar blocks)
> 4. Files named utils/helpers/common/misc
> 5. Business logic inside JSX
> 6. Direct external-service calls outside `data/`
> 7. `any` type usage
> 8. Inconsistent naming
>
> Propose fixes for me to review before any code changes.

Run weekly. Apply what's worth applying.

### 5. Code review (human)

Tooling catches the obvious. A human catches:

- Scope creep
- Hallucinated APIs
- Architectural drift
- Magical thinking ("should work" code that wasn't tested)

Read every diff. Run the actual feature.

## Prompt patterns that work

### One logical unit per task

Don't ask for "build the cart, checkout, account, and admin." Break it down. AI follows structure better with smaller scopes.

### Always specify file paths

Generic: "add a price formatter"
Specific: "add `formatPrice(cents: number): string` in `src/lib/format-price.ts`"

### Describe the contract, not the implementation

Generic: "make the filter work"
Specific: "the filter sidebar reads/writes filter state via the URL search params. Filter changes update the URL with `replace`. Page reload preserves filter state."

### Ask for a plan before implementation on big changes

For anything spanning >5 files:

> Before implementing, give me your plan: list every file you'd create or modify with a one-line description. Wait for my approval before writing code.

### Run lint/typecheck yourself

After a task finishes, run them locally. If something fails, paste the error back — don't trust the AI's "all good!" claim.

## Anti-patterns to refuse

When the AI suggests these, push back:

- **"I'll just add an `any` type here for now"** — no, model the type
- **"I'll create a `helpers.ts` for this"** — no, name it by purpose
- **"I'll inline this logic in the JSX since it's small"** — no, extract anyway
- **"I'll skip typecheck since the change is small"** — no, run it
- **"I'll use `@ts-ignore` to bypass this error"** — no, fix the type
- **"I'll add this feature while I'm here"** — no, do exactly what was asked

## When the AI is right and you're wrong

It happens. If the AI insists on something you didn't ask for and it's actually correct (security fix, real bug fix), accept it — but require a separate commit so the diff stays reviewable.

## Bottom line

Architecture isn't something you do — it's a system you set up so you can't *not* do it. CLAUDE.md + ESLint + specific prompts + periodic audits + human review = five layers of net.
Optional: CLAUDE.md skeleton for new projects
Generic template — fill in the stack section per project:

# CLAUDE.md — [PROJECT NAME]

Read this file before writing any code. These rules are not optional.

## Project summary

- **Stack:** [list libraries — e.g. React 18 + Vite + TypeScript, Tailwind, shadcn/ui, wouter, react-helmet-async, @tanstack/react-query]
- **Backend:** [e.g. Supabase / Express / none]
- **Payments:** [e.g. Stripe Checkout / none]
- **Deploy:** [e.g. Cloudflare Pages / Vercel]
- **Monorepo:** [path to app if applicable]

## Folder structure — do not deviate

See `docs/architecture.md` for the canonical layout.

## Hard rules

See `skills/coding-rules.md` for the full list. The highlights:

- No file over 300 lines
- No function over 80 lines
- No files named utils/helpers/common/misc
- No direct external-service calls outside `src/data/`
- No `any` type, no `@ts-ignore`

## Before finishing any task

1. `pnpm run typecheck` — must pass
2. `pnpm run lint` — must pass with zero warnings
3. If any file you created or grew crosses 250 lines, propose how to split it

## When in doubt

Stop and ask. Don't guess. Don't invent file paths. Don't add features that weren't requested.

## Related docs

- `docs/architecture.md` — folder shape, layer rule, file size, refactor signals
- `skills/coding-rules.md` — hard rules, naming, banned patterns
- `skills/ai-pairing-skill.md` — how to work with AI assistants productively
