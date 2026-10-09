# TorQIevo Website: Agent Build Specification

> **Audience:** AI coding agent (Antigravity or similar).
> **Goal:** Build the complete, production-ready marketing and inquiry website for **TorQIevo** as a pure Next.js application, with automated tests and a GitHub Actions CI/CD pipeline that deploys to **Vercel**.
> **Authority order:** (1) this file for structure, behavior, SEO and integrations; (2) `docs/Torqievo_design.txt` for ALL visual decisions (colors, type, spacing, radii, shadows, components). If they conflict on visuals, the design file wins. If they conflict on anything else, this file wins.

---

## 0. How to Work

1. Read this entire file and `docs/Torqievo_design.txt` **before writing any code**. The owner has placed the design file at `docs/Torqievo_design.txt`; if it is missing, stop and ask for it.
2. Read `AGENTS.md` at the repo root and the Next.js docs it points to (the scaffold uses a very recent Next.js; do not rely on memory of older APIs). Follow `AGENTS.md` where it does not conflict with this file.
3. The project is **already scaffolded** (Section 0.1). Do **not** re-run `create-next-app` or create a second app. Work inside the existing project.
4. Build in the phase order in Section 15. Do not skip phases.
5. After each phase, run `npm run lint`, `npm run typecheck`, `npm run test` (once tests exist) and `npm run build`. Fix errors before continuing.
6. Do not invent colors, fonts, or styles. Map every design token from the design file into CSS variables and the Tailwind `@theme` block first, then use only those tokens.
7. Where this file says `TODO_*` or `[PLACEHOLDER]`, centralize the value in `src/config/site.ts` so the owner can change it in one place. Never hardcode these values elsewhere.
8. **Secrets policy (Section 13A) and CI/CD policy (Section 13C) are mandatory.** Never print, log, commit, or echo secret values.
9. When finished, produce `README.md` with setup, the env/secrets workflow, Google Sheets setup, Vercel + GitHub Actions setup, and the post-launch SEO checklist.

### 0.1 Current project state (already done by the owner)

- Repo root (the Next.js project): `E:\TorQIevo\Torqievo_website\Torqievo_website\torqievo_website`
- Package name: `torqievo_website`
- Created with `create-next-app@16.4.0`: **TypeScript, ESLint, Tailwind CSS (CSS-first / v4 style), `src/` directory, App Router, npm, default import alias `@/*`, `AGENTS.md` included, React Compiler = No.**
- The owner was also asked about *Cache Components* and *agent feedback* during scaffolding; the answers are not recorded here. **Inspect `next.config.ts` first.** If `cacheComponents` is enabled, follow the current Next.js docs for it (dynamic data must be wrapped in `<Suspense>`; route handlers are unaffected) and keep all marketing pages statically prerendered. If it is not enabled, do not enable it.
- Dependencies currently installed: only the scaffold defaults. Everything else in Section 2 must be installed by the agent.
- npm reported 5 high-severity audit findings and a pending `allowScripts` notice for `unrs-resolver`. The agent must: (a) run `npm audit` and resolve what can be fixed **without breaking changes**; (b) run `npm approve-scripts` only for packages that legitimately need install scripts (e.g. `unrs-resolver`, used by the ESLint toolchain) and explain each approval in the README; (c) list anything unresolved in the README. Do not run `npm audit fix --force`.

---

## 1. Brand & Company

| Field | Value |
|---|---|
| Brand name | **TorQIevo** (letters: T-O-R-Q-I-E-V-O) |
| Tagline | Technology that moves ideas forward. |
| Type | Technology company / software product studio |
| Team | Two founding partners |
| Market | Clients across India and worldwide |
| Offer | Scalable web applications and mobile applications for any kind of business or individual, delivered at **affordable, minimal charges** for building, scaling, and hosting |

**Brand name handling:** Define `BRAND_NAME = "TorQIevo"` in `src/config/site.ts` and use it everywhere (UI, metadata, JSON-LD, llms.txt, emails). The design file writes it as "Torqievo"; the owner's preferred display is **TorQIevo**. Use the constant so it can be changed in one place. Never write "Torqivo", "Torquivo", or any other variant.

**Voice:** professional, warm, clear, confident, plain English. No jargon walls, no hype, no fake claims. Do not invent client names, testimonials, statistics, or awards. Use clearly marked placeholder content that the owner can replace.

### Services (used on Home, Inquiry form, FAQ, SEO, llms.txt)

1. Web Application Development
2. Mobile Application Development (Android, iOS, cross-platform)
3. Application Rebuild & Modernization (rebuild older websites and apps)
4. Technical Support & Maintenance (help with existing applications)
5. Scaling & Hosting (deployment, performance, uptime)
6. Technical Consulting & Pricing Guidance

---

## 2. Tech Stack (fixed)

- **Next.js** (the version already installed by the scaffold, 16.x) with **App Router**, **TypeScript** (strict)
- **Tailwind CSS** as already installed (CSS-first). **Do not create `tailwind.config.ts`/`.js`.** Map all design tokens with the `@theme` directive in `src/app/globals.css`.
- **Inter** via `next/font/google` (the only font family)
- **lucide-react** for icons (outline style only, consistent stroke width)
- **framer-motion** for subtle animations (respect `prefers-reduced-motion`)
- **react-hook-form** + **zod** for form handling and validation
- **googleapis** for Google Sheets (server-side only, see Section 8)
- **dotenv** for loading `.env_production` (Section 13A)
- **Testing (Section 13B):** Vitest + React Testing Library + jsdom (unit/component), Playwright (end-to-end smoke tests)
- **tsx** (dev dependency) for build scripts such as the `llms.txt` generator
- **No database. No ORM. No separate backend service.**
- Package manager: `npm` (commit `package-lock.json`; CI uses `npm ci`)
- Runtime: Node.js 22 LTS (Next 16 requires Node ≥ 20.9). Set `"engines": { "node": ">=20.9" }` in `package.json` and use Node 22 in CI.
- **Deployment target: Vercel only.** Netlify is **not** used (Section 13C).

Keep dependencies minimal. Do not add UI kits that bring their own visual style.

---

## 3. Project Structure

