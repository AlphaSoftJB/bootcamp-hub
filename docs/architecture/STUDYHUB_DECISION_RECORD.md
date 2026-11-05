# StudyHub — Decision Record

**Version:** 1.0  
**Date:** 2026-11-04  
**Status:** Phase 0 — All decisions below are confirmed and locked unless explicitly reopened by the owner  
**Author:** Phase 0 Audit (Claude Sonnet 4.6, session https://claude.ai/code/session_01GrXQBHN1FfE3b1PuBWsGTt)

---

## Purpose

This document records every architectural and product decision made during Phase 0. Each decision has:

- A unique ID (DR-NNN)
- Status: **Confirmed** (locked), **Deferred** (not yet decided), or **Superseded** (changed by a later decision)
- The decision itself
- The rationale
- The consequences (what this rules in or out)

Confirmed decisions are authoritative. Reopening a confirmed decision requires an explicit instruction from the owner, logged here as a superseding entry.

---

## 1. Platform Decisions

### DR-001 — Platform Support
**Status:** Confirmed  
**Date confirmed:** Pre-Phase 0

**Decision:** StudyHub supports native iOS, native Android, and Web.

**Rationale:** A multi-discipline learning platform targeting both a global (IT certs) and Nigerian (WAEC/NECO/JAMB) audience requires mobile-first reach. Native iOS and Android provide first-class mobile experiences. Web is included as the primary surface for the Phase 1 controlled pilot.

**Consequences:**
- Web is the primary surface for Phase 1 (90-day controlled pilot period)
- Native iOS and Android are confirmed targets but implemented in Phase 2
- Native must be accounted for in Phase 0 architecture and feasibility audit (done — see STUDYHUB_TARGET_ARCHITECTURE.md)
- Native implementation follows approval gate: Phase 1 retrospective → Phase 2 authorization
- API design in Phase 1 must be API-first (no web-only contracts)

---

### DR-002 — Web Phase 1 is Controlled Pilot
**Status:** Confirmed  
**Date confirmed:** Pre-Phase 0

**Decision:** The web deployment in Phase 1 is a controlled pilot — not a public launch. Access is limited to invited learners.

**Rationale:** A controlled pilot reduces risk, allows for content and AI quality monitoring, and enables course-correction before public launch.

**Consequences:**
- Registration may be invite-only or PIN-gated (specific mechanism: Deferred — see DR-OPEN-002)
- Pilot size is bounded — not an open public launch
- AI usage costs are predictable and manageable during pilot
- Feedback can be collected directly from known users

---

### DR-003 — Repository Identity
**Status:** Confirmed  
**Date confirmed:** Phase 0

**Decision:** The platform will be named StudyHub, owned by AlphaSoft LLC. The repository will be renamed from `bootcamp-hub` to `studyhub`.

**Rationale:** The existing name `bootcamp-hub` is too narrow for a multi-discipline platform. StudyHub accurately represents the broader mission.

**Consequences:**
- Repository rename (`AlphaSoftJB/bootcamp-hub` → `AlphaSoftJB/studyhub`) must happen at Phase 1 start
- `package.json` name, `vite.config.ts` base path, all GitHub Pages configuration must be updated
- Existing GitHub Pages URL will break — acceptable since Phase 1 moves to a new deployment target
- All internal documentation uses "StudyHub" from this point forward

---

## 2. AI Integration Decisions

### DR-004 — AI Scope (Phase 1)
**Status:** Confirmed  
**Date confirmed:** Pre-Phase 0

**Decision:** Phase 1 AI capabilities are limited to these five:
1. Lesson-specific AI Tutor (Q&A during lesson)
2. Simplified explanations (on request)
3. Examples and guided hints (on request during quiz)
4. Explanations of incorrect assessment answers (auto-triggered)
5. Revision sessions based on approved learning content

**Rationale:** These five capabilities directly support learner outcomes without requiring autonomous agents or unreviewed content generation. They are grounded in approved content and have predictable cost and risk profiles.

**Consequences:**
- Adaptive learning pathways are deferred (not Phase 1)
- Predictive mastery modelling is deferred
- Autonomous agents are deferred
- Complex personalized curriculum generation is deferred
- All five capabilities must be grounded in approved lesson content — no free-form generation
- AI-generated content in USMLE path → DRAFT state only, never auto-publish (see DR-007)

---

### DR-005 — AI Architecture (Backend Only)
**Status:** Confirmed  
**Date confirmed:** Phase 0

**Decision:** All AI calls route through the backend. No AI provider API keys are exposed to web or native clients.

**Rationale:** Exposing provider keys to clients creates cost and abuse risks that cannot be controlled. Backend orchestration enables rate limiting, usage tracking, cost controls, and provider switching without client changes.

**Consequences:**
- Backend must implement `AIOrchestrationService` interface before any AI feature ships
- Frontend makes calls to `/api/ai/*` endpoints only — never to provider APIs directly
- Native apps in Phase 2 use the same backend endpoints — no native-specific AI plumbing
- Provider can be changed (Anthropic → another) without client changes
- Rate limiting and cost controls are backend-enforced, not client-enforced

---

### DR-006 — AI Controls Required at Launch
**Status:** Confirmed  
**Date confirmed:** Phase 0

**Decision:** The following controls must be in place before any AI feature is deployed to production:
- Per-user daily usage tracking
- Per-user rate limiting (bucket per minute)
- Maximum token limits per request type (configurable)
- Cost-per-request logging in `ai_usage_log` table
- Circuit breaker (graceful degradation if AI provider is unavailable)
- Content grounding: every prompt includes approved lesson content as context

**Rationale:** Without these controls, a small number of users could incur significant costs or receive unreliable AI responses. These are non-negotiable prerequisites for pilot launch.

**Consequences:**
- Sprint 9–10 (AI Integration) must implement all six controls before moving to Sprint 11
- AI features do not ship to staging until controls are verified
- Freemium tier differentiation can use the rate-limiting infrastructure already in place

---

### DR-007 — Medical Content AI Rule
**Status:** Confirmed  
**Date confirmed:** Phase 0

**Decision:** AI-generated content on the USMLE pathway must never automatically become published educational content. AI-generated USMLE content lands in DRAFT state only. A qualified medical professional must review and approve before it reaches PUBLISHED state.

**Rationale:** Medical education errors have patient safety implications. Automated content publishing on a medical learning path is not acceptable without human expert review.

**Consequences:**
- Backend content lifecycle enforcement: USMLE + AI-generated → DRAFT only (backend enforced, not just convention)
- A qualified medical reviewer must be identified before USMLE content pipeline begins (see DR-OPEN-007)
- Content workflow UI for reviewer must be built before USMLE launch (may be post-Phase 1)
- USMLE content without a reviewer in place → no USMLE in Phase 1

---

## 3. Architecture Decisions

### DR-008 — Target Stack
**Status:** Confirmed  
**Date confirmed:** Phase 0

**Decision:** Target platform is React SPA + Spring Boot modular monolith + MySQL.

**Frontend:** React 18 + TypeScript + Vite 5 + Tailwind CSS 3  
**Backend:** Spring Boot 3.x + Java 21 + Spring Security + Spring Data JPA  
**Database:** MySQL 8.x  
**Native (Phase 2):** React Native + Expo (Managed Workflow)

**Rationale:** 
- React frontend: existing codebase, strong ecosystem, shared types with React Native
- Spring Boot: strong security ecosystem (Spring Security), mature JPA, familiar to enterprise Java ecosystem, excellent documentation for a developer learning backend
- Modular monolith: simpler to operate than microservices for a solo developer; clean domain boundaries allow future extraction if needed
- MySQL: ACID compliance, strong tooling, compatible with Spring Data JPA, low operational overhead

**Consequences:**
- Developer must learn Spring Boot if not already proficient — budget 30% extra on early backend sprints
- Microservices are not the target — do not design for them in Phase 1 or 2
- PostgreSQL is not chosen — do not use PostgreSQL-specific SQL features or assume PG in any design
- Flyway for database migrations (not Liquibase, not raw SQL)

---

### DR-009 — Authentication Pattern
**Status:** Confirmed  
**Date confirmed:** Phase 0

**Decision:** JWT access tokens stored in-memory only (never localStorage). Refresh tokens in httpOnly cookies (never accessible to JavaScript).

**Rationale:** Storing access tokens in localStorage exposes them to XSS attacks. The in-memory + httpOnly cookie pattern is the correct security baseline for web applications.

**Consequences:**
- GistSyncContext pattern of storing tokens in localStorage must NOT be replicated
- Access tokens lost on page refresh → silent refresh endpoint required (`POST /api/auth/refresh`)
- 401 handling in Axios interceptor must attempt silent refresh before redirecting to login
- Native apps (Phase 2) use a different token storage mechanism (SecureStore for Expo)

---

### DR-010 — Routing Change
**Status:** Confirmed  
**Date confirmed:** Phase 0

**Decision:** Replace `HashRouter` with `BrowserRouter` in Phase 1. Requires server-configured fallback to `index.html` for all routes.

**Rationale:** HashRouter was required for GitHub Pages static hosting (which cannot handle client-side routes). Phase 1 moves to a proper CDN or static host that supports fallback configuration. BrowserRouter produces cleaner URLs and is required for PWA and social sharing.

**Consequences:**
- GitHub Pages is no longer the deployment target for Phase 1
- Hosting provider must support SPA fallback (all modern CDN providers do)
- All existing route paths (`/`, `/cyber`, `/ibm`, `/ai`) must be remapped to the new five-area structure
- `vite.config.ts` base path `'/bootcamp-hub/'` changes to `'/'`

---

### DR-011 — State Management
**Status:** Confirmed  
**Date confirmed:** Phase 0

**Decision:** Server state via TanStack Query; UI state via Zustand; localStorage for UI preferences only (theme, font size).

**Rationale:** TanStack Query handles caching, background refresh, and loading/error states for API data, eliminating significant boilerplate. Zustand replaces the existing context-heavy pattern for UI state. localStorage retention is strictly limited to non-sensitive UI preferences.

**Consequences:**
- Remove all learning progress from localStorage (progress moves to backend)
- Remove GistSyncContext entirely (cross-device sync is now server-authoritative)
- XPContext, SettingsContext logic migrates: XP → backend; theme/fontSize → Zustand + localStorage
- No user data (progress, goals, scores) in localStorage in Phase 1 or later

---

### DR-012 — Content Lifecycle
**Status:** Confirmed  
**Date confirmed:** Phase 0

**Decision:** All content follows a five-state lifecycle: `DRAFT → IN_REVIEW → APPROVED → PUBLISHED → DEPRECATED`. Only `PUBLISHED` content is visible to learners.

**Rationale:** Content quality control is essential for an educational platform. The lifecycle ensures reviewed, accurate content reaches learners.

**Consequences:**
- Backend enforces content status filtering on all learner-facing content endpoints
- Existing bootcamp content (Cybersecurity, IBM) seeds as `PUBLISHED` after validation
- AI-generated content seeds as `DRAFT` (enforced at the AI orchestration layer)
- USMLE AI-generated content requires qualified reviewer approval before `APPROVED` (see DR-007)
- An admin/reviewer content management interface is needed before USMLE launch

---

### DR-013 — Mastery Model
**Status:** Confirmed  
**Date confirmed:** Phase 0

**Decision:** Progress is tracked as competency mastery per topic/domain — not as completion percentage.

**Rationale:** Completion percentage ("I read 80% of lessons") is a weaker signal than mastery ("I can correctly answer 85% of questions on subnetting"). Competency mastery aligns with how certification exams are structured and gives learners a more accurate picture of exam readiness.

**Consequences:**
- `user_mastery` table tracks mastery score per topic, not lesson completion count
- Home dashboard shows mastery map, not completion bar
- Assessment results feed mastery scores (a 90% exam score on "Network Protocols" domain updates that domain's mastery)
- XP and streaks remain (engagement mechanism) but mastery is the primary progress signal

---

### DR-014 — PWA Evaluation
**Status:** Deferred  
**Date deferred:** Phase 0  
**Decision required from:** Owner

**Question:** Should StudyHub support PWA (Progressive Web App) features in Phase 1?

**Analysis:** PWA would add offline study capability, home screen install, and potentially push notifications on the web before native apps exist in Phase 2. `vite-plugin-pwa` with Workbox makes this tractable. Recommendation: implement after core features are stable in Phase 1 (Sprint 11–12 scope).

**What is blocked:** Sprint 11–12 task list includes PWA implementation conditional on this decision.

---

## 4. Product Decisions

### DR-015 — Freemium Model
**Status:** Confirmed  
**Date confirmed:** Pre-Phase 0

**Decision:** StudyHub uses a freemium subscription model with localized pricing (NGN / USD).

**Rationale:** Freemium lowers the barrier to initial engagement. Localized pricing makes the platform accessible to Nigerian learners for whom USD pricing may be prohibitive.

**Consequences:**
- Backend `users` table includes `subscription_tier` field
- Payment integration: Paystack (NGN) + Stripe (USD) — deferred to post-Phase 1 (see DR-OPEN-005)
- Free tier limitations must be defined before Phase 1 launch (AI usage caps are the primary lever)
- Phase 1 pilot may be entirely free for invited users

---

### DR-016 — Visual Direction
**Status:** Confirmed  
**Date confirmed:** Phase 0

**Decision:** StudyHub uses a dark-first visual design. Core palette: deep navy/charcoal backgrounds (`#07090f` base), violet/indigo accents, teal/green for progress indicators. Typography: Inter (body) + JetBrains Mono (code). The existing design token system in `src/index.css` is preserved and extended.

**Rationale:** The existing design system is well-constructed, cohesive, and appropriate for a technical learning platform. Dark-first reduces eye strain for learners in extended study sessions.

**Consequences:**
- Light theme is supported (existing implementation preserved) but dark is the default
- New discipline-specific accent tokens extend rather than replace the existing token system
- Full visual direction document: `docs/design/STUDYHUB_VISUAL_DIRECTION.md`
- Native apps (Phase 2) are inspired by but not bound to these tokens — React Native design is built separately

---

### DR-017 — Five Navigation Areas
**Status:** Confirmed  
**Date confirmed:** Phase 0

**Decision:** StudyHub has five navigation areas: Home, Explore, Learn, Practice, My Progress.

**Rationale:** These five areas map cleanly to the learner journey: discover (Explore), engage (Home/Learn), assess (Practice), and reflect (My Progress). The existing four-item nav (Hub/Cyber/IBM/AI) is discipline-specific and does not scale to a multi-discipline platform.

**Consequences:**
- Current `NAV_ITEMS` array in `App.tsx` is replaced with five-area navigation in Phase 1
- Discipline selection moves from navigation to Explore page (browse and select, then enter Learn mode)
- Current bottom nav (mobile) and side nav (desktop) layouts are preserved but content changes
- SideNav width `w-20 lg:w-56` is an appropriate responsive pattern — preserved

---

### DR-018 — Accessibility Target
**Status:** Confirmed  
**Date confirmed:** Phase 0

**Decision:** StudyHub targets WCAG 2.2 AA compliance.

**Rationale:** Accessibility is a baseline requirement for an educational platform. AA is the standard that most accessibility regulations and procurement requirements reference.

**Consequences:**
- `eslint-plugin-jsx-a11y` added in Sprint 11–12 and all P0 violations fixed before pilot launch
- Color contrast ratios must meet AA minimums for text and interactive elements
- Keyboard navigation must work for all primary learner flows
- Screen reader support is required for lesson content, quiz questions, and navigation
- WCAG 2.2 AA audit is included in Phase 1 hardening sprint (Sprint 11–12)

---

## 5. Data and Migration Decisions

### DR-019 — Existing Content Migration Approach
**Status:** Confirmed  
**Date confirmed:** Phase 0

**Decision:** Existing bootcamp content (Cybersecurity, IBM Python+ML) migrates to the database via a one-time Node.js seed script that calls the backend content API. AI Engineering content requires structural mapping before migration can begin.

**Rationale:** A seed script validates the API contract and respects the content lifecycle middleware. Direct SQL insertion bypasses application logic and can introduce inconsistencies.

**Consequences:**
- Seed script is a Phase 1 Sprint 3–4 deliverable
- `cyberDays.js` (103 days), `ibmCurriculum.js` (57 days), all question files, and all lab files are inputs to the seed script
- AI Engineering (`aiMonths.js`, `aiQuiz.js`, `aiLabs.js`, `aiNotes.js`) blocked on structural mapping decision (see DR-OPEN-009)
- `cyberLabsNew.js` (72 entries, zero imports) requires a decision before migration (see DR-OPEN-008)
- Stable ID mapping required: `lab_done_1-1` localStorage keys → database IDs for any pilot user progress migration

---

### DR-020 — GistSync Removal
**Status:** Confirmed  
**Date confirmed:** Phase 0

**Decision:** `GistSyncContext` and all Gist sync functionality are removed in Phase 1. Cross-device progress sync is replaced by server-side storage.

**Rationale:** GistSync was acceptable for a personal single-user tool. It is not acceptable for a multi-user platform: PAT tokens in localStorage, single-user GitHub account dependency, no real-time sync. Server-side progress (see DR-011) makes GistSync obsolete.

**Consequences:**
- `src/context/GistSyncContext.tsx` is deleted in Phase 1 Sprint 1–2
- Gist sync section removed from `SettingsPanel.tsx`
- Any user who set up Gist sync on the existing bootcamp-hub tool loses that sync when they move to StudyHub — acceptable since pilot users are invited fresh
- `LS_TOKEN_KEY = 'gist_token'` localStorage key is cleared and not reused

---

## 6. Open Decisions (Required from Owner Before Phase 1 Begins)

These are explicitly unresolved. Phase 1 implementation cannot begin until all of these are answered.

| ID | Question | Impact of deferral |
|----|----------|-------------------|
| DR-OPEN-001 | Confirm timing of repository rename (`bootcamp-hub` → `studyhub`) | Blocks Sprint 1 setup |
| DR-OPEN-002 | Pilot registration: invite-only (manual), PIN/token gated, or open? | Affects Sprint 1 auth implementation |
| DR-OPEN-003 | Hosting for Phase 1 backend: Render, Railway, DigitalOcean, Fly.io? | Affects Sprint 11–12 deployment |
| DR-OPEN-004 | Static hosting for Phase 1 SPA: Cloudflare Pages, Vercel, Netlify? | Affects Sprint 11–12 deployment |
| DR-OPEN-005 | Payment integration in Phase 1 or deferred? | If Phase 1: add ~30hrs to Sprint 11–12 |
| DR-OPEN-006 | PWA support in Phase 1? (Recommended: yes, Sprint 11–12) | Affects Sprint 11–12 task list |
| DR-OPEN-007 | USMLE: who is the qualified medical content reviewer? | USMLE cannot launch without an answer |
| DR-OPEN-008 | `cyberLabsNew.js`: integrate into Cybersecurity content or discard? | Affects content seed script scope |
| DR-OPEN-009 | AI Engineering curriculum mapping: who does the structural mapping work, and by when? | AI Engineering content cannot migrate until this is done |
| DR-OPEN-010 | Pilot size: how many invited learners, and which disciplines at launch? | Informs infrastructure sizing and content completeness bar |

---

*End of STUDYHUB_DECISION_RECORD.md*
