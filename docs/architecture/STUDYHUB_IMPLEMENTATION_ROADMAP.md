# StudyHub — Implementation Roadmap

**Version:** 1.0  
**Date:** 2026-11-04  
**Status:** Phase 0 — Authoritative plan, requires owner approval before Phase 1 begins  
**Author:** Phase 0 Audit (Claude Sonnet 4.6, session https://claude.ai/code/session_01GrXQBHN1FfE3b1PuBWsGTt)

---

## 1. Scope and Assumptions

### 1.1 Developer Capacity

**Solo developer, estimated 15–20 focused hours per week.**

This roadmap is sized for one developer working part-time. Estimates are deliberate and do not compress to match arbitrary deadlines. The original "90-day" plan from earlier strategy documents is not achievable at this capacity for the full platform. The roadmap below is what is realistic.

**Estimation approach:**
- Hours are realistic "focused development time" excluding context switching, debugging rabbitholes, learning unfamiliar technology
- Each task is estimated assuming the developer is competent in React/TypeScript; has learned Spring Boot during the planning phase; and is not writing backend Java professionally yet
- Add 30% buffer to any task involving a new technology (Spring Boot, MySQL, Docker)

### 1.2 Approval Gates

This roadmap has explicit STOP points requiring owner review and approval before the next phase begins. The developer must present deliverables and receive written approval before proceeding.

**Gate 1:** Phase 0 → Phase 1: This document + 5 other audit documents  
**Gate 2:** Phase 1 → Phase 2: Phase 1 pilot retrospective + native implementation plan

---

## 2. Phase 0 — Audit and Planning (CURRENT PHASE)

**Goal:** Understand what we have, design what we're building, make decisions before writing code.  
**Status:** In progress  
**Duration:** 2–3 weeks

### Deliverables (all required)

| # | Deliverable | Status |
|---|-------------|--------|
| 1 | `STUDYHUB_CURRENT_STATE_AUDIT.md` | ✅ Complete |
| 2 | `STUDYHUB_TARGET_ARCHITECTURE.md` | ✅ Complete |
| 3 | `STUDYHUB_IMPLEMENTATION_ROADMAP.md` | ✅ This document |
| 4 | `STUDYHUB_CONTENT_AUDIT.md` | ✅ Complete |
| 5 | `STUDYHUB_UX_AUDIT.md` | ✅ Complete |
| 6 | `STUDYHUB_DECISION_RECORD.md` | ✅ Complete |
| 7 | `docs/design/STUDYHUB_VISUAL_DIRECTION.md` | ✅ Complete |

### Open decisions required from owner before Phase 1

See `STUDYHUB_DECISION_RECORD.md` section 3 for the full list. Summary:

- [ ] Repository rename from `bootcamp-hub` to `studyhub` — confirm timing
- [ ] Hosting provider for Phase 1 backend (Render, Railway, DigitalOcean, Fly.io)
- [ ] Static hosting for Phase 1 SPA (Cloudflare Pages, Vercel, Netlify)
- [ ] PWA support in Phase 1? (Recommended: yes, after core features)
- [ ] Payment integration in Phase 1 or deferred?
- [ ] Invite-only vs open registration for pilot?
- [ ] Pilot size: how many users, which disciplines?
- [ ] USMLE: who is the qualified medical content reviewer?
- [ ] `cyberLabsNew.js`: integrate or discard?
- [ ] AI Engineering curriculum mapping: who does this work?

**→ STOP. Present Phase 0 deliverables to owner. Await written approval before Phase 1 begins.**

---

## 3. Phase 1 — Web-First Controlled Pilot

**Goal:** A working, deployed StudyHub web application with real users, real content, and AI-assisted learning — for a controlled set of learners on existing IT certification bootcamps.  
**Prerequisite:** Phase 0 approval + all open decisions answered  
**Estimated duration:** 20–26 weeks (5–6.5 months) at 15–20 hours/week  
**Pilot scope:** Cybersecurity + IBM Python+ML bootcamps (AI Engineering after structural fix)

### Phase 1 is NOT:

- Native iOS or Android (Phase 2)
- USMLE content (Phase 2+, unless content is ready and medical reviewer engaged)
- WAEC/NECO/JAMB (Phase 2+)
- Full adaptive learning pathways or autonomous AI agents (future phases)
- Public launch — this is a controlled pilot with invited learners

---

### 3.1 Sprint Map

**Sprint = 2 weeks, ~30 hours of work**

#### Sprint 1–2: Foundation (Weeks 1–4) — ~60 hours

**Goal:** Working project skeleton, local dev environment, first API endpoint.

Tasks:
- [ ] Rename repository (`bootcamp-hub` → `studyhub`), update `package.json`, `vite.config.ts`, GitHub Pages base path
- [ ] Initialize Spring Boot project (Spring Initializr: Web, Security, Data JPA, MySQL, Validation, Actuator)
- [ ] `docker-compose.yml` for local backend + MySQL
- [ ] Configure `application.yml` / `application-local.yml` with environment variable overrides
- [ ] Implement first domain skeleton: Identity (User entity, UserRepository, basic registration/login)
- [ ] JWT token issuance (access + refresh) — in-memory + httpOnly cookie pattern
- [ ] `GET /api/health` endpoint with Actuator
- [ ] GitHub Actions CI: run frontend lint + build; run backend tests (zero tests initially)
- [ ] Vitest configured on frontend (zero tests initially — scaffold only)
- [ ] Frontend: install TanStack Query + Zustand + React Hook Form + Zod
- [ ] Frontend: Axios instance with JWT interceptor
- [ ] Frontend: `BrowserRouter` routing (replace HashRouter)
- [ ] Frontend: Auth pages (`/auth/login`, `/auth/register`) with working API integration

Estimate: ~60 hours  
Checkpoint: Local signup + login working end-to-end

---

#### Sprint 3–4: Core Data Layer (Weeks 5–8) — ~55 hours

**Goal:** Database schema for curriculum and content; seed data loaded; content API working.

Tasks:
- [ ] MySQL schema: disciplines, courses, modules, lessons (see target architecture schema)
- [ ] Flyway for database migrations (all schema changes via migration files)
- [ ] JPA entities for all tables
- [ ] Repository interfaces
- [ ] `GET /api/disciplines` — list disciplines
- [ ] `GET /api/courses/{disciplineId}` — list courses for discipline
- [ ] `GET /api/lessons/{id}` — get lesson content
- [ ] Content lifecycle enforcement: only `PUBLISHED` lessons visible to learners
- [ ] Content seed script: load Cybersecurity (103 days) and IBM (57 days) curriculum
- [ ] Content seed script: load 500 Cybersecurity + 100 IBM assessment questions
- [ ] Content seed script: load ~130 labs across both disciplines
- [ ] Admin content status management (even if just via direct SQL for pilot)
- [ ] Frontend: Explore page (discipline listing, course listing)
- [ ] Frontend: basic Lesson view (fetch + render lesson content via MarkdownRenderer)

Estimate: ~55 hours  
Checkpoint: Can browse disciplines and read a lesson (unauthenticated reading fine for pilot)

---

#### Sprint 5–6: Progress Domain (Weeks 9–12) — ~50 hours

**Goal:** Learning state tracked on the backend; XP and streak server-authoritative.

Tasks:
- [ ] MySQL schema: user_lesson_progress, user_xp, user_goals, user_mastery
- [ ] `POST /api/progress/lesson/{lessonId}/complete` — mark lesson complete, award XP
- [ ] `GET /api/progress/me` — get learner's overall progress summary
- [ ] `GET /api/progress/mastery/{disciplineId}` — domain mastery breakdown
- [ ] `POST /api/goals` — create learning goal (target exam + date)
- [ ] XP engine on backend (port logic from XPContext.tsx)
- [ ] Streak calculation: server-authoritative, timezone-aware
- [ ] Frontend: Home dashboard (goal card, resume card, discipline cards, XP/streak display)
- [ ] Frontend: My Progress page (mastery chart by domain, streak calendar, XP history)
- [ ] Frontend: XPBar updated to fetch from backend (not localStorage)
- [ ] Remove GistSyncContext; remove Gist sync from SettingsPanel

Estimate: ~50 hours  
Checkpoint: Learning progress tracked on server; Home dashboard shows real data

---

#### Sprint 7–8: Assessment Domain (Weeks 13–16) — ~55 hours

**Goal:** Working mock exam engine with server-side scoring; quiz embedded in lessons.

Tasks:
- [ ] MySQL schema: questions, exams, exam_attempts
- [ ] `GET /api/exams/{examId}/questions` — fetch randomized question set
- [ ] `POST /api/exams/{examId}/submit` — submit answers, score, record attempt
- [ ] `GET /api/exams/attempts/{userId}` — exam history
- [ ] Domain mastery update after exam (update `user_mastery` from exam results)
- [ ] Frontend: `CertExam.tsx` adapted to use server-fetched questions
- [ ] Frontend: `QuizEngine.tsx` adapted for server data
- [ ] Frontend: Practice page (exam selection, start exam, exam timer)
- [ ] Frontend: Post-exam review (score breakdown by domain, correct/incorrect review)
- [ ] Frontend: Incorrect answer review shows AI explanation (placeholder text for now — AI in next sprint)

Estimate: ~55 hours  
Checkpoint: Can take a full 100-question mock exam with server scoring and domain breakdown

---

#### Sprint 9–10: AI Integration (Weeks 17–20) — ~65 hours

**Goal:** AI Tutor working in lesson view; incorrect answer explanations; revision sessions.

Tasks:
- [ ] AI orchestration service (Spring Boot): interface + Anthropic Claude implementation
- [ ] Rate limiting per user per day (Bucket4j or simple DB counter)
- [ ] Usage tracking: `ai_usage_log` table, populated on every AI call
- [ ] Cost controls: configurable max tokens per request type, circuit breaker
- [ ] `POST /api/ai/lesson-tutor` — lesson context grounded Q&A
- [ ] `POST /api/ai/explain-answer` — incorrect answer explanation (triggered automatically)
- [ ] `POST /api/ai/hint` — quiz hint (never reveals answer)
- [ ] `POST /api/ai/revision-session` — generate targeted revision content from weak domains
- [ ] `POST /api/ai/simplify` — simplified explanation of lesson section
- [ ] Frontend: AI Tutor panel in Lesson view (collapsible, persistent)
- [ ] Frontend: Quick prompt buttons (Explain simply, Give an example, Ask)
- [ ] Frontend: Incorrect answer explanation auto-display after wrong answer
- [ ] Frontend: Hint button in quiz (before submitting answer)
- [ ] Frontend: Revision session launcher in My Progress (weak domain → revision)
- [ ] Frontend: AI usage indicator (if rate-limited on free tier)
- [ ] Security audit: confirm no AI provider keys exposed to frontend

Estimate: ~65 hours  
Checkpoint: AI Tutor functional in lesson view; explanations working for wrong answers

---

#### Sprint 11–12: Hardening + Pilot Launch (Weeks 21–24) — ~55 hours

**Goal:** Production-ready for controlled pilot. Invite pilot users. Monitor.

Tasks:
- [ ] Deploy backend to production (Docker on chosen VPS)
- [ ] Deploy frontend to production (chosen CDN/static host)
- [ ] Configure HTTPS (Let's Encrypt or provider-managed)
- [ ] Environment variables in production (no secrets in code or git)
- [ ] GitHub Actions: CI/CD to staging on PR merge, production on main merge
- [ ] Add accessibility audit: install eslint-plugin-jsx-a11y, fix P0 violations
- [ ] Font size control: test against all pages, ensure no overflow
- [ ] Mobile QA: test on iOS Safari + Android Chrome
- [ ] Error boundaries on all major page components
- [ ] Session expiry handling (silent refresh or redirect to login)
- [ ] PWA implementation (if approved): vite-plugin-pwa + manifest + service worker
- [ ] Write 20–30 Vitest unit tests (XP engine, scoring logic, component smoke tests)
- [ ] Write 5–10 Playwright E2E tests (login, complete lesson, take exam)
- [ ] Load test with expected pilot user count (k6 or similar — simple)
- [ ] Invite 5–10 pilot users; collect feedback week 1
- [ ] Monitor: AI usage costs, API latency, error rates (use Render/Railway metrics or simple logging)

Estimate: ~55 hours  
Checkpoint: Pilot live with real users

---

#### Weeks 25–26: Pilot Feedback + Iteration (Buffer)

**Purpose:** Address pilot feedback. Fix bugs. This is intentional buffer — not "extra features."

**What may happen here:**
- UI improvements based on pilot user feedback
- Performance issues found in production
- Content gaps identified by real learners
- AI quality issues (hallucination, unhelpful responses)
- Accessibility issues reported by pilot users

Estimate: ~30 hours  
Checkpoint: Pilot retrospective complete; Phase 2 planning begins

---

### 3.2 Phase 1 Total Estimate

| Sprint Block | Weeks | Hours |
|-------------|-------|-------|
| Sprint 1–2: Foundation | 1–4 | 60 |
| Sprint 3–4: Data Layer | 5–8 | 55 |
| Sprint 5–6: Progress | 9–12 | 50 |
| Sprint 7–8: Assessment | 13–16 | 55 |
| Sprint 9–10: AI Integration | 17–20 | 65 |
| Sprint 11–12: Hardening + Launch | 21–24 | 55 |
| Weeks 25–26: Pilot Feedback | 25–26 | 30 |
| **Total** | **26 weeks** | **370 hours** |

At 15 hours/week: **~25 weeks** (6.2 months)  
At 20 hours/week: **~18 weeks** (4.5 months)  
**Realistic target: 5–6 months from Phase 0 approval to pilot launch**

---

### 3.3 Phase 1 Risk Register

| Risk | Likelihood | Impact | Mitigation |
|------|-----------|--------|------------|
| Spring Boot learning curve extends backend tasks | High | Medium | Allocate extra 30% to first 3 backend sprints; use Spring Boot tutorials proactively |
| AI costs exceed budget in pilot | Medium | Medium | Rate limiting + usage caps per user per day; monitor weekly |
| AI-generated explanations are inaccurate | Medium | High | Grounding in approved content only; human QA sample during pilot |
| Content seed script issues (data format mismatches) | Medium | Medium | Test seed script on staging first; validate all 103 Cyber days before launch |
| ciberLabsNew.js decision delays content migration | Low | Low | Decision made in Phase 0; don't block on it |
| AI Engineering curriculum not mapped before launch | Medium | Low | Phase 1 launches with Cyber + IBM only; AI Engineering can follow |
| Pilot users churn before providing feedback | Low | Medium | Recruit motivated users; weekly check-in; personal feedback sessions |

---

## 4. Phase 2 — Native iOS and Android

**Prerequisite:** Phase 1 pilot retrospective complete + owner approval  
**Estimated start:** Month 7–8 from Phase 0 approval  
**Technology:** React Native + Expo (Managed Workflow)

**Phase 2 is explicitly NOT designed during Phase 1.** A proper Phase 2 architecture document is produced at the Phase 1 → Phase 2 gate. This section is an intent statement only.

### High-level scope

- iOS app (App Store submission)
- Android app (Google Play submission)
- Feature parity with web pilot (same content, same AI features, same progress sync)
- Native patterns: React Navigation, platform-specific UI conventions
- Expo Over-The-Air (OTA) updates to avoid store review delays for content changes
- Push notifications for streak reminders, new content

### Native-ready decisions made in Phase 1

The Phase 1 API is designed API-first. No web-only endpoints. All progress, content, and AI is served via REST. The native apps are first-class API clients — they don't require any backend changes.

The web SPA's design system (tokens, component patterns) informs the native design but does not block it. React Native components are built separately.

---

## 5. Subsequent Phases (High Level Only)

After Phase 2 is approved and delivered, subsequent phases address platform expansion. These are not designed in Phase 0.

### Platform Expansion
- USMLE Step 1 content (requires medical content pipeline + qualified reviewer)
- WAEC/NECO/JAMB content
- Subscription and payment integration (Paystack + Stripe)
- Localized pricing (NGN + USD)

### Advanced Learning Intelligence
- Adaptive learning pathways (content sequencing based on performance)
- Predictive mastery modelling
- Personalized curriculum generation
- Learning analytics dashboard for self-insight

**These are explicitly deferred. They are not designed in Phase 0 and are not committed as roadmap items until Phase 2 is complete.**

---

## 6. What This Roadmap Is Not Promising

To be explicit about what is out of scope across all defined phases:

- No autonomous learning agents
- No AI that independently certifies learner mastery
- No adaptive curriculum generation in Phase 1
- No public marketplace of user-generated content
- No social/community features (forums, study groups) in Phase 1 or 2
- No video content pipeline in Phase 1

---

*End of STUDYHUB_IMPLEMENTATION_ROADMAP.md*
