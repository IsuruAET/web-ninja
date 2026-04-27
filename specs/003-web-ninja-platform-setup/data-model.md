# Data Model: Web Ninja Platform

**Phase**: 1 — Design  
**Branch**: `003-web-ninja-platform-setup`  
**Spec**: [spec.md](./spec.md) | **Research**: [research.md](./research.md)

---

## Overview

Two databases:
- **PostgreSQL** (via Prisma): All structured relational data — users, content, progress, challenges, mock interviews.
- **MongoDB** (via Mongoose): AI chat sessions — flexible document model for variable-length conversation histories.

**Redis** is used as a cache/ephemeral layer, not a persistent store — no schema definition required.

---

## PostgreSQL Schema (`prisma/schema.prisma`)

```prisma
// prisma/schema.prisma

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// ─────────────────────────────────────────────
// AUTH (Auth.js v5 required models)
// ─────────────────────────────────────────────

model User {
  id            String    @id @default(cuid())
  email         String    @unique
  name          String?
  avatar        String?
  passwordHash  String?
  emailVerified DateTime?
  role          Role      @default(USER)
  theme         Theme     @default(DARK)
  notifyStreak  Boolean   @default(true)
  notifyWeekly  Boolean   @default(true)
  learningGoal  String?   // e.g. "Land a senior frontend role at FAANG"
  deletedAt     DateTime? // Soft delete — GDPR account deletion (FR-047)

  accounts     Account[]
  sessions     Session[]

  progress     UserProgress[]
  questionCompletions QuestionCompletion[]
  bookmarks    Bookmark[]
  notes        Note[]
  streak       StreakRecord?
  userBadges   UserBadge[]
  mockInterviews MockInterview[]
  challengeCompletions DailyChallengeCompletion[]

  createdAt    DateTime  @default(now())
  updatedAt    DateTime  @updatedAt

  @@index([email])
}

enum Role {
  USER
  ADMIN
}

enum Theme {
  DARK
  LIGHT
}

model Account {
  id                String  @id @default(cuid())
  userId            String
  type              String
  provider          String
  providerAccountId String
  refresh_token     String? @db.Text
  access_token      String? @db.Text
  expires_at        Int?
  token_type        String?
  scope             String?
  id_token          String? @db.Text
  session_state     String?

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@unique([provider, providerAccountId])
}

model Session {
  id           String   @id @default(cuid())
  sessionToken String   @unique
  userId       String
  expires      DateTime

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
}

model VerificationToken {
  identifier String
  token      String   @unique
  expires    DateTime

  @@unique([identifier, token])
}

// ─────────────────────────────────────────────
// CONTENT HIERARCHY
// ─────────────────────────────────────────────

// FR-007: Categories — Foundation, Frontend, Backend, Databases, Architecture,
//          DevOps, Testing, AI Engineering, Soft Skills
model Category {
  id          String   @id @default(cuid())
  slug        String   @unique  // e.g. "frontend-engineering"
  name        String             // e.g. "Frontend Engineering"
  description String?
  icon        String?            // Lucide icon name
  order       Int                // Display order in roadmap/sidebar

  modules     Module[]

  @@index([order])
}

// FR-008: Modules within a category (e.g. "JavaScript Deep Dive" in Frontend)
model Module {
  id          String     @id @default(cuid())
  slug        String     @unique  // e.g. "javascript-deep-dive"
  title       String
  description String?
  order       Int
  difficulty  Difficulty @default(BEGINNER)
  categoryId  String

  category Category @relation(fields: [categoryId], references: [id])
  lessons  Lesson[]

  @@index([categoryId, order])
}

enum Difficulty {
  BEGINNER
  INTERMEDIATE
  ADVANCED
}

// FR-009, FR-010, FR-011: Lessons within a module
model Lesson {
  id               String     @id @default(cuid())
  slug             String     @unique  // e.g. "event-loop-deep-dive"
  title            String
  order            Int
  difficulty       Difficulty
  estimatedMinutes Int        @default(15)
  isPreview        Boolean    @default(false) // FR-000a: first lesson per category public

  moduleId String
  module   Module @relation(fields: [moduleId], references: [id])

  sections         LessonSection[]
  questions        InterviewQuestion[]
  bookmarks        Bookmark[]
  notes            Note[]
  userProgress     UserProgress[]

  // PostgreSQL full-text search vector (FR-006)
  // Updated via DB trigger or manual raw query on seed/update
  searchVector Unsupported("tsvector")?

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  @@index([moduleId, order])
  @@index([slug])
}

// Structured lesson content — each section maps to a spec-defined lesson section type (FR-009)
model LessonSection {
  id       String      @id @default(cuid())
  type     SectionType
  order    Int
  // Flexible JSON payload per type:
  //   EXPLANATION:         { markdown: string }
  //   REAL_WORLD_EXAMPLE:  { markdown: string }
  //   CODE_SNIPPET:        { language: string, code: string, caption?: string }
  //   DIAGRAM:             { source: string (Mermaid syntax), caption?: string }
  //   COMMON_MISTAKES:     { items: string[] }
  //   PERFORMANCE_TIPS:    { items: string[] }
  //   INTERVIEW_TIPS:      { markdown: string }
  //   HOW_INTERVIEWERS_ASK: { examples: string[] }
  content  Json

  lessonId String
  lesson   Lesson @relation(fields: [lessonId], references: [id], onDelete: Cascade)

  @@index([lessonId, order])
}

enum SectionType {
  EXPLANATION
  REAL_WORLD_EXAMPLE
  CODE_SNIPPET
  DIAGRAM
  COMMON_MISTAKES
  PERFORMANCE_TIPS
  INTERVIEW_TIPS
  HOW_INTERVIEWERS_ASK
}

// FR-013–FR-018: Interview questions per lesson (min 25 per lesson)
model InterviewQuestion {
  id         String       @id @default(cuid())
  question   String       @db.Text
  answer     String       @db.Text
  difficulty Difficulty
  type       QuestionType
  order      Int          // Display order within the lesson

  lessonId String
  lesson   Lesson @relation(fields: [lessonId], references: [id], onDelete: Cascade)

  completions QuestionCompletion[]

  // PostgreSQL FTS vector for cross-topic search
  searchVector Unsupported("tsvector")?

  @@index([lessonId, difficulty])
  @@index([lessonId, type])
}

enum QuestionType {
  FAANG        // FAANG-company-style question
  SCENARIO     // Scenario-based question
  CONCEPTUAL   // Pure concept/definition question
}

// ─────────────────────────────────────────────
// USER PROGRESS & GAMIFICATION
// ─────────────────────────────────────────────

// FR-011, FR-028: Tracks lesson completion per user
model UserProgress {
  id          String   @id @default(cuid())
  userId      String
  lessonId    String
  completedAt DateTime @default(now())
  timeSpentMs Int      @default(0)

  user   User   @relation(fields: [userId], references: [id], onDelete: Cascade)
  lesson Lesson @relation(fields: [lessonId], references: [id], onDelete: Cascade)

  @@unique([userId, lessonId])
  @@index([userId])
}

// FR-017: Tracks individual question completion per user
model QuestionCompletion {
  id          String   @id @default(cuid())
  userId      String
  questionId  String
  completedAt DateTime @default(now())

  question InterviewQuestion @relation(fields: [questionId], references: [id], onDelete: Cascade)

  @@unique([userId, questionId])
  @@index([userId])
}

// FR-027: Daily study streak tracking
model StreakRecord {
  id             String    @id @default(cuid())
  userId         String    @unique
  currentStreak  Int       @default(0)
  longestStreak  Int       @default(0)
  lastActiveDate DateTime? // Date (not datetime) of last qualifying activity
  totalDaysActive Int      @default(0)

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)
}

// FR-029: Achievement badges
model Badge {
  id          String         @id @default(cuid())
  slug        String         @unique
  name        String
  description String
  icon        String         // SVG path or Lucide icon name
  condition   BadgeCondition
  threshold   Int            // e.g. 7 for "7-day streak"

  userBadges UserBadge[]
}

enum BadgeCondition {
  LESSONS_COMPLETED    // threshold = number of lessons
  STREAK_DAYS          // threshold = consecutive days
  QUESTIONS_ANSWERED   // threshold = number of questions
  CATEGORIES_COMPLETED // threshold = number of categories
  CHALLENGES_COMPLETED // threshold = number of challenges
}

model UserBadge {
  id       String   @id @default(cuid())
  userId   String
  badgeId  String
  earnedAt DateTime @default(now())

  badge Badge @relation(fields: [badgeId], references: [id])

  @@unique([userId, badgeId])
  @@index([userId])
}

// ─────────────────────────────────────────────
// BOOKMARKS & NOTES
// ─────────────────────────────────────────────

// FR-031, FR-033: User lesson bookmarks
model Bookmark {
  id        String   @id @default(cuid())
  userId    String
  lessonId  String
  createdAt DateTime @default(now())

  user   User   @relation(fields: [userId], references: [id], onDelete: Cascade)
  lesson Lesson @relation(fields: [lessonId], references: [id], onDelete: Cascade)

  @@unique([userId, lessonId])
  @@index([userId])
}

// FR-032, FR-034: User personal notes on lessons
model Note {
  id       String @id @default(cuid())
  userId   String
  lessonId String
  content  String @db.Text // Max enforced at service layer (~5000 chars)

  // PostgreSQL FTS for note search (FR-034)
  searchVector Unsupported("tsvector")?

  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt

  user   User   @relation(fields: [userId], references: [id], onDelete: Cascade)
  lesson Lesson @relation(fields: [lessonId], references: [id], onDelete: Cascade)

  @@unique([userId, lessonId]) // One note per user per lesson
  @@index([userId])
}

// ─────────────────────────────────────────────
// DAILY CHALLENGES
// ─────────────────────────────────────────────

// FR-035–FR-037: Globally curated daily challenge
model DailyChallenge {
  id          String        @id @default(cuid())
  date        DateTime      @unique @db.Date // The calendar date this challenge is active
  type        ChallengeType
  title       String
  description String        @db.Text
  difficulty  Difficulty
  // Flexible content payload per type:
  //   CODING:       { prompt: string, starterCode?: string, language: string }
  //   CONCEPT:      { question: string, answer: string, hints: string[] }
  //   SYSTEM_DESIGN: { scenario: string, requirements: string[], hints: string[] }
  content     Json

  completions DailyChallengeCompletion[]

  createdAt DateTime @default(now())

  @@index([date])
}

enum ChallengeType {
  CODING
  CONCEPT
  SYSTEM_DESIGN
}

model DailyChallengeCompletion {
  id          String   @id @default(cuid())
  userId      String
  challengeId String
  completedAt DateTime @default(now())

  user      User           @relation(fields: [userId], references: [id], onDelete: Cascade)
  challenge DailyChallenge @relation(fields: [challengeId], references: [id], onDelete: Cascade)

  @@unique([userId, challengeId])
  @@index([userId])
}

// ─────────────────────────────────────────────
// MOCK INTERVIEWS
// ─────────────────────────────────────────────

// FR-023–FR-026: Mock interview sessions
model MockInterview {
  id        String            @id @default(cuid())
  userId    String
  type      MockInterviewType
  prompt    String            @db.Text  // The question/scenario presented
  language  String?           // "javascript" | "typescript" | "python" (for CODING type)
  response  String?           @db.Text  // User's submitted code or text response
  feedback  Json?             // AI feedback: { correctness, efficiency, approach, summary }
  timerSeconds Int            @default(1800) // 30 min default

  startedAt   DateTime  @default(now())
  completedAt DateTime?

  user User @relation(fields: [userId], references: [id], onDelete: Cascade)

  @@index([userId])
}

enum MockInterviewType {
  CODING
  SYSTEM_DESIGN
  BEHAVIORAL
}

// ─────────────────────────────────────────────
// CONTENT EXTRAS
// ─────────────────────────────────────────────

// FR-038: System design interview walkthroughs
model SystemDesignGuide {
  id          String   @id @default(cuid())
  slug        String   @unique
  title       String
  description String?
  // Array of sections: { type: "text"|"diagram"|"steps", content: ... }
  sections    Json
  tags        String[] // e.g. ["scalability", "distributed-systems", "caching"]
  order       Int

  createdAt DateTime @default(now())

  @@index([order])
}

// FR-040: Company-specific FAANG interview prep guides
model CompanyGuide {
  id          String   @id @default(cuid())
  slug        String   @unique // e.g. "google", "meta", "amazon"
  name        String           // e.g. "Google"
  description String?
  logoUrl     String?
  // Array of question sets: [{ topic: string, questions: QuestionSummary[] }]
  questionSets Json

  createdAt DateTime @default(now())
}
```

