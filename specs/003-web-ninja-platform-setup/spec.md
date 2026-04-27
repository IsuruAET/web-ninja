# Feature Specification: Web Ninja — AI-Powered Interview Preparation Platform

**Feature Branch**: `003-web-ninja-platform-setup`  
**Created**: 2026-04-27  
**Status**: Draft  
**Input**: User description: "initial platform setup - AI-powered web interview preparation platform called 'Web Ninja'"

---

## Clarifications

### Session 2026-04-27

- Q: Can visitors browse lessons and the roadmap without creating an account, or is all content behind a login wall? → A: Guest preview with limited access — public landing page + first lesson per category is publicly browsable; all remaining lessons and interactive features require a registered account.
- Q: How is feedback on mock coding interview submissions generated? → A: AI-evaluated feedback — submitted code is reviewed by the AI (Sensei) which provides qualitative feedback on correctness, efficiency, and approach without executing the code.
- Q: Is the Daily Challenge globally curated (same for all users) or personalized per user? → A: Globally curated — one challenge per day, identical for all users, authored/selected in advance.
- Q: What is the data privacy and account deletion stance for the platform? → A: GDPR-aware baseline — users can request full account and data deletion; all personal data (progress, notes, chat history, resume input) is purged on deletion; no formal compliance certification required for v1.
- Q: What is the expected availability/uptime target for the platform? → A: 99.5% monthly uptime (~3.6 hours allowable downtime/month); single-region deployment with automated restarts and basic health checks.

---

## User Scenarios *(mandatory)*

### User Story 1 - Onboard and Explore the Learning Roadmap (Priority: P1)

A full-stack developer signs up for Web Ninja and lands on the dashboard. They see a visual roadmap of all learning modules organized by category (Foundation, Frontend, Backend, etc.) with their current progress highlighted. They pick a topic — e.g., "JavaScript Deep Dive" — and begin a structured lesson with explanations, code examples, diagrams, and curated interview questions.

**Why this priority**: This is the core value proposition. Without structured content discovery and lesson consumption, the platform delivers no value. Every other feature depends on content being accessible and navigable.

**Manual Verification**: Navigate to the dashboard, open the roadmap timeline, click any module, verify a lesson page loads with explanation, code snippets, diagrams, and interview questions.

**Acceptance Scenarios**:

1. **Given** a new user lands on the dashboard, **When** they view the roadmap, **Then** they see all learning modules organized under named categories with visual progress indicators.
2. **Given** a user selects a module, **When** they open a lesson, **Then** the lesson displays: simple explanation, real-world example, code snippet, diagram, interview tips, and 25 interview questions.
3. **Given** a user completes a lesson, **When** they return to the roadmap, **Then** the module is marked as completed with a visual badge.

---

### User Story 2 - Practice Interview Questions with Flashcards (Priority: P2)

A developer wants to drill interview questions for a specific topic. They navigate to the interview questions section of a lesson, where flashcard-style questions are presented one at a time. They can reveal the answer, mark questions as done, and filter by difficulty (Easy / Medium / Hard) or type (FAANG-style, scenario-based).

**Why this priority**: Interview question practice is the second most important action on the platform. It directly reduces anxiety and improves recall for real interviews.

**Manual Verification**: Open any lesson's interview questions tab, flip through flashcards, reveal answers, mark as completed, apply difficulty filter — verify state persists across page refresh.

**Acceptance Scenarios**:

1. **Given** a user is on a lesson's questions section, **When** they tap a flashcard, **Then** the answer is revealed with a smooth flip animation.
2. **Given** a user marks a question as completed, **When** they return later, **Then** the question shows a completion indicator and can be filtered out.
3. **Given** a user applies a "Hard" difficulty filter, **When** questions are displayed, **Then** only Hard-labeled questions are shown.

---

### User Story 3 - Get Help from the AI Mentor "Sensei" (Priority: P2)

A developer is studying "Event Loop" and doesn't understand a concept. They open the AI chat panel (Sensei) docked to the side of the lesson, type their question in natural language, and receive a context-aware explanation, followed by a code example. The conversation history is preserved within the session.