```
torqievo_website/                 (repo root, already exists)
├─ .github/workflows/ci-cd.yml    (Section 13C)
├─ docs/Torqievo_design.txt
├─ public/
│  ├─ favicon.ico, icon.svg, apple-touch-icon.png
│  ├─ og-default.png              (1200x630, brand-styled)
│  ├─ llms.txt                    (GENERATED by scripts/generate-llms.ts; see Section 10.4)
│  ├─ llms-full.txt               (GENERATED; see Section 10.4)
│  └─ projects/                   (project preview images)
├─ scripts/
│  ├─ generate-llms.ts            (builds llms.txt + llms-full.txt from data/config)
│  ├─ load-env.ts                 (loads .env_production; shared by next.config.ts and scripts)
│  ├─ check-env-hygiene.mjs       (fails if stray env files exist or are tracked by git)
│  └─ sync-env.mjs                (pushes .env_production to Vercel + GitHub; Section 13A)
├─ src/
│  ├─ app/
│  │  ├─ globals.css              (CSS variables + @theme tokens from design file Section 45)
│  │  ├─ layout.tsx
│  │  ├─ page.tsx                 (Home)
│  │  ├─ about/page.tsx
│  │  ├─ inquiry/page.tsx
│  │  ├─ support/page.tsx
│  │  ├─ faq/page.tsx             (full FAQ page; FAQ also embedded on Home)
│  │  ├─ privacy/page.tsx
│  │  ├─ terms/page.tsx
│  │  ├─ not-found.tsx
│  │  ├─ error.tsx
│  │  ├─ robots.ts
│  │  ├─ sitemap.ts
│  │  ├─ manifest.ts
│  │  ├─ opengraph-image.tsx      (optional dynamic OG)
│  │  └─ api/inquiry/route.ts     (Google Sheets writer)
│  ├─ components/
│  │  ├─ layout/ (Header, SlideMenu, Footer, SplashScreen, PageTransition)
│  │  ├─ home/ (Hero, ServicesGrid, Projects, TechStack, Process, WhyUs, FAQPreview, CTASection)
│  │  ├─ inquiry/ (InquiryForm, CategoryPicker, SuccessState)
│  │  ├─ faq/ (FAQAccordion)
│  │  ├─ seo/ (JsonLd)
│  │  └─ ui/ (Button, Card, Badge, Input, Textarea, Select, Container, Section)
│  ├─ config/site.ts              (brand, contacts, nav, socials, services)
│  ├─ data/ (projects.ts, faqs.ts, services.ts, tech.ts, process.ts)
│  └─ lib/ (env.ts, sheets.ts, validation.ts, rateLimit.ts, seo.ts, utils.ts)
├─ tests/
│  ├─ unit/                       (Vitest)
│  └─ e2e/                        (Playwright)
├─ vercel.json                    (Section 13C)
├─ vitest.config.ts
├─ playwright.config.ts
├─ .gitignore                     (Section 13A rules)
├─ AGENTS.md                      (from scaffold; keep)
├─ README.md
└─ TORQIEVO_AGENT.md              (this file)
```

---

## 4. Design System Implementation (non-negotiable)

Implement the design file exactly. Summary of what must be wired up:

1. Put the **CSS variables from design file Section 45** into `src/app/globals.css` verbatim. Add `--color-dark-hover: #2B2838` (dark button hover, design Section 13).
2. Register every token in the `@theme` block of `src/app/globals.css` (Tailwind v4 CSS-first; no `tailwind.config`) so every token is available as a utility, e.g. `bg-brand-secondary`, `text-text-secondary`, `border-border`, `rounded-card`, `shadow-md`. Do **not** leave Tailwind's default palette in use for anything visible.
3. Typography scale (design Section 9) as reusable classes/components: Display, H1 to H4, Body Large, Body, Small, Caption, with the specified desktop/tablet/mobile sizes, weights, line heights, letter spacing.
4. Spacing: 8px system only (design Section 31). Section vertical spacing and container widths per Sections 32 to 33 (max width 1200px; horizontal padding 20/32/40px mobile/tablet/desktop).
5. Radii: badge 8, input 10, button 12, card 18, large card 24, hero container 28, modal 24.
6. Shadows: only `sm`, `md`, `lg` from Section 18.
7. Animations: 200ms default, 300ms large, `ease`. Allowed: fade, small translate, card lift, button/nav transitions. Forbidden list in design Section 34 applies.
8. Section background sequence (design Section 24): Hero ivory, Services white, About/Approach ivory, Technology lavender mist, Projects white, CTA dark, Footer dark.
9. Semantic states (error, success, warning, info) use design Sections 38 to 41 exactly.
10. Hero visuals: **build with CSS/SVG/React** (floating UI panels, abstract app interfaces, a mobile device frame). No stock photos, no neon, no robots, no cyberpunk (design Section 22).
11. Logo: build as an inline SVG/React component. Wordmark in `#171522` with a controlled `#8B7CF6` accent detail (design Section 11). Include a compact mark for favicon and splash.

**Accessibility floor:** WCAG AA contrast, visible focus rings (use the input focus ring spec), semantic landmarks, keyboard navigable menu and accordion, `aria-*` where needed, `prefers-reduced-motion` respected everywhere.

---

## 5. Global Layout & Navigation

### 5.1 Top navigation bar (Header)

- Sticky. Ivory background (`#F8F6F0`); after scrolling past ~16px switch to the floating variant (white, `#E5E1EA` border, `shadow-sm`) with a 200ms transition.
- Left: logo (links to `/`).
- Center/right (desktop ≥1024px): `Home`, `About`, `Support`, `FAQ` as text links (color `#171522`, hover/active `#6C5CE7`; do not use underlines as the hover treatment).
- Right: primary CTA button **"Start an Inquiry"** → `/inquiry`. (One primary CTA per visual group.)
- Active route clearly highlighted.
- Below 1024px: show logo + hamburger button that opens the **sliding menu**.

### 5.2 Sliding menu (mobile and tablet; also usable on desktop via a menu button if desired)

- Slides in from the **right** (300ms ease) with a dimmed backdrop (`rgba(23,21,34,0.4)`; no heavy blur).
- Panel: white surface, 24px left radius, large shadow.
- Contents: logo, close button, large nav links (Home, About, Inquiry, Support, FAQ), primary CTA "Start an Inquiry", business email and phone/WhatsApp (from config), social links.
- Behavior requirements:
  - Closes on backdrop click, `Esc`, link click, and route change.
  - Focus trap while open; focus returns to the hamburger button on close.
  - Locks body scroll while open.
  - `aria-modal`, `role="dialog"`, `aria-label`.
  - Staggered fade-in of links is allowed (subtle, short).

### 5.3 Footer

Dark (`#171522`), premium and understated, following design Section 26 exactly (text `#FFFFFF`, secondary `#B9B5C2`, links `#DCD6FF` with hover `#8B7CF6`, borders `rgba(255,255,255,0.12)`).

Columns:
1. **Brand**: logo (light version), tagline, one-sentence description.
2. **Company**: Home, About, Support, FAQ, Inquiry.
3. **Services**: the six services (each linking to `/inquiry?category=<slug>`).
4. **Contact**: business emails (see Section 9), phone/WhatsApp, location "India, serving clients worldwide", response-time note.

Bottom bar: `© {currentYear} TorQIevo. All rights reserved.`, Privacy/Terms placeholders (create simple `/privacy` and `/terms` pages with clearly marked placeholder copy), and a "Back to top" control.

### 5.4 Splash screen (brand intro)

