# Tasks: Web Ninja — AI-Powered Interview Preparation Platform

**Input**: Design documents from `/specs/003-web-ninja-platform-setup/`
**Prerequisites**: plan.md ✅, spec.md ✅, research.md ✅, data-model.md ✅, contracts/ ✅

**Tests**: Automated testing is PROHIBITED by constitution (Principle V — No Testing, NON-NEGOTIABLE). Zero test files, no test framework, no test tasks.

**Organization**: Tasks are grouped by user story to enable independent implementation and testing of each story.

## Format: `[ID] [P?] [Story?] Description`

- **[P]**: Can run in parallel (different files, no dependencies on incomplete tasks in phase)
- **[Story]**: Which user story this task belongs to (US1–US7)
- File paths included in all descriptions

---

## Phase 1: Setup (Shared Infrastructure)

**Purpose**: Project initialization, tooling configuration, and base dependencies

- [ ] T001 Initialize Next.js 16.2.4 project with TypeScript at repository root via `pnpm create next-app`
- [ ] T002 [P] Create `pnpm-workspace.yaml` for monorepo-ready setup at project root
- [ ] T003 [P] Create `docker-compose.yml` with PostgreSQL 16, MongoDB 7, and Redis 7-alpine services at project root
- [ ] T004 [P] Create `.env.example` documenting all required environment variables: DATABASE_URL, MONGODB_URI, REDIS_URL, AUTH_SECRET, AUTH_URL, AUTH_GOOGLE_ID, AUTH_GOOGLE_SECRET, OPENAI_API_KEY, OPENAI_MODEL_FAST, OPENAI_MODEL_SMART, CRON_SECRET, NEXT_PUBLIC_APP_URL, NEXT_PUBLIC_APP_NAME
- [ ] T005 Install all runtime dependencies via pnpm: `next-auth@beta @auth/prisma-adapter prisma @prisma/client mongoose ioredis openai @tanstack/react-query zustand react-hook-form @hookform/resolvers zod framer-motion date-fns react-markdown remark-gfm rehype-highlight mermaid node-cron bcryptjs lucide-react clsx tailwind-merge class-variance-authority`
- [ ] T006 [P] Install Monaco Editor and Excalidraw dependencies: `@monaco-editor/react @excalidraw/excalidraw`
- [ ] T007 Initialize shadcn/ui via `pnpm dlx shadcn@latest init` with New York style and Slate base color, generating `components.json` at root
- [ ] T008 Add shadcn/ui components via CLI into `src/components/ui/`: button, card, dialog, input, badge, skeleton, progress, tooltip, dropdown-menu, sheet, command, scroll-area, tabs, separator, avatar
- [ ] T009 [P] Configure `next.config.ts` with security headers (Content-Security-Policy, HSTS, X-Frame-Options, X-Content-Type-Options), image domains, and TypeScript strict mode
- [ ] T010 [P] Configure `tsconfig.json` with strict TypeScript settings and `@/*` path alias resolving to `src/`

**Checkpoint**: Project scaffold ready — dependencies installed, tooling configured

---

## Phase 2: Foundational (Blocking Prerequisites)

**Purpose**: Core infrastructure that MUST be complete before ANY user story can be implemented

**⚠️ CRITICAL**: No user story work can begin until this phase is complete

