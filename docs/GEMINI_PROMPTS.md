# Gemini Build Prompts

These are copy-paste prompts for building the Sungaru Dev website with **Gemini**. Give them **one at a time, in order**. Each prompt is a small, testable step that ends with the site still working.

## How to use

1. **Recommended: use [Gemini CLI](https://github.com/google-gemini/gemini-cli)** (free with a Google account) inside your local clone of this repo:
   ```bash
   git clone https://github.com/sungarudev-ctrl/Sungaru_dev.git
   cd Sungaru_dev
   npx @google/gemini-cli     # or: npm i -g @google/gemini-cli && gemini
   ```
   Gemini CLI reads `GEMINI.md` automatically, so it always knows the project rules. It can also read the `docs/` files, run commands, and edit files.
   *You can also use Gemini Code Assist in VS Code, or paste the prompts into gemini.google.com. In that case, paste `GEMINI.md` and the docs it mentions first, then apply the code yourself.*
2. **Start a new branch for each phase**, e.g. `git checkout -b phase-0`. Merge it into `main` once the step works.
3. **After each prompt:**
   - Check the changes with `git diff`.
   - Run `npm run dev` and look at the result in your browser.
   - Commit only when you're happy with it.
4. If something breaks, use the **Fix-it prompt** at the end of this file.
5. You'll need to supply a few values yourself, marked `<LIKE_THIS>`.

---

## Prompt 0: Orientation (no code)

```
Read GEMINI.md, docs/PLAN.md, docs/BRAND.md and docs/SCHEMA.md in full.
Do not write any code yet.

Reply with:
1. A 5-line summary of what we are building.
2. The build order you will follow (phases and steps).
3. Any contradictions or unclear points you found in the docs, as questions.
```

---

## Phase 0: Foundations

### Prompt 1: Project scaffold

```
Scaffold the project in the repository root (keep the existing docs/, README.md, GEMINI.md).

- Vite + React 19 + TypeScript (strict mode, "noUncheckedIndexedAccess": true).
- Path alias "@/" -> "src/" in both tsconfig and vite config.
- Install: react-router, @tanstack/react-query, zod, react-hook-form, @hookform/resolvers,
  framer-motion, lucide-react, clsx, tailwind-merge. Use the latest stable versions.
- Dev tools: ESLint (flat config, typescript-eslint, react-hooks, jsx-a11y), Prettier,
  Vitest + @testing-library/react + jsdom, and one example test that passes.
- package.json scripts: dev, build, preview, lint, typecheck (tsc --noEmit), test (vitest run),
  test:watch, format.
- public/_redirects containing: /*  /index.html  200   (Cloudflare Pages SPA fallback)
- .env.example with VITE_DATA_SOURCE=mock, VITE_SUPABASE_URL=, VITE_SUPABASE_ANON_KEY=,
  VITE_FEEDBACK_FORM_URL=, VITE_FEEDBACK_APP_FIELD=  (each with a one-line comment).
  Make sure .env.local and .env are in .gitignore.
- .github/workflows/ci.yml: on push and pull_request, Node LTS, npm ci, then lint,
  typecheck, test, build.
- Create the folder structure from README "Project structure" with placeholder index files
  only where needed.

Run lint, typecheck, test and build. All must pass. Show me the output, then commit:
"Scaffold Vite + React + TypeScript project".
```

### Prompt 2: Design tokens, fonts, Tailwind, base components

```
Set up the design system from docs/BRAND.md.

1. Tailwind CSS v4 via @tailwindcss/vite. In src/styles/globals.css:
   - Copy the :root and .dark CSS variables from BRAND §10 exactly.
   - Add the teal scale (BRAND §4.2) as --teal-50 … --teal-950.
   - Map the tokens into Tailwind with @theme (colors: bg, surface, surface-2, border, text,
     muted, brand, brand-contrast, brand-soft, accent, accent-strong, success, warning,
     danger, info; radius sm/md/lg/xl; font-display, font-body, font-mono).
   - Use @custom-variant dark (&:where(.dark, .dark *)) so dark mode follows the .dark class.
   - Base styles: body uses --bg, --text and font-body; headings use font-display;
     :focus-visible shows a 2px var(--focus) outline with a 2px offset.
   - Type scale utilities from BRAND §5 (text-display, text-h1, text-h2, text-h3,
     text-body-lg, text-small, text-caption) with the clamp() sizes, line heights,
     weights and letter spacing listed there.
   - Motion tokens from BRAND §8, and a prefers-reduced-motion block that disables
     transitions and animations.
2. Self-host the fonts with @fontsource: Plus Jakarta Sans (600, 700, 800), Inter (400, 500, 600),
   JetBrains Mono (400). Use font-display: swap.
3. Initialise shadcn/ui for Vite + Tailwind v4 and add: button, card, dialog, tabs, accordion,
   input, textarea, label, switch, select, badge, tooltip, dropdown-menu, sonner (toasts).
   Restyle Button, Card, Badge and Input to match BRAND §9 exactly (heights, radii,
   variants primary / secondary / ghost / destructive). Use tokens only; no hex colors.
4. Create a temporary /dev/styleguide page that shows every color token, the type scale,
   and every button, card, badge and input variant in both themes, so I can review them.

Run all checks, then commit: "Add design tokens, fonts and base UI components".
```

### Prompt 3: Theme switcher

```
Implement light / dark / system theme switching (PLAN §6, BRAND §9 "Theme toggle").

- src/app/providers/ThemeProvider.tsx: stores "light" | "dark" | "system" in localStorage
  (key "sungaru-theme", with every read and write wrapped in try/catch), sets or removes the
  "dark" class on <html>, updates <meta name="color-scheme">, listens for OS theme changes
  when set to "system", and exposes useTheme().
- An inline script in index.html <head>, placed BEFORE any CSS, that applies the saved theme
  so there is never a flash of the wrong theme on load.
- src/components/ThemeToggle.tsx: an icon button (Sun / Moon / Monitor) with a
  dropdown-menu to pick the theme, an aria-label, a tooltip, and full keyboard support.
- Unit tests: the provider sets the class correctly for all three modes, and the saved
  choice persists.

Run all checks, then commit: "Add theme provider and toggle".
```

### Prompt 4: Logo and icons

```
Create logo v1 as described in BRAND §3.

- public/brand/logo-mark.svg: a rounded geometric "S" made of two arcs flowing into each
  other, centered in a squircle (rounded square). Squircle fill #1F6F68, S stroke #FFFFFF,
  viewBox 0 0 64 64, clean paths, no embedded fonts.
- public/brand/logo-mark-dark.svg: the same shape with squircle fill #4FB3A9 and S stroke #0E1113.
- src/components/brand/Logo.tsx: an inline SVG React component with the props
  variant: "lockup" | "mark" | "wordmark" and size. The lockup is the mark plus the text
  "Sungaru" (font-display, 800) and " Dev" (font-display, 500). Colors come from CSS
  variables so it switches with the theme automatically. Include role="img" and an
  aria-label of "Sungaru Dev".
- public/favicon.svg (the mark), plus a script at scripts/generate-icons.mjs that uses sharp
  (dev dependency) to create apple-touch-icon.png (180), icon-192.png, icon-512.png and
  og-default.png (1200x630: brand teal background, lockup and the tagline "Software that just
  makes sense." in white). Add an "icons" npm script and commit the generated PNGs.
- A site.webmanifest file, and the icon and manifest links in index.html.

Show me the mark and the lockup on the /dev/styleguide page in both themes.
Commit: "Add logo v1, favicon and app icons".
```

### Prompt 5: Router, layouts, all routes

```
Build the routing skeleton from PLAN §3 using React Router v7 data mode.

- src/app/router.tsx with createBrowserRouter:
  - RootLayout (header, main, footer) for all public routes.
  - Routes: / , /apps , /apps/:slug (AppLayout with nested index, faq, help, feedback),
    /support , /request , /about , /admin/login , /admin (AdminLayout, guarded) with
    index, apps/new, apps/:slug/edit, and * (NotFound).
  - Every page is lazy-loaded (the route "lazy" property). Each route has an errorElement
    with a friendly message and a "Go home" button.
  - Put <ScrollRestoration /> in RootLayout.
- Header: Logo (lockup, links to /), nav links (Apps, Support, Request, About) with the
  active state shown, and ThemeToggle. On mobile (below md) the nav collapses into an
  accessible menu (Dialog/Sheet) with a focus trap. Include a "Skip to content" link.
- Footer: logo mark, © year Sungaru Dev, links (Apps, Support, Request, About),
  support@<YOUR_DOMAIN> as a mailto link.
- Every page for now: an <h1> and a short placeholder paragraph written in the brand voice.
- A PageMeta component (react-helmet-async or React 19 <title>/<meta> hoisting) that sets
  the title as "<Page> · Sungaru Dev", plus a description and og: tags. Use it on every page.
- AdminLayout: for now, if the user is not signed in, redirect to /admin/login
  (a stub hook useAuth() that returns { user: null } until Phase 2).
- Page transitions: a subtle fade (200ms) with framer-motion, disabled when reduced motion is on.

Test: rendering "/unknown" shows NotFound; each nav link navigates correctly.
Run all checks, then commit: "Add router, layouts and page skeletons".
```

---

## Phase 1: Public site with mock data

### Prompt 6: Schema, seed data, mock API

```
Implement the data layer from docs/SCHEMA.md.

1. src/lib/schema/app.ts: copy the Zod schema from SCHEMA §6 exactly, and export the types.
2. src/data/seed-apps.json: 3 realistic sample apps that validate against the App schema,
   based on SCHEMA §7:
   - "snip": Play Store + Windows + macOS + web; teal "ocean" theme.
   - "pocket-budget": App Store + Play Store only; warm light theme with a solid background.
   - "focus-timer": web only; dark-leaning theme with a gradient background and glass cards.
   Use images from placehold.co (e.g. https://placehold.co/1600x1000/webp?text=Snip)
   as the "path" values for now, and write good alt text for each.
3. src/lib/api/types.ts: the DataSource interface with every function in SCHEMA §8.
4. src/lib/api/mock.ts: implements DataSource using seed-apps.json plus localStorage for
   admin changes (wrapped in try/catch), with ~300ms simulated latency.
   Only published apps are returned to the public.
5. src/lib/api/index.ts: exports the active data source based on VITE_DATA_SOURCE
   (default "mock"). Leave a clear TODO where the Supabase source will plug in.
6. src/lib/api/hooks.ts: TanStack Query hooks: useApps(filters), useApp(slug),
   useSubmitRequest(), plus query keys. Add the QueryClientProvider in providers.
7. src/lib/media.ts: a helper that turns an ImageRef into { src, srcSet, sizes, width,
   height, alt } (it resolves the Supabase public URL later; for now it uses the path as-is).
8. Tests: every seed app parses with App; AppPublish rejects an app with no platform links;
   the mock source hides drafts from the public.

Run all checks, then commit: "Add app schema, seed data and mock data source".
```

### Prompt 7: Home page and app catalogue

```
Build the Home page (/) and the catalogue (/apps). Follow BRAND §2 (voice) and §9 (components).

AppCard (src/components/app-card/AppCard.tsx):
- The whole card is one <Link> to /apps/:slug. It shows the app icon (56px, radius xl),
  the name (h3), the tagline (muted, clamped to 2 lines) and platform badges
  (Android, iOS, Desktop, Web — only the ones the app has).
- A subtle tint from the app's theme.light.accent (or dark) at about 6% opacity.
- On hover or focus: lift 2px, lifted shadow, border color = the app accent. Respect reduced motion.
- Loading skeleton variant.

Catalogue /apps:
- h1 "Our apps" + a one-line intro.
- A search input (filters by name/tagline, debounced 200ms) and platform filter chips
  (All, Android, iOS, Desktop, Web). The filters sync to URL search params (?q=&platform=)
  so filtered views can be shared.
- A responsive grid: 1 / 2 / 3 columns. Empty state: "No apps match "<q>". Try a different word."
- Prefetch the app data when a card is hovered or focused (queryClient.prefetchQuery).

Home /:
- Hero: text-display headline "Apps that remove everyday friction.", a sub-line from the
  mission, primary button "Explore our apps" -> /apps, secondary "Request a feature" -> /request.
  Simple geometric teal/sand decoration (CSS or SVG), nothing loud.
- "Featured apps": the first 3 published apps by sort order, using AppCard.
- "How we build": 3 short value cards (Solve real problems / Respect your time / Keep it simple).
- A closing CTA band linking to /support.

Test: filtering by platform shows only matching apps; the card links to the right slug.
Check the layout at 360px, 768px and 1280px. Commit: "Add home page and app catalogue".
```

### Prompt 8: App detail page and per-app theming

```
Build the app detail page (/apps/:slug), following PLAN §4–§6 and BRAND §10.

Theming engine (src/lib/theme/):
- contrast.ts: relativeLuminance(hex) and contrastRatio(a, b) (WCAG 2.1), plus unit tests
  (e.g. #16181A on #FAFAF9 ≈ 17.04).
- deriveDark.ts: builds a dark ThemeVariant from a light one when theme.dark is missing
  (dark page background, lifted card surface, lightened accent that reaches ≥ 4.5:1
  on the dark background, light text). Unit-test that the result always passes AA.
- appThemeToCssVars(theme, mode): returns the CSS variables --app-bg, --app-card-bg,
  --app-accent, --app-accent-contrast, --app-text, --app-muted, --app-card-radius,
  --app-card-border, --app-card-shadow. Backgrounds support solid, gradient and image
  (image + overlay colour/opacity).
- AppThemeScope component: a wrapper that applies these variables for the current site
  theme (light/dark). The site header and footer are NOT themed (BRAND §10).

AppLayout (/apps/:slug/*):
- Loads the app with a route loader + useApp. If the slug doesn't exist, it shows a themed
  "App not found" page with a link back to /apps.
- Sub-navigation tabs: Overview · FAQ · Help · Feedback (NavLinks, accessible tablist styling).
- PageMeta with the app name, tagline and first screenshot as og:image.

Overview sections (src/components/app-page/):
- Hero: supports the layouts "centered" | "split" | "banner" from theme.hero. It shows
  the icon, name (h1), tagline, version badge and DownloadLinks. If hero.showScreenshot is
  true, it shows the first screenshot.
- DownloadLinks: renders ONLY the platforms that have URLs. Play Store and App Store use
  the official badge artwork; Windows / macOS / Linux / Web use brand-style buttons with
  Lucide icons. Links open in a new tab with rel="noopener noreferrer".
- Description: renders markdown safely (react-markdown, no raw HTML).
- Gallery: a horizontal scroll-snap strip of screenshots using srcset. Clicking opens a
  Lightbox (Dialog) with prev/next, arrow keys, Esc to close, swipe on touch, captions,
  and focus returned to the thumbnail on close.
- FeatureGrid + FeatureCard: 1/2/3 columns; icon (Lucide by name, with a fallback), title,
  description, optional image; applies the per-feature cardStyle overrides.
- SupportStrip: cards linking to FAQ, Help and Feedback.
- All cards use the --app-card-* variables.

Test: DownloadLinks renders nothing for missing platforms; the lightbox keyboard controls work.
Check all 3 seed apps in light and dark mode at 360px and 1280px.
Commit: "Add app detail page with per-app theming".
```

### Prompt 9: FAQ, Help, Feedback, Support, Request, About

```
Build the remaining public pages.

- /apps/:slug/faq: a searchable Accordion of the app's FAQs (markdown answers). Empty state
  with a link to Help.
- /apps/:slug/help: renders support.help as markdown, with an auto-generated table of
  contents from its h2/h3 headings, plus a contact card (support.contactEmail, falling back
  to support@<YOUR_DOMAIN>).
- /apps/:slug/feedback: a FeedbackEmbed component.
  - The URL is app.support.feedbackFormUrl, or otherwise VITE_FEEDBACK_FORM_URL with
    "?usp=pp_url&entry.<VITE_FEEDBACK_APP_FIELD>=<encoded app name>&embedded=true".
  - A lazy-loaded <iframe> (title="Feedback form for <App>", about 900px tall,
    responsive width), a loading skeleton, and an "Open in Google Forms" link underneath.
  - If no form URL is configured, show a friendly message with the support email instead.
- /support: a short intro, "Get help with an app" (a list of apps linking to their help
  pages), a contact card, and a link to /request.
- /request: a form with React Hook Form + the RequestInput Zod schema. Fields: type (select:
  Feature idea / New app idea / Bug report / Other), app (optional select, populated from
  the apps list), name (optional), email, message, and a hidden honeypot field "website".
  If the honeypot is filled, show success but don't submit. Show inline errors, a disabled
  and loading state while sending, a success message ("Thanks. Your request has been
  sent."), and a friendly error for a rate limit.
- /about: the mission, personality and a contact section, written in the BRAND §1–2 voice.

Test: the request form shows validation errors and doesn't submit when the honeypot is filled;
the feedback URL builder encodes the app name correctly.
Commit: "Add FAQ, help, feedback, support, request and about pages".
```

---

## Phase 2: Backend (Supabase Free)

> **Before Prompt 10, you need to:**
> 1. Create a free project at supabase.com. There's no card needed; pick the region nearest your users.
> 2. Copy the **Project URL** and **anon public key** into `.env.local`.
> 3. Create the site-wide **Google Form** (fields: App as short answer, Rating 1–5, Feedback as paragraph, Email optional) and link it to a Google Sheet (Responses → Link to Sheets).
> 4. Use ⋮ → **Get pre-filled link** to find the App field's `entry.<number>`, then put the form URL and that number in `.env.local` as `VITE_FEEDBACK_FORM_URL` and `VITE_FEEDBACK_APP_FIELD`.

### Prompt 10: Database migration and Supabase data source

```
Connect Supabase.

1. supabase/migrations/0001_init.sql: combine SCHEMA §2, §3, §4 and §5 into one migration,
   exactly as written (tables, enums, functions, triggers, RLS policies, storage bucket and
   policies). Add a header comment explaining how to run it (Supabase dashboard ->
   SQL Editor -> paste -> Run) and the admin insert from SCHEMA §4 as a commented-out
   final step.
2. src/lib/supabase.ts: creates the client from VITE_SUPABASE_URL / VITE_SUPABASE_ANON_KEY,
   and throws a clear error if they're missing while VITE_DATA_SOURCE=supabase.
3. src/lib/api/supabase.ts: implements the full DataSource interface.
   - snake_case <-> camelCase mapping in one place, and every row validated with the Zod App schema.
   - submitRequest inserts WITHOUT .select() (the public can't read requests); map the
     P0001 rate-limit error to a friendly message.
   - uploadImage(appId, file, kind): resize in the browser (canvas / createImageBitmap) to
     the widths in SCHEMA §5, encode as WebP (quality 0.82), upload to
     app-media/apps/{appId}/{kind}/{imageId}-{width}.webp, and return an ImageRef
     with the original width and height.
   - deleteApp also removes the storage folder apps/{appId}/.
4. Update src/lib/media.ts to build public Storage URLs when using Supabase.
5. src/lib/api/index.ts chooses between mock and supabase from VITE_DATA_SOURCE.
6. scripts/seed-supabase.mjs: an optional script that inserts the seed apps (it uses the
   service role key from a local, git-ignored env var, never committed, and clearly warns
   about this).
7. .github/workflows/keep-alive.yml: runs on a cron every 3 days (plus workflow_dispatch)
   and curls GET $SUPABASE_URL/rest/v1/apps?select=id&limit=1 with the apikey header, using
   the repo secrets SUPABASE_URL and SUPABASE_ANON_KEY. The job fails if the HTTP status
   isn't 200.
8. Update README "Getting started" with the Supabase setup steps.

Tests: the row mapping round-trips; the mock and supabase sources share a contract test
suite (run against the mock in CI).
Commit: "Add Supabase migration, data source and keep-alive job".
```

### Prompt 11: Admin authentication

```
Implement admin sign-in (PLAN §2, SCHEMA §4).

- src/lib/auth.tsx: an AuthProvider using supabase.auth (email + password). It exposes
  useAuth() -> { user, isAdmin, loading, signIn, signOut }. isAdmin is checked by
  selecting the user's row from public.admins (RLS only lets admins see it).
  In mock mode, signIn accepts any email with the password "admin" and logs a console warning.
- /admin/login: a form (email, password) with errors ("Email or password is incorrect."),
  a loading state, and a redirect to the page the user originally wanted.
  There is NO sign-up link.
- AdminLayout guard: while loading, show a spinner; if not signed in, redirect to login;
  if signed in but not an admin, show "You don't have access" with a sign-out button.
- AdminLayout: a sidebar/topbar with Dashboard, Create App, Requests and
  "Feedback sheet" (an external link from VITE_FEEDBACK_SHEET_URL; add it to .env.example),
  the user's email and a sign-out button.
- Add <meta name="robots" content="noindex"> on /admin routes.

Tests: the guard redirects signed-out users, and non-admins see the access message.
Commit: "Add admin authentication and route guard".
```

---

## Phase 3: App builder

### Prompt 12: Admin dashboard and requests inbox

```
Build /admin (Dashboard) and a Requests inbox.

Dashboard:
- A prominent primary "Create App" button -> /admin/apps/new.
- An apps table/list: icon, name, slug, status badge (Draft / Published), last updated,
  and actions (View, Edit, Publish/Unpublish, Delete with a confirmation dialog that asks
  you to type the app name).
- Drag-to-reorder (dnd-kit), plus accessible up/down buttons as an alternative, saved with
  reorderApps.
- Empty state: "No apps yet. Create your first one."

Requests (/admin/requests, add the route):
- A list filtered by status tabs (New, Reviewing, Planned, Done, Declined), showing the type,
  app, email (mailto), the message (expandable) and the date. A status select updates it.
- Show a count badge for New in the admin nav.

Use toasts for success and failure. Commit: "Add admin dashboard and requests inbox".
```

### Prompt 13: App builder — structure, Basics, Media

```
Build the app builder at /admin/apps/new and /admin/apps/:slug/edit (PLAN §5).

Structure:
- A two-column layout on lg+: the form on the left, a sticky LIVE PREVIEW on the right
  (a placeholder panel for now). Below lg: a stepper with Next/Back and a "Preview" toggle
  that opens the preview full screen.
- Sections: 1 Basics, 2 Media, 3 Features, 4 Downloads, 5 Support, 6 Theme. Each is a
  component in src/components/builder/sections/. Build 1 and 2 now; 3–6 are placeholders.
- React Hook Form with zodResolver. It validates with AppDraft for "Save Draft" and
  AppPublish for "Add App". The error summary at the top links to (and focuses) each
  invalid field.
- Footer action bar: "Save Draft" (secondary) and "Add App" (primary; on the edit page it
  reads "Save Changes"; for a published app it shows "Update").
  Add App -> publish -> toast -> navigate to /apps/:slug.
- Autosave the form state to localStorage every 2 seconds (key per app, wrapped in
  try/catch), offer "Restore unsaved changes?" on load, and warn before leaving with unsaved
  changes (useBlocker + beforeunload).
- New apps get a default theme copied from the "ocean" preset (create
  src/lib/theme/presets.ts with the presets: ocean, paper, midnight, sand, forest, each a
  complete AppTheme that passes AA contrast).

Section 1 Basics: name; slug (auto-generated from the name until edited manually, with a
live "unique?" check); tagline with a character counter (80); description as a markdown
textarea with a Write/Preview toggle; version; sort order.

Section 2 Media, with an ImageUploader component:
- Drag-and-drop or click, with instant local previews, uploaded through
  api.uploadImage (resize -> WebP).
- Icon: one square image (warn if it isn't square). Screenshots: up to 12, reorderable by
  drag or with up/down buttons, each with required alt text and an optional caption, and
  removable.
- Rejects files over 10 MB before resizing and any non-image types, and shows progress
  and errors.

Commit: "Add app builder with basics and media sections".
```

### Prompt 14: App builder — Features, Downloads, Support

```
Complete builder sections 3, 4 and 5.

Section 3 Features (useFieldArray):
- Add / remove / duplicate / reorder feature cards (max 12).
- Each has: title (60), description (300, with a counter), an icon picker (a searchable grid
  of Lucide icons, storing the name), an optional image (ImageUploader, single) and
  "Customize card" (an optional background as solid/gradient and an accent colour,
  using the ColorField from Prompt 15; stub it with a native color input for now).

Section 4 Downloads:
- A row per platform: Google Play, App Store, Windows, macOS, Linux, Web app. Each has a
  Switch to enable it and an https URL input shown when enabled. Disabling clears the value.
- Light validation hints: the Play Store URL should contain "play.google.com", and the
  App Store URL should contain "apps.apple.com" (show a warning, not a hard error).
- Show the "at least one platform to publish" error here.

Section 5 Support:
- The feedback form URL (optional), with help text: "Leave empty to use the main
  feedback form. The app name is filled in automatically."
- An FAQ editor (useFieldArray): question + markdown answer, reorderable.
- Help content: a markdown editor with preview.
- Contact email (optional).

Commit: "Add features, downloads and support builder sections".
```

### Prompt 15: Theme editor and live preview

```
Build section 6 Theme and the live preview. This is the most important customization feature.

Components (src/components/builder/theme/):
- ColorField: a swatch + hex input + native color picker + the brand swatches
  (the BRAND §4 palette) as quick picks. Validates hex.
- BackgroundField: a segmented control Solid / Gradient / Image.
  Solid: ColorField. Gradient: from, to, and an angle slider (0–360) with a preview.
  Image: ImageUploader (kind "backgrounds") + overlay ColorField + an opacity slider.
- ContrastBadge: shows the ratio and AA / AAA / "Fails" for a text-on-background pair
  (for gradients and images, check against the worst case: both gradient ends, or the
  overlay colour blended at its opacity).

Theme section:
- Presets row: clickable preset cards (ocean, paper, midnight, sand, forest) that apply a
  full theme, with a confirmation if the current theme has been customized.
- Tabs: Light | Dark. The Dark tab starts with "Automatic (generated from light)" switched
  on; switching it off unlocks manual editing (pre-filled from deriveDark).
- For each mode: Page background, Card background, Accent, Text, Muted text,
  with ContrastBadges for text/page, muted/page, text/card and accent-contrast/accent.
- Hero layout (centered / split / banner, shown as visual radio cards) + "Show screenshot" switch.
- Card style: radius, border, shadow (radio chips) + a Glass switch.
- If any required pair fails AA, show a warning banner. "Add App" is blocked until it's
  fixed or explicitly acknowledged with a "Publish anyway" checkbox.

Live preview:
- Renders the REAL app page components (Hero, Gallery, FeatureGrid, DownloadLinks,
  SupportStrip) inside AppThemeScope, using the current form values (watch() + useDeferredValue).
- A toolbar: Light/Dark toggle (preview only), and device width Mobile 375 / Tablet 768 /
  Desktop 1280 (scaled to fit the panel).
- Also shows how the AppCard will look in the catalogue.

Tests: ContrastBadge thresholds; applying a preset sets every theme field; the preview
updates when the accent changes.
Commit: "Add theme editor, presets, contrast checks and live preview".
```

---

## Phase 4: Polish and launch

### Prompt 16: Quality pass

```
Do a full quality pass. Don't add new features.

1. Accessibility: run @axe-core/playwright on every public route in both themes and fix all
   violations. Check the keyboard-only flow: header nav, mobile menu, catalogue filters,
   card, gallery + lightbox, request form, and the whole builder.
2. Performance: route-level code splitting is working; images use width/height, lazy
   loading and srcset; fonts preload the display font weights actually used;
   there are no layout shifts. Run Lighthouse (mobile) on /, /apps and /apps/snip, get every
   category ≥ 95, and report the scores.
3. SEO: PageMeta on every route, canonical URLs using VITE_SITE_URL (add it to
   .env.example), a JSON-LD SoftwareApplication block on app pages, public/robots.txt
   (disallow /admin), and scripts/generate-sitemap.mjs that builds sitemap.xml from the
   published apps at build time.
4. Playwright e2e tests (tests/e2e):
   - Visitor: home -> catalogue -> filter by platform -> open an app -> only the configured
     download buttons are visible -> the lightbox opens and closes with the keyboard -> the
     theme toggle persists after a reload.
   - Admin (mock mode): log in -> Create App -> fill in the required fields -> Add App ->
     the app appears in /apps and at /apps/<slug>.
   Use the pre-installed browser when PLAYWRIGHT_BROWSERS_PATH is set; add a test:e2e
   script and run e2e in CI against the mock data source.
5. Remove /dev/styleguide from production builds (only register it when import.meta.env.DEV).

Commit: "Quality pass: accessibility, performance, SEO and e2e tests".
```

### Prompt 17: Deployment guide

```
Write docs/DEPLOY.md: a step-by-step guide for a non-expert to launch the site at $0/month
(plus the domain). Cover:

1. Cloudflare account -> Workers & Pages -> Create -> Pages -> Connect to Git ->
   sungarudev-ctrl/Sungaru_dev, production branch main, build command "npm run build",
   output directory "dist", Node version env var, and the environment variables
   (VITE_DATA_SOURCE=supabase, VITE_SUPABASE_URL, VITE_SUPABASE_ANON_KEY,
   VITE_FEEDBACK_FORM_URL, VITE_FEEDBACK_APP_FIELD, VITE_FEEDBACK_SHEET_URL, VITE_SITE_URL).
2. Buying the domain with Cloudflare Registrar, attaching it to Pages as a custom domain
   (apex + www redirect), and automatic HTTPS.
3. Supabase: Auth -> URL configuration (Site URL = the domain), disable sign-ups,
   create your admin user, run the admin insert SQL.
4. Cloudflare Email Routing: support@ and hello@ -> your Gmail.
5. Cloudflare Web Analytics (free) for the site.
6. GitHub repo secrets for the keep-alive workflow, and how to run it manually once.
7. A post-launch checklist: test each route on the live domain, add your first real app with
   the builder, and check the feedback form posts to the Google Sheet.
8. Free-tier limits to watch (from PLAN §2.1) and what to do if you get close to them.

Also update the README with a "Deploying" section linking to it.
Commit: "Add deployment guide".
```

---

## Fix-it prompt (use any time)

```
Something is wrong. Here is what I see:
<describe the problem, paste the error message / terminal output, and the URL or screen>

Steps:
1. Reproduce the problem and explain the root cause in 2–3 sentences before changing code.
2. Make the smallest fix that solves it; don't refactor unrelated code.
3. Add or update a test that would have caught it.
4. Run lint, typecheck, test and build, and show the output.
5. Commit with a message describing the fix.
```

## Review prompt (after each phase)

```
Review everything changed since <branch or commit> against GEMINI.md, docs/PLAN.md,
docs/BRAND.md and docs/SCHEMA.md. List:
1. Anything missing from the phase's checklist in PLAN §7.
2. Bugs, accessibility problems, hard-coded colors, or data access that bypasses src/lib/api.
3. Security issues (secrets in code, unsafe HTML, RLS gaps).
Don't change code yet. Give me a prioritized list, then wait for my go-ahead.
```

---

**Tip:** tick the matching boxes in `docs/PLAN.md` §7 as each prompt is completed, so the plan always shows where the build is.