- Shown **once per browser session** on first load (use `sessionStorage` flag; guard for SSR). Do not show on subsequent client-side navigations.
- Background `#F8F6F0`. Center: animated logo mark drawing/fading in, then the wordmark "TorQIevo", then the tagline in muted text. A thin lavender progress line may be used.
- Total duration about **1.6 to 2.2 seconds**, then a fade/slide-up exit (300ms) revealing the page.
- Must **not** block SEO: server-render the real page content underneath; the splash is a client overlay only. Must not cause layout shift or hurt LCP. Crawlers and no-JS users must see the full page immediately.
- Respect `prefers-reduced-motion` (show a static brief fade instead).
- Allow skipping on click/tap/keypress.
- Prevent flash of content before the splash mounts (use an inline script or CSS class on `<html>` set early, or render the overlay on the server with a client component that removes itself).

### 5.5 Page transitions & loading

- Subtle fade/translate-in on route changes (no large page transitions).
- Add `loading.tsx` skeletons where relevant. Use `next/image` and `next/link` throughout.
- Smooth scrolling for in-page anchors, with header offset.

---

## 6. Pages

### 6.1 Home (`/`)

Sections in this order, with backgrounds per design Section 24:

1. **Hero** (ivory, optional very subtle gradient from the design file)
   - Badge: "Web & Mobile Application Studio".
   - Display heading, e.g. "Build Better **Digital Experiences**" (highlighted phrase in `#6C5CE7`). Keep the final copy keyword-aware: it must naturally convey *affordable web and mobile app development*.
   - Supporting paragraph: scalable web and mobile apps for businesses in India and worldwide, with affordable pricing for building, scaling, and hosting.
   - CTAs: primary "Start an Inquiry" (`/inquiry`), secondary "View Our Work" (anchors to Projects).
   - Trust row (no fake numbers): "Affordable pricing", "Scalable architecture", "Hosting & support included", "Clients across India & worldwide".
   - Right-side visual: composed floating UI panels + phone frame built with CSS/SVG. Gentle one-time entrance animation and small hover movement only.
2. **Services** (white): six service cards (icon in `#E9E6FF` container, icon `#6C5CE7`). Each card links to `/inquiry?category=<slug>`.
3. **How We Work / Approach** (ivory): 4 steps: Discuss → Plan & Quote → Build & Review → Launch, Host & Support.
4. **Technology** (lavender mist `#E9E6FF`): tech cards (white, `#E5E1EA` border), monochrome/dark logos or outline icons per design Section 36. Examples: Next.js, React, Node.js, TypeScript, React Native, Flutter, Firebase, PostgreSQL, Tailwind CSS, Vercel. Only list technologies the team confirms; make the list editable in `data/tech.ts`.
5. **Latest Projects** (white): project cards per design Section 37. Grid of 3 to 6 projects from `data/projects.ts` with: preview image (CSS/SVG mock if no image), category label (`#6C5CE7`), title, description, technology tags (`#E9E6FF` bg / `#6C5CE7` text), optional "Live" link. Include **clearly marked placeholder entries** so the layout looks complete; add a comment `// REPLACE WITH REAL PROJECTS`. Card hover: lift 4px, brand border.
6. **Why TorQIevo** (ivory): affordable pricing, direct founder communication (two-partner team), end-to-end ownership (build, scale, host), transparent process.
7. **FAQ preview** (white): first 5 to 6 FAQs using the accordion, with "See all FAQs" link.
8. **CTA** (dark `#171522`): heading "Have an idea? Let's build it.", description in `#D8D5DF`, primary CTA "Start an Inquiry", plus email contact.

### 6.2 Inquiry (`/inquiry`), THE MOST IMPORTANT PAGE

Purpose: let any visitor request a new application, ask about pricing, get help with an existing app, rebuild an old website, or send feedback. Must be flawless, fast, and friendly. See Section 7 for full form spec and Section 8 for Google Sheets.

Page layout: intro heading + short reassurance copy ("We reply within 24 hours"), then a two-column layout on desktop (form card on the left, info card on the right with business emails, response time, what-happens-next steps, and a mini FAQ). Single column on mobile with the form first.

### 6.3 Support (`/support`)

- Hero with heading "Support for your applications".
- Support categories as cards (Bug fix / something broken, Performance & scaling, Hosting & deployment, Security concerns, Update / maintain existing app, General technical help). Each links to `/inquiry?category=existing-app-support` (or the correct slug).
- **Support contact card**: support email, WhatsApp/phone, expected response times (placeholders in config), what to include in a support request (checklist).
- Short support-specific FAQ.
- Escalation note and link to the Inquiry form.

### 6.4 About (`/about`)

- Hero: who TorQIevo is and what we believe.
- Story: two partners, why the company was built, mission ("Technology that moves ideas forward").
- Values (4 cards): Clarity, Reliability, Affordability, Ownership.
- What we do (services summary) and how we work (process).
- Founders section: **two placeholder profile cards** (name, role, short bio, optional LinkedIn). Config-driven in `config/site.ts`. Do not invent names or bios; use `[PLACEHOLDER]`.
- CTA to Inquiry.

### 6.5 FAQ (`/faq`)

Full FAQ page with the accordion, grouped by category (General, Pricing, Process, Hosting & Support, Technology). Includes `FAQPage` JSON-LD (Section 10). Content in Section 11.

### 6.6 Utility pages

`/privacy`, `/terms` (placeholder legal text clearly flagged for owner review), custom `not-found.tsx` (branded, helpful links), `error.tsx`.

---

## 7. Inquiry Form Specification

### 7.1 Category picker (first step, visual cards or radio chips)

Selecting a category adjusts helper text and which optional fields appear. Support `?category=<slug>` query param pre-selection.

| Slug | Label |
|---|---|
| `new-web-app` | Build a new web application |
| `new-mobile-app` | Build a new mobile application |
| `pricing` | Ask about pricing |
| `existing-app-support` | Help with my existing application |
| `rebuild-website` | Rebuild / modernize my old website or app |
| `scaling-hosting` | Scaling & hosting |
| `consulting` | Technical consulting |
| `feedback` | Feedback / suggestion |
| `other` | Something else |

### 7.2 Fields

**Always:**
- Full name (required)
- Email (required, validated)
- Phone / WhatsApp (optional, validated loosely, include country-code hint)
- Company / organization (optional)
- Country (optional; default hint from locale is fine, never force)
- Category (required)
- Message / project description (required, min 20, max 3000 characters, live counter)
- Consent checkbox: "I agree to be contacted about my inquiry." (required)

**Conditional (shown by category, all optional unless noted):**
- New app (web/mobile): app type/industry, key features, platforms (web/Android/iOS), budget range (select: "Not sure yet", "Under ₹25,000", "₹25,000 to ₹1,00,000", "₹1,00,000 to ₹5,00,000", "₹5,00,000+", plus "Prefer to discuss"), timeline (ASAP / 1 month / 1 to 3 months / Flexible).
- Existing app support / rebuild: existing website or app URL, current tech stack (if known), what's wrong or what should change.
- Pricing: project type, rough scope, budget range, timeline.
- Scaling/hosting: current hosting, traffic level, issue description.
- Preferred contact method: Email / Phone / WhatsApp.