- [ ] T011 Create Prisma schema in `prisma/schema.prisma` with all models: User (with Role, Theme enums), Account, Session, VerificationToken, Category, Module, Lesson, LessonSection (SectionType enum), InterviewQuestion (QuestionType enum), UserProgress, QuestionCompletion, StreakRecord, Badge (BadgeCondition enum), UserBadge, Bookmark, Note, DailyChallenge (ChallengeType enum), DailyChallengeCompletion, MockInterview (MockInterviewType enum), SystemDesignGuide, CompanyGuide; Difficulty enum; all `tsvector` columns as `Unsupported("tsvector")?`
- [ ] T012 Run `pnpm prisma migrate dev --name init`, then create FTS trigger SQL in `prisma/migrations/fts-triggers.sql` for Lesson, InterviewQuestion, and Note search vectors (GIN indexes), and apply via `pnpm prisma db execute --file prisma/migrations/fts-triggers.sql`
- [ ] T013 Create database seed script in `prisma/seed.ts` seeding: 9 categories, ~54 modules, ~216 lessons with skeleton content, 3–5 sample questions per lesson (all 3 types + all 3 difficulties), 15 badges (LESSONS_COMPLETED, STREAK_DAYS, QUESTIONS_ANSWERED, CATEGORIES_COMPLETED, CHALLENGES_COMPLETED conditions), 90 daily challenges (CODING/CONCEPT/SYSTEM_DESIGN mix), 6 company guides (Google, Meta, Amazon, Netflix, Apple, Microsoft), 10 system design walkthroughs
- [ ] T014 [P] Create Prisma client singleton in `src/lib/db/prisma.ts` (dev: global instance to prevent connection exhaustion in HMR, prod: new PrismaClient instance)
- [ ] T015 [P] Create Mongoose ChatSession schema and model in `src/lib/db/schemas/chat-session.schema.ts` with nested ChatMessage sub-schema (role, content, timestamp, codeBlocks, tokenCount), compound index on `{ userId, lessonContext, isActive }`
- [ ] T016 [P] Create MongoDB connection singleton in `src/lib/db/mongodb.ts` (cached connection for dev/prod)
- [ ] T017 [P] Create ioredis client singleton in `src/lib/cache/redis.ts` using REDIS_URL environment variable
- [ ] T018 [P] Create sliding window rate limiter in `src/lib/cache/rate-limit.ts` supporting per-user (`rate:ai:{userId}`) and per-IP (`rate:signup:{ip}`) counters with 60s TTL
- [ ] T019 Create Auth.js v5 configuration in `src/lib/auth/config.ts` with PrismaAdapter, CredentialsProvider (email/password + bcrypt validation), Google OAuth provider, JWT strategy, session callbacks, and RBAC role propagation to session token
- [ ] T020 [P] Create server-side session helpers in `src/lib/auth/session.ts` — export `auth()` wrapper and `getUser()` helper that throws if unauthenticated
- [ ] T021 Create Auth.js route handler in `app/api/auth/[...nextauth]/route.ts` exporting GET and POST handlers from Auth.js config
- [ ] T022 Create Next.js middleware in `middleware.ts` — intercepts all `/(platform)/**` routes, calls `auth()`, redirects unauthenticated users to `/login`; allows public `/api/auth/**` and cron routes
- [ ] T023 [P] Create TypeScript type definitions: `src/types/auth.ts` (Session, User, Role), `src/types/content.ts` (Category, Module, Lesson, LessonSection, SectionType), `src/types/progress.ts` (UserProgress, Streak, Badge, WeeklyActivity), `src/types/ai.ts` (ChatMessage, SenseiRequest, FeedbackResponse, QuizQuestion), `src/types/index.ts` (re-exports all)
- [ ] T024 [P] Create utility functions: `src/lib/utils/cn.ts` (tailwind-merge + clsx `cn()` helper), `src/lib/utils/dates.ts` (date-fns wrappers: formatStreakDate, generateHeatmapGrid, isConsecutiveDay), `src/lib/utils/streak.ts` (incrementStreak, resetStreak, shouldResetStreak logic)
- [ ] T025 [P] Create Zustand stores: `src/stores/ui.store.ts` (sidebar open/closed, theme DARK/LIGHT, Sensei panel open), `src/stores/sensei.store.ts` (messages array max 50, streaming state, sessionId), `src/stores/progress.store.ts` (optimistic lesson/question completions map)
- [ ] T026 Create TanStack Query provider client component in `src/components/providers/QueryProvider.tsx` wrapping `QueryClientProvider` with default staleTime/gcTime config
- [ ] T027 Create root layout in `app/layout.tsx` with QueryProvider, ThemeProvider, `next/font/google` (Inter + JetBrains Mono), `<html>` default `dark` class, global metadata
- [ ] T028 [P] Create `app/globals.css` with Tailwind v4 `@import "tailwindcss"`, `@theme` directive defining oklch CSS variables: --color-background, --color-foreground, --color-primary (purple-blue), --color-accent (cyan), light/dark token overrides, and base font CSS
- [ ] T029 [P] Create `app/error.tsx` global error boundary component and `app/not-found.tsx` 404 page with navigation link
- [ ] T030 Create signUp, signIn, and signOut server actions in `src/app/actions/auth.ts` with Zod validation: signUp hashes password with bcryptjs (12 rounds), creates User + StreakRecord, signs in; signIn delegates to Auth.js credentials; signOut calls Auth.js signOut
- [ ] T031 Create sign-up page with react-hook-form + Zod in `app/(marketing)/signup/page.tsx` — fields: name, email, password, confirmPassword; calls signUp action via useActionState
- [ ] T032 Create sign-in page with react-hook-form + Zod in `app/(marketing)/login/page.tsx` — email, password fields; Google OAuth button; calls signIn action
- [ ] T033 [P] Create marketing layout in `app/(marketing)/layout.tsx` — minimal unauthenticated shell with a simple header nav (logo + login/signup links)

**Checkpoint**: Foundation ready — DB schema migrated, auth wired, utilities available — user story implementation can now begin

---

## Phase 3: User Story 1 — Onboard and Explore the Learning Roadmap (Priority: P1) 🎯 MVP

**Goal**: User can sign up, land on the dashboard, browse the visual roadmap, open a module, read a full lesson (explanation, code snippets, diagrams, interview tips), and mark it complete.

**Independent Test**: Navigate to `/dashboard`, open the roadmap at `/learn`, click a module, verify a lesson page loads with all section types (explanation, code block with copy, Mermaid diagram, interview tips, completion overlay). Mark lesson complete and verify roadmap badge updates.

### Implementation for User Story 1