---

## Entity Relationship Summary

```
Category (1) ──< Module (many)
Module   (1) ──< Lesson (many)
Lesson   (1) ──< LessonSection (many)
Lesson   (1) ──< InterviewQuestion (many)

User (1) ──< UserProgress (many) >── Lesson
User (1) ──< QuestionCompletion (many) >── InterviewQuestion
User (1) ──  StreakRecord (1)
User (1) ──< UserBadge (many) >── Badge
User (1) ──< Bookmark (many) >── Lesson
User (1) ──< Note (many) >── Lesson
User (1) ──< MockInterview (many)
User (1) ──< DailyChallengeCompletion (many) >── DailyChallenge
User (1) ──< Account (many)          [Auth.js]
User (1) ──< Session (many)          [Auth.js]
```

---

## MongoDB Schema (Mongoose)

**Collection**: `chat_sessions`

Chat sessions are stored in MongoDB because:
- Conversation histories have a variable, potentially large number of messages.
- Schema changes frequently as AI features evolve (e.g. adding tool call records, citations, token counts).
- No relational joins are needed — always fetched as a single document by session ID.

```typescript
// src/lib/db/schemas/chat-session.schema.ts

import mongoose, { Schema, Document, Model } from 'mongoose'

interface ChatMessage {
  role: 'user' | 'assistant' | 'system'
  content: string
  timestamp: Date
  // Present when the assistant response contains a code block
  codeBlocks?: Array<{
    language: string
    code: string
  }>
  // Token usage for cost monitoring (assistant messages only)
  tokenCount?: number
}

export interface ChatSessionDocument extends Document {
  userId: string          // References PostgreSQL User.id
  sessionId: string       // Unique session identifier (cuid)
  lessonContext?: string  // Lesson slug for context-aware Sensei responses
  categoryContext?: string // Category slug for broader context
  messages: ChatMessage[]
  isActive: boolean
  totalTokensUsed: number
  createdAt: Date
  updatedAt: Date
}

const ChatMessageSchema = new Schema<ChatMessage>(
  {
    role: { type: String, enum: ['user', 'assistant', 'system'], required: true },
    content: { type: String, required: true },
    timestamp: { type: Date, default: Date.now },
    codeBlocks: [
      {
        language: String,
        code: String,
      },
    ],
    tokenCount: Number,
  },
  { _id: false }
)

const ChatSessionSchema = new Schema<ChatSessionDocument>(
  {
    userId: { type: String, required: true, index: true },
    sessionId: { type: String, required: true, unique: true },
    lessonContext: String,
    categoryContext: String,
    messages: [ChatMessageSchema],
    isActive: { type: Boolean, default: true },
    totalTokensUsed: { type: Number, default: 0 },
  },
  {
    timestamps: true, // adds createdAt and updatedAt
  }
)

// Compound index: efficiently fetch active session for a user in a lesson context
ChatSessionSchema.index({ userId: 1, lessonContext: 1, isActive: 1 })

export const ChatSession: Model<ChatSessionDocument> =
  mongoose.models.ChatSession ??
  mongoose.model<ChatSessionDocument>('ChatSession', ChatSessionSchema)
```

