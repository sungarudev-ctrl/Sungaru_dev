# Sungaru Dev: Data Schema

This is the single reference for how data is stored and validated. There are three layers, and they must stay in sync:

1. **Database**: Supabase Postgres tables, security policies, and storage bucket (§2–§5). This becomes `supabase/migrations/0001_init.sql`.
2. **Validation and types**: the Zod schema in `src/lib/schema/app.ts` (§6). The app form, the API layer, and the TypeScript types all come from it.
3. **Example data**: a sample app record (§7). This is used by `src/data/seed-apps.json` and the mock data source.

---

## 1. Overview

```
auth.users (Supabase)          public.admins            public.apps                 public.requests
┌──────────────┐  1 ─── 0..1   ┌──────────────┐         ┌───────────────────┐ 1 ── * ┌──────────────────┐
│ id           │──────────────▶│ user_id (PK) │         │ id (PK)           │◀───────│ app_id (FK, null)│
│ email        │               │ email        │         │ slug (unique)     │        │ type, email, ... │
└──────────────┘               └──────────────┘         │ status, content…  │        └──────────────────┘
                                                         └───────────────────┘
storage.buckets: app-media  (public read, admin write)  →  icons, screenshots, feature images, backgrounds
```

- **Columns vs. JSON:** fields we filter or sort on (slug, status, order, name) are real columns. Rich nested content (screenshots, features, platforms, support, theme) is stored as **JSONB** and validated by Zod. Adding a new field to the form then needs no database migration.
- **Feedback is not stored here.** It goes to Google Forms (see PLAN §2).

---

## 2. Tables

```sql
-- supabase/migrations/0001_init.sql

create type public.app_status     as enum ('draft', 'published');
create type public.request_type   as enum ('feature', 'new_app', 'bug', 'other');
create type public.request_status as enum ('new', 'reviewing', 'planned', 'done', 'declined');

-- ── Admins ─────────────────────────────────────────────────────────────
create table public.admins (
  user_id    uuid primary key references auth.users (id) on delete cascade,
  email      text not null unique,
  created_at timestamptz not null default now()
);

-- ── Apps ───────────────────────────────────────────────────────────────
create table public.apps (
  id           uuid primary key default gen_random_uuid(),
  slug         text not null unique
               check (slug ~ '^[a-z0-9]+(-[a-z0-9]+)*$' and char_length(slug) <= 60),
  status       public.app_status not null default 'draft',
  sort_order   integer not null default 0,
  name         text not null check (char_length(name) between 1 and 60),
  tagline      text not null default '' check (char_length(tagline) <= 80),
  description  text not null default '' check (char_length(description) <= 5000),
  version      text check (char_length(version) <= 20),
  icon         jsonb,                                  -- ImageRef
  screenshots  jsonb not null default '[]'::jsonb,     -- ImageRef[]
  features     jsonb not null default '[]'::jsonb,     -- Feature[]
  platforms    jsonb not null default '{}'::jsonb,     -- Platforms
  support      jsonb not null default '{}'::jsonb,     -- Support
  theme        jsonb not null default '{}'::jsonb,     -- AppTheme
  published_at timestamptz,
  created_at   timestamptz not null default now(),
  updated_at   timestamptz not null default now(),

  -- A published app must have the essentials; drafts may be incomplete.
  constraint apps_publish_ready check (
    status = 'draft' or (
      icon is not null
      and char_length(tagline) > 0
      and char_length(description) > 0
      and jsonb_array_length(features) > 0
      and platforms <> '{}'::jsonb
    )
  )
);

create index apps_catalogue_idx on public.apps (sort_order, name) where status = 'published';

-- ── User requests ──────────────────────────────────────────────────────
create table public.requests (
  id         uuid primary key default gen_random_uuid(),
  type       public.request_type not null,
  app_id     uuid references public.apps (id) on delete set null,
  name       text check (char_length(name) <= 80),
  email      text not null
             check (char_length(email) <= 254 and email ~* '^[^@\s]+@[^@\s]+\.[^@\s]+$'),
  message    text not null check (char_length(message) between 10 and 4000),
  status     public.request_status not null default 'new',
  created_at timestamptz not null default now()
);

create index requests_inbox_idx on public.requests (status, created_at desc);
create index requests_email_recent_idx on public.requests (lower(email), created_at desc);
```

---

## 3. Functions and triggers