- [ ] T034 Create `src/services/lesson.service.ts` with functions: `getAllCategories()`, `getCategoryWithModules(slug)`, `getModuleWithLessons(slug)`, `getLessonBySlug(slug)`, `getLessonSections(lessonId)`, `isLessonPreview(lessonId)`
- [ ] T035 [P] [US1] Create `src/components/features/roadmap/RoadmapTimeline.tsx` — SVG/CSS timeline rendering all categories with visual connectors between modules, progress ring per category
- [ ] T036 [P] [US1] Create `src/components/features/roadmap/CategorySection.tsx` — grouped category container with category icon, progress ring (shadcn Progress), and collapsed/expanded module list
- [ ] T037 [P] [US1] Create `src/components/features/roadmap/ModuleCard.tsx` — individual module card with title, difficulty badge, lesson count, completion badge (Framer Motion entrance animation), and link to module overview
- [ ] T038 [US1] Create learn roadmap overview page at `app/(platform)/learn/page.tsx` — Server Component using `getAllCategories()`, renders RoadmapTimeline with all CategorySections; `export const unstable_instant = true` for instant navigation
- [ ] T039 [US1] Create category modules page at `app/(platform)/learn/[category]/page.tsx` — Server Component using `getCategoryWithModules(slug)`, renders ModuleCards for the category with progress overlay
- [ ] T040 [US1] Create module overview page at `app/(platform)/learn/[category]/[module]/page.tsx` — Server Component using `getModuleWithLessons(slug)`, renders ordered lesson list with difficulty badges and completion indicators
- [ ] T041 [P] [US1] Create `src/components/features/lesson/LessonLayout.tsx` — lesson page shell with sticky section nav (list of section types), keyboard navigation support, and lesson metadata header
- [ ] T042 [P] [US1] Create `src/components/features/lesson/LessonSection.tsx` — renders a single LessonSection by SectionType: EXPLANATION/INTERVIEW_TIPS → react-markdown; CODE_SNIPPET → CodeBlock; DIAGRAM → DiagramRenderer; COMMON_MISTAKES/PERFORMANCE_TIPS/HOW_INTERVIEWERS_ASK → formatted list
- [ ] T043 [P] [US1] Create `src/components/features/lesson/CodeBlock.tsx` — terminal-style code block with syntax highlighting (rehype-highlight themes), copy-to-clipboard button with success feedback, language label badge (FR-042)
- [ ] T044 [P] [US1] Create `src/components/features/lesson/DiagramRenderer.tsx` — Client Component with `dynamic(() => import('mermaid'), { ssr: false })`, renders Mermaid source string to SVG on mount (FR-043)
- [ ] T045 [P] [US1] Create `src/components/features/lesson/LessonProgressBar.tsx` — sticky reading progress indicator using IntersectionObserver on section anchors (FR-010)
- [ ] T046 [P] [US1] Create `src/components/features/lesson/LessonComplete.tsx` — completion overlay with badge icon, Framer Motion celebration animation, "Mark Complete" button that calls completeLesson action
- [ ] T047 [P] [US1] Create `src/components/shared/SkeletonCard.tsx` — reusable shadcn Skeleton placeholder card (FR-046)
- [ ] T048 [P] [US1] Create `src/components/shared/EmptyState.tsx` — no results component with SVG illustration and optional CTA button
- [ ] T049 [P] [US1] Create `src/components/shared/GlassCard.tsx` — glassmorphism card wrapper with backdrop-blur, border-opacity styling
- [ ] T050 [P] [US1] Create `src/components/shared/PageTransition.tsx` — Framer Motion `AnimatePresence` + `motion.div` page wrapper for smooth route transitions (FR-044)
- [ ] T051 [P] [US1] Create `src/components/shared/DifficultyBadge.tsx` — color-coded BEGINNER/INTERMEDIATE/ADVANCED badge using shadcn Badge with variant mapping
- [ ] T052 Create completeLesson server action in `src/app/actions/lesson.ts` — upserts `UserProgress`, calls `progress.service.updateStreak()`, calls `progress.service.evaluateBadges()`, invalidates Redis `streak:{userId}` and `progress:{userId}:summary`, revalidates `/dashboard` and `/learn` paths
- [ ] T053 Create `src/services/progress.service.ts` with functions: `getUserProgress(userId)`, `updateStreakRecord(userId)` (increment if same/consecutive day, reset if gap), `evaluateBadgeConditions(userId)` (check thresholds, insert UserBadge), `getCategoryProgressSummary(userId)`, `getWeeklyActivity(userId)`
- [ ] T054 [US1] Create lesson viewer page at `app/(platform)/learn/[category]/[module]/[lesson]/page.tsx` — Server Component using `getLessonBySlug()` + `getLessonSections()`; renders LessonLayout with LessonSection list, LessonProgressBar, LessonComplete; `generateMetadata()` for title/description/OG; JSON-LD structured data; `export const unstable_instant = true`
- [ ] T055 [US1] Create `src/components/layout/Sidebar.tsx` — collapsible left nav with category tree (accordion), module links, active route highlighting, collapses to bottom sheet on mobile (FR-004); reads categories from `getAllCategories()` or store
- [ ] T056 [US1] Create `src/components/layout/TopNav.tsx` — search input (CommandMenu trigger), StreakBadge slot, overall progress indicator, notifications bell, user profile dropdown (sign out link), Sensei AI toggle button (FR-005)
- [ ] T057 [P] [US1] Create `src/components/layout/MobileNav.tsx` — fixed bottom navigation bar with icons for Dashboard, Learn, Questions, Challenges, Notes (FR-003)
- [ ] T058 [P] [US1] Create `app/(platform)/@sensei/default.tsx` — default parallel route slot (empty fragment) for Sensei panel
- [ ] T059 [US1] Create platform layout at `app/(platform)/layout.tsx` — authenticated app shell: verifies session, renders Sidebar, TopNav, MobileNav, PageTransition wrapper, `{children}` + `{sensei}` parallel slot
- [ ] T060 [US1] Create dashboard page at `app/(platform)/dashboard/page.tsx` — Server Component: renders RoadmapTimeline overview, StreakBadge (placeholder data), CategoryProgress cards, DailyChallengeCard slot; `generateMetadata()`
- [ ] T061 [US1] Create public landing page at `app/(marketing)/page.tsx` — value proposition hero, featured modules grid, testimonial section, sign-up and login CTAs; mobile-first responsive layout
- [ ] T062 [P] [US1] Create `app/sitemap.ts` generating XML sitemap from all Lesson slugs fetched via Prisma and `app/robots.ts` blocking `/api/**`

**Checkpoint**: US1 fully functional — user can sign up, browse roadmap, read full lessons, mark complete, see dashboard

---

## Phase 4: User Story 2 — Practice Interview Questions with Flashcards (Priority: P2)

**Goal**: User can open a lesson's interview questions as flashcards, flip to reveal answers with 3D animation, filter by difficulty/type, mark questions as done, and complete a full deck.

**Independent Test**: Navigate to `/questions/[topic]`, flip through cards, apply "Hard" difficulty filter — verify only Hard cards show; mark a card complete, reload page — verify completion persists; complete all cards — verify DeckComplete summary screen.

### Implementation for User Story 2