**Why this priority**: AI assistance dramatically increases learning retention and reduces the friction of being stuck, making Web Ninja feel like a personal tutor rather than a static reference.

**Manual Verification**: Open a lesson, activate the Sensei chat panel, type "explain event loop with an example," verify a coherent response with a code snippet is returned within a reasonable time.

**Acceptance Scenarios**:

1. **Given** a user is viewing any lesson, **When** they open Sensei and submit a question, **Then** a relevant response is returned within a few seconds.
2. **Given** a user continues asking follow-up questions, **When** Sensei responds, **Then** the conversation history is maintained and the context of prior messages is reflected.
3. **Given** a user closes and reopens the chat panel, **When** they view it again in the same session, **Then** their conversation history is still visible.

---

### User Story 4 - Track Daily Progress and Maintain a Study Streak (Priority: P3)

A developer wants to build a study habit. Each day they log in, complete at least one lesson or challenge, and see their streak counter increment. The dashboard shows their overall completion percentage per category, time spent, and weekly activity chart.

**Why this priority**: Habit-forming mechanics (streaks, progress bars, badges) drive retention. Without them, users disengage after initial exploration.

**Manual Verification**: Complete a lesson on day 1, return on day 2, complete another lesson — verify streak counter shows "2 days," verify progress bars and analytics update accordingly.

**Acceptance Scenarios**:

1. **Given** a user completes a lesson or challenge on a given calendar day, **When** they view the dashboard, **Then** their streak counter reflects the consecutive days of activity.
2. **Given** a user misses a day, **When** they return, **Then** their streak resets to 1.
3. **Given** a user views progress analytics, **When** the page loads, **Then** they see completion percentages per category, total time studied, and a weekly activity heatmap.

---

### User Story 5 - Simulate a Mock Interview Session (Priority: P3)

A developer preparing for an upcoming interview starts a mock interview session. They are presented with a timed coding challenge or system design question, write their solution in an embedded playground, and receive feedback. They can also simulate a behavioral interview with STAR-method guided prompts.

**Why this priority**: Mock interviews are the closest practice to the real thing. They build confidence and surface weaknesses before actual interviews.

**Manual Verification**: Start a mock interview session, receive a prompt, write code in the playground, submit — verify a feedback summary is displayed.

**Acceptance Scenarios**:

1. **Given** a user starts a mock interview, **When** the session loads, **Then** a question prompt, timer, and code editor (or whiteboard for system design) are visible.
2. **Given** a user submits their solution, **When** evaluation completes, **Then** they receive structured feedback on correctness, efficiency, and communication quality.
3. **Given** a user chooses a behavioral mock interview, **When** prompted, **Then** STAR-method guiding cues are provided and their response can be self-evaluated against a rubric.

---

### User Story 6 - Save Bookmarks and Personal Notes (Priority: P4)

A developer reading a lesson finds a key insight they want to revisit. They bookmark the lesson and add a personal note. Later, they access their saved items from a "My Notes" section in the sidebar, search across notes, and return directly to the bookmarked content.

**Why this priority**: Notes and bookmarks convert passive reading into active review material, improving long-term retention.

**Manual Verification**: Bookmark a lesson, add a note to it, navigate to My Notes section — verify both appear and clicking returns to the correct lesson page.

**Acceptance Scenarios**:

1. **Given** a user bookmarks a lesson, **When** they visit My Notes, **Then** the bookmarked lesson appears with a thumbnail and link.
2. **Given** a user writes a note on a lesson, **When** they search for a keyword from that note, **Then** the note appears in search results.

---

### User Story 7 - Complete Daily Interview Challenges (Priority: P4)

A developer opens Web Ninja in the morning and sees the "Daily Challenge" card on the dashboard — a curated question or coding problem aligned with their current learning path. They complete it, earn a badge, and their streak is maintained.

**Why this priority**: Daily challenges create a lightweight re-engagement loop, especially for users who may not have time for full lessons every day.

**Manual Verification**: Load the dashboard on a new day, confirm a new Daily Challenge is visible, complete it, verify a success/badge notification appears and streak is maintained.

