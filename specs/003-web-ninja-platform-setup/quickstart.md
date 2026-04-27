# Quickstart: Web Ninja Development Setup

**Branch**: `003-web-ninja-platform-setup`  
**Time to first run**: ~15 minutes

---

## Prerequisites

| Tool | Version | Install |
|---|---|---|
| Node.js | ≥20 LTS | [nodejs.org](https://nodejs.org) |
| pnpm | ≥9 | `npm install -g pnpm` |
| Docker | ≥24 | [docker.com](https://docker.com) (for local PostgreSQL + Redis + MongoDB) |
| Git | any | pre-installed |

---

## 1. Clone and Install

```bash
git clone <repo-url>
cd web-ninja
pnpm install
```

---

## 2. Start Local Services (Docker)

Create `docker-compose.yml` in the repo root:

```yaml
services:
  postgres:
    image: postgres:16
    environment:
      POSTGRES_DB: webninja
      POSTGRES_USER: webninja
      POSTGRES_PASSWORD: webninja_dev
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  mongodb:
    image: mongo:7
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"

volumes:
  postgres_data:
  mongo_data:
```

```bash
docker compose up -d
```

---

## 3. Environment Variables

Copy `.env.example` to `.env.local`:

```bash
cp .env.example .env.local
```

Edit `.env.local`:

```bash
# ── Database ──────────────────────────────────────────────
DATABASE_URL="postgresql://webninja:webninja_dev@localhost:5432/webninja"
MONGODB_URI="mongodb://localhost:27017/webninja"
REDIS_URL="redis://localhost:6379"

# ── Auth.js v5 ─────────────────────────────────────────────
AUTH_SECRET="<generate with: openssl rand -base64 32>"
AUTH_URL="http://localhost:3000"

# Google OAuth (optional for local dev — skip to use email/password only)
AUTH_GOOGLE_ID=""
AUTH_GOOGLE_SECRET=""

# ── OpenAI ────────────────────────────────────────────────
OPENAI_API_KEY="sk-..."
OPENAI_MODEL_FAST="gpt-4o-mini"
OPENAI_MODEL_SMART="gpt-4o"

# ── Cron ──────────────────────────────────────────────────
CRON_SECRET="<generate with: openssl rand -base64 32>"

# ── App ───────────────────────────────────────────────────
NEXT_PUBLIC_APP_URL="http://localhost:3000"
NEXT_PUBLIC_APP_NAME="Web Ninja"
```

---

## 4. Install shadcn/ui

```bash
pnpm dlx shadcn@latest init
```

When prompted:
- Style: **New York**
- Base color: **Slate**
- CSS variables: **Yes**
- `components.json` will be created at the root.

Then add the required components:

```bash
pnpm dlx shadcn@latest add button card dialog input badge skeleton progress tooltip dropdown-menu sheet command scroll-area tabs separator avatar
```

---

## 5. Install Additional Dependencies

```bash
pnpm add next-auth@beta @auth/prisma-adapter \
  prisma @prisma/client \
  mongoose ioredis \
  openai \
  @tanstack/react-query \
  zustand \
  react-hook-form @hookform/resolvers zod \
  framer-motion \
  date-fns \
  react-markdown remark-gfm rehype-highlight \
  mermaid \
  node-cron \
  bcryptjs \
  lucide-react

pnpm add -D @types/bcryptjs @types/node-cron prisma
```

Install Monaco Editor and Excalidraw (large, lazy-loaded only):

```bash
pnpm add @monaco-editor/react @excalidraw/excalidraw
```

---

## 6. Initialize Prisma

```bash
pnpm prisma init --datasource-provider postgresql
```

Replace `prisma/schema.prisma` with the schema from [data-model.md](./data-model.md).

Run initial migration:

```bash
pnpm prisma migrate dev --name init
```

Apply the FTS triggers (copy SQL from data-model.md "Full-Text Search Setup"):

```bash
pnpm prisma db execute --file prisma/migrations/fts-triggers.sql
```

Generate Prisma client:

```bash
pnpm prisma generate
```

---

## 7. Seed the Database

```bash
pnpm prisma db seed
```

The seed script (`prisma/seed.ts`) inserts:
- 9 categories, ~54 modules, ~216 lessons (skeleton content to start)
- Sample interview questions (3–5 per lesson for initial dev, expanded to 25 before launch)
- 15 badges
- 90 daily challenges
- 6 company guides
- 10 system design walkthroughs

---

## 8. Run Development Server

```bash
pnpm dev
```

Open [http://localhost:3000](http://localhost:3000).

---

## Development URLs

| Path | Description |
|---|---|
| `/` | Public landing page |
| `/login` | Sign in |
| `/signup` | Create account |
| `/dashboard` | Main app dashboard (auth required) |
| `/learn` | Learning roadmap |
| `/learn/[category]/[module]/[lesson]` | Lesson viewer |
| `/questions/[topic]` | Flashcard deck |
| `/mock-interviews` | Mock interview picker |
| `/analytics` | Progress dashboard |
| `/notes` | Bookmarks and notes |
| `/challenges` | Daily challenge |
| `/settings` | Profile and preferences |

---

## Prisma Studio (DB Inspector)

```bash
pnpm prisma studio
```

Opens at [http://localhost:5555](http://localhost:5555).

---

## Common Development Scripts

```bash
pnpm dev                    # Start dev server (Next.js HMR)
pnpm build                  # Production build
pnpm start                  # Start production server
pnpm lint                   # ESLint
pnpm prisma migrate dev     # Create and apply a new migration
pnpm prisma generate        # Regenerate Prisma client after schema changes
pnpm prisma db seed         # Re-seed the database
pnpm prisma studio          # Open Prisma Studio
```

---

## Production Environment Variables

For Vercel deployment, add these in the Vercel project settings:

```bash
DATABASE_URL          # Neon PostgreSQL connection string
MONGODB_URI           # MongoDB Atlas connection string  
REDIS_URL             # Upstash Redis REST URL
AUTH_SECRET           # Same as local, keep secret
AUTH_URL              # https://web-ninja.vercel.app
AUTH_GOOGLE_ID        # Google OAuth client ID
AUTH_GOOGLE_SECRET    # Google OAuth client secret
OPENAI_API_KEY        # OpenAI API key
OPENAI_MODEL_FAST     # gpt-4o-mini
OPENAI_MODEL_SMART    # gpt-4o
CRON_SECRET           # Vercel Cron authorization header value
NEXT_PUBLIC_APP_URL   # https://web-ninja.vercel.app
NEXT_PUBLIC_APP_NAME  # Web Ninja
```

---

## Vercel Cron Configuration

Add to `vercel.json`:

```json
{
  "crons": [
    {
      "path": "/api/cron/rotate-challenge",
      "schedule": "0 0 * * *"
    }
  ]
}
```

---

## Adding New shadcn/ui Components

```bash
pnpm dlx shadcn@latest add <component-name>
```

Components are generated into `src/components/ui/`.

---

## Deployment Checklist

- [ ] Run `pnpm prisma migrate deploy` (not `migrate dev`) in CI for production migrations.
- [ ] Verify `CRON_SECRET` is set in Vercel environment.
- [ ] Confirm `AUTH_URL` matches the production domain.
- [ ] Seed production database once via Vercel CLI: `vercel env pull && pnpm prisma db seed`.
- [ ] Verify Redis connection (Upstash Redis REST URL format differs from local `redis://`).
- [ ] Test Google OAuth callback URL is registered in Google Cloud Console: `https://<domain>/api/auth/callback/google`.