- [ ] T063 Create `src/services/question.service.ts` with functions: `getQuestionsByLesson(lessonId, filters)`, `getCompletedQuestionIds(userId, lessonId)`, `toggleQuestionCompletion(userId, questionId, completed)`, `getRandomQuestionsAcrossTopics(userId, filters, limit)`
- [ ] T064 [P] [US2] Create `app/api/questions/route.ts` — GET handler: validates `lessonId` + optional `difficulty` and `type` query params with Zod, joins with QuestionCompletion for `completed` field, returns questions array
- [ ] T065 [P] [US2] Create `app/api/questions/[id]/route.ts` — PATCH handler: validates `{ completed: boolean }` body, upserts or deletes QuestionCompletion record, returns `{ id, completed }`
- [ ] T066 [P] [US2] Create `src/components/features/flashcards/FlashcardFlip.tsx` — Client Component with Framer Motion 3D `rotateY` flip animation (perspective transform), shows question on front face, answer (react-markdown rendered) on back face (FR-014)
- [ ] T067 [P] [US2] Create `src/components/features/flashcards/DifficultyFilter.tsx` — filter buttons for BEGINNER/INTERMEDIATE/ADVANCED difficulty and FAANG/SCENARIO/CONCEPTUAL type, active state styling, clears filter option (FR-016)
- [ ] T068 [P] [US2] Create `src/components/features/flashcards/DeckComplete.tsx` — summary screen showing completion count/total, options: "Retry All (Shuffled)", "Filter Incomplete", "Next Lesson" link (edge case handler)
- [ ] T069 [US2] Create `src/components/features/flashcards/FlashcardDeck.tsx` — Client Component deck container: integrates FlashcardFlip, DifficultyFilter, DeckComplete; progress counter (X of Y); previous/next navigation; mark-complete toggle button; CSS `contain: content` for performance on >25 items
- [ ] T070 [P] [US2] Create `src/hooks/useProgress.ts` — TanStack Query hooks: `useQuestionProgress(lessonId)` (fetches completed IDs), `useToggleQuestion()` mutation with optimistic update via `progress.store`
- [ ] T071 [US2] Create cross-topic flashcard mode page at `app/(platform)/questions/page.tsx` — Server Component with topic selector; Client Component shell rendering FlashcardDeck with randomized questions from `getRandomQuestionsAcrossTopics()` (FR-018)
- [ ] T072 [US2] Create topic-specific flashcard page at `app/(platform)/questions/[topic]/page.tsx` — Server Component fetching lesson by topic slug, renders FlashcardDeck; `generateMetadata()`; `export const unstable_instant = true`

**Checkpoint**: US2 fully functional — flashcards work with filters, 3D animation, completion persistence

---

## Phase 5: User Story 3 — Get Help from the AI Mentor "Sensei" (Priority: P2)

**Goal**: User can open the Sensei chat panel from any page, ask questions in natural language with lesson context, receive streaming responses with code examples, and have conversation history persist within the session.

**Independent Test**: Open Sensei panel from `/learn/[any-lesson]`, type "explain event loop with an example", verify a streamed response with a code snippet appears within 5 seconds. Close and reopen panel — verify conversation history is still visible.

### Implementation for User Story 3

- [ ] T073 Create OpenAI client singleton in `src/lib/ai/client.ts` — exports `openai` instance configured with `OPENAI_API_KEY`, exports `FAST_MODEL` and `SMART_MODEL` constants from env
- [ ] T074 [P] [US3] Create system prompt templates in `src/lib/ai/prompts.ts` — `buildSenseiSystemPrompt(lessonContext?, categoryContext?)` injecting platform identity, current lesson context, and instruction to always include code examples for technical topics
- [ ] T075 [P] [US3] Create Sensei streaming chat logic in `src/lib/ai/sensei.ts` — `streamSenseiResponse(sessionId, userMessage, lessonContext, categoryContext)`: loads/creates ChatSession from MongoDB, builds 10-message rolling window, calls `openai.chat.completions.stream()`, returns ReadableStream of SSE tokens, saves assistant message + token count on stream end
- [ ] T076 [US3] Create POST `/api/ai/chat` route handler in `app/api/ai/chat/route.ts` — authenticates user, applies Redis rate limit (20 req/min per user via `rate-limit.ts`), calls `streamSenseiResponse()`, returns `text/event-stream` response with `data: {"token":"..."}` events and final `data: {"done":true,"sessionId":"..."}` event (FR-019, FR-020)
- [ ] T077 [P] [US3] Create `src/components/features/sensei/ChatMessage.tsx` — user/assistant message bubble: user bubble right-aligned, assistant bubble left-aligned with avatar; renders content via react-markdown with custom CodeBlock component for code highlighting (FR-021)
- [ ] T078 [P] [US3] Create `src/components/features/sensei/ChatInput.tsx` — Client Component textarea (auto-resize) with send button, streaming loading indicator (animated dots), disabled state while streaming, Enter-to-send (Shift+Enter for newline)
- [ ] T079 [P] [US3] Create `src/components/features/sensei/SenseiUnavailable.tsx` — graceful fallback card displayed when AI service returns 5xx error: "Sensei is resting — try again in a moment" message without disrupting lesson (FR-022)
- [ ] T080 [US3] Create `src/components/features/sensei/ChatWindow.tsx` — Client Component: scrollable message list (shadcn ScrollArea), auto-scrolls to bottom on new message, renders ChatMessage list, max 50 messages visible (Zustand slice), shows SenseiUnavailable on error state
- [ ] T081 [US3] Create `src/components/layout/SenseiPanel.tsx` — Client Component using shadcn Sheet (slide-out from right on desktop, bottom Sheet on mobile); contains ChatWindow + ChatInput; reads `ui.store.senseiOpen`; passes current `lessonContext` + `categoryContext` to useSenseiChat
- [ ] T082 [US3] Create `src/hooks/useSenseiChat.ts` — Client hook: POSTs to `/api/ai/chat` with fetch, reads ReadableStream via `getReader()`, parses SSE `data:` lines, appends tokens to `sensei.store.messages`, handles `done` event to finalize message and store `sessionId`
- [ ] T083 [US3] Wire SenseiPanel into `app/(platform)/layout.tsx` and connect Sensei toggle button in TopNav to `ui.store.setSenseiOpen()`; pass current route context (lessonSlug, categorySlug) as props to SenseiPanel

**Checkpoint**: US3 fully functional — Sensei streaming chat with context awareness, history persistence, graceful fallback

---

## Phase 6: User Story 4 — Track Daily Progress and Maintain a Study Streak (Priority: P3)

**Goal**: User completing lessons/challenges sees streak counter increment each day. Dashboard shows per-category completion percentages and a weekly activity heatmap. Missing a day resets streak to 0.

**Independent Test**: Complete a lesson, verify streak increments on dashboard. Return next calendar day (or mock date), complete another lesson — streak shows 2. Check `/analytics` for weekly heatmap, category percentages, and badge gallery.

### Implementation for User Story 4

