# Implementation Plan: Web Ninja — AI-Powered Interview Preparation Platform

**Branch**: `003-web-ninja-platform-setup` | **Date**: 2026-04-27 | **Spec**: [spec.md](./spec.md)  
**Input**: Feature specification from `/specs/003-web-ninja-platform-setup/spec.md`

---

## Summary

Build **Web Ninja** — a production-ready, scalable, AI-assisted web interview preparation platform for full-stack developers. The platform delivers structured lessons, FAANG-style flashcard questions, a streaming AI mentor ("Sensei"), mock interviews with a code editor and whiteboard, progress tracking with gamification, and a daily challenge system.

**Technical approach**: Single Next.js 16.2.4 App Router application with a feature-based folder structure. PostgreSQL (Prisma) for structured relational data, MongoDB (Mongoose) for AI chat sessions, Redis (ioredis) for caching and rate limiting. Auth.js v5 for email/password + Google OAuth. OpenAI API for Sensei. Deployed to Vercel (frontend) with managed PostgreSQL, MongoDB Atlas, and Redis Cloud.

---

## Technical Context

**Language/Version**: TypeScript 5 / Next.js 16.2.4  
**Primary Dependencies**: Next.js 16.2.4, React 19.2.4, Tailwind CSS ^4, shadcn/ui, Framer Motion ^11, TanStack Query v5, Zustand ^5, Auth.js v5, Prisma ^6, OpenAI SDK ^4 — see [research.md](./research.md) for full matrix  
**Storage**: PostgreSQL 16 (structured data via Prisma), MongoDB 7 (AI chat sessions via Mongoose), Redis 7 (caching + rate limiting via ioredis)  
**Testing**: N/A — automated testing is prohibited by constitution (Principle V)  
**Target Platform**: Web — mobile-first responsive (≥320px), tablet (≥768px), desktop (≥1280px)  
**Project Type**: Full-stack web application (single Next.js app; monorepo-ready)  
**Performance Goals**: Sensei responses ≤5s (SC-003), 500 concurrent users without degradation (SC-007), dashboard-to-lesson navigation ≤30s (SC-001), 99.5% monthly uptime (SC-011)  
**Constraints**: No offline mode, no native mobile app, no server-side code execution for mock interviews, no automated tests  
**Scale/Scope**: ~9 categories × ~50 topics × ~25 questions = ~11,250 questions at launch; 500 concurrent users target; single-region Vercel deployment

---

## Constitution Check

*GATE: Must pass before Phase 0 research. Re-check after Phase 1 design.*

- [x] **I. Clean Code**: Feature-based folder structure with single-purpose service functions, typed interfaces, and no dead code. Comments reserved for non-obvious constraints only.
- [x] **II. Simple UX**: Dashboard → Roadmap → Lesson → Questions flow operable without documentation. Sensei AI chat accessible from any page. No onboarding tooltips required.
- [x] **III. Responsive Design**: All layouts built mobile-first with Tailwind CSS `sm:` / `md:` / `lg:` prefixes. Sidebar collapses to bottom nav on mobile. Touch targets ≥44×44px.
- [x] **IV. Minimal Dependencies**: 20+ new runtime dependencies added. Every dependency is justified in the Complexity Tracking table below and in research.md. No dependency is redundant with another or replaceable by a built-in.
- [x] **V. No Testing**: Zero test files, no test framework, no test-related tasks in any task list.

**Post-Design Re-check** (after Phase 1): All entities in data-model.md trace back to spec requirements. No speculative models added. API contracts cover only what the spec mandates.

---

## Project Structure

### Documentation (this feature)

```text
specs/003-web-ninja-platform-setup/
├── plan.md              ← this file
├── research.md          ← Phase 0: technology decisions
├── data-model.md        ← Phase 1: Prisma schema + MongoDB schema
├── quickstart.md        ← Phase 1: dev setup and seed instructions
├── contracts/
│   ├── api-routes.md    ← Phase 1: HTTP Route Handler contracts
│   └── server-actions.md ← Phase 1: Next.js Server Actions
└── tasks.md             ← Phase 2 output (/speckit-tasks command)
```