```sql
-- Is the signed-in user an admin? (used by every policy)
create function public.is_admin() returns boolean
language sql stable security definer set search_path = public as $$
  select exists (select 1 from public.admins where user_id = auth.uid());
$$;

-- Keep updated_at / published_at current
create function public.apps_touch() returns trigger
language plpgsql as $$
begin
  new.updated_at := now();
  if new.status = 'published' and (tg_op = 'INSERT' or old.status <> 'published') then
    new.published_at := now();
  end if;
  return new;
end $$;

create trigger apps_touch before insert or update on public.apps
  for each row execute function public.apps_touch();

-- Simple anti-spam: max 3 requests per email per hour, 50 in total per hour
create function public.requests_rate_limit() returns trigger
language plpgsql security definer set search_path = public as $$
begin
  if (select count(*) from public.requests
      where lower(email) = lower(new.email) and created_at > now() - interval '1 hour') >= 3
     or (select count(*) from public.requests
      where created_at > now() - interval '1 hour') >= 50 then
    raise exception 'Too many requests. Please try again later.' using errcode = 'P0001';
  end if;
  new.status := 'new';   -- the public can't set their own status
  return new;
end $$;

create trigger requests_rate_limit before insert on public.requests
  for each row execute function public.requests_rate_limit();
```

The request form also has a hidden **honeypot** field. If a bot fills it in, the site pretends the request was sent and doesn't save it.

---

## 4. Row-Level Security (who can do what)

```sql
alter table public.apps     enable row level security;
alter table public.requests enable row level security;
alter table public.admins   enable row level security;

-- Apps: everyone can read published apps; admins can do everything
create policy apps_read   on public.apps for select using (status = 'published' or public.is_admin());
create policy apps_insert on public.apps for insert to authenticated with check (public.is_admin());
create policy apps_update on public.apps for update to authenticated using (public.is_admin()) with check (public.is_admin());
create policy apps_delete on public.apps for delete to authenticated using (public.is_admin());

-- Requests: anyone can submit; only admins can read or manage them
create policy requests_submit on public.requests for insert to anon, authenticated with check (true);
create policy requests_read   on public.requests for select to authenticated using (public.is_admin());
create policy requests_update on public.requests for update to authenticated using (public.is_admin());
create policy requests_delete on public.requests for delete to authenticated using (public.is_admin());

-- Admins: admins can see the list; changes are made only from the Supabase dashboard
create policy admins_read on public.admins for select to authenticated using (public.is_admin());
```

| Who | apps | requests | admins | app-media files |
|---|---|---|---|---|
| Visitor (not signed in) | Read **published** | Insert only | No access | Read |
| Admin | Full access (incl. drafts) | Read, update, delete | Read | Upload, replace, delete |

> The public form inserts **without asking for the new row back** (`.insert(row)` with no `.select()`), because visitors aren't allowed to read requests.

### Auth settings (Supabase dashboard)
- **Disable new sign-ups.** Only accounts you create can sign in.
- Turn on the **email + password** provider only.

**Making yourself admin (one time):**
```sql
insert into public.admins (user_id, email)
select id, email from auth.users where email = '<your-admin-email>';
```

**Adding an admin later:** create their user in *Authentication → Users*, then run the same insert with their email.

---

## 5. File storage

```sql
insert into storage.buckets (id, name, public, file_size_limit, allowed_mime_types)
values ('app-media', 'app-media', true, 2097152,                    -- 2 MB per file
        array['image/webp', 'image/png', 'image/jpeg']);             -- no SVG (can contain scripts)

create policy media_insert on storage.objects for insert to authenticated
  with check (bucket_id = 'app-media' and public.is_admin());
create policy media_update on storage.objects for update to authenticated
  using (bucket_id = 'app-media' and public.is_admin());
create policy media_delete on storage.objects for delete to authenticated
  using (bucket_id = 'app-media' and public.is_admin());
```

Because the bucket is public, anyone can view the images through their URL, but only admins can upload or delete them.

**File path convention**
```
app-media/
└─ apps/{app_id}/
   ├─ icon-{256|512}.webp
   ├─ screenshots/{image_id}-{480|960|1600}.webp
   ├─ features/{image_id}-{480|960}.webp
   └─ backgrounds/{image_id}-{960|1920}.webp
```

The browser resizes each image and converts it to **WebP** before uploading, at the widths above. Supabase's own image resizing is a paid feature, so we don't use it. The site then serves the right size with `srcset`. When an app is deleted, its `apps/{app_id}/` folder is deleted too.

---

## 6. Validation schema (Zod, `src/lib/schema/app.ts`)