- [ ] T084 [P] [US4] Create GET `/api/progress` route handler in `app/api/progress/route.ts` — authenticates user, checks Redis `progress:{userId}:summary` cache (5min TTL), falls back to `progress.service.getUserProgress()`, returns completedLessons, completedQuestions, categoryProgress, streak, badges, weeklyActivity
- [ ] T085 [P] [US4] Create POST `/api/progress` route handler in `app/api/progress/route.ts` — validates `{ lessonId, timeSpentMs }` with Zod, calls `progress.service.updateStreakRecord()`, evaluates badges, invalidates Redis `streak:{userId}` and `progress:{userId}:summary`
- [ ] T086 [P] [US4] Create `src/components/features/progress/StreakBadge.tsx` — flame icon (Lucide Flame) + streak count; Framer Motion pulse animation when streak increments; color shifts green → orange → red based on streak length
- [ ] T087 [P] [US4] Create `src/components/features/progress/ActivityHeatmap.tsx` — Client Component: GitHub-style 7-column weekly grid using `dates.generateHeatmapGrid()`; cell color intensity based on `lessonsCompleted + questionsAnswered`; tooltip on hover showing date + activity count (FR-030)
- [ ] T088 [P] [US4] Create `src/components/features/progress/CategoryProgress.tsx` — progress ring per category using shadcn Progress (circular via CSS) + category icon; percentage label; animated fill on mount with Framer Motion (FR-028)
- [ ] T089 [P] [US4] Create `src/components/features/progress/BadgeGallery.tsx` — grid of Badge icons; earned badges in full color, locked badges in grayscale; tooltip with badge name + condition description (FR-029)
- [ ] T090 [P] [US4] Create `src/hooks/useStreak.ts` — TanStack Query hook fetching streak data from `/api/progress`; staleTime 5min; invalidated after lesson/challenge completion mutations
- [ ] T091 [US4] Create analytics/progress dashboard at `app/(platform)/analytics/page.tsx` — Server Component: renders ActivityHeatmap, CategoryProgress grid, BadgeGallery, time-studied stat, modules-completed stat, questions-answered stat; `generateMetadata()` (FR-030)
- [ ] T092 [US4] Update dashboard page at `app/(platform)/dashboard/page.tsx` — wire in live StreakBadge (from `useStreak` hook), CategoryProgress summary cards, and overall progress percentage in TopNav

**Checkpoint**: US4 fully functional — streaks update on lesson completion, analytics page shows heatmap + badges + category rings

---

## Phase 7: User Story 5 — Simulate a Mock Interview Session (Priority: P3)

**Goal**: User can start a timed coding, behavioral, or system design mock interview, write their solution in an embedded editor or whiteboard, submit, and receive AI-evaluated structured feedback.

**Independent Test**: Start a coding mock interview at `/mock-interviews/coding`, receive a prompt, write code in Monaco editor, submit — verify structured AI feedback panel appears with correctness/efficiency/approach sections. Open `/mock-interviews/system-design` — verify Excalidraw whiteboard loads.

### Implementation for User Story 5

- [ ] T093 Create `src/services/mock-interview.service.ts` with functions: `createSession(userId, type, language)` (selects random prompt by type), `getSession(sessionId, userId)`, `saveResponse(sessionId, response)`, `saveFeedback(sessionId, feedback)`, `listUserSessions(userId)`
- [ ] T094 [P] [US5] Add interview feedback prompt builder to `src/lib/ai/prompts.ts` — `buildInterviewFeedbackPrompt(type, prompt, response)` returning structured JSON schema request for correctness, efficiency, approach, summary, score
- [ ] T095 [P] [US5] Create POST `/api/mock-interviews` route handler in `app/api/mock-interviews/route.ts` — validates `{ type, language? }`, calls `mock-interview.service.createSession()`, returns session with prompt, timerSeconds, startedAt (FR-023)
- [ ] T096 [P] [US5] Create GET + PATCH `/api/mock-interviews/[id]` route handlers in `app/api/mock-interviews/[id]/route.ts` — GET returns session; PATCH validates `{ response }`, calls OpenAI (non-streaming structured output) for feedback, saves via `mock-interview.service.saveFeedback()`, returns feedback object (FR-025)
- [ ] T097 [P] [US5] Create `src/components/features/mock-interview/CodeEditor.tsx` — Client Component with Monaco Editor via `dynamic(() => import('@monaco-editor/react'), { ssr: false })`, supports JS/TS/Python language selection, dark theme, auto-indent (FR-024)
- [ ] T098 [P] [US5] Create `src/components/features/mock-interview/TimerBar.tsx` — countdown timer displaying MM:SS, Framer Motion progress bar width animation, color transitions: green (>50%) → yellow (20–50%) → red (<20%), pulse on critical
- [ ] T099 [P] [US5] Create `src/components/features/mock-interview/FeedbackPanel.tsx` — AI feedback display: correctness reasoning, efficiency analysis, approach quality (each as collapsible section), overall score badge, summary paragraph
- [ ] T100 [P] [US5] Create `src/components/features/mock-interview/WhiteboardCanvas.tsx` — Client Component with Excalidraw via `dynamic(() => import('@excalidraw/excalidraw'), { ssr: false })`, serializes canvas state to JSON for storage in `MockInterview.response` (FR-038)
- [ ] T101 [P] [US5] Create `src/components/features/mock-interview/BehavioralPrompt.tsx` — STAR-method guidance panel: expandable sections for Situation, Task, Action, Result with guiding questions and self-evaluation rubric (FR-026)
- [ ] T102 [US5] Create mock interview picker page at `app/(platform)/mock-interviews/page.tsx` — session type cards (CODING, SYSTEM_DESIGN, BEHAVIORAL) with descriptions and difficulty selector
- [ ] T103 [US5] Create coding interview setup page at `app/(platform)/mock-interviews/coding/page.tsx` — language selector (JS/TS/Python) + difficulty selector; POST to `/api/mock-interviews` on start → redirect to active session
- [ ] T104 [US5] Create active coding session page at `app/(platform)/mock-interviews/coding/[sessionId]/page.tsx` — Client Component: CodeEditor + TimerBar; submit button POSTs response to PATCH `/api/mock-interviews/[id]`; FeedbackPanel appears on completion; prevents navigation with `useBeforeUnload`
- [ ] T105 [US5] Create behavioral mock interview page at `app/(platform)/mock-interviews/behavioral/page.tsx` — BehavioralPrompt + STAR-method text areas + self-evaluation rubric (FR-026)
- [ ] T106 [US5] Create system design whiteboard page at `app/(platform)/mock-interviews/system-design/page.tsx` — WhiteboardCanvas + scenario prompt panel + submit for AI feedback on exported JSON
- [ ] T107 Create AI quiz generation logic in `src/lib/ai/quiz.ts` and POST `/api/ai/quiz` route handler in `app/api/ai/quiz/route.ts` — validates `{ topic, difficulty, count }`, calls OpenAI with JSON schema for question/answer/explanation triples, applies rate limit (10 req/min)
- [ ] T108 Create AI resume analysis logic in `src/lib/ai/resume.ts` and POST `/api/ai/resume` route handler in `app/api/ai/resume/route.ts` — validates `{ resumeText }` (max 8000 chars), calls OpenAI SMART model for structured feedback (summary, strengths, improvements, keywords, score), applies rate limit (5 req/min) (FR-039)
- [ ] T109 [US5] Create resume analyzer page at `app/(platform)/resume/page.tsx` — plain-text `<textarea>` paste area + analyze button; displays structured AI feedback from `/api/ai/resume` with score, strengths, and improvement suggestions
- [ ] T110 [US5] Create system design guides library page at `app/(platform)/system-design/page.tsx` — Server Component rendering SystemDesignGuide cards (10 walkthroughs) with tag filtering
- [ ] T111 [US5] Create system design guide detail page at `app/(platform)/system-design/[slug]/page.tsx` — Server Component rendering guide sections (text/diagram/steps) with DiagramRenderer for architecture diagrams; `generateMetadata()`