### Source Code (repository root)

```text
web-ninja/
│
├── app/                                  # Next.js App Router
│   ├── (marketing)/                      # Unauthenticated public routes
│   │   ├── layout.tsx                    # Minimal marketing layout
│   │   ├── page.tsx                      # Landing page (/)
│   │   ├── login/
│   │   │   └── page.tsx
│   │   └── signup/
│   │       └── page.tsx
│   │
│   ├── (platform)/                       # Authenticated app shell
│   │   ├── layout.tsx                    # App shell: sidebar + topnav + Sensei panel slot
│   │   ├── @sensei/                      # Parallel route slot for Sensei chat panel
│   │   │   └── default.tsx
│   │   ├── dashboard/
│   │   │   └── page.tsx                  # /dashboard
│   │   ├── learn/
│   │   │   ├── page.tsx                  # /learn — roadmap overview
│   │   │   └── [category]/
│   │   │       ├── page.tsx              # /learn/[category] — category modules
│   │   │       └── [module]/
│   │   │           ├── page.tsx          # /learn/[category]/[module] — module overview
│   │   │           └── [lesson]/
│   │   │               └── page.tsx      # /learn/[category]/[module]/[lesson]
│   │   ├── questions/
│   │   │   ├── page.tsx                  # /questions — cross-topic flashcard mode
│   │   │   └── [topic]/
│   │   │       └── page.tsx              # /questions/[topic]
│   │   ├── mock-interviews/
│   │   │   ├── page.tsx                  # /mock-interviews — session picker
│   │   │   ├── coding/
│   │   │   │   ├── page.tsx              # Setup: language + difficulty
│   │   │   │   └── [sessionId]/
│   │   │   │       └── page.tsx          # Active coding session
│   │   │   ├── behavioral/
│   │   │   │   └── page.tsx
│   │   │   └── system-design/
│   │   │       └── page.tsx              # Excalidraw whiteboard
│   │   ├── system-design/
│   │   │   ├── page.tsx                  # /system-design — walkthrough library
│   │   │   └── [slug]/
│   │   │       └── page.tsx
│   │   ├── resume/
│   │   │   └── page.tsx                  # /resume — analyzer
│   │   ├── notes/
│   │   │   └── page.tsx                  # /notes — bookmarks + notes
│   │   ├── analytics/
│   │   │   └── page.tsx                  # /analytics — progress dashboard
│   │   ├── challenges/
│   │   │   └── page.tsx                  # /challenges — today's challenge + history
│   │   └── settings/
│   │       └── page.tsx                  # /settings — profile, theme, account deletion
│   │
│   ├── api/
│   │   ├── auth/
│   │   │   └── [...nextauth]/
│   │   │       └── route.ts              # Auth.js handler (signin, signout, OAuth callback)
│   │   ├── ai/
│   │   │   ├── chat/
│   │   │   │   └── route.ts              # POST — Sensei streaming chat
│   │   │   ├── quiz/
│   │   │   │   └── route.ts              # POST — AI quiz question generation
│   │   │   ├── resume/
│   │   │   │   └── route.ts              # POST — resume analysis
│   │   │   └── interview-feedback/
│   │   │       └── route.ts              # POST — mock interview feedback
│   │   ├── progress/
│   │   │   └── route.ts                  # GET + POST lesson completions
│   │   ├── questions/
│   │   │   ├── route.ts                  # GET questions by lessonId
│   │   │   └── [id]/
│   │   │       └── route.ts              # PATCH — mark question complete
│   │   ├── bookmarks/
│   │   │   ├── route.ts                  # GET + POST
│   │   │   └── [id]/
│   │   │       └── route.ts              # DELETE
│   │   ├── notes/
│   │   │   ├── route.ts                  # GET + POST
│   │   │   └── [id]/
│   │   │       └── route.ts              # PUT + DELETE
│   │   ├── challenges/
│   │   │   ├── today/
│   │   │   │   └── route.ts              # GET today's challenge
│   │   │   └── complete/
│   │   │       └── route.ts              # POST completion
│   │   ├── mock-interviews/
│   │   │   ├── route.ts                  # POST create session
│   │   │   └── [id]/
│   │   │       └── route.ts              # GET + PATCH
│   │   ├── search/
│   │   │   └── route.ts                  # GET ?q= full-text search
│   │   ├── user/
│   │   │   └── route.ts                  # PATCH profile, DELETE account
│   │   └── cron/
│   │       └── rotate-challenge/
│   │           └── route.ts              # GET — Vercel Cron daily challenge rotation
│   │
│   ├── layout.tsx                        # Root layout (ThemeProvider, QueryProvider, fonts)
│   ├── globals.css                       # Tailwind v4 base + CSS variables for dark theme
│   ├── not-found.tsx
│   └── error.tsx
│
├── src/
│   ├── components/
│   │   ├── ui/                           # shadcn/ui generated components
│   │   │   ├── button.tsx
│   │   │   ├── card.tsx
│   │   │   ├── dialog.tsx
│   │   │   ├── input.tsx
│   │   │   ├── badge.tsx
│   │   │   ├── skeleton.tsx
│   │   │   ├── progress.tsx
│   │   │   ├── tooltip.tsx
│   │   │   ├── dropdown-menu.tsx
│   │   │   ├── sheet.tsx                 # Slide-out panel (Sensei on mobile)
│   │   │   ├── command.tsx               # ⌘K command menu
│   │   │   ├── scroll-area.tsx
│   │   │   └── tabs.tsx
│   │   │
│   │   ├── layout/                       # App chrome components
│   │   │   ├── Sidebar.tsx               # Collapsible left nav with category tree
│   │   │   ├── TopNav.tsx                # Search, streak, progress, notifications, profile
│   │   │   ├── SenseiPanel.tsx           # AI chat slide-out panel (right side)
│   │   │   ├── CommandMenu.tsx           # ⌘K global search overlay
│   │   │   └── MobileNav.tsx             # Bottom navigation for mobile
│   │   │
│   │   ├── features/
│   │   │   ├── roadmap/
│   │   │   │   ├── RoadmapTimeline.tsx   # SVG/CSS timeline with module nodes
│   │   │   │   ├── CategorySection.tsx   # Grouped category with progress ring
│   │   │   │   └── ModuleCard.tsx        # Individual module card with completion badge
│   │   │   │
│   │   │   ├── lesson/
│   │   │   │   ├── LessonLayout.tsx      # Lesson page shell with section nav
│   │   │   │   ├── LessonSection.tsx     # Renders a single section by type
│   │   │   │   ├── CodeBlock.tsx         # Terminal-style block with copy button
│   │   │   │   ├── DiagramRenderer.tsx   # Mermaid client-side renderer
│   │   │   │   ├── LessonProgressBar.tsx # Sticky reading progress indicator
│   │   │   │   └── LessonComplete.tsx    # Completion overlay with badge
│   │   │   │
│   │   │   ├── flashcards/
│   │   │   │   ├── FlashcardDeck.tsx     # Deck container with progress counter
│   │   │   │   ├── FlashcardFlip.tsx     # 3D flip animation (Framer Motion)
│   │   │   │   ├── DifficultyFilter.tsx  # Easy/Medium/Hard + type filter
│   │   │   │   └── DeckComplete.tsx      # Summary screen when deck is finished
│   │   │   │
│   │   │   ├── sensei/
│   │   │   │   ├── ChatWindow.tsx        # Scrollable message list
│   │   │   │   ├── ChatMessage.tsx       # User/assistant bubble with code rendering
│   │   │   │   ├── ChatInput.tsx         # Textarea with send button + streaming indicator
│   │   │   │   └── SenseiUnavailable.tsx # Graceful fallback (FR-022)
│   │   │   │
│   │   │   ├── mock-interview/
│   │   │   │   ├── CodeEditor.tsx        # Monaco wrapper (dynamic import)
│   │   │   │   ├── TimerBar.tsx          # Countdown timer with visual urgency states
│   │   │   │   ├── FeedbackPanel.tsx     # AI feedback display after submission
│   │   │   │   ├── WhiteboardCanvas.tsx  # Excalidraw wrapper (dynamic import)
│   │   │   │   └── BehavioralPrompt.tsx  # STAR-method guidance panel
│   │   │   │
│   │   │   ├── progress/
│   │   │   │   ├── StreakBadge.tsx        # Flame icon + count
│   │   │   │   ├── ActivityHeatmap.tsx   # GitHub-style weekly grid
│   │   │   │   ├── CategoryProgress.tsx  # Progress ring per category
│   │   │   │   └── BadgeGallery.tsx      # Achievement badges display
│   │   │   │
│   │   │   ├── notes/
│   │   │   │   ├── NoteEditor.tsx        # Textarea with character count + autosave
│   │   │   │   └── BookmarkList.tsx      # Saved lessons list with search
│   │   │   │
│   │   │   └── daily-challenge/
│   │   │       ├── DailyChallengeCard.tsx # Dashboard card with challenge preview
│   │   │       └── ChallengeComplete.tsx  # Completion badge + streak preserved
│   │   │
│   │   └── shared/
│   │       ├── SkeletonCard.tsx           # Reusable skeleton placeholder
│   │       ├── EmptyState.tsx             # No results + illustration
│   │       ├── PageTransition.tsx         # Framer Motion page wrapper
│   │       ├── DifficultyBadge.tsx        # Easy/Medium/Hard color-coded badge
│   │       └── GlassCard.tsx             # Glassmorphism card wrapper
│   │
│   ├── lib/
│   │   ├── db/
│   │   │   ├── prisma.ts                  # Prisma client singleton (dev: global, prod: new)
│   │   │   └── mongodb.ts                 # Mongoose connection singleton
│   │   ├── auth/
│   │   │   ├── config.ts                  # Auth.js config (providers, Prisma adapter, callbacks)
│   │   │   └── session.ts                 # Server-side session helpers (auth(), getUser())
│   │   ├── ai/
│   │   │   ├── client.ts                  # OpenAI client singleton
│   │   │   ├── sensei.ts                  # Streaming chat logic
│   │   │   ├── quiz.ts                    # Quiz generation logic
│   │   │   ├── resume.ts                  # Resume analysis logic
│   │   │   └── prompts.ts                 # System prompt templates
│   │   ├── cache/
│   │   │   ├── redis.ts                   # ioredis client singleton
│   │   │   └── rate-limit.ts              # Sliding window rate limiter
│   │   ├── search/
│   │   │   └── full-text.ts               # PostgreSQL FTS query builders
│   │   └── utils/
│   │       ├── cn.ts                      # tailwind-merge + clsx helper
│   │       ├── dates.ts                   # date-fns wrappers (streak calc, heatmap)
│   │       └── streak.ts                  # Streak update logic (reset, increment)
│   │
│   ├── hooks/
│   │   ├── useProgress.ts                 # TanStack Query: lesson + question progress
│   │   ├── useStreak.ts                   # TanStack Query: streak data
│   │   ├── useSenseiChat.ts               # Streaming chat via fetch + ReadableStream
│   │   ├── useBookmarks.ts                # TanStack Query: bookmarks CRUD
│   │   ├── useNotes.ts                    # TanStack Query: notes CRUD
│   │   ├── useCommandMenu.ts              # Keyboard shortcut (⌘K) + search state
│   │   └── useTheme.ts                    # Dark/light mode toggle (persisted)
│   │
│   ├── stores/
│   │   ├── ui.store.ts                    # sidebar open/closed, theme, Sensei panel
│   │   ├── sensei.store.ts                # chat messages buffer, streaming state
│   │   └── progress.store.ts              # optimistic lesson/question completions
│   │
│   ├── services/                          # Server-side data access (used in Server Components and Route Handlers)
│   │   ├── lesson.service.ts              # Lesson + section fetching
│   │   ├── question.service.ts            # Question fetching + completion writes
│   │   ├── progress.service.ts            # UserProgress + streak writes
│   │   ├── challenge.service.ts           # DailyChallenge fetch + completion
│   │   ├── note.service.ts                # Note CRUD
│   │   ├── bookmark.service.ts            # Bookmark CRUD
│   │   ├── mock-interview.service.ts      # Session creation + feedback storage
│   │   ├── search.service.ts              # FTS query execution
│   │   └── user.service.ts               # Profile updates + account deletion
│   │
│   └── types/
│       ├── index.ts                       # Re-exports all types
│       ├── auth.ts                        # Session, User types
│       ├── content.ts                     # Category, Module, Lesson, Section types
│       ├── progress.ts                    # UserProgress, Streak, Badge types
│       └── ai.ts                          # ChatMessage, SenseiRequest, FeedbackResponse types
│
├── prisma/
│   ├── schema.prisma                      # All PostgreSQL models
│   └── seed.ts                            # Content seed: categories, modules, lessons, questions
│
├── public/
│   └── images/
│       └── og-image.png
│
├── .env.example                           # All required environment variables documented
├── components.json                        # shadcn/ui configuration
├── next.config.ts
├── tailwind.config.ts                     # CSS variable theme tokens for dark mode
├── pnpm-workspace.yaml
└── tsconfig.json
```