```ts
import { z } from 'zod';

const hex = z.string().regex(/^#([0-9a-f]{6}|[0-9a-f]{8})$/i, 'Use a hex color like #1F6F68');
const url = z.string().url().max(500);
const httpsUrl = url.refine((u) => u.startsWith('https://'), 'Link must start with https://');

export const ImageRef = z.object({
  id: z.string().uuid(),
  alt: z.string().min(1, 'Describe the image for screen readers').max(200),
  caption: z.string().max(120).optional(),
  width: z.number().int().positive(),              // original size, used to prevent layout shift
  height: z.number().int().positive(),
  sources: z.array(z.object({ width: z.number().int(), path: z.string() })).min(1), // srcset
});

export const Background = z.discriminatedUnion('type', [
  z.object({ type: z.literal('solid'), color: hex }),
  z.object({ type: z.literal('gradient'), from: hex, to: hex, angle: z.number().min(0).max(360) }),
  z.object({
    type: z.literal('image'),
    image: ImageRef,
    overlay: hex,
    overlayOpacity: z.number().min(0).max(1),
  }),
]);

export const CardStyle = z.object({
  radius: z.enum(['sm', 'md', 'lg', 'xl']).default('lg'),
  border: z.enum(['none', 'subtle', 'accent']).default('subtle'),
  shadow: z.enum(['none', 'soft', 'lifted']).default('soft'),
  glass: z.boolean().default(false),
});

export const ThemeVariant = z.object({
  pageBackground: Background,
  cardBackground: Background,
  accent: hex,
  text: hex,
  mutedText: hex,
});

export const AppTheme = z.object({
  preset: z.string().optional(),                   // e.g. "ocean": the preset it started from
  light: ThemeVariant,
  dark: ThemeVariant.optional(),                   // generated from light if omitted
  hero: z.object({
    layout: z.enum(['centered', 'split', 'banner']).default('split'),
    showScreenshot: z.boolean().default(true),
  }),
  card: CardStyle,
});

export const Feature = z.object({
  id: z.string().uuid(),
  title: z.string().min(1).max(60),
  description: z.string().min(1).max(300),
  icon: z.string().max(40).optional(),             // Lucide icon name, e.g. "zap"
  image: ImageRef.optional(),
  cardStyle: z.object({ background: Background.optional(), accent: hex.optional() }).optional(),
});

export const Platforms = z.object({
  playStore: httpsUrl.optional(),
  appStore: httpsUrl.optional(),
  desktop: z.object({
    windows: httpsUrl.optional(),
    macos: httpsUrl.optional(),
    linux: httpsUrl.optional(),
  }).optional(),
  web: httpsUrl.optional(),
});

export const Support = z.object({
  feedbackFormUrl: httpsUrl.optional(),            // empty → site-wide Google Form, app pre-filled
  faqs: z.array(z.object({
    id: z.string().uuid(),
    question: z.string().min(1).max(200),
    answer: z.string().min(1).max(2000),           // markdown
  })).default([]),
  help: z.string().max(10000).default(''),         // markdown
  contactEmail: z.string().email().optional(),
});

const slug = z.string().regex(/^[a-z0-9]+(-[a-z0-9]+)*$/, 'Lowercase letters, numbers and dashes').max(60);

/** Everything the builder form edits. "Save Draft" validates with this. */
export const AppDraft = z.object({
  slug,
  status: z.enum(['draft', 'published']),
  sortOrder: z.number().int().default(0),
  name: z.string().min(1).max(60),
  tagline: z.string().max(80).default(''),
  description: z.string().max(5000).default(''),  // markdown
  version: z.string().max(20).optional(),
  icon: ImageRef.nullable(),
  screenshots: z.array(ImageRef).max(12).default([]),
  features: z.array(Feature).max(12).default([]),
  platforms: Platforms.default({}),
  support: Support.default({}),
  theme: AppTheme,
});

/** "Add App" (publish) validates with this stricter version. */
export const AppPublish = AppDraft.extend({
  status: z.literal('published'),
  icon: ImageRef,
  tagline: z.string().min(1).max(80),
  description: z.string().min(1).max(5000),
  screenshots: z.array(ImageRef).min(1).max(12),
  features: z.array(Feature).min(1).max(12),
}).refine(
  (a) => Boolean(a.platforms.playStore || a.platforms.appStore || a.platforms.web ||
    a.platforms.desktop?.windows || a.platforms.desktop?.macos || a.platforms.desktop?.linux),
  { path: ['platforms'], message: 'Add at least one download or web link' },
);

/** A row as read from the database. */
export const App = AppDraft.extend({
  id: z.string().uuid(),
  publishedAt: z.string().datetime().nullable(),
  createdAt: z.string().datetime(),
  updatedAt: z.string().datetime(),
});

export type App = z.infer<typeof App>;
export type AppDraft = z.infer<typeof AppDraft>;
export type AppTheme = z.infer<typeof AppTheme>;
export type Feature = z.infer<typeof Feature>;

export const RequestInput = z.object({
  type: z.enum(['feature', 'new_app', 'bug', 'other']),
  appId: z.string().uuid().optional(),
  name: z.string().max(80).optional(),
  email: z.string().email().max(254),
  message: z.string().min(10, 'Tell us a bit more (10+ characters)').max(4000),
  website: z.string().max(0).optional(),           // honeypot: must stay empty
});
```

