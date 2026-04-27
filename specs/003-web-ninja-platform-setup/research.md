# Research: Web Ninja Platform Setup

**Phase**: 0 — Research & Decision Log  
**Branch**: `003-web-ninja-platform-setup`  
**Spec**: [spec.md](./spec.md)

---

## Summary of Unknowns Resolved

All items marked `NEEDS CLARIFICATION` in the Technical Context have been researched and resolved below.

---

## Decision 1: Authentication Library

**Decision**: Use **Auth.js v5** (`next-auth@5`) with the Prisma adapter.

**Rationale**:
- Next.js documentation explicitly recommends Auth.js as a proven option for Next.js App Router.
- Provides built-in support for credentials (email + password) and Google OAuth — both required by the spec.
- Prisma adapter gives a type-safe integration with the PostgreSQL models we already need (User, Account, Session, VerificationToken).
- JWT + database session hybrid: JWT in cookies for stateless API auth, database sessions for server-side validation.
- Supports middleware-level route protection via `auth()` wrapper — no additional RBAC library needed for MVP.

**Alternatives considered**:
- **Better Auth**: Newer, also has excellent TypeScript support and Prisma adapter. Less community precedent but listed first in Next.js docs. Deferred as Auth.js v5 is more widely tested.
- **Clerk**: Managed auth with beautiful UI; introduces a paid SaaS dependency and limits self-hosting. Rejected per Principle IV (Minimal Dependencies / control).
- **Custom JWT**: Full control but requires building session management, refresh logic, and OAuth from scratch. Security liability rejected.

---

## Decision 2: AI Provider (Sensei)

**Decision**: Use **OpenAI API** with `openai` Node.js SDK (`openai@^4`). Model: `gpt-4o-mini` for standard queries, `gpt-4o` for complex analysis (resume review, mock interview feedback).

**Rationale**:
- Most mature AI SDK with first-class TypeScript support and streaming via `openai.chat.completions.stream()`.
- Next.js Route Handlers support streaming `ReadableStream` responses natively — enables token-by-token Sensei streaming (SC-003: responses within 5s).
- `gpt-4o-mini` offers excellent cost/quality balance for the high-frequency chat use case.
- Provider-agnostic abstraction layer (`src/lib/ai/client.ts`) allows future swap to Anthropic, Gemini, etc.

**Alternatives considered**:
- **Anthropic Claude**: Comparable quality; less established Node.js streaming ergonomics at time of research.
- **Google Gemini**: Good for multimodal; overkill for text-only chat/quiz use case.
- **Self-hosted (Ollama)**: Eliminates API cost but introduces hosting complexity beyond the 99.5% uptime SLA target.

---

## Decision 3: State Management

**Decision**: **Zustand** (not Redux Toolkit) for client-side UI state.

**Rationale**:
- Next.js App Router runs page components as Server Components by default. Redux's global store singleton doesn't fit this model safely without careful provider wrapping.
- Zustand stores are created outside React's render tree and accessed in Client Components only — aligns cleanly with the `'use client'` boundary model.
- Three stores cover the entire MVP UI state need: `ui.store.ts` (sidebar, theme, Sensei panel open/closed), `sensei.store.ts` (chat messages buffer, loading state), `progress.store.ts` (optimistic lesson/question completions).
- Zero boilerplate vs Redux Toolkit's slice/action/reducer ceremony.

**Alternatives considered**:
- **Redux Toolkit**: More ecosystem support, Redux DevTools; the added complexity is unjustified for 3 small stores.
- **React Context**: Sufficient for theme; causes unnecessary re-renders for high-frequency state (Sensei chat, sidebar toggle). Rejected for dynamic state.
- **Jotai**: Valid alternative to Zustand; Zustand's flat store pattern is easier to reason about for this use case.

---

## Decision 4: Server State / Data Fetching

**Decision**: **TanStack Query v5** (`@tanstack/react-query`) for client-driven data fetching from Route Handlers. Next.js Server Components directly use `prisma` or service functions for server-side data (no React Query on the server).

**Rationale**:
- TanStack Query handles cache invalidation, background refresh, optimistic updates, and loading/error states for the client-heavy features: flashcard completion, bookmarks, notes, mock interview sessions.
- Complements Next.js Server Components: SSR data is fetched in Server Components directly; client interactions go through React Query + Route Handlers.
- `useMutation` + optimistic updates for marking questions as completed and lesson progress (FR-017) gives immediate UI feedback without a loading state.

**Alternatives considered**:
- **SWR**: Simpler API but fewer features (no mutation helpers, no optimistic updates built-in).
- **Native fetch + useEffect**: Reimplements caching manually; not worth the effort.

