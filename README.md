# Sungaru Dev

> Software that solves real user pain points and improves everyday UX.

This repository contains the **Sungaru Dev website**: a single-page application that showcases every app Sungaru Dev builds. Each app gets its own fully customizable page with details, screenshots, key features, download links, FAQs, help, and a feedback channel. New apps are added through a built-in form, with no code changes needed.

> **Status:** Planning. The full build plan is in [`docs/PLAN.md`](docs/PLAN.md).

---

## Features

### Public site
- **App catalogue**: a grid of app cards, each with an icon, name, and short description. Clicking a card opens that app's page.
- **App detail pages**: each has:
  - App name, icon, and tagline
  - A curated screenshot gallery with a lightbox
  - A short "what it's about" description
  - **Key features**, each shown as its own styled card
  - **Download links**, shown only when set: Google Play, App Store, Desktop (Windows / macOS / Linux), and Web App
  - Links to **Feedback** (an embedded Google Form with the app name pre-filled), **FAQs**, and **Help**
- **Per-app theming**: every app page has its own background, card colors, and accent color, with separate values for light and dark mode.
- **Support and user requests**: a general support page plus a form for feature and app requests.
- **Dark / light mode switcher**: follows the system setting by default, and the visitor's choice is remembered.
- **Client-side routing**: deep-linkable URLs, scroll restoration, lazy-loaded routes, and a 404 page.

### Admin (app management)
- **Create App** button: opens the app builder form.
- **Add App** button: validates the form and publishes the app. Its page is generated from the form data straight away.
- **Live preview**: the form shows the finished app page, in light or dark mode, while you edit.
- **Detailed customization**: page background (solid color, gradient, or image), hero style, card background, card border and radius, accent color, and per-feature card color overrides.
- **Contrast guard**: warns you when your chosen colors make text hard to read.
- Save as **Draft** or **Publish**. You can also edit, reorder, or unpublish apps later.

---

## Brand

| Token | Light | Dark | Use |
|---|---|---|---|
| `brand` (Sungaru Teal) | `#1F6F68` | `#4FB3A9` | Primary actions, links, focus rings |
| `brand-soft` | `#E6F2F1` | `#12302D` | Tinted surfaces, badges |
| `accent` (Warm Sand) | `#C98B4B` | `#E0A96D` | Rare highlights only |
| `bg` | `#FAFAF9` | `#0E1113` | Page background |
| `surface` | `#FFFFFF` | `#161A1D` | Cards, panels |
| `text` | `#16181A` | `#ECEDEE` | Body text |
| `muted` | `#5B6168` | `#9AA1A8` | Secondary text |
| `border` | `#E4E6E8` | `#262B30` | Dividers, card borders |

The palette is calm and modern: a deep, slightly desaturated teal with neutral greys and a warm accent used sparingly. All text/background pairs meet WCAG AA contrast.

**Typography**
- Headings: **Plus Jakarta Sans** (600–800)
- Body/UI: **Inter** (400–600), with a fluid type scale using `clamp()`
- Code/versions: **JetBrains Mono**

**Logo:** v1 is a simple SVG "S" monogram plus a "Sungaru Dev" wordmark in the brand teal. The favicon and app icons are generated from the same file, and it can be swapped for a professionally designed logo later.

---

## Tech stack