**Acceptance Scenarios**:

1. **Given** a user loads the dashboard each day, **When** the page renders, **Then** a new Daily Challenge card is visible.
2. **Given** a user completes the Daily Challenge, **When** they submit it, **Then** a completion badge is awarded and their streak is preserved.

---

### Edge Cases

- What happens when a user opens Sensei and the AI service is temporarily unavailable? → Display a graceful error message ("Sensei is resting — try again in a moment") without crashing the lesson page.
- What happens when a user completes all questions in a lesson's flashcard deck? → Show a completion summary screen with options to retry (shuffled), filter unseen, or move to the next lesson.
- What happens when a user hasn't logged in for more than 24 hours? → Streak is reset to 0; display a motivational message encouraging them to restart.
- What happens when a user searches for a topic that doesn't exist yet? → Display "No results found" with suggestions for related existing topics.
- What happens when a user's note exceeds reasonable length? → Allow saving but display a character count indicator and warn at 90% of the limit.
- What happens on mobile when viewing code snippets wider than the screen? → Horizontal scrolling within the code block, no clipping or overflow breaking layout.

---

## Requirements *(mandatory)*

### Functional Requirements

#### Platform Foundation

- **FR-000**: The platform MUST include a public marketing landing page accessible without authentication, showcasing the platform's value proposition, featured modules, and a call-to-action to sign up or log in.
- **FR-000a**: The first lesson within each category MUST be publicly accessible as a free preview without requiring a user account. All remaining lessons and all interactive features (flashcards, Sensei, mock interviews, notes, bookmarks, progress tracking) MUST require a registered and authenticated account.
- **FR-001**: The platform MUST present a dashboard as the primary landing page after login, displaying the learning roadmap, daily challenge, progress summary, and streak counter.
- **FR-002**: The platform MUST support dark mode and light mode, with dark mode as the default, and persist the user's preference.
- **FR-003**: The platform MUST be fully responsive and functional on mobile, tablet, and desktop screen sizes.
- **FR-004**: The platform MUST include a persistent left sidebar navigation listing all learning categories and modules, collapsible on smaller screens.
- **FR-005**: The platform MUST include a top navigation bar with: global search, streak indicator, overall progress, notifications bell, user profile menu, and Sensei AI chat toggle.
- **FR-006**: The platform MUST support full-text search across all lesson titles, content, and interview questions.

#### Learning Roadmap & Lessons

- **FR-007**: The platform MUST organize all content into the defined category hierarchy: Foundation, Frontend Engineering, Backend Engineering, Databases & Data, Architecture & System Design, DevOps & Cloud, Testing & Quality, AI Engineering, Soft Skills & Leadership.
- **FR-008**: Each category MUST contain its defined sub-topics (modules) in the specified order.
- **FR-009**: Each lesson MUST include the following sections: Simple Explanation, Real-World Examples, JavaScript/TypeScript Code Snippets, Architecture/Flow Diagrams, Common Mistakes, Performance Tips, Interview Tips, "How Interviewers Ask This Topic," and Interview Questions.
- **FR-010**: Lessons MUST support a beginner-to-advanced content progression, clearly indicated visually.
- **FR-011**: Lesson completion MUST be tracked and reflected in the roadmap with visual badges.
- **FR-012**: The roadmap MUST display as a timeline view with visual connectors between modules, showing prerequisite relationships.

#### Interview Questions

- **FR-013**: Each lesson MUST include at least 25 interview questions presented in flashcard style.
- **FR-014**: Flashcards MUST support reveal/hide answer toggle with smooth animation.
- **FR-015**: Each question MUST be labeled with a difficulty level: Easy, Medium, or Hard.
- **FR-016**: Questions MUST be filterable by difficulty and by type (FAANG-style, scenario-based, conceptual).
- **FR-017**: Users MUST be able to mark individual questions as completed, with state persisted across sessions.
- **FR-018**: The platform MUST include a dedicated Quiz & Flashcard mode allowing randomized practice across topics.

#### AI Mentor — Sensei