---

## Decision 5: Code Editor (Mock Interviews)

**Decision**: **Monaco Editor** via `@monaco-editor/react`.

**Rationale**:
- The same editor that powers VS Code — instantly familiar to developer users.
- Supports JavaScript, TypeScript, and Python out of the box (all required by spec assumptions).
- Client Component only (`'use client'`); lazy-loaded with dynamic import to avoid bundle bloat.
- Built-in syntax highlighting, auto-indentation, bracket matching.

**Alternatives considered**:
- **CodeMirror 6**: More lightweight and customizable; Monaco better matches developer expectations ("feels like VS Code").
- **Ace Editor**: Legacy; Monaco is the modern standard.

---

## Decision 6: Markdown / Lesson Content Rendering

**Decision**: **`react-markdown`** with `remark-gfm` (GitHub Flavored Markdown) and `rehype-highlight` (Highlight.js syntax themes) for rendering lesson content stored as Markdown strings.

**Rationale**:
- Lesson content (FR-009) is authored as Markdown and stored in `LessonSection.content` JSON field.
- `react-markdown` renders safely (no dangerouslySetInnerHTML), supports plugins, and produces accessible HTML.
- `rehype-highlight` uses Highlight.js tokens — maps to terminal-style theming via CSS class overrides.
- For inline code blocks with a copy button (FR-042), a custom `CodeBlock` component replaces the default `<code>` renderer in `react-markdown`'s `components` prop.

**Alternatives considered**:
- **Next.js MDX**: For static MDX files, excellent. For DB-stored dynamic content, `@next/mdx` doesn't apply — we need runtime rendering.
- **`marked`**: Simpler but outputs raw HTML string — requires `dangerouslySetInnerHTML`, XSS risk.
- **`rehype-pretty-code` + Shiki**: Better syntax highlighting quality (Shiki uses TextMate grammars). Adds significant bundle size. Future enhancement candidate.

---

## Decision 7: Architecture Diagrams in Lessons

**Decision**: **Mermaid.js** (`mermaid`) for rendering architecture and flow diagrams inline.

**Rationale**:
- Diagrams are stored as Mermaid text syntax in `LessonSection.content`. No SVG files to manage.
- Mermaid renders client-side — a `DiagramRenderer.tsx` Client Component calls `mermaid.render()` on mount.
- Supports flowcharts, sequence diagrams, class diagrams, ER diagrams — all relevant for web engineering topics.
- Zero server-side complexity.

**Alternatives considered**:
- **Excalidraw (for lessons)**: Designed for interactive drawing, not static diagram rendering. Overkill.
- **PlantUML**: Requires server-side rendering. Adds infra complexity.
- **Pre-rendered SVG stored in DB**: Inflexible for content updates; Mermaid source is more maintainable.

---

## Decision 8: System Design Whiteboard

**Decision**: **Excalidraw** (`@excalidraw/excalidraw`) embedded as a Client Component for the system design whiteboard in mock interviews.

**Rationale**:
- Provides a fully interactive whiteboard with shapes, arrows, text — exactly what system design sessions require.
- State (the drawn elements) is serializable JSON — easily stored in `MockInterview.response`.
- Used in many well-known developer tools; users are familiar with the UX.

**Alternatives considered**:
- **tldraw**: Equally capable; Excalidraw has broader brand recognition in developer community.
- **Custom canvas**: Far too much implementation effort for MVP.

---

## Decision 9: Full-Text Search

**Decision**: **PostgreSQL `tsvector` / `tsquery`** for full-text search on lessons, questions, and notes. Indexed columns on `Lesson.searchVector`, `InterviewQuestion.searchVector`, and `Note.searchVector`. Redis caches top search results for 5 minutes.

**Rationale**:
- Eliminates an external search service (Elasticsearch, Algolia) for MVP — Principle IV.
- PostgreSQL FTS is production-quality for the target scale (< 10k lessons/questions at launch).
- `ts_rank` for relevance ordering.
- Prisma's `Unsupported("tsvector")` type + raw queries for FTS operations.

**Alternatives considered**:
- **Algolia**: Excellent DX but external SaaS cost and dependency. Deferred to post-MVP.
- **Typesense**: Self-hosted Algolia alternative. Additional infra to maintain. Post-MVP.
- **`pg_trgm`**: Trigram similarity search — simpler but less accurate for multi-word queries.

---

## Decision 10: Monorepo Structure

**Decision**: **Single Next.js application** for MVP. No separate NestJS backend service. All API logic in Next.js Route Handlers and Server Actions. The `pnpm-workspace.yaml` is pre-configured for future package extraction but no packages are split out now.

