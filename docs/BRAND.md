# Sungaru Dev: Brand Identity

This is the reference for how Sungaru Dev looks, sounds, and behaves on the website and in every app listing. The design tokens here map one-to-one to the CSS variables in `src/styles/globals.css` (see §10).

---

## 1. Brand essence

| | |
|---|---|
| **Name** | Sungaru Dev |
| **What we do** | We build apps that solve real user pain points and improve everyday UX. |
| **Tagline** | *Software that just makes sense.* |
| **Alternatives** | *Less friction. Better apps.* · *Built around the people who use it.* |
| **Mission** | Find the small frustrations people put up with every day, and remove them with software that is simple, fast, and respectful of their time. |
| **Promise to users** | Every Sungaru app is easy to learn, pleasant to use, and honest about what it does. |

### Personality

| We are | We are not |
|---|---|
| **Calm**: quiet confidence, nothing shouts | Loud, hyped, flashy |
| **Clear**: plain words, obvious next steps | Clever at the cost of clarity |
| **Thoughtful**: details are considered | Rushed or cluttered |
| **Reliable**: consistent and dependable | Trendy for the sake of it |
| **Human**: warm and helpful | Corporate or cold |

---

## 2. Voice and tone

- **Write like a helpful person, not a company.** Use "you" and "we".
- **Lead with the user's problem, then the fix.** For example: "Tired of re-typing the same notes? Snip saves them once and fills them in anywhere."
- **Keep it short.** Headlines should be ≤ 8 words and sentences ≤ 20 words where possible.
- **Be specific.** "Syncs in under a second" beats "blazing fast".
- **Avoid jargon, exclamation marks, and buzzwords** ("revolutionary", "next-gen", "synergy").

| Situation | ✅ Do | ❌ Don't |
|---|---|---|
| Hero headline | Apps that remove everyday friction. | The world's most revolutionary apps!!! |
| Button | Get the app · View details · Send feedback | Click here · Submit |
| Error | We couldn't load this app. Check your connection and try again. | Error 500: request failed |
| Empty state | No apps match "photo". Try a different word. | No results. |
| Success | Thanks! Your request has been sent. | Request successfully submitted to server. |

---

## 3. Logo

### Concept
The **mark** is a rounded, geometric **"S"** built from two arcs that flow into each other, like a smooth path through a problem. It sits inside a soft-cornered square (a squircle, like an app icon) so it reads naturally next to app icons.

The **wordmark** is "Sungaru" in Plus Jakarta Sans ExtraBold (800) with "Dev" in Medium (500). The weight contrast keeps "Sungaru" as the name people remember.

```
 ┌─────────┐
 │  ╭──╮   │   Sungaru Dev
 │  ╰──╮   │   ───────  ───
 │  ╰──╯   │   800      500
 └─────────┘
   mark         wordmark
```

### Variants
| Variant | Use |
|---|---|
| **Horizontal lockup** (mark + wordmark) | Site header, footer, emails |
| **Mark only** | Favicon, app-touch icon, social avatar, loading screen |
| **Wordmark only** | Where the mark is already visible nearby |

Each variant comes in **teal on light**, **light teal on dark**, and **one-color** (black or white) versions.

### Rules
- **Clear space:** keep at least the height of the "S" stroke × 2 empty around the logo.
- **Minimum size:** mark 16px (favicon), lockup 96px wide on screen.
- **Don't:** stretch, rotate, recolor outside the palette, add shadows or glows, or place it on busy images without a solid backdrop.

**Files (Phase 0):** `public/brand/logo-mark.svg`, `logo-lockup.svg`, `logo-lockup-dark.svg`, `favicon.svg`, `apple-touch-icon.png` (180px), `icon-512.png`, `og-default.png` (1200×630).

---

## 4. Color

### 4.1 Brand colors

| Name | Hex | Role |
|---|---|---|
| **Sungaru Teal** | `#1F6F68` | Primary brand color: buttons, links, logo, focus |
| **Teal Light** | `#4FB3A9` | Primary color in dark mode |
| **Warm Sand** | `#C98B4B` | Accent: small highlights, illustrations, "New" dots. **Not for text on light backgrounds** |
| **Sand Strong** | `#9A6330` | Accent when it must be readable text on light backgrounds |
| **Ink** | `#16181A` | Main text (light mode) |
| **Paper** | `#FAFAF9` | Page background (light mode) |
| **Night** | `#0E1113` | Page background (dark mode) |

**Usage ratio:** about 80% neutrals, 15% teal, and ≤ 5% sand. This keeps the brand calm and "not loud".

### 4.2 Teal scale

