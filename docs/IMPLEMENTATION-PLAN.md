# SKILLEZE INTERVIEW HUB — Production Implementation Plan

**Organization:** SKILLEZE TECHNOLOGIES, powered by DIGITEEASE  
**Domain:** https://interview.digiteease.com  
**Repository:** https://github.com/Digiteease/SKILLEZE-INTERVIEW-HUB  
**Timezone:** Asia/Kolkata

## Confirmed decisions
- Login: email/password and Google OAuth.
- Meetings: Google Meet.
- Candidate access: public applications and invitation-only candidates.
- Assessments: HR, technical, practical coding, communication.
- AI: Google Gemini through server-side API only.
- Deployment: existing GitHub and hosting accounts.
- UI: professional light theme.
- Roles: Admin, HR/Interviewer, Candidate, and restricted Sourcing Admin permission.

## Programme catalogue — 17 specializations
1. Agentic AI Program
2. AI for Business
3. AI Mastery Program
4. Generative AI
5. No-Code Automation
6. Prompt Engineering
7. Graphic Design
8. Digital Marketing Using AI
9. Advanced Video Editing Using AI
10. Application Development
11. Data Analysis
12. Full Stack .NET Development
13. Full Stack Python Development
14. Full Stack Web Development
15. MERN Stack Development
16. Web Designing
17. Web Development

Each programme gets its own competency matrix, question bank, practical-task template, scoring rubric and suggested follow-up questions.

## Architecture
- Frontend: React + Vite, responsive candidate, HR/interviewer and admin portals.
- API: Node.js + Express with schema validation, rate limits and audit events.
- Database: PostgreSQL / Supabase with migrations and row-level security.
- Authentication: Supabase Auth email/password + Google OAuth.
- Storage: private resume and assessment storage with short-lived signed URLs.
- AI: Google Gemini through a server-side adapter; never expose keys in browser code.
- Meetings: Google Calendar API with Meet conference creation and organizer OAuth.
- Email: transactional notifications for applications, invitations, reminders and status changes.
- Hosting: Vercel frontend, Node-compatible API hosting and managed PostgreSQL.
- Domain: interview.digiteease.com with HTTPS.

## Portals

### Candidate
Public application and invitation flow, email verification, profile, programme selection, resume upload, consent, application status, interview schedule, Meet link, instructions and assessment submission.

### HR / Interviewer
Assigned candidate leads, resume access, interview schedule, question bank, HR/technical/coding/communication assessments, scorecards, evidence-based notes and feedback.

### Admin
User and role management, programmes, interview templates, schedules, reports, integrations, retention settings and restricted source-provenance access.

## Lead-source privacy — critical requirement
HR must see the candidate lead and application details, but must NOT see the job portal/provider that supplied the lead.

Keep provenance in a separate restricted table, such as application_source_private, not in the normal candidate record. Enforce access on the server and in database RLS:
- Admin/Sourcing Admin with the explicit lead_sources.read permission may see provider/source, external application ID and ingestion integration.
- HR/Interviewer sees candidate/application data only. Source fields must not be included in their API responses, search results, exports, reports, notifications or audit views.
- Candidate cannot see internal provenance.
- Do not rely on hiding a UI label. Add automated tests for API, exports and database access.
- Audit every restricted source-provenance read.

## Job-board lead ingestion
Use approved official APIs, partner integrations, documented webhooks or permitted exports. Do not scrape portals or bypass their access controls/terms.

### Indeed
Indeed documents Candidate Sync APIs and Indeed Apply application delivery. Production access requires partner onboarding and the appropriate credentials/scopes. Until approved, mark Indeed integration as pending and use a permitted CSV/manual import workflow.
Official documentation:
- https://docs.indeed.com/
- https://docs.indeed.com/api-guides/
- https://docs.indeed.com/candidate-sync-apis/send-candidates-api
- https://docs.indeed.com/job-sync-api/integrate-with-job-sync-api

### Other job portals
Create a provider adapter for each source only after confirming API/webhook/export authorization. Each adapter needs its own credentials, field mapping, signature validation where supported, idempotency key, retry queue, deduplication rules and integration-health monitoring.