---

## Redis Key Schema

Redis is used for ephemeral/cache data only. All keys are prefixed and include a separator `:`.

| Key Pattern | Type | TTL | Purpose |
|---|---|---|---|
| `rate:ai:{userId}` | String (counter) | 60s | AI endpoint rate limit (20 req/min per user) |
| `rate:signup:{ip}` | String (counter) | 60s | Signup rate limit (5 req/min per IP) |
| `session:{sessionToken}` | Hash | 24h | Cached Auth.js session data |
| `challenge:today` | String (JSON) | Until midnight UTC | Cached today's DailyChallenge |
| `search:{queryHash}` | String (JSON) | 5min | Cached search results |
| `streak:{userId}` | String (JSON) | 1h | Cached StreakRecord |
| `progress:{userId}:summary` | String (JSON) | 5min | Cached category progress percentages |

---

## Full-Text Search Setup

PostgreSQL `tsvector` columns on `Lesson`, `InterviewQuestion`, and `Note` models.

**Trigger SQL** (applied via Prisma raw migration):

```sql
-- Lesson search vector: title + module title (weighted)
CREATE OR REPLACE FUNCTION update_lesson_search_vector()
RETURNS TRIGGER AS $$
BEGIN
  NEW."searchVector" :=
    setweight(to_tsvector('english', coalesce(NEW.title, '')), 'A') ||
    setweight(to_tsvector('english', coalesce(
      (SELECT title FROM "Module" WHERE id = NEW."moduleId"), ''
    )), 'B');
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER lesson_search_vector_update
BEFORE INSERT OR UPDATE ON "Lesson"
FOR EACH ROW EXECUTE FUNCTION update_lesson_search_vector();

CREATE INDEX lesson_search_idx ON "Lesson" USING GIN("searchVector");

-- InterviewQuestion search vector: question text
CREATE OR REPLACE FUNCTION update_question_search_vector()
RETURNS TRIGGER AS $$
BEGIN
  NEW."searchVector" :=
    to_tsvector('english', coalesce(NEW.question, ''));
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER question_search_vector_update
BEFORE INSERT OR UPDATE ON "InterviewQuestion"
FOR EACH ROW EXECUTE FUNCTION update_question_search_vector();

CREATE INDEX question_search_idx ON "InterviewQuestion" USING GIN("searchVector");

-- Note search vector: note content
CREATE OR REPLACE FUNCTION update_note_search_vector()
RETURNS TRIGGER AS $$
BEGIN
  NEW."searchVector" :=
    to_tsvector('english', coalesce(NEW.content, ''));
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER note_search_vector_update
BEFORE INSERT OR UPDATE ON "Note"
FOR EACH ROW EXECUTE FUNCTION update_note_search_vector();

CREATE INDEX note_search_idx ON "Note" USING GIN("searchVector");
```

---

## Seed Data Structure

`prisma/seed.ts` seeds the following at launch:

```
9 Categories × avg 6 modules × avg 4 lessons × 25 questions
= ~54 modules, ~216 lessons, ~5,400 questions (Phase 1 content)

Categories (in order):
1. foundations          → 8 modules
2. frontend-engineering → 12 modules
3. backend-engineering  → 10 modules
4. databases-data       → 8 modules
5. architecture-system-design → 8 modules
6. devops-cloud         → 7 modules
7. testing-quality      → 4 modules
8. ai-engineering       → 7 modules
9. soft-skills-leadership → 6 modules

System Design Guides: 10 walkthroughs (URL shortener, chat app, Netflix, etc.)
Company Guides: 6 (Google, Meta, Amazon, Netflix, Apple, Microsoft)
Badges: 15 achievements
Daily Challenges: 90 days pre-loaded
```