**Structure Decision**: Single Next.js app with route groups — `(marketing)` for public pages, `(platform)` for the authenticated shell. All shared code lives in `src/` co-located by concern (components, lib, hooks, stores, services, types). Prisma schema at root-level `prisma/`. The `pnpm-workspace.yaml` is pre-configured for future monorepo package extraction when the backend outgrows Next.js Route Handlers.

---

## Architecture Decisions

### Authentication & Authorization

Auth.js v5 (`next-auth@5`) with Prisma adapter. Email/password credentials + Google OAuth. Sessions stored as JWT in `httpOnly` cookies; Prisma session table for server-side session validation.

**Route Protection**:
- Next.js Middleware (`middleware.ts`) intercepts all `/(platform)/**` routes, calls `auth()`, and redirects unauthenticated users to `/login`.
- Public preview: first lesson per category (marked `isPreview: true` in DB) is accessible without auth via the `(marketing)` route group or a conditional in the platform layout.
- Admin routes (cron, seed) protected by `CRON_SECRET` / `ADMIN_SECRET` headers.

**RBAC**: Two roles — `USER` (default) and `ADMIN` (content management, challenge curation). Role checked in Route Handlers via `session.user.role`. No RBAC middleware library needed at MVP scale.

### API Architecture

All API logic in Next.js Route Handlers (`app/api/**`). Pattern:
1. Route Handler validates request (Zod schema).
2. Calls a service function from `src/services/`.
3. Service function calls Prisma/Mongoose directly.
4. Returns `Response.json()` or streaming `Response` for AI.