**Name mapping:** the database uses `snake_case` (`sort_order`, `published_at`) and TypeScript uses `camelCase` (`sortOrder`, `publishedAt`). The conversion happens in one place, `lib/api/apps.ts`.

**Checks happen in two places.** Zod gives friendly messages in the form. The database constraints and security policies are the final safeguard, even if someone bypasses the form.

---

## 7. Example record (`src/data/seed-apps.json`)

```json
{
  "id": "5b1c7a4e-3f0d-4b8a-9d62-1a2b3c4d5e6f",
  "slug": "snip",
  "status": "published",
  "sortOrder": 1,
  "name": "Snip",
  "tagline": "Save text once, paste it anywhere.",
  "description": "Snip keeps the phrases you type every day (addresses, replies, code) one shortcut away.",
  "version": "1.4.0",
  "icon": {
    "id": "0c9e…", "alt": "Snip app icon", "width": 512, "height": 512,
    "sources": [{ "width": 256, "path": "apps/5b1c…/icon-256.webp" }, { "width": 512, "path": "apps/5b1c…/icon-512.webp" }]
  },
  "screenshots": [
    { "id": "8f21…", "alt": "Snippet library in dark mode", "caption": "All your snippets in one place",
      "width": 1600, "height": 1000,
      "sources": [{ "width": 480, "path": "…-480.webp" }, { "width": 960, "path": "…-960.webp" }, { "width": 1600, "path": "…-1600.webp" }] }
  ],
  "features": [
    { "id": "a1…", "title": "Instant paste", "description": "Type ;addr and your address appears.", "icon": "zap" },
    { "id": "a2…", "title": "Syncs everywhere", "description": "Phone, laptop and browser stay in step.", "icon": "refresh-cw",
      "cardStyle": { "accent": "#2E9187" } }
  ],
  "platforms": {
    "playStore": "https://play.google.com/store/apps/details?id=dev.sungaru.snip",
    "desktop": { "windows": "https://…/Snip-Setup.exe", "macos": "https://…/Snip.dmg" },
    "web": "https://snip.<your-domain>"
  },
  "support": {
    "faqs": [{ "id": "f1…", "question": "Is Snip free?", "answer": "Yes. All core features are free." }],
    "help": "## Getting started\n1. Install Snip…",
    "contactEmail": "support@<your-domain>"
  },
  "theme": {
    "preset": "ocean",
    "light": {
      "pageBackground": { "type": "gradient", "from": "#EEF7F6", "to": "#FFFFFF", "angle": 180 },
      "cardBackground": { "type": "solid", "color": "#FFFFFF" },
      "accent": "#1F6F68", "text": "#16181A", "mutedText": "#5B6168"
    },
    "hero": { "layout": "split", "showScreenshot": true },
    "card": { "radius": "lg", "border": "subtle", "shadow": "soft", "glass": false }
  },
  "publishedAt": "2026-10-01T09:00:00Z",
  "createdAt": "2026-09-30T12:00:00Z",
  "updatedAt": "2026-10-01T09:00:00Z"
}
```

This record renders with: a Google Play button, Windows and macOS buttons, and a web app button. There is **no** App Store or Linux button, because those links aren't set. Its dark theme is generated automatically because `dark` is omitted.

---

## 8. Data access API (`src/lib/api/`)

The same interface has two implementations: **mock** (`seed-apps.json` plus local storage) and **supabase**. The `VITE_DATA_SOURCE` setting chooses between them.

| Function | Who | Purpose |
|---|---|---|
| `listApps({ platform?, search? })` | Public | Published apps for the catalogue |
| `getApp(slug)` | Public | One published app (admins also see drafts) |
| `submitRequest(input)` | Public | Save a user request |
| `listAllApps()` | Admin | All apps, including drafts |
| `createApp(draft)` / `updateApp(id, draft)` | Admin | Save from the builder |
| `publishApp(id)` / `unpublishApp(id)` | Admin | Change status |
| `reorderApps(ids[])` | Admin | Set `sort_order` |
| `deleteApp(id)` | Admin | Delete the app and its media folder |
| `uploadImage(appId, file, kind)` | Admin | Resize → WebP → upload → returns `ImageRef` |
| `listRequests(status?)` / `setRequestStatus(id, status)` | Admin | Request inbox |

---

## 9. Keep-alive (free-tier pause prevention)

`.github/workflows/keep-alive.yml` runs every 3 days and calls:

```
GET {SUPABASE_URL}/rest/v1/apps?select=id&limit=1
apikey: {SUPABASE_ANON_KEY}
```

The URL and key are stored as GitHub Actions secrets. The key is the public one, so this is safe.
