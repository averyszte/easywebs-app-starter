# CLAUDE.md — [PROJECT NAME]

Read this file before writing any code. These rules are not optional.

## On session start (read this first)

Before responding to my first message in any new session:

1. Read every file in `/docs/` (architecture, api-plan, auth-roles, client-context, current-task, database-plan)
2. Read every file in `/skills/` (coding-rules, ai-pairing-skill, plus the domain skills)
3. Reply with a short confirmation (<100 words) containing:
   - The stack (1 line)
   - The layer rule (1 line)
   - 3 specific hard rules from coding-rules.md
   - The current task from docs/current-task.md, or "no current task set"
4. Then wait for my actual instruction. Do NOT start writing code yet.

If anything I ask later conflicts with a rule in those docs, stop and flag the conflict instead of silently breaking the rule.

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

## MCP usage:
- Use shadcn for reliable component structure.
- Use Magic UI only for subtle visual polish.
- Use 21st.dev Magic for UI inspiration and component variations before major UI builds.
- Do not blindly copy registry components.
- Customize all spacing, typography, colors, copy, and hierarchy.
- Make the final site feel like a custom local business website, not a component dump.

## Important:
Do not reuse the starter layout as the final design.

For every new website, create a fresh visual direction based on the business type, audience, offer, market, and conversion goal.

The starter code exists only as a technical foundation.

Each site must have:
- a distinct hero layout
- a distinct color system
- a distinct CTA treatment
- varied section composition
- business-specific trust signals
- service-specific copy
- custom layout rhythm

Avoid making multiple sites feel like reskins of the same template.
