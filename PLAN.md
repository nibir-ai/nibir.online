# nibir.online — "Headspace for Reading" Product & Build Plan

## Context

`nibir.online` is a greenfield project (empty repo). The goal is a **mobile-first reading
webapp that mirrors Headspace's product design language and habit-building UX — applied to
reading instead of meditation**. A calm, friendly, gamified experience that turns reading into
a daily ritual: daily bite-sized reads + full books, reading streaks, daily goals, a
distraction-free reader, and a curated library users can also add their own books to.

This file is a **product design plan + technical architecture** intended to be handed to a
build pass ("ultracode"). It is detailed enough to execute screen-by-screen.

### Decisions locked with the user
- **Product:** Reading app modeled on Headspace's UX patterns & structure.
- **Content:** Curated public-domain catalog **+** user uploads.
- **Formats:** Both bite-sized daily reads **and** full books (chapters).
- **Monetization:** Headspace-style subscription paywall in the MVP (Free + Premium, Stripe).
- **Stack:** Next.js (App Router) + React + Tailwind. **Mobile-first.**
- **Backend:** Full — auth + database.
- **Deploy:** Cloudflare Pages.

> **Legal note:** We replicate Headspace's *UX patterns, IA, and habit mechanics* — NOT its
> branding, copyrighted illustrations, color assets, or content. All visual assets, copy,
> palette, and illustrations are original. Catalog content is public-domain (Project Gutenberg /
> Standard Ebooks) or user-supplied.

---

## 1. Product Vision & Principles

**One-liner:** "A calmer way to build a reading habit — a few minutes a day."

Design principles (Headspace-derived):
1. **Calm over busy** — generous whitespace, soft warm palette, gentle motion, no clutter.
2. **One clear action per screen** — "Today's read", "Continue", a single primary CTA.
3. **Habit-first** — streaks, daily goal ring, reminders, celebratory end-of-session moments.
4. **Friendly & human** — rounded shapes, organic blob illustrations, encouraging copy.
5. **Mobile-first, thumb-friendly** — bottom tab nav, large tap targets, single-column layouts.

---

## 2. Information Architecture & Navigation

**Bottom tab bar (4 tabs)** — persistent, mobile-first:

1. **Today** — home: daily read, continue reading, streak + daily goal ring, recommendations.
2. **Discover** — curated catalog: collections, categories, search, browse public-domain books.
3. **Shelf** — My books: in-progress, saved, finished, and user uploads.
4. **Profile** — stats, streak calendar, achievements, settings, subscription.

The **Reader** and **Onboarding/Auth/Paywall** are full-screen flows *outside* the tab shell.

---

## 3. Core Screens & User Flows

### A. First-run / Onboarding (Headspace-style personalization)
1. **Welcome carousel** — 3 value-prop slides with original illustrations.
2. **Personalization** (one question per screen, progress dots):
   - "What brings you here?" (build a habit / read more books / learn / unwind)
   - "Pick topics you love" (genre chips: fiction, philosophy, history, sci-fi, self-growth…)
   - "Your daily goal" (5 / 10 / 15 / 20 min)
   - "Daily reminder" (time picker, optional)
3. **Auth** — sign up / sign in (email+password and Google OAuth).
4. **Paywall** — Free vs Premium, free-trial offer (can skip → Free).
5. Land on **Today**, prefilled with picks from chosen topics.

### B. Today (Home)
- Greeting + **daily goal progress ring** (minutes read today).
- **Streak badge** (🔥 current streak) with subtle flame animation.
- **"Today's Read"** hero card — a curated bite-sized session (5–15 min).
- **Continue reading** carousel (in-progress books, % progress bars).
- **Recommended for you** rows (by chosen topics/collections).

### C. Discover (Library)
- Search bar (title/author, hits Gutendex + local catalog).
- **Collections** (curated, e.g. "Short reads for tonight", "Stoic classics", "10-min stories").
- Category chips/grid. Book cards (cover, title, author, premium lock icon if gated).

### D. Book Detail
- Cover, title, author, blurb, length/est. time, genre tags, source badge.
- Primary CTA: **Start reading / Continue**. Secondary: **Add to Shelf**.
- Chapter list (for full books). Premium lock state if gated.