- **FR-019**: The platform MUST provide an AI chat assistant called "Sensei" accessible from any page via the top navigation.
- **FR-020**: Sensei MUST maintain conversation context within a session and display the full conversation history.
- **FR-021**: Sensei MUST respond with relevant code examples when explaining technical topics.
- **FR-022**: The platform MUST display a graceful fallback message if the AI service is unavailable, without disrupting the rest of the page.

#### Mock Interviews

- **FR-023**: The platform MUST include a Mock Interview section offering: timed coding challenges, system design whiteboard exercises, and behavioral interview prompts.
- **FR-024**: Mock coding interviews MUST include an embedded code editor with syntax highlighting.
- **FR-025**: Mock coding interviews MUST provide AI-evaluated feedback upon submission: the user's submitted code is sent to the AI (Sensei) for qualitative review covering correctness reasoning, efficiency analysis, and approach quality. No server-side code execution or test-case running is required.
- **FR-026**: Behavioral mock interviews MUST include STAR-method guidance and self-evaluation prompts.

#### Progress & Gamification

- **FR-027**: The platform MUST track and display a daily study streak, resetting when a day is missed.
- **FR-028**: The platform MUST display per-category completion percentages and an overall progress percentage.
- **FR-029**: The platform MUST award completion badges for finishing lessons, categories, and challenges.
- **FR-030**: The platform MUST include a progress analytics dashboard showing: weekly activity heatmap, time studied, modules completed, and questions answered.

#### Bookmarks & Notes

- **FR-031**: Users MUST be able to bookmark any lesson for quick re-access.
- **FR-032**: Users MUST be able to write and save personal notes on any lesson.
- **FR-033**: Notes and bookmarks MUST be accessible from a dedicated "My Notes" section in the sidebar.
- **FR-034**: Notes MUST be searchable by keyword.

#### Daily Challenges

- **FR-035**: The platform MUST surface a new Daily Challenge on the dashboard each calendar day. The challenge is globally curated — identical for all users on a given day, authored or selected in advance by platform administrators.
- **FR-036**: Completing the Daily Challenge MUST count toward the user's study streak.
- **FR-037**: Daily Challenges MUST include a mix of coding questions, concept questions, and system design prompts.

#### Additional Sections

- **FR-038**: The platform MUST include a System Design section with whiteboard-style diagramming support and curated system design interview walkthroughs.
- **FR-039**: The platform MUST include a Resume Analyzer section where users can input resume content and receive structured improvement feedback.
- **FR-040**: The platform MUST include company-specific interview preparation guides (e.g., FAANG-focused question sets).
- **FR-041**: The platform MUST include a Settings/Profile page where users can configure: display name, avatar, dark/light mode, notification preferences, and learning goals.
- **FR-047**: The platform MUST provide an account deletion option in the Settings page. Upon confirmed deletion, all personal data associated with the account (progress records, notes, bookmarks, Sensei chat history, and any resume content) MUST be permanently purged. Anonymised aggregated data (e.g., global challenge completion counts) may be retained.

#### UI & Interaction

- **FR-042**: Code snippets MUST be displayed in animated, syntax-highlighted code blocks with a copy-to-clipboard button.
- **FR-043**: Diagrams MUST render inline within lessons (architecture diagrams, flowcharts, lifecycle diagrams).
- **FR-044**: The platform MUST use smooth page transitions and micro-animations for a polished interactive experience.
- **FR-045**: The platform MUST include terminal-style UI components for command-line-related lessons.
- **FR-046**: Loading states MUST be handled with skeleton screens, not spinner-only states.

### Key Entities *(include if feature involves data)*

