# Lead Ingestion and Source Privacy

## Product rule
HR users must see the candidate lead and application details, but must not see the source/provider that supplied the lead. Only users with the explicit lead_sources.read permission (normally Admin/Sourcing Admin) may view source provenance.

## Data separation
Keep provenance outside the ordinary candidate/application record in a restricted application_source_private table:
- id
- application_id
- provider_code
- provider_application_id
- provider_job_id
- ingestion_integration_id
- received_at
- payload_fingerprint
- dedupe_key
- created_at

Do not add source, source_name, job_board or provider IDs to the HR-facing candidate response.

## Authorization matrix

| Capability | Candidate | HR / Interviewer | Admin / Sourcing Admin |
|---|---:|---:|---:|
| View own application | Yes | No, except assigned workflow | Yes |
| View assigned candidate lead | No | Yes | Yes |
| View source provenance | No | No | Yes, with permission |
| Export candidate list | Own data only | Candidate fields only | Permission-controlled |
| Manage integrations | No | No | Yes |
| Read ingestion failures | No | No | Yes |

## Enforcement
1. Use database RLS to restrict source-provenance rows.
2. Enforce permissions in every API handler; UI hiding is not security.
3. Use separate response DTOs/allowlists for HR and Admin.
4. Filter exports, search indexes, analytics, emails, notifications and audit views.
5. Keep raw provider payloads in restricted integration logs; never copy them into candidate notes.
6. Audit every provenance read and export.
7. Test that HR responses contain no source fields and source endpoints return 403/404.
8. Do not expose source through filenames or portal-specific application IDs.

## Ingestion
Receive webhook over HTTPS, verify authenticity, enforce idempotency, normalize only necessary fields, store candidate data separately from provenance, quarantine malformed records, retry according to provider contract, and notify HR without naming the source.

## Source onboarding
Prefer official APIs, approved partner integrations and documented webhooks. Use CSV/manual handoff only where terms allow. Do not scrape job boards or bypass login controls. Keep integrations disabled until credentials, terms and privacy notice are approved.

## Indeed
Indeed Partner Docs describe Candidate Sync APIs and Indeed Apply application delivery. Access requires partner onboarding and suitable OAuth credentials/scopes. It is not a public anonymous feed.
- https://docs.indeed.com/
- https://docs.indeed.com/api-guides/
- https://docs.indeed.com/candidate-sync-apis/send-candidates-api
- https://docs.indeed.com/job-sync-api/integrate-with-job-sync-api

## Privacy and retention
Publish a candidate privacy notice covering purposes, data categories, recipients, AI use, retention and contact process. Obtain appropriate consent/lawful basis before sending content to external AI services. Minimize personal data sent to Gemini. Define retention/deletion for applications, resumes, recordings and provider payloads. Recording/video capture is opt-in with notice and off by default.