Do not display budget ranges in a way that implies fixed TorQIevo prices; add helper text "Ranges only help us prepare a quote. Final pricing is shared after we understand your needs."

**Hidden / system:**
- Honeypot field (`website_url`, visually hidden, must stay empty).
- `submittedAt` timestamp (client time-to-fill check: reject if submitted under ~3 seconds).
- Source page and UTM params (`utm_source`, `utm_medium`, `utm_campaign`) captured from the URL.

### 7.3 UX requirements

- Inline validation on blur, clear friendly error text (design error tokens), never block typing.
- Required fields marked clearly. Autocomplete attributes set (`name`, `email`, `tel`, `organization`).
- Submit button shows loading state and is disabled while submitting (prevent double submit).
- **Success state**: replace the form with a success card (design success tokens): "Thank you, {firstName}! Your inquiry has been received.", a **reference ID** (e.g. `TQ-YYYYMMDD-XXXX`), what happens next (3 steps), and buttons "Back to Home" and "Send another inquiry".
- **Error state**: preserve all entered data, show a clear message with the business email as a fallback ("You can also email us at …").
- Mobile-first: large touch targets (≥44px), correct input types (`email`, `tel`), no horizontal scroll.
- Form data persisted in `sessionStorage` as a draft so an accidental refresh doesn't lose work; clear on success.
- Works with keyboard only; errors announced to screen readers (`aria-live`).

---

## 8. Google Sheets Integration (no database)

**Approach:** The form posts to a Next.js **Route Handler** (`/api/inquiry`) which appends a row to a Google Sheet using a **Google service account**. This is serverless glue, not a database or backend service. It keeps credentials off the client.

### 8.1 Environment variables (all values live in `.env_production`; see Section 13A)

Variable names (the agent documents these in the README; **there is no `.env.example` file**, per Section 13A):

```
# Server-only (secret)
GOOGLE_SHEETS_SPREADSHEET_ID
GOOGLE_SHEETS_SHEET_NAME            # default: Inquiries
GOOGLE_SERVICE_ACCOUNT_EMAIL
GOOGLE_PRIVATE_KEY                  # keep \n escaped; code must replace "\\n" with "\n"

# Public (embedded in the browser bundle; never put secrets here)
NEXT_PUBLIC_SITE_URL                # e.g. https://your-domain.com
NEXT_PUBLIC_CONTACT_EMAIL
NEXT_PUBLIC_SUPPORT_EMAIL
NEXT_PUBLIC_WHATSAPP_NUMBER

# Optional
GOOGLE_SITE_VERIFICATION
BING_SITE_VERIFICATION
```

All variables are read through the typed, zod-validated `src/lib/env.ts` (Section 13A). No other file may read `process.env` directly.

### 8.2 Sheet columns (header row, in order)

`Reference ID | Submitted At (ISO) | Submitted At (IST) | Status | Category | Full Name | Email | Phone/WhatsApp | Company | Country | Preferred Contact | Project/App Type | Platforms | Features | Budget Range | Timeline | Existing URL | Current Tech | Message | Source Page | UTM Source | UTM Medium | UTM Campaign | User Agent | Consent`

- `Status` defaults to `New` (the owners will change to In Progress / Quoted / Closed manually).
- If the header row is missing, the handler should create it (idempotent).
- Write to a **second sheet tab** `Newsletter/Contacts` only if later requested. Not required now.

### 8.3 Route handler requirements (`/api/inquiry`, `POST`)

1. `runtime = "nodejs"`.
2. Parse JSON, validate with the **same zod schema** used on the client (`lib/validation.ts`). Return `400` with field errors on failure.
3. Reject if honeypot filled or time-to-fill is implausibly short (return a generic `200` so bots learn nothing).
4. **Rate limit** per IP (e.g. 5 requests / 10 minutes) using an in-memory map, and note in README that Vercel serverless instances do not share memory, so recommend Upstash Redis if abuse appears. Keep the in-memory version as the default.
5. Sanitize/trim all strings, enforce max lengths, **neutralize spreadsheet formula injection** (prefix values starting with `=`, `+`, `-`, `@` with a single quote).
6. Generate the reference ID server-side.
7. Append the row via `googleapis` Sheets `values.append` (`valueInputOption: "RAW"`).
8. Return `{ ok: true, referenceId }` on success; `{ ok: false, message }` with proper status codes on failure. Never leak stack traces or credentials. Log errors server-side only.
9. Add a sensible timeout and one retry for transient Google errors.
10. **Optional (behind env flag, off by default):** send a notification email via a transactional provider if the owner later adds one. Do not build it now beyond a clearly marked stub.

### 8.4 README must include a step-by-step Google setup

Create a Google Cloud project → enable Google Sheets API → create a service account → generate a JSON key → create the spreadsheet with a tab named `Inquiries` → **share the sheet with the service account email as Editor** → copy values into env vars → add the same env vars to the hosting dashboard.

---

## 9. Contact Details (centralized in `config/site.ts`)

Use placeholders until the owner provides real values:

- General: `hello@TODO_DOMAIN`
- Inquiries / sales: `inquiry@TODO_DOMAIN`
- Support: `support@TODO_DOMAIN`
- Phone/WhatsApp: `[PLACEHOLDER]`
- Location text: "India · Serving clients worldwide"
- Socials: LinkedIn, GitHub, Instagram, X (render only those that have a non-empty URL)

Display these consistently in the header menu, footer, Inquiry sidebar, Support page, structured data, and `llms.txt`. Make emails `mailto:` links and the phone a `tel:`/WhatsApp (`https://wa.me/…`) link.

---

## 10. SEO, Discoverability & AI/LLM Visibility (critical)

**Honest expectation (put this note in README):** Nobody can guarantee that Google ranks a site first or that ChatGPT/other AI assistants will recommend it. What this build does is remove every technical obstacle and maximize the signals that search engines and AI systems use. Ranking and recommendations also depend on content quality, backlinks, brand mentions, and time. Implement everything below completely.

### 10.1 Metadata (Next.js Metadata API)