- **User**: Represents a registered learner. Key attributes: profile information, preferences (theme, goals), streak count, total progress.
- **Category**: A top-level grouping of learning modules (e.g., "Frontend Engineering"). Ordered collection of Modules.
- **Module**: A single topic within a category (e.g., "JavaScript Deep Dive"). Contains one or more Lessons.
- **Lesson**: A structured unit of content within a Module. Composed of sections: explanation, code snippets, diagrams, tips, and interview questions.
- **InterviewQuestion**: An individual question associated with a Lesson. Attributes: question text, answer, difficulty level, type (FAANG/scenario/conceptual), completion status per user.
- **UserProgress**: Tracks a User's completion state per Lesson and per InterviewQuestion. Also stores streak data and badge awards.
- **Bookmark**: A user-saved reference to a specific Lesson.
- **Note**: A user-authored text note attached to a specific Lesson.
- **DailyChallenge**: A globally curated question or task surfaced on the dashboard each calendar day, identical for all users. Linked to each user's own completion record to track whether they have completed that day's challenge.
- **MockInterview**: A session record for a mock interview attempt, storing the prompt type, user response, and feedback.
- **ChatSession**: A record of a Sensei AI conversation session, storing the sequence of messages.

---

## Success Criteria *(mandatory)*

### Measurable Outcomes

- **SC-001**: A new user can navigate from the dashboard to a lesson and begin reading content in under 30 seconds, without requiring any tutorial.
- **SC-002**: A user can complete a full lesson (read content, review code snippets, flip through 25 flashcards) in under 20 minutes, supporting bite-sized learning sessions.
- **SC-003**: Sensei responds to any user question within 5 seconds under normal conditions.
- **SC-004**: At least 90% of lessons are navigable on mobile without horizontal scrolling or layout breakage.
- **SC-005**: A user can find any lesson topic via global search in under 10 seconds using a keyword.
- **SC-006**: Progress state (completed lessons, flashcard answers, streaks, notes) persists correctly across browser sessions with zero data loss.
- **SC-007**: The platform supports at least 500 concurrent learners without noticeable performance degradation.
- **SC-011**: The platform achieves 99.5% monthly uptime (~3.6 hours maximum allowable downtime per month), verified via uptime monitoring. Planned maintenance windows are excluded from this calculation.
- **SC-008**: Daily active users maintain a 7-day average study streak at a rate comparable to habit-focused learning platforms (target: 40%+ of engaged users maintain streaks beyond 7 days).
- **SC-009**: Users report the platform as feeling "production-quality" and "professional" in usability feedback (target: 80%+ positive qualitative rating in early user testing).
- **SC-010**: The roadmap view renders all modules in the correct category order on first load without any reordering or visual flash.

---

## Assumptions

- Users are practicing full-stack developers (junior to senior level) targeting technical interviews; non-technical users are out of scope.
- Authentication will use a standard email/password flow with optional social login (Google OAuth). Multi-factor authentication is out of scope for the initial platform setup. Unauthenticated visitors can access the public landing page and the first lesson per category as a free preview; all other content and interactive features require an authenticated account.
- All lesson content (explanations, code snippets, interview questions) for the initial platform will be pre-authored and seeded into the system; a content management interface for authors is out of scope for this spec.
- The AI mentor (Sensei) will be powered by an external AI API; the selection of the specific AI provider is an implementation decision.
- Diagram rendering will use an embeddable diagramming library; diagram source files will be stored as structured data alongside lesson content.
- The mock interview code editor will support JavaScript and TypeScript as the primary languages, with Python as a secondary option; other languages are out of scope for v1.
- Resume Analyzer will accept pasted plain text in v1; PDF parsing is a future enhancement.
- The platform assumes stable internet connectivity; offline mode is out of scope for the initial setup.
- Mobile-first responsive design is required, but a dedicated native mobile app is out of scope.
- Company-specific interview prep guides will be pre-authored for a curated list of companies (e.g., Google, Meta, Amazon, Netflix, Apple, Microsoft); community-submitted content is out of scope for v1.
- Pricing and subscription tiers are out of scope for this specification; the platform is assumed to be accessible without a paywall during the initial setup phase.
- Accessibility compliance targets WCAG 2.1 AA as the baseline standard.
- Data privacy follows a GDPR-aware baseline: users have the right to delete their account and all associated personal data. Full GDPR compliance certification, consent management flows, and data processing agreements are out of scope for v1.
- The platform targets 99.5% monthly uptime using a single-region deployment with automated restarts and basic health checks. Multi-region active-active architecture is out of scope for v1.