Server Actions are used for form submissions (signup, login, profile update, note save) — they run on the server and return `ActionState` objects consumed by `useActionState` in Client Components.

### AI Architecture (Sensei)

```
Client (useSenseiChat hook)
  → POST /api/ai/chat
  → Rate limit check (Redis, 20 req/min per user)
  → Build messages array with system prompt + lesson context
  → openai.chat.completions.stream()
  → Return ReadableStream (Server-Sent Events)
  → Hook reads stream and appends tokens to Zustand sensei.store
  → ChatWindow re-renders incrementally
```

System prompts in `src/lib/ai/prompts.ts` inject:
- Platform identity ("You are Sensei, Web Ninja's AI mentor...")
- Current lesson context (title, category, topic)
- User's current progress level

### State Management Strategy

| State Domain | Solution | Location |
|---|---|---|
| UI chrome (sidebar, Sensei open) | Zustand `ui.store` | Client Components |
| Sensei chat messages | Zustand `sensei.store` | Client Components |
| Optimistic progress | Zustand `progress.store` | Client Components |
| Server data (lessons, questions) | Server Components + fetch | Server |
| Client-mutated server data | TanStack Query | Client Components |
| Theme preference | Zustand `ui.store` + `localStorage` | Client Components |
| Auth session | Auth.js cookies | Server + Client |