The whole stack runs on free tiers ($0/month); the only cost is the custom domain (about $10–15/year). See [Zero-cost budget](docs/PLAN.md#21-zero-cost-budget).


| Concern | Choice | Why |
|---|---|---|
| Framework | **React 19 + TypeScript** | Component model, type safety for the app schema |
| Build tool | **Vite** | Fast dev server and small production builds |
| Routing | **React Router (data router)** | Nested layouts, loaders, lazy routes, scroll restoration |
| Styling | **Tailwind CSS v4** + CSS variables | Design tokens, `dark:` mode, per-app runtime theming |
| UI primitives | **Radix UI** (via shadcn/ui) | Accessible dialogs, tabs, accordions, and switches |
| Forms | **React Hook Form + Zod** | One schema shared by the form, validation, and the database |
| Data fetching | **TanStack Query** | Caching, loading states, and data updates after admin edits |
| Backend | **Supabase Free plan** (Postgres, Storage, Auth) | Stores apps, uploaded images, and admin sign-in at $0 with no card required |
| Animation | **Framer Motion** | Subtle page and card transitions |
| Icons | **Lucide** | Clean, consistent icon set |
| Testing | **Vitest + Testing Library + Playwright** | Unit, component, and end-to-end tests |
| Feedback | **Google Forms** | Free; responses go to a Google Sheet |
| Domain | **Cloudflare Registrar** + free DNS / Email Routing | At-cost custom domain, the only running cost |
| Hosting | **Cloudflare Pages (Free)** | Unlimited bandwidth, commercial use allowed, SPA rewrites, preview deploys |

---

## Project structure (planned)

```
sungaru_dev/
├─ docs/
│  └─ PLAN.md                 # Build plan & roadmap
├─ public/                    # Favicons, OG images, robots.txt
├─ src/
│  ├─ app/
│  │  ├─ router.tsx           # Route tree (lazy-loaded)
│  │  ├─ providers.tsx        # Theme, Query, Auth providers
│  │  └─ layouts/             # RootLayout, AppLayout, AdminLayout
│  ├─ pages/
│  │  ├─ Home.tsx
│  │  ├─ Apps.tsx             # Catalogue
│  │  ├─ app/                 # AppDetail, AppFaq, AppHelp, AppFeedback
│  │  ├─ Support.tsx
│  │  ├─ Request.tsx
│  │  ├─ About.tsx
│  │  ├─ admin/               # Dashboard, AppBuilder, Login
│  │  └─ NotFound.tsx
│  ├─ components/
│  │  ├─ ui/                  # Button, Card, Dialog, Tabs… (shadcn)
│  │  ├─ app-card/            # Catalogue card
│  │  ├─ app-page/            # Hero, Gallery, FeatureCard, DownloadLinks, FaqList
│  │  ├─ builder/             # Form sections, ColorPicker, ImageUploader, LivePreview
│  │  └─ ThemeToggle.tsx
│  ├─ lib/
│  │  ├─ schema/app.ts        # Zod schema = single source of truth
│  │  ├─ api/apps.ts          # Data access (Supabase + local mock)
│  │  ├─ theme/               # Token helpers, contrast checker
│  │  └─ utils.ts
│  ├─ data/seed-apps.json     # Local sample data for development
│  └─ styles/globals.css      # Tailwind + design tokens
├─ supabase/
│  └─ migrations/             # SQL schema, RLS policies, storage buckets
├─ tests/                     # Playwright e2e
├─ .env.example
└─ package.json
```

---

## Getting started (after Phase 1)

```bash
# 1. Install dependencies
npm install

# 2. Configure environment
cp .env.example .env.local
#   VITE_SUPABASE_URL=...
#   VITE_SUPABASE_ANON_KEY=...
#   VITE_DATA_SOURCE=mock      # "mock" uses src/data/seed-apps.json, "supabase" uses the backend

# 3. Run the dev server
npm run dev

# Other scripts
npm run build       # Production build
npm run preview     # Preview the production build
npm run lint        # ESLint
npm run typecheck   # tsc --noEmit
npm run test        # Vitest
npm run test:e2e    # Playwright
```

You can build and try out the whole site with `VITE_DATA_SOURCE=mock` before a Supabase project exists.

---

## Adding a new app

1. Sign in at `/admin/login`.
2. Click **Create App**.
3. Fill in the builder sections: **Basics → Media → Features → Downloads → Support → Theme**.
4. Check the page in the **Live Preview** panel and switch between light and dark mode.
5. Click **Add App** to publish, or **Save Draft** to keep working later.
6. The app card appears in the catalogue, and its page is live at `/apps/<slug>`.

---

## Roadmap

See [`docs/PLAN.md`](docs/PLAN.md) for the phase-by-phase plan, the data model, the route map, and acceptance criteria.

## License

© Sungaru Dev. All rights reserved.