**Checkpoint**: US5 fully functional — all 3 mock interview types work, AI feedback renders, resume analyzer operational

---

## Phase 8: User Story 6 — Save Bookmarks and Personal Notes (Priority: P4)

**Goal**: User can bookmark any lesson, write a personal note on a lesson, access all bookmarks and notes from `/notes`, and search note content via the command menu.

**Independent Test**: Bookmark a lesson → visit `/notes` → confirm it appears in Bookmarks tab. Write a note on a lesson, search for a unique keyword in CommandMenu — verify the note appears in results. Click result → navigate back to correct lesson.

### Implementation for User Story 6

- [ ] T112 Create `src/services/note.service.ts` with functions: `upsertNote(userId, lessonId, content)`, `deleteNote(noteId, userId)`, `getNotesByUser(userId)`, `getNoteByLesson(userId, lessonId)`, `updateNoteSearchVector(noteId)` (raw Prisma query to refresh tsvector)
- [ ] T113 Create `src/services/bookmark.service.ts` with functions: `toggleBookmark(userId, lessonId)` (create if absent, delete if exists), `getBookmarksByUser(userId)` (includes lesson + module + category), `isLessonBookmarked(userId, lessonId)`
- [ ] T114 [P] [US6] Create GET + POST `/api/bookmarks` route handler in `app/api/bookmarks/route.ts` — GET returns enriched bookmark list; POST validates `{ lessonId }`, calls `bookmark.service.toggleBookmark()`
- [ ] T115 [P] [US6] Create DELETE `/api/bookmarks/[id]` route handler in `app/api/bookmarks/[id]/route.ts` — verifies ownership, deletes Bookmark record
- [ ] T116 [P] [US6] Create GET + POST `/api/notes` route handler in `app/api/notes/route.ts` — GET with optional `lessonId` filter; POST validates `{ lessonId, content }` (max 5000 chars), upserts Note
- [ ] T117 [P] [US6] Create PUT + DELETE `/api/notes/[id]` route handler in `app/api/notes/[id]/route.ts` — PUT updates note content; DELETE verifies ownership and deletes
- [ ] T118 Create saveNote and deleteNote server actions in `src/app/actions/notes.ts` — upserts Note via `note.service`, refreshes FTS searchVector via raw query, revalidates `/notes`
- [ ] T119 Create toggleBookmark server action in `src/app/actions/bookmarks.ts` — calls `bookmark.service.toggleBookmark()`, revalidates `/notes` path, returns `{ bookmarked: boolean }`
- [ ] T120 [P] [US6] Create `src/hooks/useBookmarks.ts` — TanStack Query hooks: `useBookmarks()` query, `useToggleBookmark()` mutation with optimistic update
- [ ] T121 [P] [US6] Create `src/hooks/useNotes.ts` — TanStack Query hooks: `useNotes(lessonId?)` query, `useSaveNote()` and `useDeleteNote()` mutations
- [ ] T122 [P] [US6] Create `src/components/features/notes/NoteEditor.tsx` — Client Component: textarea with character count display (warn at 90% of 5000 limit), autosave debounce (1000ms) via `useSaveNote` mutation (FR-032, edge case handler)
- [ ] T123 [P] [US6] Create `src/components/features/notes/BookmarkList.tsx` — Client Component: saved lessons list with lesson title, module/category breadcrumb, creation date, remove bookmark button; local keyword filter input (FR-031, FR-033)
- [ ] T124 [US6] Create notes & bookmarks page at `app/(platform)/notes/page.tsx` — Server Component with two shadcn Tabs: "Notes" (NoteEditor + note list) and "Bookmarks" (BookmarkList); `generateMetadata()`
- [ ] T125 Create `src/services/search.service.ts` with function `fullTextSearch(userId, query, type, limit)` — builds `websearch_to_tsquery` queries, executes raw Prisma queries on Lesson, InterviewQuestion, Note tsvector columns with `ts_rank`, merges and returns ranked results
- [ ] T126 [P] [US6] Create `src/lib/search/full-text.ts` — helper functions: `buildTsQuery(query)`, `searchLessons(query, limit)`, `searchQuestions(query, limit)`, `searchNotes(userId, query, limit)` using Prisma `$queryRaw`
- [ ] T127 [US6] Create GET `/api/search` route handler in `app/api/search/route.ts` — validates `q` (max 100 chars), `type`, `limit` params; checks Redis `search:{queryHash}` (5min TTL); calls `search.service.fullTextSearch()`; caches result (FR-006)
- [ ] T128 [P] [US6] Create `src/hooks/useCommandMenu.ts` — Zustand-backed hook: ⌘K keyboard shortcut listener, open/close state, debounced search query (300ms), clears on close
- [ ] T129 [US6] Create `src/components/layout/CommandMenu.tsx` — Client Component using shadcn Command: ⌘K global overlay, live search input wired to `/api/search`, displays grouped results (Lessons/Questions/Notes), keyboard navigation, click navigates to result URL (FR-005, FR-006)

