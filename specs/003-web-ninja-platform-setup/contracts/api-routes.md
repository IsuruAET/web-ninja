# API Route Contracts

**Type**: Next.js Route Handlers (`app/api/**`)  
**Auth**: All routes except `/api/auth/**` and `/api/search` require a valid Auth.js session cookie.  
**Validation**: Every route validates its request body with a Zod schema before processing.  
**Error format**: All error responses: `{ error: string, code?: string }`  
**Success format**: All success responses: `{ data: T }` or `T` directly for resource endpoints.

---

## Authentication Routes

> Handled by Auth.js v5. The `[...nextauth]` catch-all handles all OAuth callbacks, signin, signout, and CSRF token endpoints automatically.

### `GET|POST /api/auth/[...nextauth]`
Auth.js handler — sign in (credentials + Google OAuth), sign out, session, CSRF.  
See [Auth.js v5 docs](https://authjs.dev/getting-started/installation?framework=Next.js) for full route list.

---

## User Routes

### `PATCH /api/user`
Update the authenticated user's profile.

**Auth required**: Yes

**Request body**:
```json
{
  "name": "string (optional)",
  "avatar": "string (optional, URL)",
  "theme": "DARK | LIGHT (optional)",
  "notifyStreak": "boolean (optional)",
  "notifyWeekly": "boolean (optional)",
  "learningGoal": "string (optional, max 200 chars)"
}
```

**Response `200`**:
```json
{
  "id": "string",
  "name": "string",
  "email": "string",
  "avatar": "string | null",
  "theme": "DARK | LIGHT",
  "learningGoal": "string | null"
}
```

---

### `DELETE /api/user`
Permanently delete the authenticated user's account and all personal data (FR-047).

**Auth required**: Yes

**Request body**:
```json
{ "confirmEmail": "string" }
```

**Behavior**:
1. Verifies `confirmEmail` matches `session.user.email`.
2. Cascading Prisma delete on `User` (removes all progress, notes, bookmarks, mock interviews, completions via cascade).
3. Deletes all `ChatSession` documents in MongoDB matching `userId`.
4. Evicts all Redis keys prefixed with `session:{userId}` and `streak:{userId}`.

**Response `200`**: `{ "deleted": true }`

---

## Progress Routes

### `GET /api/progress`
Get the authenticated user's overall progress summary.

**Auth required**: Yes

**Query params**: None

**Response `200`**:
```json
{
  "completedLessons": "string[]",
  "completedQuestions": "string[]",
  "categoryProgress": [
    {
      "categoryId": "string",
      "categorySlug": "string",
      "categoryName": "string",
      "totalLessons": "number",
      "completedLessons": "number",
      "percentage": "number"
    }
  ],
  "overallPercentage": "number",
  "streak": {
    "current": "number",
    "longest": "number",
    "lastActiveDate": "string | null"
  },
  "badges": ["string"],
  "weeklyActivity": [
    { "date": "string (YYYY-MM-DD)", "lessonsCompleted": "number", "questionsAnswered": "number" }
  ]
}
```

---

### `POST /api/progress`
Mark a lesson as complete and update the user's streak.

**Auth required**: Yes

**Request body**:
```json
{
  "lessonId": "string",
  "timeSpentMs": "number"
}
```

**Behavior**:
1. Upserts `UserProgress` record.
2. Calls `streak.service.ts` to update `StreakRecord` (increment if same/next day, reset if gap).
3. Evaluates badge conditions and inserts new `UserBadge` records if thresholds crossed.
4. Invalidates Redis `streak:{userId}` and `progress:{userId}:summary` keys.

**Response `201`**:
```json
{
  "lessonId": "string",
  "completedAt": "string (ISO 8601)",
  "streakCurrent": "number",
  "newBadges": ["string"]
}
```

---

## Questions Routes

### `GET /api/questions`
Get interview questions for a lesson, optionally filtered.

**Auth required**: Yes

**Query params**:
- `lessonId` (required): `string`
- `difficulty` (optional): `BEGINNER | INTERMEDIATE | ADVANCED`
- `type` (optional): `FAANG | SCENARIO | CONCEPTUAL`

**Response `200`**:
```json
{
  "questions": [
    {
      "id": "string",
      "question": "string",
      "answer": "string",
      "difficulty": "BEGINNER | INTERMEDIATE | ADVANCED",
      "type": "FAANG | SCENARIO | CONCEPTUAL",
      "order": "number",
      "completed": "boolean"
    }
  ]
}
```

---

### `PATCH /api/questions/[id]`
Toggle a question's completion status for the authenticated user.

**Auth required**: Yes

**Request body**:
```json
{ "completed": "boolean" }
```

**Behavior**:
- `completed: true` → upsert `QuestionCompletion` record.
- `completed: false` → delete `QuestionCompletion` record if it exists.

**Response `200`**:
```json
{ "id": "string", "completed": "boolean" }
```

---

## Bookmarks Routes

### `GET /api/bookmarks`
Get all bookmarks for the authenticated user.

**Auth required**: Yes

**Response `200`**:
```json
{
  "bookmarks": [
    {
      "id": "string",
      "lessonId": "string",
      "lessonSlug": "string",
      "lessonTitle": "string",
      "moduleTitle": "string",
      "categorySlug": "string",
      "createdAt": "string (ISO 8601)"
    }
  ]
}
```

---

### `POST /api/bookmarks`
Bookmark a lesson.

**Auth required**: Yes

**Request body**:
```json
{ "lessonId": "string" }
```

**Response `201`**:
```json
{ "id": "string", "lessonId": "string", "createdAt": "string" }
```

---

### `DELETE /api/bookmarks/[id]`
Remove a bookmark.

**Auth required**: Yes

**Response `200`**: `{ "deleted": true }`

---

## Notes Routes

### `GET /api/notes`
Get all notes for the authenticated user.

**Auth required**: Yes

**Query params**:
- `lessonId` (optional): Filter to a specific lesson's note.

**Response `200`**:
```json
{
  "notes": [
    {
      "id": "string",
      "lessonId": "string",
      "lessonSlug": "string",
      "lessonTitle": "string",
      "content": "string",
      "updatedAt": "string"
    }
  ]
}
```

---

### `POST /api/notes`
Create a note for a lesson.

**Auth required**: Yes

**Request body**:
```json
{
  "lessonId": "string",
  "content": "string (max 5000 chars)"
}
```

**Response `201`**:
```json
{ "id": "string", "lessonId": "string", "content": "string", "updatedAt": "string" }
```

---

### `PUT /api/notes/[id]`
Update an existing note.

**Auth required**: Yes

**Request body**:
```json
{ "content": "string (max 5000 chars)" }
```

**Response `200`**:
```json
{ "id": "string", "content": "string", "updatedAt": "string" }
```

---

### `DELETE /api/notes/[id]`
Delete a note.

**Auth required**: Yes

**Response `200`**: `{ "deleted": true }`

---

## Daily Challenge Routes

### `GET /api/challenges/today`
Get today's daily challenge and the user's completion status.

**Auth required**: Yes

**Response `200`**:
```json
{
  "challenge": {
    "id": "string",
    "date": "string (YYYY-MM-DD)",
    "type": "CODING | CONCEPT | SYSTEM_DESIGN",
    "title": "string",
    "description": "string",
    "difficulty": "BEGINNER | INTERMEDIATE | ADVANCED",
    "content": "object"
  },
  "completed": "boolean",
  "completedAt": "string | null"
}
```

---

### `POST /api/challenges/complete`
Mark today's daily challenge as complete.

**Auth required**: Yes

**Request body**:
```json
{ "challengeId": "string" }
```

**Behavior**:
1. Upserts `DailyChallengeCompletion`.
2. Calls streak update (same as lesson completion path).
3. Evaluates challenge-related badge conditions.

**Response `201`**:
```json
{
  "completedAt": "string",
  "streakCurrent": "number",
  "newBadges": ["string"]
}
```

---

## Mock Interview Routes

### `POST /api/mock-interviews`
Create a new mock interview session.

**Auth required**: Yes

**Request body**:
```json
{
  "type": "CODING | SYSTEM_DESIGN | BEHAVIORAL",
  "language": "javascript | typescript | python (required if type=CODING)"
}
```

**Behavior**: Selects a random prompt appropriate for the type and language, creates a `MockInterview` record with `startedAt`.

**Response `201`**:
```json
{
  "id": "string",
  "type": "string",
  "prompt": "string",
  "language": "string | null",
  "timerSeconds": "number",
  "startedAt": "string"
}
```

---

### `GET /api/mock-interviews/[id]`
Get a mock interview session (including feedback if completed).

**Auth required**: Yes

**Response `200`**:
```json
{
  "id": "string",
  "type": "string",
  "prompt": "string",
  "response": "string | null",
  "feedback": "object | null",
  "startedAt": "string",
  "completedAt": "string | null"
}
```

---

### `PATCH /api/mock-interviews/[id]`
Submit a mock interview response (triggers AI feedback generation).

**Auth required**: Yes

**Request body**:
```json
{ "response": "string" }
```

**Behavior**:
1. Saves `response` to `MockInterview`.
2. Calls `ai.service.ts` → `openai.chat.completions.create()` (non-streaming; feedback generation is a one-shot structured response).
3. Saves structured feedback JSON and sets `completedAt`.

**Response `200`**:
```json
{
  "feedback": {
    "correctness": "string",
    "efficiency": "string",
    "approach": "string",
    "summary": "string",
    "score": "number (0-10)"
  },
  "completedAt": "string"
}
```

---

## AI Routes

### `POST /api/ai/chat`
Stream a Sensei AI response.

**Auth required**: Yes  
**Rate limit**: 20 requests/minute per user (Redis sliding window)

**Request body**:
```json
{
  "sessionId": "string (existing) | null (start new session)",
  "message": "string (max 2000 chars)",
  "lessonContext": "string (lesson slug, optional)",
  "categoryContext": "string (category slug, optional)"
}
```

**Behavior**:
1. Load or create `ChatSession` in MongoDB.
2. Append user message.
3. Build messages array: system prompt + lesson context + last 10 messages (rolling window to control tokens).
4. Stream `openai.chat.completions.stream()` response.
5. On stream end, save assistant message + token count to MongoDB.

**Response `200`**: `text/event-stream` (Server-Sent Events)  
Each event: `data: {"token": "string"}`  
Final event: `data: {"done": true, "sessionId": "string"}`

---

### `POST /api/ai/quiz`
Generate AI quiz questions for a topic.

**Auth required**: Yes  
**Rate limit**: 10 requests/minute per user

**Request body**:
```json
{
  "topic": "string",
  "difficulty": "BEGINNER | INTERMEDIATE | ADVANCED",
  "count": "number (1-10, default 5)"
}
```

**Response `200`**:
```json
{
  "questions": [
    {
      "question": "string",
      "answer": "string",
      "explanation": "string"
    }
  ]
}
```

---

### `POST /api/ai/resume`
Analyze resume content and return structured improvement feedback.

**Auth required**: Yes  
**Rate limit**: 5 requests/minute per user

**Request body**:
```json
{ "resumeText": "string (max 8000 chars, plain text)" }
```

**Response `200`**:
```json
{
  "feedback": {
    "summary": "string",
    "strengths": ["string"],
    "improvements": [
      { "section": "string", "issue": "string", "suggestion": "string" }
    ],
    "keywords": ["string"],
    "score": "number (0-100)"
  }
}
```

---

### `POST /api/ai/interview-feedback`
Used internally by `PATCH /api/mock-interviews/[id]`. Returns structured feedback on a code submission. Not intended to be called directly from the client.

---

## Search Route

### `GET /api/search`
Full-text search across lessons, questions, and notes.

**Auth required**: Yes (notes are user-specific)

**Query params**:
- `q` (required): search query string (max 100 chars)
- `type` (optional): `lessons | questions | notes | all` (default: `all`)
- `limit` (optional): `number` (default: 10, max: 30)

**Response `200`**:
```json
{
  "lessons": [
    { "id": "string", "slug": "string", "title": "string", "moduleTitle": "string", "categorySlug": "string", "rank": "number" }
  ],
  "questions": [
    { "id": "string", "question": "string", "lessonSlug": "string", "lessonTitle": "string", "difficulty": "string", "rank": "number" }
  ],
  "notes": [
    { "id": "string", "lessonSlug": "string", "lessonTitle": "string", "excerpt": "string" }
  ]
}
```

---

## Cron Route

### `GET /api/cron/rotate-challenge`
Rotate the daily challenge. Called by Vercel Cron at `0 0 * * *` (00:00 UTC).

**Auth**: Validates `Authorization: Bearer {CRON_SECRET}` header.

**Behavior**:
1. Finds tomorrow's `DailyChallenge` by date (pre-seeded).
2. Updates Redis `challenge:today` cache with the new challenge.
3. Logs rotation.

**Response `200`**: `{ "rotated": true, "date": "string" }`