| 50 | 100 | 200 | 300 | 400 | 500 | 600 | 700 | 800 | 900 | 950 |
|---|---|---|---|---|---|---|---|---|---|---|
| `#EEF7F6` | `#D5ECE9` | `#ABD8D3` | `#7CC2BA` | `#4FB3A9` | `#2E9187` | **`#1F6F68`** | `#1A5A55` | `#164845` | `#12302D` | `#0B1F1D` |

### 4.3 Semantic tokens

| Token | Light | Dark | Use |
|---|---|---|---|
| `--bg` | `#FAFAF9` | `#0E1113` | Page background |
| `--surface` | `#FFFFFF` | `#161A1D` | Cards, panels, header |
| `--surface-2` | `#F2F3F4` | `#1D2226` | Inputs, hover states, nested panels |
| `--border` | `#E4E6E8` | `#262B30` | Dividers, card borders |
| `--text` | `#16181A` | `#ECEDEE` | Body text, headings |
| `--muted` | `#5B6168` | `#9AA1A8` | Secondary text, captions |
| `--brand` | `#1F6F68` | `#4FB3A9` | Primary actions, links |
| `--brand-contrast` | `#FFFFFF` | `#0E1113` | Text on a `--brand` button |
| `--brand-soft` | `#E6F2F1` | `#12302D` | Tinted backgrounds, badges |
| `--accent` | `#C98B4B` | `#E0A96D` | Decorative highlights |
| `--success` | `#1E7A45` | `#4CC38A` | Success messages |
| `--warning` | `#8A5A00` | `#F0B849` | Warnings |
| `--danger` | `#B42318` | `#F97066` | Errors, destructive actions |
| `--info` | `#1D5FA8` | `#6AA8F0` | Informational notes |
| `--focus` | `#1F6F68` | `#4FB3A9` | 2px focus ring, 2px offset |

### 4.4 Checked contrast (WCAG 2.1)

| Pair | Light | Dark |
|---|---|---|
| Text on background | 17.0 : 1 | 16.2 : 1 |
| Muted text on background | 6.0 : 1 | 7.3 : 1 |
| Brand (link) on background | 5.7 : 1 | 7.5 : 1 |
| Button text on brand | 6.0 : 1 | 7.5 : 1 |
| Brand on brand-soft (badge) | 5.2 : 1 | 5.6 : 1 |
| Accent on background | 2.8 : 1 ⚠️ decorative only | 9.1 : 1 |
| Sand Strong on background | 4.8 : 1 | n/a |
| Success / warning / danger / info on surface | 5.4 / 5.9 / 6.6 / 6.5 | 7.9 / 9.7 / 6.3 / 7.1 |

All pairs used for text pass **AA (≥ 4.5 : 1)**. Light-mode Warm Sand does not, so use it only for decoration and switch to Sand Strong when sand-colored text is needed.

---

## 5. Typography

| Role | Family | Weights | Source |
|---|---|---|---|
| Display and headings | **Plus Jakarta Sans** | 600, 700, 800 | Google Fonts (free) |
| Body and UI | **Inter** | 400, 500, 600 | Google Fonts (free) |
| Code, versions | **JetBrains Mono** | 400 | Google Fonts (free) |

The fonts are self-hosted as `woff2` through `@fontsource` packages, which avoids extra network requests and cookie banners.

### Type scale (fluid, 360px → 1280px)

| Token | Size | Line height | Weight | Letter spacing | Use |
|---|---|---|---|---|---|
| `display` | `clamp(2.5rem, 1.6rem + 4vw, 4rem)` | 1.05 | 800 | -0.03em | Home hero |
| `h1` | `clamp(2rem, 1.5rem + 2.2vw, 3rem)` | 1.1 | 800 | -0.02em | Page titles, app name |
| `h2` | `clamp(1.5rem, 1.25rem + 1.1vw, 2rem)` | 1.2 | 700 | -0.015em | Section titles |
| `h3` | `1.25rem` | 1.3 | 700 | -0.01em | Card titles, feature titles |
| `body-lg` | `1.125rem` | 1.6 | 400 | 0 | Intros, taglines |
| `body` | `1rem` | 1.6 | 400 | 0 | Default text |
| `small` | `0.875rem` | 1.5 | 500 | 0 | Labels, meta |
| `caption` | `0.75rem` | 1.4 | 500 | 0.02em | Badges, screenshot captions |

Long text is limited to about 70 characters per line (`max-width: 68ch`).

---

## 6. Layout, spacing, shape

- **Spacing scale (4px base):** 4 · 8 · 12 · 16 · 24 · 32 · 48 · 64 · 96 · 128
- **Container:** max-width 1200px, side margin 16px on mobile, 24px on tablet, 32px on desktop
- **Grid:** 4 columns on mobile, 8 on tablet, 12 on desktop, with a 24px gap; app cards are 1 / 2 / 3 per row
- **Breakpoints:** `sm 640` · `md 768` · `lg 1024` · `xl 1280`