**Checkpoint**: US6 fully functional — bookmarks and notes persist, full-text search operational in CommandMenu

---

## Phase 9: User Story 7 — Complete Daily Interview Challenges (Priority: P4)

**Goal**: User sees a new Daily Challenge card on the dashboard each calendar day, completes it, earns a badge, and maintains their streak.

**Independent Test**: Load dashboard on a fresh day — verify DailyChallengeCard shows today's challenge. Complete the challenge → verify ChallengeComplete badge and streak maintained. Hit `/api/cron/rotate-challenge` with CRON_SECRET header → verify Redis cache is updated.

### Implementation for User Story 7

- [ ] T130 Create `src/services/challenge.service.ts` with functions: `getTodayChallenge()` (checks Redis `challenge:today`, falls back to DB by date), `completeChallengeForUser(userId, challengeId)` (upserts DailyChallengeCompletion, updates streak, evaluates badges), `isChallengeCompleted(userId, challengeId)`
- [ ] T131 [P] [US7] Create GET `/api/challenges/today` route handler in `app/api/challenges/today/route.ts` — authenticates user, calls `challenge.service.getTodayChallenge()`, includes `completed` and `completedAt` for the current user (FR-035)
- [ ] T132 [P] [US7] Create POST `/api/challenges/complete` route handler in `app/api/challenges/complete/route.ts` — validates `{ challengeId }`, calls `challenge.service.completeChallengeForUser()`, returns `{ completedAt, streakCurrent, newBadges }` (FR-036)
- [ ] T133 [P] [US7] Create GET `/api/cron/rotate-challenge` route handler in `app/api/cron/rotate-challenge/route.ts` — validates `Authorization: Bearer {CRON_SECRET}` header, finds next day's DailyChallenge from DB, updates Redis `challenge:today` key (TTL until midnight UTC), returns `{ rotated, date }`
- [ ] T134 [P] [US7] Create `src/components/features/daily-challenge/DailyChallengeCard.tsx` — Client Component: challenge title, type badge (CODING/CONCEPT/SYSTEM_DESIGN), difficulty, description preview, "Complete Today's Challenge" button; completed state shows checkmark + timestamp (FR-035)
- [ ] T135 [P] [US7] Create `src/components/features/daily-challenge/ChallengeComplete.tsx` — Framer Motion celebration: completion badge icon, "Streak Preserved!" message, new badge award notification if triggered (FR-036)
- [ ] T136 [US7] Create challenges history page at `app/(platform)/challenges/page.tsx` — Server Component: today's DailyChallengeCard at top, completion history list below; `generateMetadata()`
- [ ] T137 [P] [US7] Create `vercel.json` at project root with cron configuration: `{ "crons": [{ "path": "/api/cron/rotate-challenge", "schedule": "0 0 * * *" }] }`
- [ ] T138 [US7] Wire DailyChallengeCard into dashboard at `app/(platform)/dashboard/page.tsx` — fetch today's challenge server-side, render DailyChallengeCard with client-side completion mutation calling `/api/challenges/complete`

**Checkpoint**: US7 fully functional — daily challenges rotate, completion updates streak, cron configured for Vercel

---

## Phase 10: Polish & Cross-Cutting Concerns

**Purpose**: Settings, account management, final integrations, deployment preparation

- [ ] T139 Create `src/services/user.service.ts` with functions: `updateUser(userId, data)`, `deleteUser(userId)` (Prisma cascading delete → MongoDB ChatSession purge → Redis key eviction for `session:{userId}`, `streak:{userId}`, `progress:{userId}:summary`)
- [ ] T140 Create updateProfile and deleteAccount server actions in `src/app/actions/user.ts` — updateProfile validates with Zod (all optional fields), calls `user.service.updateUser()`, revalidates `/settings`; deleteAccount validates `confirmEmail`, calls `user.service.deleteUser()`, calls Auth.js signOut, redirects to `/` (FR-047)
- [ ] T141 [P] Create PATCH `/api/user` route handler in `app/api/user/route.ts` — validates profile update payload, calls `user.service.updateUser()`, returns updated user object
- [ ] T142 [P] Create DELETE `/api/user` route handler in `app/api/user/route.ts` — validates `{ confirmEmail }` matches session email, calls `user.service.deleteUser()`, returns `{ deleted: true }` (FR-047)
- [ ] T143 Create settings page at `app/(platform)/settings/page.tsx` — profile form (name, avatar URL, learning goal, theme toggle, notification preferences) using react-hook-form + Zod + updateProfile action; danger zone section with account deletion dialog using shadcn AlertDialog requiring email confirmation; `generateMetadata()`
- [ ] T144 [P] Create `src/hooks/useTheme.ts` — Client hook: reads `ui.store.theme`, toggles `dark` class on `document.documentElement`, persists preference to `localStorage`; syncs on mount from `localStorage` (FR-002)
- [ ] T145 Add `generateMetadata()` exports to all remaining platform pages missing them: `/questions`, `/questions/[topic]`, `/mock-interviews`, `/mock-interviews/coding`, `/mock-interviews/behavioral`, `/mock-interviews/system-design`, `/resume`, `/system-design`, `/challenges`
- [ ] T146 Integrate StreakBadge component into TopNav at `src/components/layout/TopNav.tsx` using `useStreak` hook; integrate dark/light mode toggle button connected to `useTheme` hook
- [ ] T147 Create GitHub Actions CI workflow at `.github/workflows/ci.yml` — on PR: `pnpm lint` (ESLint) + `pnpm tsc --noEmit` (TypeScript type check) + Vercel preview deployment trigger
- [ ] T148 Run full manual smoke-check walkthrough per `specs/003-web-ninja-platform-setup/quickstart.md`: start Docker services → migrate + seed DB → verify all 7 user story flows end-to-end → verify mobile layout on 375px viewport → verify dark/light mode toggle