### Dark Mode Implementation

Default dark mode via Tailwind v4 CSS custom properties. Root layout adds `dark` class to `<html>` by default. `useTheme` hook toggles the class and persists to `localStorage`. Tailwind v4 uses `@theme` directive with CSS variables — no `tailwind.config.js` `darkMode` needed.

```css
/* app/globals.css */
@import "tailwindcss";

@theme {
  --color-background: oklch(9% 0.01 240);
  --color-foreground: oklch(95% 0.01 240);
  --color-primary: oklch(65% 0.25 260); /* Purple-blue developer aesthetic */
  --color-accent: oklch(70% 0.2 185);   /* Cyan accent */
  /* ... */
}
```

### Performance Architecture

| Concern | Solution |
|---|---|
| Lesson pages | Server Components + `use cache` directive |
| Roadmap initial load | Server Component, static prerender |
| Flashcard deck | Client Component, lazy-loaded |
| Code editor (Monaco) | `dynamic(() => import(...), { ssr: false })` |
| Whiteboard (Excalidraw) | `dynamic(() => import(...), { ssr: false })` |
| Mermaid diagrams | `dynamic(() => import(...), { ssr: false })` |
| Images | `next/image` with WebP |
| Fonts | `next/font/google` (no FOUT) |
| Questions list (>25 items) | Virtual scroll via CSS `contain: content` |
| AI chat history | Virtualized scroll (Zustand slice with max 50 messages visible) |
| Search | Debounced input (300ms) + Redis cache |
| `unstable_instant` | Export from roadmap, lesson, and questions routes for instant client-side navigation |

