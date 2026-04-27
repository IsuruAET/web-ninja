# Server Actions Contracts

**Type**: Next.js Server Actions (`'use server'`)  
**Usage**: Form submissions and mutations that originate from Client Components via `useActionState` or direct `action={actionFn}` on `<form>` elements.  
**Auth**: All actions except `signUp` and `signIn` require an authenticated session (checked via `auth()` at the top of each action).  
**Return type**: All actions return `ActionState`:

```typescript
type ActionState =
  | { success: true; data?: unknown; redirect?: string }
  | { success: false; error: string; fieldErrors?: Record<string, string[]> }
```

---

## Auth Actions (`src/app/actions/auth.ts`)

### `signUp(formData: FormData) → ActionState`

Creates a new user account.

**Fields**: `name`, `email`, `password`, `confirmPassword`

**Validation** (Zod):
- `email`: valid email format, max 255 chars
- `password`: min 8 chars, at least 1 uppercase, 1 number
- `confirmPassword`: must match `password`
- `name`: min 2 chars, max 100 chars

**Behavior**:
1. Check email is not already registered.
2. Hash password with `bcryptjs` (12 rounds).
3. Create `User` in PostgreSQL.
4. Create `StreakRecord` with `currentStreak: 0`.
5. Sign the user in via Auth.js `signIn('credentials', ...)`.
6. Return `{ success: true, redirect: '/dashboard' }`.

**Errors**: `{ success: false, error: "Email already in use" }` or field-level Zod errors.

---

### `signIn(formData: FormData) → ActionState`

Sign in with email and password.

**Fields**: `email`, `password`

**Behavior**:
Delegates to Auth.js `signIn('credentials', { email, password, redirect: false })`.

**Errors**: `{ success: false, error: "Invalid email or password" }`

---

### `signOut() → ActionState`

Sign the current user out.

**Behavior**: Calls Auth.js `signOut({ redirect: false })`.

**Returns**: `{ success: true, redirect: '/' }`

---

## User Actions (`src/app/actions/user.ts`)

### `updateProfile(formData: FormData) → ActionState`

Update the current user's profile settings.

**Fields**: `name`, `avatar` (URL), `theme`, `notifyStreak`, `notifyWeekly`, `learningGoal`

**Validation**: Zod — all fields optional; `name` max 100 chars; `learningGoal` max 200 chars; `avatar` must be a valid HTTPS URL.

**Behavior**:
1. Call `user.service.ts updateUser()`.
2. Revalidate `/settings` path via `revalidatePath`.

**Returns**: `{ success: true, data: { name, theme } }`

---

### `deleteAccount(formData: FormData) → ActionState`

Permanently delete the authenticated user's account (FR-047).

**Fields**: `confirmEmail`

**Validation**: `confirmEmail` must exactly match `session.user.email`.

**Behavior**:
1. Prisma cascading delete on `User`.
2. Delete all MongoDB `ChatSession` documents for this `userId`.
3. Evict Redis keys: `session:{userId}`, `streak:{userId}`, `progress:{userId}:summary`.
4. Sign out via Auth.js.

**Returns**: `{ success: true, redirect: '/' }`

---

## Note Actions (`src/app/actions/notes.ts`)

### `saveNote(formData: FormData) → ActionState`

Create or update a personal note on a lesson (upsert — one note per user per lesson).

**Fields**: `lessonId`, `content`

**Validation**: `content` max 5000 chars; `lessonId` must be a valid cuid.

**Behavior**:
1. Upsert `Note` record in Prisma.
2. Update PostgreSQL `searchVector` for the note (via raw query).
3. Revalidate `/notes` path.

**Returns**: `{ success: true, data: { id, content, updatedAt } }`

---

### `deleteNote(formData: FormData) → ActionState`

Delete a note by ID.

**Fields**: `noteId`

**Behavior**: Deletes `Note` only if `userId` matches the session user (ownership check).

**Returns**: `{ success: true }`

---

## Lesson Actions (`src/app/actions/lesson.ts`)

### `completeLesson(formData: FormData) → ActionState`

Mark a lesson as completed and update progress/streak.

**Fields**: `lessonId`, `timeSpentMs`

**Behavior**:
1. Upsert `UserProgress`.
2. Update `StreakRecord` via `streak.service.ts`.
3. Check and award badges.
4. Invalidate Redis `streak:{userId}` and `progress:{userId}:summary`.
5. Revalidate `/dashboard` and `/learn` paths.

**Returns**: `{ success: true, data: { streakCurrent, newBadges } }`

---

## Bookmark Actions (`src/app/actions/bookmarks.ts`)

### `toggleBookmark(formData: FormData) → ActionState`

Add or remove a bookmark on a lesson (toggle).

**Fields**: `lessonId`

**Behavior**: If bookmark exists, delete it. If not, create it.

**Returns**: `{ success: true, data: { bookmarked: boolean } }`