| Radius | Value | Use |
|---|---|---|
| `sm` | 6px | Inputs, badges |
| `md` | 10px | Buttons |
| `lg` | 16px | Cards |
| `xl` | 24px | Hero panels, app icon frame |
| `full` | 9999px | Pills, avatars |

| Shadow | Light | Dark |
|---|---|---|
| `soft` | `0 1px 2px rgb(16 24 32 / .06), 0 1px 3px rgb(16 24 32 / .08)` | none (use a border) |
| `lifted` | `0 8px 24px -6px rgb(16 24 32 / .12)` | `0 8px 24px -6px rgb(0 0 0 / .5)` |

---

## 7. Iconography and imagery

- **Icons:** Lucide at 1.75px stroke, 20px in UI and 24px in feature cards, colored `currentColor`.
- **Platform badges:** use the official Google Play and App Store badge artwork, as required by their guidelines. Desktop and Web use Lucide `monitor` and `globe` icons in brand-styled buttons.
- **Screenshots:** real app screens only; rounded corners (`lg`), shown at their original aspect ratio, with alt text always provided. Optional device frames come later (v2).
- **Illustrations:** simple geometric shapes in teal and sand, used sparingly (hero, empty states, 404).
- **Photos:** not used on the brand site in v1.

---

## 8. Motion

| Token | Duration | Easing | Use |
|---|---|---|---|
| `fast` | 120ms | `cubic-bezier(.2,0,0,1)` | Hover, press, toggle |
| `base` | 200ms | `cubic-bezier(.2,0,0,1)` | Card lift, dropdowns |
| `slow` | 320ms | `cubic-bezier(.2,0,0,1)` | Page transitions, dialogs |

Motion should be small (≤ 8px movement, fades) and never block the user. With `prefers-reduced-motion`, all transitions become instant fades or are removed.

---

## 9. Core components

| Component | Spec |
|---|---|
| **Primary button** | `--brand` background, `--brand-contrast` text, radius `md`, height 44px, `small`/600 text; hover darkens 8%, and there is a visible focus ring |
| **Secondary button** | Transparent, 1px `--border`, `--text` color; hover `--surface-2` |
| **Ghost button** | No border; `--brand` text; for less important actions |
| **App card** | `--surface`, radius `lg`, 1px `--border`, 24px padding; 56px app icon, name (h3), tagline (muted, 2 lines max), platform badges; on hover it lifts 2px, gets the `lifted` shadow, and its border takes the app's accent color |
| **Feature card** | Icon in a 40px tinted circle, title (h3), description (body); styled by the app theme |
| **Badge** | `--brand-soft` background, `--brand` text, `caption` size, radius `full` |
| **Input** | Height 44px, `--surface` background, 1px `--border`, radius `sm`; focus shows a `--focus` ring; error shows a `--danger` border and message |
| **Theme toggle** | Icon button in the header (sun / moon / monitor) with a tooltip and `aria-label` |

---

## 10. Brand site vs. app themes

- The **header, footer, navigation, and admin area always use the Sungaru brand**. This keeps the site recognisable.
- The **app detail pages** (`/apps/:slug/*`) use that app's own theme (page background, card background, and accent) for the content area.
- App themes can override colors but **not** fonts, spacing, or components, so every app page still looks like part of the Sungaru family.
- The builder's contrast checker enforces **AA contrast** for text on each app's backgrounds.

### Tokens in code

```css
:root {
  --bg: #FAFAF9;  --surface: #FFFFFF;  --surface-2: #F2F3F4;  --border: #E4E6E8;
  --text: #16181A; --muted: #5B6168;
  --brand: #1F6F68; --brand-contrast: #FFFFFF; --brand-soft: #E6F2F1;
  --accent: #C98B4B; --accent-strong: #9A6330;
  --success: #1E7A45; --warning: #8A5A00; --danger: #B42318; --info: #1D5FA8;
  --focus: var(--brand);
  --radius-sm: 6px; --radius-md: 10px; --radius-lg: 16px; --radius-xl: 24px;
  --font-display: "Plus Jakarta Sans", system-ui, sans-serif;
  --font-body: "Inter", system-ui, sans-serif;
  --font-mono: "JetBrains Mono", ui-monospace, monospace;
}
.dark {
  --bg: #0E1113;  --surface: #161A1D;  --surface-2: #1D2226;  --border: #262B30;
  --text: #ECEDEE; --muted: #9AA1A8;
  --brand: #4FB3A9; --brand-contrast: #0E1113; --brand-soft: #12302D;
  --accent: #E0A96D; --accent-strong: #E0A96D;
  --success: #4CC38A; --warning: #F0B849; --danger: #F97066; --info: #6AA8F0;
}
```