### SEO

- `generateMetadata()` on every page — title, description, OG image.
- Structured data (`JSON-LD`) on lesson pages.
- `sitemap.ts` generated from DB for all public lesson slugs.
- `robots.ts` allows all public pages; blocks `/api/**`.

### Security

| Concern | Mechanism |
|---|---|
| Authentication | Auth.js JWT in httpOnly cookie |
| Authorization | Middleware + service-layer user ID assertion |
| Rate limiting | Redis sliding window on `/api/ai/**` (20/min user) and `/api/auth/signup` (5/min IP) |
| Input validation | Zod schemas on every Route Handler and Server Action |
| XSS | `react-markdown` safe rendering; no `dangerouslySetInnerHTML` |
| CSRF | Next.js built-in CSRF protection on Server Actions |
| Secure headers | `next.config.ts` `headers()` — CSP, HSTS, X-Frame-Options |
| Account deletion | Cascading Prisma deletes on User + MongoDB chat purge + Redis key eviction (FR-047) |

### Deployment Strategy

```
Vercel (Production)
  ├── Next.js app (auto-scaled serverless + edge functions)
  ├── Vercel Cron Job → /api/cron/rotate-challenge (daily at 00:00 UTC)
  └── Environment: production / preview (PR previews) / development

External Services
  ├── Neon PostgreSQL (serverless PostgreSQL, free tier to start)
  ├── MongoDB Atlas (M0 free tier for chat sessions)
  └── Upstash Redis (serverless Redis, free tier)
```

**CI/CD**: GitHub Actions workflow:
1. On PR: lint (`eslint`) + type-check (`tsc --noEmit`) + Vercel preview deployment.
2. On merge to `main`: Vercel production deployment + Prisma migration run.

**Environments**:
- `.env.local` — local development
- `.env.development` — Vercel preview
- `.env.production` — Vercel production

---

## Complexity Tracking