### Ingestion pipeline
1. Receive HTTPS webhook or authorized import.
2. Verify signature/token and timestamp where supported.
3. Enforce idempotency and provider-specific duplicate rules.
4. Normalize only necessary candidate/application fields.
5. Store source provenance separately with restricted access.
6. Quarantine malformed records for admin review.
7. Notify HR of the new candidate without disclosing source.
8. Monitor retries, failures and delivery latency.

## Gemini capabilities
- Generate programme-specific HR and technical questions.
- Draft interview plans and follow-up questions.
- Summarize answers and interviewer notes with evidence references.
- Suggest rubric-aligned feedback for human review.
- Assess answer relevance, completeness and technical correctness.
- Identify possible text similarity only where there is a defensible comparison corpus and lawful basis.

AI-authorship warning: text-only AI detectors cannot reliably prove authorship. AI indicators must not be a guilt score, automatic failure, rejection or ranking signal. Use them only as non-decisional prompts for neutral follow-up questions. Disclose AI processing, obtain appropriate consent and minimize personal data sent to Gemini.

Gemini references:
- https://ai.google.dev/gemini-api/docs/function-calling
- https://ai.google.dev/gemini-api/docs/structured-output

## Suggested database domains
profiles, roles, permissions, role_permissions, programmes, competencies, question_bank, assessment_templates, candidates, applications, candidate_consents, candidate_documents, application_source_private, interviews, interview_participants, meetings, assessment_attempts, answers, coding_submissions, evaluations, evaluation_criteria, ai_review_runs, ai_review_evidence, integration_accounts, webhook_events, ingestion_jobs, audit_events, notifications and retention_jobs.

## Delivery phases
### Phase 0 — Repository and environment
Confirm deployment targets, environments, secrets, branch protection, CI, linting, tests and migration workflow.

### Phase 1 — Secure foundation
PostgreSQL migrations, Supabase Auth, Google OAuth, role permissions, RLS, private file storage, signed URLs, consent records and audit logs.

### Phase 2 — Candidate and programme workflows
Programme catalogue, public application, invitation links, resume intake, candidate profiles, duplicate handling, status tracking and email notifications.

### Phase 3 — Interview operations
Interview scheduling, interviewer assignment, Google Calendar/Meet, question banks, HR/technical/communication/coding assessments, rubrics and reports.

### Phase 4 — Lead ingestion
Restricted source provenance, Indeed integration after approval, adapters for other authorized portals, signed webhooks, idempotency, retries, import review and HR privacy tests.

### Phase 5 — Gemini
Server-only Gemini integration, structured outputs, prompt/version logging, programme-specific question generation, summaries, rubric assistance and human review.

### Phase 6 — Production hardening
Security and privacy review, RLS tests, backup/restore, load/accessibility/mobile/end-to-end tests, domain/TLS, monitoring, retention and deletion workflows.

## Acceptance criteria
1. Candidate can apply publicly or accept an invitation, verify email and track status.
2. Admin can manage programmes, users, interviews and assignments.
3. HR can manage assigned leads but cannot retrieve source metadata through UI, API, exports or database queries.
4. Authorized Admin/Sourcing Admin can view provenance and access is audited.
5. Job-board ingestion uses approved authenticated channels and is idempotent.
6. Google Meet links are created through authorized Google Calendar integration.
7. All four assessment types are supported.
8. Gemini outputs are advisory, traceable and human-editable.
9. No AI signal can automatically reject a candidate.
10. Production release passes security, privacy, accessibility and end-to-end tests.

## Owner-provided access still required
Configure secrets in encrypted deployment settings, never in chat or source control:
- Supabase URL, anon key and server-only service-role key.
- Google OAuth client credentials and redirect URIs.
- Google Calendar API enablement and organizer authorization.
- Gemini API key.
- Transactional email API key and verified sender domain.
- Indeed partner approval, credentials and scopes when granted.
- Credentials/approval for each other portal.
- Vercel/API-host and DNS access for interview.digiteease.com.

## Repository status
The repository was checked before implementation and contained only a minimal README. This document is the first tracked implementation specification; it does not claim that external integrations or production deployment are already active.
