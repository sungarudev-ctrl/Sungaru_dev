# Sungaru Dev Website: Build Plan

## 1. Goals

1. Showcase every Sungaru Dev app on a clean, modern catalogue page.
2. Give each app a rich, **individually themed** detail page.
3. Let the owner add new apps **through a form** (Create App → Add App), with no code changes.
4. Provide support channels: per-app FAQs, Help, and Feedback, plus site-wide support and user requests.
5. Build it as a fast, accessible **single-page application** with solid routing and a dark/light mode switcher.

### Non-goals for v1
- User accounts for the public (visitors don't sign in).
- Payments or licensing.
- Multi-language support (the code will keep text separate so this can be added later).

---

## 2. Key decisions

| Decision | Choice | Notes |
|---|---|---|
| SPA framework | React 19 + TypeScript + Vite | |
| Router | React Router data router (`createBrowserRouter`) | Loaders prefetch app data, `lazy()` splits routes, `<ScrollRestoration/>`, `errorElement` per route |
| Styling | Tailwind v4 + CSS custom properties | Site tokens on `:root`/`.dark`, per-app tokens scoped to the app page wrapper |
| Persistence | Supabase **Free plan**: `apps` table (JSONB columns for flexible content), Storage bucket for images, Auth for the admin | A local JSON "mock" adapter behind the same interface allows offline development |
| Validation | One Zod schema used by the form, the API layer, and the TypeScript types | |
| Admin access | Supabase email + password auth + Row-Level Security; only admin emails can write | Password rather than magic link, because the free built-in email sender is heavily rate-limited. Public users can only read published apps |
| Images | Resized and converted to WebP **in the browser before upload**, then served from Supabase Storage at several sizes (`srcset`) with long cache headers | Keeps usage inside the free 1 GB storage / 5 GB egress |
| Hosting | **Cloudflare Pages (Free)** with SPA fallback (`_redirects`: `/* /index.html 200`) | Unlimited bandwidth, commercial use allowed, preview deploys per branch |
| Domain | **Custom domain** registered through **Cloudflare Registrar** (sold at cost, no markup, about $10–15/yr depending on the extension) with DNS on Cloudflare (free) | The only running cost of the project. Free Cloudflare Email Routing forwards `support@<domain>` to your Gmail |
| Feedback | **Google Forms** (free), with responses collected in a Google Sheet | Nothing stored in Supabase; Google handles spam, storage and notifications |

> **Why Supabase rather than a static JSON file?** The "Add App" button has to save data somewhere that the live site reads from. A static site would need a rebuild for every new app. Supabase lets a new app appear immediately after you click Add App, and it also handles image uploads and sign-in.
> **Why not Firebase?** Since February 2026, Cloud Storage for Firebase requires the Blaze (pay-as-you-go) plan with a billing account attached, even when usage stays in the free allowance. We need image uploads, so Firebase would mean attaching a card. Supabase's free plan includes storage with no card required.

### 2.1 Zero-cost budget

| Service | Plan | Free allowance | Our expected use |
|---|---|---|---|
| Cloudflare Pages | Free | Unlimited sites and bandwidth, 500 builds/month | A few builds per week |
| Supabase | Free | 500 MB database, 1 GB file storage, 5 GB egress/month, 50k MAU, 2 projects | Tiny database; about 150 KB per WebP screenshot fits roughly 6,000 images |
| GitHub | Free | Repo, Actions (2,000 min/month on private repos) | CI + keep-alive ping |

**Free-tier risks and mitigations**
- **Supabase pauses projects after 7 days of inactivity.** A scheduled GitHub Actions workflow (`.github/workflows/keep-alive.yml`) runs a tiny read query every 3 days, so the project never pauses.
- **5 GB/month egress.** Screenshots are compressed in the browser before upload, served in responsive sizes, and cached by browsers for a long time. The catalogue loads only app icons, and screenshots load lazily.
- **Built-in auth emails are rate-limited.** The admin signs in with email + password, so no emails are needed for normal use.
- **Upgrading later is optional.** If traffic outgrows the free tier, move to Supabase Pro. The code doesn't change.

---

## 3. Route map

| Path | Page | Notes |
|---|---|---|
| `/` | Home | Hero, featured apps, value props, CTA to catalogue |
| `/apps` | App catalogue | Grid of app cards, search + platform filter (Android / iOS / Desktop / Web) |
| `/apps/:slug` | App detail | Themed with that app's colors |
| `/apps/:slug/faq` | App FAQs | Accordion, searchable |
| `/apps/:slug/help` | App help | Rich text / guides, contact info |
| `/apps/:slug/feedback` | App feedback | Embedded Google Form with the app name pre-filled, plus an "Open in Google Forms" button as a fallback |
| `/support` | General support | Contact form, links to each app's help |
| `/request` | User requests | Feature request / new app idea form |
| `/about` | About Sungaru Dev | Mission, contact |
| `/admin/login` | Admin sign-in | |
| `/admin` | Dashboard | List of apps (draft/published), **Create App** button, inbox of user requests, link to the feedback Google Sheet |
| `/admin/apps/new` | App builder | Form + live preview, **Add App** button |
| `/admin/apps/:slug/edit` | Edit app | Same builder, pre-filled |
| `*` | 404 | Friendly not-found page |

**Routing details**
- `RootLayout` (header with nav + theme toggle, footer) wraps all public routes; `AdminLayout` wraps `/admin/*` behind an auth guard.
- `AppLayout` wraps `/apps/:slug/*`: it loads the app once, applies its theme, and shows sub-tabs (Overview · FAQ · Help · Feedback) as nested routes.
- Route loaders + TanStack Query prefetch on link hover, so pages open instantly.
- Unknown slugs show a themed "App not found" `errorElement`.
- `<title>`/meta tags update for each route (react-helmet-async) so shared links have proper previews.

---

## 4. Data model

### 4.1 `App` (Zod schema = TypeScript type = DB row)

```ts
App {
  id: uuid
  slug: string                       // auto-generated from name, editable, unique
  status: 'draft' | 'published'
  order: number                      // position in catalogue
  name: string
  tagline: string                    // ≤ 80 chars, shown on the card
  description: string                // markdown, "what it's about"
  icon: ImageRef
  screenshots: ImageRef[]            // curated, reorderable, with captions
  features: Feature[]
  platforms: {
    playStore?: url
    appStore?:  url
    desktop?:   { windows?: url; macos?: url; linux?: url }
    web?:       url
  }
  support: {
    feedbackFormUrl?: url            // per-app Google Form; if empty, the site-wide form is used with the app name pre-filled
    faqs: { question: string; answer: string }[]
    help: string                     // markdown
    contactEmail?: email
  }
  theme: AppTheme
  version?: string
  updatedAt: timestamp
  createdAt: timestamp
}

Feature {
  id: string
  title: string
  description: string
  icon?: string                      // Lucide icon name or uploaded image
  image?: ImageRef
  cardStyle?: Partial<CardStyle>     // per-feature override
}

ImageRef { url: string; alt: string; width: number; height: number }
```

### 4.2 `AppTheme` (the customization options)

```ts
AppTheme {
  light: ThemeVariant
  dark:  ThemeVariant
  hero:  { layout: 'centered' | 'split' | 'banner'; showScreenshot: boolean }
  card:  CardStyle
}

ThemeVariant {
  pageBackground: Background
  cardBackground: Background
  accent: color                      // buttons, links, highlights on this page
  text: color
  mutedText: color
}

Background =
  | { type: 'solid';    color: color }
  | { type: 'gradient'; from: color; to: color; angle: number }
  | { type: 'image';    image: ImageRef; overlay: color; overlayOpacity: number }

CardStyle {
  radius: 'sm' | 'md' | 'lg' | 'xl'
  border: 'none' | 'subtle' | 'accent'
  shadow: 'none' | 'soft' | 'lifted'
  glass: boolean                     // translucent + backdrop-blur
}
```

The theme is converted into CSS variables (`--app-bg`, `--app-card-bg`, `--app-accent`, …) on the app page wrapper, so the components themselves need no per-app code. If you only set light-mode colors, dark-mode values are generated automatically; you can override them.

### 4.3 Other tables
- `requests` (`id, type: 'feature' | 'new-app' | 'bug' | 'other', app_id?, name?, email, message, created_at, status`)
- `admins` (`user_id`, `email`), used by RLS policies. Starts with one row (you). **Adding an admin later** means creating their user in the Supabase dashboard and adding a row here, with no code change. Public sign-ups are disabled, so nobody else can create an account.

**Row-Level Security**
- `apps`: public `select` where `status = 'published'`; admins have full access.
- `requests`: public `insert` only (rate-limited, with a honeypot field against spam); admins can `select`/`update`.
- Storage `app-media`: public read; admins write.

---

## 5. App builder (Create App → Add App)

**Flow:** Dashboard → **Create App** → builder opens with an empty form and a side-by-side live preview → fill in the sections → **Add App** validates the form and saves it → redirects to `/apps/:slug`.

**Form sections** (a stepper on mobile, collapsible panels on desktop):

1. **Basics**: name, slug (auto-filled), tagline, description (markdown editor), version, status.
2. **Media**: icon upload; drag-and-drop screenshot upload with reordering, captions, and alt text (required, for accessibility).
3. **Key features**: add/remove/reorder features; each has a title, description, icon picker, optional image, and optional card color override.
4. **Downloads**: toggle each platform on or off and enter its URL. Only enabled platforms appear on the page.
5. **Support**: optional per-app Google Form link (defaults to the site-wide feedback form), FAQ list editor, help content, contact email.
6. **Theme**:
   - Light/dark tabs
   - Page background: solid / gradient (two color pickers + angle) / image (upload + overlay)
   - Card background: same options
   - Accent, text, and muted text colors
   - Card style: radius, border, shadow, glass
   - Hero layout
   - Theme presets (e.g. "Midnight", "Paper", "Ocean") as starting points
   - **Contrast checker**: shows AA/AAA badges and a warning when colors are hard to read

**Behavior**
- Unsaved work is autosaved to local storage, and you are warned before leaving the page.
- **Save Draft** skips the "required for publish" checks; **Add App** enforces them all.
- Validation errors link to the field they refer to.
- Editing an existing app reuses the same builder, and its main button reads **Save Changes**.

---

## 6. UI and design system

- **Tokens:** defined in `styles/globals.css` as CSS variables for `:root` and `.dark` (see the palette in the README).
- **Theme toggle:** Light / Dark / System. The choice is saved in `localStorage`, and an inline script in `index.html` applies it before the page renders, so there is no flash of the wrong theme.
- **Typography:** Plus Jakarta Sans (headings), Inter (body), fluid `clamp()` scale, 65–75 character line length for long text.
- **Layout:** 12-column container with a max width of about 1200px and 16px side margins on mobile; responsive card grid (1 → 2 → 3 columns).
- **App cards:** icon, name, tagline, platform badges; a small lift and border-color change on hover; the whole card is one link; the card is tinted with the app's accent color.
- **Motion:** subtle fade and slide between pages, which is turned off for users who set `prefers-reduced-motion`.
- **Accessibility:** semantic landmarks, visible focus rings, keyboard-operable gallery and lightbox, alt text required for screenshots, AA contrast on site colors.

---

## 7. Phases and tasks

### Phase 0: Foundations
- [ ] Scaffold Vite + React + TS; ESLint, Prettier, Vitest, path aliases
- [ ] Tailwind v4 + design tokens + fonts
- [ ] shadcn/ui base components (Button, Card, Dialog, Tabs, Accordion, Input, Textarea, Switch, Select)
- [ ] ThemeProvider + ThemeToggle (no flash of the wrong theme)
- [ ] Router skeleton with layouts, lazy routes, 404, scroll restoration
- [ ] **Logo v1**: a simple SVG monogram ("S" mark) + "Sungaru Dev" wordmark in Plus Jakarta Sans, using the brand teal; light/dark variants, favicon, and app-touch icons generated from the same SVG (free, no designer needed; replaceable later)
- [ ] CI (GitHub Actions): lint, typecheck, test, build

### Phase 1: Public site with mock data
- [ ] Zod `App` schema + seed data (2–3 sample apps)
- [ ] `lib/api/apps.ts` interface + mock adapter
- [ ] Home page
- [ ] Catalogue page with AppCard grid, search, and platform filter
- [ ] App detail page: Hero, Gallery + lightbox, Description, FeatureCards, DownloadLinks (shown only if set), Support links
- [ ] Per-app theming via CSS variables (light and dark)
- [ ] FAQ, Help, and Feedback sub-routes
- [ ] Support, Request, and About pages
- [ ] SEO/meta tags for each route

### Phase 2: Backend
- [ ] Supabase project, SQL migrations, RLS policies, storage bucket
- [ ] Supabase adapter implementing the same API interface
- [ ] Request form writes to the DB (honeypot field + basic rate limit)
- [ ] Create the site-wide **Google Form** (fields: App, Rating, Feedback, Email (optional)) linked to a Google Sheet, with email notifications on new responses; store its URL and the App field's pre-fill `entry.<id>` in `.env` (`VITE_FEEDBACK_FORM_URL`, `VITE_FEEDBACK_APP_FIELD`)
- [ ] `FeedbackEmbed` component: iframe embed (`?embedded=true&entry.<id>=<App name>`), lazy-loaded, with an "Open in Google Forms" fallback link
- [ ] Admin auth (email + password) + route guard
- [ ] Keep-alive GitHub Actions workflow so the free Supabase project never pauses

### Phase 3: App builder
- [ ] Admin dashboard: app list, status, reordering, **Create App** button
- [ ] Builder form sections 1–6 (React Hook Form + Zod)
- [ ] Image uploader (drag-and-drop, resizing/WebP conversion, reordering, alt text)
- [ ] Theme editor + presets + contrast checker
- [ ] Live preview using the real app page components
- [ ] **Add App** (publish), **Save Draft**, edit, unpublish, delete (with confirmation)
- [ ] Draft autosave + warning before leaving with unsaved changes
- [ ] Admin inbox for user requests + link to the feedback Google Sheet

### Phase 4: Polish and launch
- [ ] Page transitions + reduced-motion support
- [ ] Lighthouse ≥ 95 on Performance, Accessibility, Best Practices, and SEO
- [ ] Playwright e2e: browse → open app → download link; admin create → add → app appears in catalogue
- [ ] Favicon, OG images, `robots.txt`, sitemap generation
- [ ] Register the custom domain on Cloudflare Registrar, attach it to Cloudflare Pages (HTTPS automatic and free), redirect `www` → apex
- [ ] Cloudflare Email Routing: `support@` and `hello@` forward to the owner's Gmail
- [ ] Cloudflare Web Analytics (free, privacy-friendly)

### Later (v2 ideas)
- Changelog / release notes per app
- Newsletter or "notify me" for upcoming apps
- Public roadmap and voting on requests
- Multi-language support
- Pre-rendering app pages for better SEO

---

## 8. Acceptance criteria

- Clicking any app card opens `/apps/:slug`, and the browser back button returns to the same scroll position.
- A download button appears **only** for platforms with a URL.
- Each app page uses its own background and card colors in both light and dark mode.
- The theme toggle works everywhere, is remembered after a reload, and the wrong theme never flashes on load.
- An admin can create an app through the form, click **Add App**, and see it in the catalogue and at its own URL without redeploying the site.
- Non-admins cannot reach `/admin/*` or write to `apps`.
- All pages work at 360px width with no horizontal scrolling.
- Lighthouse Accessibility ≥ 95.

---

## 9. Open questions for the owner

1. ~~**Hosting / backend**~~: decided. Cloudflare Pages + Supabase Free, $0/month (see §2.1).
2. ~~**Logo**~~: decided. A simple SVG monogram + wordmark is built in Phase 0.
3. ~~**Domain**~~: decided. Custom domain via Cloudflare Registrar. **Still to choose: the name** (e.g. `sungaru.dev`, `sungarudev.com`).
4. ~~**Admin users**~~: decided. One admin (the owner) for now; more can be added later without code changes.
5. ~~**Feedback**~~: decided. Google Forms (free), with the app name pre-filled.
6. **User requests**: keep the built-in request form (stored in Supabase), or move it to a Google Form as well?
7. **Launch apps**: which apps (and their details and screenshots) go live first?