> **Violations of Constitution Principle IV — recorded here per governance requirements**

| Dependency | Why Needed | Simpler Alternative Rejected Because |
|---|---|---|
| `shadcn/ui` | Accessible, styled component primitives (Button, Card, Dialog, Sheet, Tabs, Badge, Skeleton, Command, ScrollArea, Progress, Tooltip, DropdownMenu) | Building 12+ accessible components from scratch takes weeks; diverts from core platform value |
| `framer-motion` | Page transitions (FR-044), flashcard 3D flip animation (FR-014), streak badge micro-animations, roadmap timeline entrance animations | CSS transitions cannot produce 3D card flip; `animate` API is the only practical solution |
| `@tanstack/react-query` | Client-driven CRUD (bookmarks, notes, question completions) with optimistic updates, cache invalidation, and background refetch | Next.js fetch caching is for Server Components; client mutations via `useEffect` + manual state reimplements caching poorly |
| `zustand` | Cross-component client state: sidebar open, Sensei panel, theme, streaming chat buffer | React Context triggers full tree re-renders for high-frequency streaming state; Redux Toolkit adds boilerplate with no benefit for 3 small stores |
| `react-hook-form` | Signup, login, profile, settings, note forms with real-time client validation | `useActionState` + Server Actions handle submission but provide no synchronous field-level validation UX |
| `zod` | Runtime type validation on all Route Handlers and Server Actions | No built-in validation primitive in Next.js; manual checks are error-prone and verbose |
| `date-fns` | Streak calculation (consecutive day detection), activity heatmap date grid generation, challenge date comparison | JavaScript `Date` arithmetic is timezone-unreliable; `Temporal` API is not yet baseline |
| `next-auth` (Auth.js v5) | Email/password + Google OAuth with Prisma session adapter | Building JWT issuance, cookie management, OAuth PKCE flow, and session refresh from scratch is a security liability |
| `prisma` + `@prisma/client` | Type-safe ORM with migration management for PostgreSQL | Raw SQL requires manual migration files and loses TypeScript inference on query results |
| `mongoose` | MongoDB ODM for `ChatSession` flexible document schema with TypeScript types | Using MongoDB Node.js driver directly requires manual schema definition and type assertions on every read |
| `ioredis` | Redis client for rate limiting, session cache, search cache, streak cache | No built-in Redis client in Node.js; `redis` (official) is valid alternative — `ioredis` chosen for cluster/pipeline support and TypeScript quality |
| `openai` | OpenAI API client with streaming support for Sensei chat | No built-in AI capability in Next.js; `fetch` directly is possible but the SDK handles token streaming, retry logic, and type safety |
| `@monaco-editor/react` | In-browser code editor for mock interviews (FR-024) with JS/TS/Python syntax | No native HTML element provides a code editor; `<textarea>` is insufficient for developer experience |
| `react-markdown` + `remark-gfm` + `rehype-highlight` | Render lesson content stored as Markdown strings from the DB (FR-009) | Next.js MDX is for static build-time content; dynamic DB-driven Markdown requires runtime rendering |
| `mermaid` | Client-side architecture and flow diagram rendering from text syntax (FR-043) | Pre-rendered SVG would make content updates expensive; server-side rendering adds infra complexity |
| `@excalidraw/excalidraw` | Interactive whiteboard for system design mock interviews (FR-038) | Building a canvas-based whiteboard from scratch is a multi-week effort outside MVP scope |
| `lucide-react` | Icon set (required by shadcn/ui, used throughout UI) | Inline SVG for 50+ icons is unmaintainable; font icons are accessibility-poor |
| `clsx` + `tailwind-merge` + `class-variance-authority` | CSS class composition utilities required by the shadcn/ui component pattern | Without these, conditional Tailwind class merging produces unpredictable CSS specificity conflicts |
| `node-cron` | Daily challenge rotation via Vercel Cron Job trigger | No built-in Next.js scheduler; the task is too simple to warrant a full queue service |