### E. Reader (immersive, full-screen)
- Clean reading surface; tap center → toggle minimal chrome.
- Top: back, title, bookmark toggle. Bottom: progress (% / location), chapter nav.
- **Reading settings sheet:** font family (serif/sans), size, line spacing, margins,
  **themes** (Light / Sepia / Dark / Night), brightness.
- **Highlights & notes** (select text → highlight color + note).
- Background **session timer** feeds streak/goal/stats.
- Pagination (epub.js) with smooth page transitions; remembers location.

### F. End-of-Session (celebratory moment)
- "Nice work" screen: minutes read, **streak +1**, daily-goal ring filling, gentle confetti.
- Any milestone/achievement unlocked shown here. CTAs: "Keep reading" / "Done".

### G. Shelf (My Books)
- Segments: **In progress** / **Saved** / **Finished** / **My uploads**.
- **Upload book** button → import `.epub` (parsed, stored), appears in My uploads.

### H. Profile & Stats
- **Streak calendar/heatmap**, current & longest streak.
- Totals: minutes read, books finished, days active.
- **Achievements** grid (first book, 7-day streak, 100 minutes, genre explorer…).
- Subscription status, settings entry.

### I. Settings
- Account (email, password, sign out, delete account).
- Reading defaults (theme, font, size). Daily goal & reminder time.
- Subscription management (upgrade / manage / cancel via Stripe portal).

---

## 4. Design System

- **Palette:** warm calming pastels — primary warm amber/orange, calming teal/indigo
  secondary, cream/off-white surfaces; full **dark/night reading theme**. (All original values.)
- **Shape language:** large border-radii, pill buttons, soft layered shadows, organic
  **blob/gradient** backgrounds, custom friendly illustrations.
- **Typography:** rounded geometric sans for UI; readable serif (e.g. Literata/Lora) for
  reading body, user-overridable in the reader.
- **Motion (Framer Motion):** gentle fades/slides between screens, progress-ring fill,
  flame/streak pulse, confetti on milestones, breathing-style idle accents. Respect
  `prefers-reduced-motion`.
- **Components (shared library):** BottomTabBar, ProgressRing, StreakBadge, BookCard,
  CollectionRow, PrimaryButton/PillButton, BottomSheet, ChipSelect, StatCard,
  AchievementBadge, Paywall card, OnboardingStep, ReaderControls, Toast, EmptyState, Skeleton.
- Tailwind config: design tokens (colors, radii, spacing, fonts) centralized in `tailwind.config`.

---

## 5. Technical Architecture (Cloudflare-native)

| Concern | Choice | Notes |
|---|---|---|
| Framework | **Next.js (App Router)**, RSC + edge runtime | Mobile-first; route handlers for API |
| Deploy | **Cloudflare Pages** via `@cloudflare/next-on-pages` | Edge runtime; (OpenNext adapter as alt.) |
| Styling | **Tailwind CSS** + design tokens | |
| DB | **Cloudflare D1** (SQLite) | Edge-native, serverless |
| ORM | **Drizzle ORM** (D1 driver) | Edge-compatible, typed, migrations |
| Auth | **Better Auth** (email/password + Google OAuth), Drizzle/D1 adapter | Edge-friendly sessions |
| File storage | **Cloudflare R2** | User-uploaded EPUBs + cached covers |
| Payments | **Stripe** subscriptions + webhook route | Test mode in MVP; customer portal |
| Reader | **epub.js** (epubjs) | Renders EPUB/HTML, pagination, location |
| Catalog source | **Gutendex API** (Project Gutenberg) + Standard Ebooks | Curated import job + on-demand |
| Animation | **Framer Motion** | |
| Client state | RSC + light **Zustand** for reader UI; server actions for mutations | |
| PWA | manifest + service worker, offline reading cache | Installable, mobile-first |

**Cloudflare bindings** (`wrangler.toml`): `DB` (D1), `BOOKS` (R2), auth/Stripe/OAuth secrets
as env vars. Note edge-runtime constraints when picking libs (all above are edge-safe).

---

## 6. Data Model (Drizzle / D1)

Auth tables (Better Auth): `user`, `session`, `account`, `verification`.

App tables:
- **profile** — `user_id`, `daily_goal_minutes`, `reminder_time`, `topics` (json), `default_theme`, `default_font`.
- **book** — `id`, `title`, `author`, `source` (gutenberg|standard_ebooks|upload), `cover_url`,
  `description`, `language`, `categories` (json), `content_ref` (R2 key or external url),
  `est_minutes`, `is_premium`, `created_by` (nullable, for uploads).