**Rationale**:
- The spec targets 500 concurrent users — Next.js Route Handlers with PostgreSQL + Redis handle this comfortably without a separate API service layer.
- NestJS separation adds: Docker networking, separate deployments, CORS config, authentication forwarding complexity. All unjustified at MVP scale.
- Clean service layer (`src/services/`) within the Next.js app ensures a clean extraction path to NestJS in the future without rewriting business logic.
- Future: when AI services, background job queues, or real-time WebSocket needs grow, extract to separate `packages/api` workspace package.

**Alternatives considered**:
- **NestJS monorepo package**: Premature. Adds ~2 weeks of infra setup for zero user-facing value in MVP.
- **Express.js API routes**: Stepping away from Next.js strengths. Rejected.

---

## Decision 11: Background Jobs (Daily Challenge Rotation)

**Decision**: **`node-cron`** within a Next.js Route Handler invoked by a Vercel Cron Job. A `GET /api/cron/rotate-challenge` endpoint rotates the daily challenge at midnight UTC. The route is protected by a `CRON_SECRET` header.

**Rationale**:
- Vercel Cron Jobs are free on Vercel's Hobby/Pro plans and natively integrate with Next.js Route Handlers.
- `node-cron` avoids the complexity of external job queue services (BullMQ, Inngest) for a single daily task.

**Alternatives considered**:
- **BullMQ + Redis**: Overkill for a single daily cron. Redis is already in the stack — BullMQ is a valid upgrade path.
- **Inngest**: Excellent developer experience; adds external service dependency. Deferred.
- **PostgreSQL pg_cron**: Extension not available on all managed PostgreSQL providers.

---

## Decision 12: Caching Strategy

**Decision**: **Redis** (`ioredis`) for:
1. Rate limiting on AI endpoints (Sensei chat, resume analyzer) — sliding window counter per user.
2. Session data caching — Auth.js sessions cached in Redis to reduce PostgreSQL load on every authenticated request.
3. Daily challenge caching — challenge object cached for 24h, invalidated on rotation.
4. Search result caching — top 50 search queries cached for 5 minutes.
5. User streak data — cached for 1 hour to reduce per-request DB reads.

**Rationale**: Redis is in the approved stack; these are the exact use cases Redis excels at with minimal code.

---

## Resolved: Technology Matrix

| Concern | Choice | Version | Justification |
|---|---|---|---|
| Framework | Next.js | 16.2.4 | Constitution baseline |
| React | React + React DOM | 19.2.4 | Constitution baseline |
| Styling | Tailwind CSS | ^4 | Constitution baseline |
| Language | TypeScript | ^5 | Constitution baseline |
| Component Library | shadcn/ui | latest | Accessible headless components; saves weeks of UI work |
| Animations | Framer Motion | ^11 | Page transitions and micro-animations are core UX requirement (FR-044) |
| Client State | Zustand | ^5 | See Decision 3 |
| Server State | TanStack Query v5 | ^5 | See Decision 4 |
| Forms | React Hook Form + Zod | ^7 / ^3 | Complex form validation with real-time client feedback |
| Dates | date-fns | ^4 | Streak arithmetic, heatmap generation, challenge date management |
| Auth | Auth.js (next-auth) v5 | ^5 | See Decision 1 |
| ORM | Prisma | ^6 | Type-safe PostgreSQL access with migration management |
| Primary DB | PostgreSQL | 16 | Structured relational data; FTS support |
| Chat DB | MongoDB | 7 | Flexible schema for variable-length AI chat sessions |
| Cache | Redis | 7 | Rate limiting, session cache, search cache |
| AI | OpenAI SDK | ^4 | See Decision 2 |
| Code Editor | Monaco Editor | ^4 | See Decision 5 |
| Markdown | react-markdown + remark-gfm + rehype-highlight | latest | See Decision 6 |
| Diagrams | Mermaid.js | ^11 | See Decision 7 |
| Whiteboard | Excalidraw | ^0.17 | See Decision 8 |
| Icons | Lucide React | latest | shadcn/ui default icon set |
| UI Primitives | Radix UI | via shadcn | Accessible headless primitives |
| Utilities | clsx + tailwind-merge + class-variance-authority | latest | shadcn/ui required utilities |
| MongoDB client | Mongoose | ^8 | ODM for chat session schema validation |
| Redis client | ioredis | ^5 | Full-featured Redis client with TypeScript support |
| Cron | node-cron (via Vercel Cron) | latest | Daily challenge rotation |