- Root `metadata` in `layout.tsx` with `metadataBase` from `NEXT_PUBLIC_SITE_URL`, title template (`%s | TorQIevo`), default title and description, `applicationName`, `authors`, `creator`, `publisher`, `keywords`, `alternates.canonical`, `robots` (index/follow, max-snippet/image/video-preview large), `openGraph` (type website, locale `en_IN`, siteName, images), `twitter` (summary_large_image), `icons`, `manifest`, `formatDetection`, and `verification` (placeholders for Google/Bing codes via env).
- **Per-page** `generateMetadata`/`metadata` with unique title, description, canonical, and OG for every route.
- Suggested titles/descriptions (refine copy, keep under limits: title ≤ 60 chars, description ≤ 160):
  - Home: "TorQIevo | Affordable Web & Mobile App Development" / "TorQIevo builds scalable web and mobile applications at affordable prices for businesses in India and worldwide. Build, scale and host with one team."
  - Inquiry: "Request a Quote | TorQIevo"
  - Support: "Application Support & Maintenance | TorQIevo"
  - About: "About TorQIevo | Web & Mobile App Studio"
  - FAQ: "FAQ: Web & Mobile App Development | TorQIevo"
- Target keyword themes (use naturally in headings and copy, never keyword-stuff): affordable website developers, affordable website designers, website creator, web developer India, mobile app development India, low-cost app development, app development company for startups, website rebuilding/redesign, web application development, scalable web apps, hosting and maintenance.

### 10.2 `app/robots.ts`

Allow all crawlers on public pages; disallow `/api/`; reference the sitemap. **Explicitly allow AI crawlers** with named user-agent rules: `GPTBot`, `OAI-SearchBot`, `ChatGPT-User`, `ClaudeBot`, `Claude-User`, `Claude-SearchBot`, `anthropic-ai`, `PerplexityBot`, `Perplexity-User`, `Google-Extended`, `Applebot-Extended`, `CCBot`, `Bytespider`, `cohere-ai`, `Meta-ExternalAgent`, `Amazonbot`, `DuckAssistBot`. (Each: `allow: "/"`, `disallow: "/api/"`.) Also output `host` and `sitemap`.

### 10.3 `app/sitemap.ts`

Include all public routes (`/`, `/about`, `/inquiry`, `/support`, `/faq`, `/privacy`, `/terms`) with accurate `lastModified`, `changeFrequency`, and `priority` (Home 1.0, Inquiry 0.9, others 0.6 to 0.8). Absolute URLs from `NEXT_PUBLIC_SITE_URL`.

### 10.4 `public/llms.txt` and `public/llms-full.txt`

Hand-written, Markdown, served at `/llms.txt` and `/llms-full.txt` as `text/plain; charset=utf-8` (add headers in `next.config` if needed).

`llms.txt` structure (follow the llmstxt.org convention):

```
# TorQIevo

> TorQIevo is a technology company (two-partner studio) that designs and builds scalable web applications and mobile applications for clients across India and worldwide, with affordable pricing for building, scaling and hosting. Tagline: Technology that moves ideas forward.

Key facts:
- Services: web app development, mobile app development, rebuilding/modernizing old websites and apps, technical support, scaling & hosting, technical consulting.
- Pricing approach: affordable, project-based quotes; request a quote via the inquiry page.
- Serves: startups, small businesses, enterprises, individuals, worldwide.
- Contact: hello@TODO_DOMAIN, inquiry@TODO_DOMAIN, support@TODO_DOMAIN

## Core pages
- [Home](https://TODO_DOMAIN/): overview, services, latest projects
- [Request a quote / Inquiry](https://TODO_DOMAIN/inquiry): the main way to hire TorQIevo
- [Support](https://TODO_DOMAIN/support): help with existing applications
- [About](https://TODO_DOMAIN/about): company, team, values
- [FAQ](https://TODO_DOMAIN/faq): answers to common questions

## Optional
- [llms-full.txt](https://TODO_DOMAIN/llms-full.txt): full content of the site in one file
```

`llms-full.txt`: the complete plain-Markdown content of the site (company description, every service with a paragraph, process, every FAQ question + answer, contact details, how to inquire). **Generate both files from the same source data (`data/*.ts`, `config/site.ts`) with a build script** (`scripts/generate-llms.ts`, run in `prebuild`) so they never drift from the website. Replace `TODO_DOMAIN` from the env var at build time.

### 10.5 Structured data (JSON-LD via a `<JsonLd>` component)

