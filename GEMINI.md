# Project context for Gemini

Gemini CLI loads this file automatically. Follow it in every session.

## What we're building
The **Sungaru Dev website**: a single-page application that showcases Sungaru Dev's apps. Each app has its own customizable detail page, and an admin can add new apps through a form (**Create App** → **Add App**).

## Source of truth: read before coding
- `docs/PLAN.md`: goals, routes, phases, acceptance criteria
- `docs/BRAND.md`: brand identity, design tokens, typography, components
- `docs/SCHEMA.md`: database SQL, security policies, Zod schema, example data, API

If a prompt conflicts with these docs, **stop and ask**. Don't guess.

## Stack (do not change without asking)
- React 19 + TypeScript (strict) + Vite
- React Router v7 (`react-router` package) in **data mode**: `createBrowserRouter`, loaders, `lazy`, `ScrollRestoration`
- Tailwind CSS v4 (`@tailwindcss/vite`) + CSS variables from BRAND §10
- shadcn/ui (Radix) components in `src/components/ui`
- React Hook Form + Zod; TanStack Query; Framer Motion; Lucide icons
- Supabase Free plan (Postgres, Storage, Auth): `@supabase/supabase-js`
- Vitest + Testing Library; Playwright for end-to-end tests
- Hosting: Cloudflare Pages (SPA fallback via `public/_redirects`)
- Feedback: Google Forms (embedded), **not** stored in Supabase

## Hard rules
1. **Zero cost.** Don't add any paid service, paid API, or anything that requires a credit card.
2. **No secrets in the repo.** Only `VITE_*` public values go in `.env.local` (git-ignored), and `.env.example` documents them. The repository is public.
3. **Colors only through tokens.** Never hard-code hex colors in components. Use CSS variables or Tailwind classes mapped to them. The only exception is per-app theme values that come from data.
4. **Accessibility:** semantic HTML, a visible focus ring, keyboard support, alt text on every image, AA contrast.
5. **Mobile first:** everything must work at 360px width with no horizontal scrolling.
6. **Import app types from `src/lib/schema/app.ts`.** Never redefine them.
7. **All data access goes through `src/lib/api/`.** Components never call Supabase directly.
8. **Small, reviewable changes.** Do one prompt's scope at a time, and don't refactor unrelated code.
9. **Before saying you're done, run:** `npm run lint && npm run typecheck && npm run test && npm run build`. Fix every error, and report the output honestly.
10. **Commit** with a clear message in the imperative mood (e.g. "Add app catalogue page").

## Conventions
- **Files and names:** components `PascalCase.tsx`; hooks `useThing.ts`; utilities `camelCase.ts`.
- **Path alias:** `@/` → `src/`.
- **Components:** function components with named exports (default exports only for lazy route modules).
- **Copy:** follow the voice rules in BRAND §2 (plain, calm, specific; no exclamation marks).