---

## Dependencies & Execution Order

### Phase Dependencies

- **Setup (Phase 1)**: No dependencies — can start immediately
- **Foundational (Phase 2)**: Depends on Phase 1 completion — **BLOCKS all user stories**
- **User Stories (Phases 3–9)**: All depend on Phase 2 completion
  - Phases 3–9 can proceed sequentially in priority order (P1 → P2 → P3 → P4)
  - Or in parallel across multiple developers if staffed
- **Polish (Phase 10)**: Depends on all desired user stories being complete

### User Story Dependencies

- **US1 (P1)**: Can start after Phase 2 — no dependencies on other stories
- **US2 (P2)**: Can start after Phase 2 — depends on US1 Lesson pages being navigable (for context) but independently testable via `/questions`
- **US3 (P2)**: Can start after Phase 2 — independent; integrates into platform layout already built in US1
- **US4 (P3)**: Can start after Phase 2 — depends on `progress.service.ts` from US1, extends it
- **US5 (P3)**: Can start after Phase 2 — fully independent of US1–US4
- **US6 (P4)**: Can start after Phase 2 — fully independent; enhances lesson pages from US1
- **US7 (P4)**: Can start after Phase 2 — fully independent; integrates into dashboard from US1

### Within Each User Story

- Service layer → Route Handlers → Components → Pages
- Shared components (SkeletonCard, DifficultyBadge, etc.) → Feature components
- Core page shell → Sub-components
- API route handler → Client hook → Component integration

### Parallel Opportunities

- All [P]-marked tasks within a phase can run concurrently (different files, no intra-phase dependencies)
- Phase 1 setup tasks T002–T006 all [P] after T001 creates the project
- Phase 2 infrastructure tasks T014–T025 all [P] after T011–T013 complete
- All shadcn component installs (T008) can run concurrently with T009/T010
- After Phase 2: US1 and US5 have zero inter-dependency and can run in parallel

---

## Parallel Example: Phase 2 Foundational

```bash
# After T011-T013 complete (schema + migration + seed):
Task: "Create Prisma client singleton in src/lib/db/prisma.ts"              # T014
Task: "Create Mongoose ChatSession schema in src/lib/db/schemas/..."        # T015
Task: "Create MongoDB connection singleton in src/lib/db/mongodb.ts"        # T016
Task: "Create ioredis client singleton in src/lib/cache/redis.ts"           # T017
Task: "Create rate limiter in src/lib/cache/rate-limit.ts"                  # T018
Task: "Create TypeScript type definitions in src/types/"                    # T023
Task: "Create utility functions in src/lib/utils/"                          # T024
Task: "Create Zustand stores in src/stores/"                                # T025
```

## Parallel Example: User Story 1 Components

```bash
# After T034 (lesson.service.ts) and T052/T053 complete:
Task: "Create RoadmapTimeline in src/components/features/roadmap/"          # T035
Task: "Create CategorySection in src/components/features/roadmap/"          # T036
Task: "Create ModuleCard in src/components/features/roadmap/"               # T037
Task: "Create LessonLayout in src/components/features/lesson/"              # T041
Task: "Create CodeBlock in src/components/features/lesson/"                 # T043
Task: "Create DiagramRenderer in src/components/features/lesson/"           # T044
Task: "Create all shared components in src/components/shared/"              # T047-T051
```

---

## Implementation Strategy

### MVP First (User Story 1 Only)

1. Complete Phase 1: Setup (T001–T010)
2. Complete Phase 2: Foundational (T011–T033) — **CRITICAL, blocks everything**
3. Complete Phase 3: User Story 1 (T034–T062)
4. **STOP and VALIDATE**: Navigate dashboard → roadmap → lesson → mark complete
5. Deploy preview — demonstrate core value proposition

### Incremental Delivery

1. **Foundation** (Phase 1 + 2) → DB migrated, auth working, layouts rendered
2. **US1** (Phase 3) → Roadmap + lessons MVP → **deploy as v0.1**
3. **US2 + US3** (Phase 4 + 5) → Flashcards + Sensei → **deploy as v0.2**
4. **US4 + US5** (Phase 6 + 7) → Progress/streaks + Mock interviews → **deploy as v0.3**
5. **US6 + US7** (Phase 8 + 9) → Notes/bookmarks + Challenges → **deploy as v0.4**
6. **Polish** (Phase 10) → Settings, account deletion, CI/CD → **deploy as v1.0**

### Parallel Team Strategy

With 3 developers after Phase 2:

- **Developer A**: US1 (roadmap, lessons, dashboard) + US2 (flashcards)
- **Developer B**: US3 (Sensei AI streaming) + US4 (progress/gamification)
- **Developer C**: US5 (mock interviews, resume, system design)
- All reconvene for US6 + US7 + Polish

---

## Notes

- [P] tasks = different files, no dependencies on incomplete sibling tasks in the same phase
- [USn] label maps each task to a specific user story for traceability (US1–US7)
- No automated tests in any task — Constitution Principle V (NON-NEGOTIABLE)
- Each user story phase should be independently completable and manually verifiable before proceeding
- Commit after completing each phase or logical group of tasks
- All AI route handlers must apply Redis rate limiting before OpenAI calls
- All Route Handlers validate request input with Zod before calling service layer
- Server Actions return `ActionState` type; Client Components consume via `useActionState`