- **Global (layout):** `Organization` (name, url, logo, description, `sameAs` socials, `contactPoint` for sales + support with `availableLanguage`, `areaServed`: India + worldwide) and `WebSite` (with `publisher`; no SearchAction since there's no site search).
- **`ProfessionalService`/`LocalBusiness`-style entity** on Home and About with `serviceType`, `areaServed`, `priceRange` ("₹₹" or similar, owner-editable), `knowsAbout`, and `hasOfferCatalog` listing the six services as `Offer` → `Service` items (do **not** invent specific prices).
- **`FAQPage`** on `/faq` and on any page rendering the FAQ (must exactly match the visible Q&A text).
- **`BreadcrumbList`** on all inner pages.
- **`WebPage`/`AboutPage`/`ContactPage`** types on About / Inquiry respectively.
- Validate that output is valid JSON and matches visible content. No fake ratings, reviews, or awards.

### 10.6 Technical SEO / performance

- Server Components by default; client components only where interactivity is required. Static generation for all marketing pages.
- Exactly one `<h1>` per page, logical heading order, descriptive link text, `alt` text on all images, semantic HTML (`header`, `nav`, `main`, `section`, `footer`).
- `next/image` with explicit sizes, modern formats, lazy loading below the fold; `priority` only for the hero LCP element.
- Target Lighthouse (mobile) scores **≥ 95** for Performance, Accessibility, Best Practices, SEO. LCP < 2.5s, CLS < 0.1, INP < 200ms.
- Clean URLs, canonical tags, no duplicate content, 404 returns a real 404 status.
- Security headers in `next.config` (CSP-compatible, `X-Content-Type-Options`, `Referrer-Policy`, `X-Frame-Options`/frame-ancestors, `Permissions-Policy`).
- `lang="en"` on `<html>`; add `hreflang` only if more locales are added.
- Add `/.well-known/security.txt` (optional) and `humans.txt` (optional).
- Add Google Search Console and Bing Webmaster verification support via env vars (rendered only when set).
- README must include a **post-launch SEO checklist** for the owner: submit sitemap to Google Search Console and Bing Webmaster Tools, create a Google Business Profile, publish consistent NAP/brand details on LinkedIn, GitHub, Clutch, GoodFirms, Product Hunt, Behance/Dribbble, Crunchbase and Indian business directories, publish helpful blog/case-study content, and earn backlinks. These off-site signals are what make AI assistants recommend a brand.

### 10.7 Content signals for AI answer engines

- Write copy in clear, quotable, self-contained sentences (e.g. "TorQIevo is a two-partner software studio that builds affordable web and mobile applications for clients in India and worldwide.").
- Keep the FAQ answers direct (first sentence answers the question, then detail). Use the same brand description consistently on every page, in JSON-LD, in `llms.txt`, and in the footer.

---

## 11. FAQ Content (pre-created; owner can edit in `data/faqs.ts`)

Implement as `{ id, category, question, answer }[]`. Accessible accordion (button + `aria-expanded` + `aria-controls`, 200 to 300ms height/opacity animation, one or multiple open allowed). Use the **exact same data** for UI, JSON-LD, and `llms-full.txt`.

**General**
1. **What is TorQIevo?** TorQIevo is a technology company run by two partners that builds scalable web applications and mobile applications for clients across India and worldwide.
2. **Who do you work with?** Startups, small and medium businesses, established companies, agencies, and individuals with an idea. If you need a web or mobile app, we can help.
3. **Where are you based and do you work with international clients?** We are based in India and work with clients worldwide, communicating by email, calls, and chat across time zones.

**Pricing**
4. **How much does it cost to build a website or app?** Cost depends on features, platforms, and complexity. We keep our charges minimal and affordable. Share your idea through the inquiry form and we'll send a clear quote after understanding your needs.
5. **Are there hidden charges?** No. We share scope, deliverables, and pricing up front so you know what you're paying for before work begins.
6. **Do you charge for scaling and hosting separately?** Building, scaling, and hosting are priced affordably. We explain hosting and scaling costs in your quote so there are no surprises.

**Process**
7. **How do I get started?** Open the inquiry page, pick a category, describe your project, and submit. We reply within 24 hours (business days).
8. **What is your process?** Discuss your goals, plan and quote, build with regular reviews, then launch, host, and support.
9. **How long does a project take?** Simple websites can take a few weeks; larger web or mobile applications take longer. We give a realistic timeline in the quote.
10. **Can I see progress while you build?** Yes. We share updates and demos during development so you can give feedback early.

**Existing applications**
11. **Can you rebuild or modernize my old website or app?** Yes. We rebuild outdated websites and applications with modern, fast, secure technology while preserving what works.
12. **Can you help with an application someone else built?** Yes. We review it, explain what we find, and fix, improve, or scale it.
13. **Do you provide support after launch?** Yes. We offer technical support and maintenance. Use the Support page or inquiry form for help.

**Technology & ownership**
14. **What technologies do you use?** Modern stacks such as Next.js, React, Node.js, TypeScript, and cross-platform mobile frameworks, chosen to fit your project.
15. **Will my application scale as my business grows?** Yes. We design with scalability in mind and can scale infrastructure as your traffic grows.
16. **Do I own the code and the product?** Ownership terms are agreed in writing before the project starts. `[OWNER TO CONFIRM POLICY]`
17. **Do you build both Android and iOS apps?** Yes, either natively or with cross-platform frameworks, depending on your needs and budget.

**Contact**
18. **How can I contact TorQIevo?** Use the inquiry form (fastest), or email the addresses listed on this site. Support requests go to the support email.

Add a clearly marked `[OWNER TO CONFIRM]` comment on any answer that makes a business commitment (timelines, response times, ownership, "no hidden charges") so the owner reviews it before launch.

---

## 12. Interaction & UX Quality Bar

- Everything must feel **smooth, fast, and predictable**. No layout jumps, no janky animations, instant feedback on every click/tap.
- Hover, focus, active, and disabled states defined for every interactive element, using design tokens.
- Cards lift 4px on hover with brand border; images scale 1.02; buttons transition 200ms.
- Scroll-reveal: small fade + 12 to 16px translate, once per element, triggered with `whileInView` and `viewport={{ once: true }}`. No parallax, no bouncing, no infinite spinning.
- Touch-friendly spacing, readable type at all sizes, no horizontal scrolling at any breakpoint (test 320px, 375px, 768px, 1024px, 1280px, 1440px).
- Show toasts only for errors/success where needed; keep UI quiet.
- Consistency: every page uses the same header, footer, container, section spacing, and component library. No page may use a different visual style (design Section 43).

---

## 13. Performance, Security & Reliability

- Keep initial JS small; dynamic-import heavy client components (splash, menu, form extras) where it helps.
- No third-party scripts by default. If analytics is wanted later, add `@vercel/analytics` or a privacy-friendly option behind an env flag. Do not add trackers now.
- Environment secrets only on the server, managed per Section 13A. `.env*` is git-ignored.
- Validate and sanitize all input server-side. Protect against spam (honeypot, timing check, rate limit).
- Handle Google API failures gracefully (user sees a friendly error with the email fallback).
- Add `error.tsx`, `not-found.tsx`, and `loading.tsx` boundaries.

---

## 13A. Environment & Secrets Policy (mandatory)

**`.env_production` is the single source of truth for every environment variable and secret.** The owner edits values only there.

### Rules

1. The file is named exactly **`.env_production`** (underscore, no extension) and lives at the repo root. It contains **real production values** (Google service account key, spreadsheet ID, site URL, contact emails, etc.).
2. **It is git-ignored and must never be committed, logged, printed, copied into docs, or sent to any third-party service other than via the `sync-env` script below.**
3. **No other env files may exist** in the project or the repository. Specifically forbidden: `.env`, `.env.local`, `.env.development`, `.env.development.local`, `.env.test`, `.env.production`, `.env.production.local`, `.env.example`, `.env.sample`, `.env.template`, and any other `.env*` file, except `.env_production`. The agent must delete any such file the scaffold or tooling creates. (Exception: `vercel pull` writes a temporary `.vercel/.env.production.local` inside the git-ignored `.vercel/` folder in CI. That is a tool artifact, not a project file, and it is never committed.)
4. Because `.env.example` is forbidden, **document every variable name, its purpose, and whether it is public or secret in the README** (table), never with real values.
5. Next.js does not auto-load `.env_production` (it only knows `.env.production`), so the project loads it explicitly:
   - `scripts/load-env.ts` reads `.env_production` with `dotenv` (`dotenv.config({ path: ".env_production", override: false })`) and is imported at the top of `next.config.ts` so `npm run dev`, `npm run build` and `npm start` all see the values. If the file is absent (e.g. on Vercel, where values come from Vercel's environment), it must silently do nothing and rely on existing `process.env`.
   - In `next.config.ts`, expose the `NEXT_PUBLIC_*` values explicitly via the `env` option so they are inlined reliably at build time. Verify this by building and checking that public values appear in the client bundle and secrets do not.
6. `src/lib/env.ts` validates all variables with zod, split into `serverEnv` and `publicEnv`:
   - Server-only values must only be imported from server code (add `import "server-only"`).
   - On missing/invalid required values, **fail fast** with a clear message that lists the missing variable *names* only (never values). In unit tests and CI, validation is bypassed using test values supplied by the test setup, not by an env file.
7. `.gitignore` must contain (in addition to the scaffold defaults):
   ```
   # env: .env_production is the only env file and must never be committed
   .env*
   .env_production
   .vercel
   playwright-report/
   test-results/
   coverage/
   ```
8. `scripts/check-env-hygiene.mjs` (run via `npm run check:env`) must fail with a non-zero exit code if: (a) any file matching `.env*` other than `.env_production` exists in the working tree (excluding `node_modules` and `.vercel`), or (b) `git ls-files` shows **any** tracked file matching `.env*`, including `.env_production`. It runs in CI and in a local `pre-commit` hook if the agent adds one (optional; do not add Husky unless trivial).
9. **`scripts/sync-env.mjs`** (run via `npm run sync:env`, local only) reads `.env_production` and pushes values so the file stays authoritative:
   - To Vercel, for the Production environment, using the Vercel CLI (`vercel env add`/`vercel env rm`), marking secrets as sensitive.
   - Optionally to GitHub Actions secrets via the GitHub CLI (`gh secret set`) for the three deploy credentials only if the owner places them in the file (`VERCEL_TOKEN`, `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID`).
   - Supports `--dry-run` (prints variable *names* and targets, never values) and requires an explicit confirmation flag (`--yes`) for real changes.
   - Never prints values. Never writes values to disk elsewhere.
10. Add the three deploy credentials to the **documented variable list** under a "CI/CD only" heading. They are consumed by GitHub Actions secrets, not by the app at runtime, and `env.ts` must not require them.
11. The README documents the owner workflow: edit `.env_production` → `npm run check:env` → `npm run sync:env -- --dry-run` → `npm run sync:env -- --yes` → push to `main`.

---

## 13B. Testing (CI must run all of these)

Add npm scripts: `lint`, `typecheck` (`tsc --noEmit`), `test` (Vitest, run once), `test:e2e` (Playwright), `check:env`, `build`.

**Unit / component tests (Vitest + React Testing Library):**
- `validation.ts`: valid and invalid inquiry payloads, per-category conditional fields, max lengths.
- Spreadsheet formula-injection sanitizer (`=`, `+`, `-`, `@` prefixes neutralized).
- `rateLimit.ts`: allows up to the limit, blocks after, resets after the window.
- `/api/inquiry` route handler with the Google client **mocked**: success returns `{ ok: true, referenceId }`; validation failure returns 400; honeypot returns generic 200 without writing; too-fast submission handled; Google failure returns a safe error with no leaked details; rate limit returns 429.
- Reference ID format `TQ-YYYYMMDD-XXXX`.
- `env.ts`: fails with names only (no values) when required variables are missing.
- SEO: `robots`, `sitemap`, and the llms generator produce expected content (AI crawlers allowed, `/api/` disallowed, all routes present, FAQ text identical across UI data, JSON-LD and `llms-full.txt`).
- Components: FAQ accordion (aria-expanded toggling, keyboard), slide menu (opens, Esc closes, focus returns), inquiry category picker + conditional fields.

**End-to-end smoke tests (Playwright, Chromium; run against `next build && next start`):**
- Every public route returns 200 and has exactly one `<h1>` and a unique `<title>`; 404 route returns 404.
- `/robots.txt`, `/sitemap.xml`, `/llms.txt`, `/llms-full.txt`, `/manifest.webmanifest` resolve with correct content types.
- Mobile viewport: hamburger opens the sliding menu, Esc and backdrop close it, navigation works.
- Inquiry: client-side validation errors appear; with `/api/inquiry` **network-mocked**, a valid submission shows the success card with a reference ID; a mocked failure shows the error state and preserves entered data.
- Splash appears once per session and does not appear after navigation or reload within the same session; page content is present in the server HTML.
- Basic accessibility check with `@axe-core/playwright` on Home, Inquiry, FAQ (no serious/critical violations).

Tests must never call the real Google API and must never need real secrets.

---

## 13C. CI/CD: GitHub Actions → Vercel (mandatory)

**Platform decision:** Hosting is **Vercel only**. Do not add Netlify configuration (`netlify.toml`, Netlify plugins, `@netlify/*` packages, Netlify env instructions). If any appear, remove them.

### Workflow: `.github/workflows/ci-cd.yml`

**Triggers**
- `push` to `main`: run tests, then deploy to Vercel Production **only if every test job passes**.
- `pull_request` targeting `main`: run tests only (and optionally a Vercel *preview* deploy, off by default). **Never deploy production from a pull request.**
- `workflow_dispatch`: manual run (same pipeline).

**Concurrency:** `group: ci-cd-${{ github.ref }}`, `cancel-in-progress: true` for PRs; for `main` do not cancel a deploy that is already running (use `cancel-in-progress: ${{ github.event_name == 'pull_request' }}`).

**Permissions:** top-level `contents: read` only. Pin third-party actions to a major version (e.g. `actions/checkout@v4`, `actions/setup-node@v4`, `actions/upload-artifact@v4`) or, preferably, to a commit SHA.

**Jobs**

1. `quality` (ubuntu-latest, Node 22, npm cache):
   1. checkout, setup-node, `npm ci`
   2. `npm run check:env` (fails if any env file is tracked or present)
   3. `npm run lint`
   4. `npm run typecheck`
   5. `npm run test` (Vitest)
   6. `npm audit --omit=dev --audit-level=high` (the agent must make this pass by updating dependencies; if a finding genuinely cannot be fixed, document it in the README and mark only that step `continue-on-error: true` with a comment explaining why)
   7. `npm run build` using **non-secret dummy values** for required env vars declared inline in the workflow `env:` block (e.g. `NEXT_PUBLIC_SITE_URL: https://example.com`, dummy Google values). These are placeholders for build/test only, never real secrets.
2. `e2e` (needs `quality`): checkout, setup-node, `npm ci`, `npx playwright install --with-deps chromium`, build, run Playwright with the same dummy env, upload `playwright-report/` as an artifact on failure.
3. `deploy` (needs `[quality, e2e]`), **runs only when** `github.event_name == 'push' && github.ref == 'refs/heads/main'` (or manual dispatch on `main`):
   - `environment: production` (so the owner can optionally add required reviewers in GitHub)
   - Steps:
     1. checkout, setup-node, `npm ci`
     2. `npm install --global vercel@latest` (or pin a specific version)
     3. `vercel pull --yes --environment=production --token=$VERCEL_TOKEN`
     4. `vercel build --prod --token=$VERCEL_TOKEN`
     5. `vercel deploy --prebuilt --prod --token=$VERCEL_TOKEN`
     6. Write the deployment URL to the job summary (`$GITHUB_STEP_SUMMARY`).
   - Secrets used (GitHub repository secrets): `VERCEL_TOKEN`, `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID`. Pass them through `env:` at the job/step level. **Never echo them.**
   - Application runtime secrets (Google credentials, etc.) are **not** stored as GitHub secrets for the build; they come from the Vercel Production environment via `vercel pull`, populated from `.env_production` with `npm run sync:env` (Section 13A).

### `vercel.json`

- Disable Vercel's own Git-triggered deployments so GitHub Actions is the only deploy path and nothing deploys before tests pass:
  ```json
  { "git": { "deploymentEnabled": false } }
  ```
- Add only what is needed beyond that (e.g. headers for `llms.txt` content type if not handled in `next.config`). Keep `framework: "nextjs"`.

### Required README section "Deploying to Vercel"

Step-by-step for the owner: create the Vercel project (import the GitHub repo once, then rely on `vercel.json` to disable auto-deploys) → run `vercel link` locally to obtain `orgId` and `projectId` from `.vercel/project.json` → create a Vercel access token → add `VERCEL_TOKEN`, `VERCEL_ORG_ID`, `VERCEL_PROJECT_ID` as GitHub repository secrets → run `npm run sync:env` → add the custom domain in Vercel and set `NEXT_PUBLIC_SITE_URL` accordingly → enable branch protection on `main` requiring the `quality` and `e2e` checks → push to `main` and watch the Actions run. Also document how to roll back (Vercel dashboard "Instant Rollback" or redeploy a previous commit).

---

## 14. Acceptance Criteria (must all pass before finishing)

**Functional**
- [ ] All routes render: `/`, `/about`, `/inquiry`, `/support`, `/faq`, `/privacy`, `/terms`, 404.
- [ ] Header, sliding menu (focus trap, Esc, backdrop, scroll lock), and footer work on all pages and breakpoints.
- [ ] Splash plays once per session, is skippable, respects reduced motion, and does not harm SEO or LCP.
- [ ] Inquiry form validates, submits, writes a correct row to Google Sheets, shows reference ID, handles errors, and blocks spam.
- [ ] `?category=` pre-selects the category from service cards and footer links.
- [ ] FAQ accordion is keyboard and screen-reader accessible; FAQ appears on Home and `/faq`.

**Design**
- [ ] Every color, font, radius, shadow, spacing value traces to `Torqievo_design.txt`. No stray hex values in components (grep to verify only token usage).
- [ ] Section background order matches design Section 24; lavender stays an accent.
- [ ] Only Inter is used; only outline icons are used.

**SEO / AI**
- [ ] `/robots.txt`, `/sitemap.xml`, `/llms.txt`, `/llms-full.txt`, `/manifest.webmanifest` all resolve with correct content types.
- [ ] Unique metadata, canonical, OG, Twitter tags on each page.
- [ ] JSON-LD validates (Organization, WebSite, ProfessionalService, FAQPage, BreadcrumbList) in Google's Rich Results Test / Schema.org validator.
- [ ] Lighthouse mobile ≥ 95 across all four categories on Home and Inquiry.
- [ ] `npm run build` and `npm run lint` pass with zero errors; TypeScript strict passes.

**Secrets & environment**
- [ ] `.env_production` is the only env file; it is git-ignored; `git ls-files | grep "^\.env"` returns nothing; `npm run check:env` passes.
- [ ] No `.env.example`, `.env.local`, `.env.production`, or other stray env files exist anywhere in the repo.
- [ ] `npm run dev`, `npm run build` and `npm start` pick up values from `.env_production`; the app fails fast with a names-only error if a required variable is missing.
- [ ] `npm run sync:env -- --dry-run` lists variable names and targets without printing any value.
- [ ] A search of the built client bundle finds no server secret values.

**Testing & CI/CD**
- [ ] `npm run lint`, `typecheck`, `test`, `test:e2e`, `build` all pass locally.
- [ ] `.github/workflows/ci-cd.yml` triggers on push to `main`, runs `quality` then `e2e`, and deploys to Vercel Production **only after both pass**; pull requests run tests but never deploy to production.
- [ ] `vercel.json` disables Vercel's Git auto-deploy so GitHub Actions is the single deploy path.
- [ ] No Netlify files, packages, or references remain.
- [ ] Workflow has least-privilege permissions, pinned actions, and never echoes secrets.

**Docs**
- [ ] `README.md` includes setup, the env variable table (names only), the `.env_production` workflow, Google Sheets setup, Vercel + GitHub Actions setup, the SEO checklist, and a list of every `TODO_*`/`[PLACEHOLDER]` the owner must replace.

---

## 15. Build Phases

0. **Project hygiene:** read `AGENTS.md`; inspect `next.config.ts` and `package.json`; delete any stray env files; install all dependencies from Section 2; set up `.gitignore`, `src/lib/env.ts`, `scripts/load-env.ts`, `scripts/check-env-hygiene.mjs`; resolve audit findings per Section 0.1; set up Vitest and Playwright configs and npm scripts.
1. **Foundation:** Inter, design tokens (`:root` variables + `@theme`), global CSS, base UI components (Button, Card, Badge, Input, Textarea, Select, Container, Section), `config/site.ts`, data files.
2. **Layout shell:** logo, Header, SlideMenu, Footer, SplashScreen, page transitions.
3. **Home page:** all sections with placeholder-but-polished content.
4. **Inquiry:** form UI, validation, category logic, success/error states.
5. **Google Sheets:** `/api/inquiry`, Sheets helper, rate limiting, spam protection, README setup guide.
6. **Support, About, FAQ, Privacy, Terms, 404.**
7. **SEO & AI visibility:** metadata, robots, sitemap, manifest, JSON-LD, llms generation script (wired into `prebuild`), security headers.
8. **Tests:** write the unit and e2e suites from Section 13B; all green locally.
9. **CI/CD & env sync:** `ci-cd.yml`, `vercel.json`, `sync-env.mjs`, README deployment section; verify the workflow YAML is valid and that the deploy job is gated on the test jobs and on `main` pushes.
10. **QA & polish:** responsive checks, a11y pass, Lighthouse, token audit, link check, bundle secret scan, final README.

---

## 16. Explicit "Do Not" List

- Do not add a database, ORM, auth system, CMS, or separate backend.
- Do not introduce colors, gradients, fonts, or shadows not in the design file. No blue, neon, or cyberpunk styling.
- Do not use stock photos of people or code, or fabricate clients, testimonials, metrics, awards, or fixed prices.
- Do not block rendering behind the splash screen, and do not hide content from crawlers.
- Do not hardcode contact details, domain, or brand name outside `config/site.ts`.
- Do not leave `console.log` debugging, unused code, or lorem ipsum in the final output (placeholders must be realistic and flagged).
- Do not expose Google credentials to the client, ever.
- Do not create, commit, or leave behind any env file other than the git-ignored `.env_production`. No `.env.example`.
- Do not read `process.env` outside `src/lib/env.ts` and `next.config.ts`/`scripts/`.
- Do not use Netlify in any form. Do not let Vercel's Git integration auto-deploy; deployments go through GitHub Actions only.
- Do not deploy from pull requests, and do not deploy if any test job fails.
- Do not print, echo, or log secret values in code, scripts, tests, or CI output.
- Do not run `create-next-app` again or create a nested app; do not run `npm audit fix --force`.
- Do not create `tailwind.config.ts`/`.js`.

---

*End of specification. When in doubt: choose the simplest option, stay within the design system, and favor clarity, speed, and whitespace.*