- **chapter** — `id`, `book_id`, `idx`, `title`, `href/content_ref`.
- **user_book** (shelf) — `user_id`, `book_id`, `status` (in_progress|saved|finished),
  `progress_percent`, `current_location`, `added_at`, `finished_at`.
- **reading_session** — `user_id`, `book_id`, `started_at`, `ended_at`, `minutes`, `units_read`.
- **bookmark** — `user_id`, `book_id`, `location`, `created_at`.
- **highlight** — `user_id`, `book_id`, `cfi_range`, `text`, `note`, `color`.
- **streak** — `user_id`, `current_streak`, `longest_streak`, `last_read_date`, `freezes`.
- **achievement** / **user_achievement** — definitions + unlocks.
- **collection** / **collection_book** — curated groupings for Discover.
- **subscription** — `user_id`, `stripe_customer_id`, `stripe_subscription_id`, `plan`,
  `status`, `current_period_end`.

---

## 7. Gamification System
- **Daily goal:** minutes-read today vs goal → Today ring; reading sessions increment it.
- **Streak:** consecutive days hitting (any reading / goal); updated on session end; freezes for misses.
- **Achievements:** seeded definitions (first book, 7/30-day streak, 100/1000 min, finish a book,
  explore 3 genres…), evaluated on session end, surfaced on End-of-Session + Profile.
- **Stats:** aggregate from `reading_session` (minutes, books finished, active days, heatmap).

---

## 8. Build Phases (hand-off order for ultracode)

- **P0 — Scaffold:** Next.js + Tailwind + tokens; Cloudflare Pages + `next-on-pages`;
  D1 + Drizzle + migrations; `wrangler.toml`, env, lint/format; PWA manifest.
- **P1 — Design system & shell:** component library, bottom-tab shell, theme provider, motion baseline.
- **P2 — Auth & onboarding:** Better Auth (email + Google), onboarding/personalization, profile, settings.
- **P3 — Catalog & shelf:** Gutendex import + curated collections, Discover, search, Book Detail, shelf CRUD.
- **P4 — Reader:** epub.js reader, themes/fonts, location persistence, bookmarks, highlights, session timer.
- **P5 — Habit layer:** Today home, daily-goal ring, streaks, stats, achievements, End-of-Session screen.
- **P6 — Uploads:** EPUB import → R2 → parse → user book in shelf.
- **P7 — Paywall & Stripe:** Free/Premium tiers, paywall, Stripe checkout + webhook + portal, premium gating.
- **P8 — Polish:** empty/loading/skeleton states, offline reading cache, accessibility (a11y, reduced-motion),
  responsive up to desktop, final motion polish.

MVP = P0–P7; P8 is polish but should be folded in continuously.

---

## 9. Critical Files / Structure (to be created)

```
app/
  (auth)/onboarding, sign-in, sign-up, paywall
  (app)/today, discover, shelf, profile, settings   # tab shell
  reader/[bookId]                                    # immersive reader
  api/ (auth, stripe/webhook, books, sessions, upload)
components/   # design-system library (section 4)
lib/         # db (drizzle client), auth, stripe, gutendex, epub, streaks, achievements
db/          # schema.ts, migrations, seed (collections + achievements)
tailwind.config.ts  # design tokens
wrangler.toml       # D1/R2 bindings, vars
public/manifest.webmanifest, sw.js
```

---

## 10. Verification (end-to-end)

1. **Local:** `wrangler` + local D1; run migrations + seed; `npm run dev` (or `next-on-pages`
   preview) — exercise full flow: onboarding → personalization → Discover → start a curated
   book → read (themes/font, bookmark, highlight) → End-of-Session shows streak +1 & goal fill
   → Profile reflects stats/achievements → upload an EPUB → it reads from shelf.
2. **Paywall:** Stripe **test mode** — upgrade to Premium, confirm gated content unlocks,
   webhook updates `subscription`, customer portal cancel reflects status.
3. **Mobile-first checks:** verify on small viewport first (375px), thumb reach of bottom tabs,
   reader legibility in all themes, reduced-motion honored.
4. **Deploy:** Cloudflare Pages preview build green; bindings (D1/R2) and secrets configured;
   smoke-test the deployed preview on a phone.
5. Per phase: typecheck + lint pass; seed/migration runs clean.
