# Implementation Plan: The Content Concierge

**Role**: Tech Lead
**Sprint**: Practice Media Strategy
**Date**: 2026-03-10
**Source Artifacts**: `04-architect/architecture.md`, `03-decide/decision-record.md`, `01-frame/problem-map.md`

---

## 1. Stack Validation

### Node.js — Primary Backend Language

**Decision**: Node.js 20 (LTS) for all three Phase 1 backend services.

**Justification**: The dominant workload is I/O-bound — Twilio webhook handling, S3 uploads, API calls to Sikka ONE and Airtable, outbound SMS. Node.js handles concurrent I/O with high throughput per container dollar. The Twilio Node SDK, AWS SDK, and Airtable SDK are all first-class and well-maintained. This also matches the Phase 2 tech stack (React Native for mobile, Next.js for dashboard), allowing shared TypeScript types and team knowledge.

**Alternatives considered**: Python (excellent for Phase 2 AI pipeline, but adds two language contexts in Phase 1), Go (fast but smaller library ecosystem for the specific integrations needed).

### PostgreSQL — Operational Database

**Decision**: PostgreSQL 15 via AWS RDS for all operational data.

**Justification**: Capture assignments, users, practices, consents, and captures are relational data with referential integrity requirements. Consent records must link correctly to patient IDs and practice IDs — foreign keys matter. PostgreSQL's JSONB support handles semi-structured playbook guidance without schema proliferation. At Phase 1 volume (10–15 practices, ~50 captures/day), a `db.t3.medium` is massively sufficient.

**Alternatives considered**: MongoDB (document model fits capture metadata, but the relational data — consents, assignments — suffers without joins), DynamoDB (wrong consistency model for consent tracking; HIPAA audit trails require strong consistency).

### AWS S3 — Media Storage

**Decision**: S3 for all raw and processed media, indefinitely.

**Justification**: HIPAA requires 7-year retention for PHI-adjacent data. S3 Intelligent Tiering handles the cost: hot tier for recent captures (under review), cool tier for processed assets, Glacier for HIPAA retention. S3's encryption (SSE-KMS), access logging, and object-level audit trail satisfy HIPAA security rule requirements. Presigned URLs (7-day expiry) allow Airtable to display thumbnails without making any media public.

**Concern**: Large video files (50–500 MB) via Twilio MMS. Twilio MMS has a 5 MB media attachment limit. Videos will come via WhatsApp (16 MB limit) or as a shared link (Phase 2 app handles this with direct S3 upload). Phase 1: accept this limitation; prompt champions to send short clips only via MMS, WhatsApp for longer video.

### Twilio — SMS + WhatsApp

**Decision**: Twilio for both SMS (MMS) and WhatsApp Business API.

**Justification**: Twilio is the only provider with both a HIPAA BAA offering and WhatsApp Business API access in a single platform. WhatsApp Business API requires Meta approval (5 business days) — submit at project start. Twilio's webhook validation (HMAC-SHA1 signature) is production-grade for inbound media. At Phase 1 volume (~100 SMS/day), cost is trivial ($0.0079/message).

**WhatsApp approval timeline risk**: WhatsApp Business API requires a Facebook Business Manager account in good standing and a phone number not previously used with WhatsApp. Begin this process in Week 1 — it can block champion onboarding if delayed.

### Airtable — Phase 1 Review Queue

**Decision**: Airtable Enterprise as the editorial review interface for Phase 1.

**Justification**: Human editors need a visual, Kanban-style interface to review incoming captures, annotate, and track publication status. Building this in-house for Phase 1 is over-engineering. Airtable's automations (trigger on record update → Slack notification) provide the workflow glue needed. Enterprise tier required for HIPAA BAA — confirm license before launch.

**Explicit constraint**: Airtable is Phase 1 ONLY. The Next.js HQ dashboard in Phase 2 replaces it entirely. Inform editors of this upfront — don't invest in complex Airtable automation that will be thrown away.

### Next.js 14 + Vercel — Admin Dashboard

**Decision**: Next.js 14 (App Router) deployed to Vercel for the internal admin dashboard.

**Justification**: The admin dashboard is primarily a read-heavy display UI with a few mutations (add practice, add champion). Server Components eliminate unnecessary client-side JS. Vercel preview deployments enable rapid iteration. This is an internal tool — SLA requirements are low; Vercel's managed infrastructure is appropriate. No need for a custom ECS deployment for a low-traffic internal app.

### Auth0 → NextAuth.js (Phase 1)

**Decision**: NextAuth.js with Google OAuth for the admin dashboard. Auth0 deferred to Phase 2.

**Justification**: Phase 1 admin users are 3–5 HQ staff. Google OAuth covers the use case with zero cost and minimal configuration. Auth0 is more powerful (RBAC, SSO federation, HIPAA compliance for patient-facing apps) but adds cost and setup overhead unnecessary for Phase 1. When Phase 2 adds practice managers and champion-level access, migrate to Auth0 for proper RBAC and session management.

---

## 2. Phase 1 Sprint Plan (6 Weeks)

### Week 1: Infrastructure + Foundation

**Backend**:
- [ ] AWS ECS cluster, VPC, subnets, security groups provisioned via Terraform
- [ ] RDS PostgreSQL instance created; schema migrations written and applied
- [ ] S3 buckets created (raw + processed) with encryption, access logging, lifecycle rules
- [ ] Secrets Manager populated with Twilio, Sikka, Airtable credentials
- [ ] Node.js monorepo scaffolded: `apps/scheduler`, `apps/capture`, `apps/admin`, `packages/db` (shared Postgres client + migrations)
- [ ] TypeScript configured; shared types for `Practice`, `User`, `CaptureAssignment`, `Capture`, `Consent`
- [ ] Twilio webhook endpoint stubbed; Twilio signature validation middleware written and tested
- [ ] S3 upload utility written and tested with test media

**Frontend**:
- [ ] Next.js 14 project created, Vercel project connected to repo
- [ ] NextAuth.js configured with Google OAuth; domain restriction to `@sgadental.com`
- [ ] shadcn/ui initialized; Tailwind theme configured (SGA brand colors)
- [ ] Admin Service API client typed (shared types from monorepo)

**Operations**:
- [ ] Twilio SMS number purchased and webhook URL registered
- [ ] WhatsApp Business API application submitted to Meta
- [ ] Airtable Enterprise base created with table structure; editors added
- [ ] Sikka ONE API test credentials obtained; test call to `/appointments` endpoint verified

---

### Week 2: Core Capture Flow

**Backend**:
- [ ] Twilio MMS webhook fully implemented: receive → download → S3 upload → PostgreSQL insert → Airtable push → confirmation SMS
- [ ] Twilio WhatsApp webhook implemented (same logic, different number format)
- [ ] SMS command parser: HELP, STOP, STATUS, SKIP
- [ ] Airtable API client: create record with presigned S3 URL; handle rate limits
- [ ] Admin Service: CRUD for practices and users; tested with Postman

**Frontend**:
- [ ] Add Practice wizard (3-step form): functional end-to-end
- [ ] Practice List page with health status cards
- [ ] Practice Detail: Overview, Champions, Config tabs

**Testing**:
- [ ] End-to-end test: send MMS from test phone → verify S3 object, PostgreSQL record, Airtable record, confirmation SMS — all within 30 seconds

---

### Week 3: Sikka ONE Sync + Daily Brief

**Backend**:
- [ ] Sikka ONE API client: `/appointments` endpoint integration with error handling and retry logic
- [ ] Nightly sync job: appointment → assignment creation logic with consent check
- [ ] Daily brief SMS job: timezone-aware scheduling, assignment aggregation, SMS formatting
- [ ] Post-op follow-up scheduler: 14/30/90-day logic
- [ ] Cron job setup in ECS Scheduled Tasks for nightly sync (11 PM UTC) and daily brief (7:30 AM local time per timezone bucket)

**Frontend**:
- [ ] Captures Monitor page: live table with SWR polling
- [ ] Assignments Monitor page
- [ ] Sync Health page: per-practice sync history, manual re-trigger

**Testing**:
- [ ] Test Sikka sync against staging PMS data for 3 practices
- [ ] Verify daily brief fires at correct local time for each timezone
- [ ] Verify post-op follow-ups trigger correctly from test data

---

### Week 4: Champion Onboarding + Pilot Go-Live

**Practices**:
- [ ] Onboard 10–15 pilot practices via Admin Dashboard
- [ ] Champions briefed via phone; test SMS sent, confirmed working
- [ ] Content consent form process established (paper Phase 1; log in `consents` table manually)

**Backend**:
- [ ] Onboarding SMS sequence implemented
- [ ] Weekly recap SMS (Monday 8 AM) implemented
- [ ] Rejection feedback SMS (triggered by Airtable automation webhook)

**Frontend**:
- [ ] Dashboard Home: metrics bar + practice grid + activity feed
- [ ] Playbook Library: guidance text editor

**Go-Live**:
- [ ] Daily briefs firing for all 10–15 practices
- [ ] CloudWatch alarms configured: webhook latency, S3 errors, sync failures
- [ ] Slack `#content-ops` alerts live
- [ ] Airtable editorial workflow operational with 2 editors

---

### Week 5–6: Stabilization + Measurement

**Focus**: Run the system, fix bugs, measure capture behavior.

- [ ] Week 5 daily capture rate analysis: which practices are capturing, which are not
- [ ] Champions not responding: re-engagement call + guidance text review
- [ ] Photo quality issues: adjust guidance text in Playbook Library based on editor feedback
- [ ] Airtable workflow friction points: adjust record schema or views based on editor feedback
- [ ] Capture → Airtable latency monitored (<30 sec target)
- [ ] Twilio delivery success rate >99% verified
- [ ] Phase 2 planning: React Native app scope locked, Phase 2 sprint kickoff

---

## 3. Phase 2 Architecture Decisions (Pre-Work for Month 2)

These decisions must be made before Phase 2 build starts. Research and validate during Phase 1 stabilization.

### Cloudinary vs. Custom Lambda for Photo Processing

**Question**: Phase 2 requires dental-specific smart crop (smile area focus), brand overlay application, and multi-format resizing. Can Cloudinary handle this or do we need custom Lambda?

**Research needed**:
- Cloudinary AI background removal + custom crop parameters: can we define "center on smile area" via landmark detection?
- AWS Rekognition FaceDetails API returns mouth coordinates — use these to define crop region, pass to Cloudinary transformation pipeline?
- Cost comparison: Cloudinary Advanced plan (~$224/month for 100K transformations) vs. Lambda + custom code (lower cost, higher dev effort)

**Recommendation direction**: Start with Cloudinary for standard formatting (resize, brand overlay). Use Rekognition mouth landmarks for crop parameters passed to Cloudinary. Custom Lambda only if Cloudinary transformation limits prove too restrictive.

### OpusClip API vs. Custom Video Highlight Detection

**Question**: OpusClip API ($0.06–0.12/minute of processed video) vs. building on top of Whisper transcription + custom scoring model.

**Research needed**:
- OpusClip API availability for enterprise/B2B use (not just creator tools)
- At Phase 2 volume (~30 videos/day), OpusClip cost: ~$90–180/month — acceptable
- Does OpusClip produce acceptable results for dental testimonial content vs. general social media content?

**Recommendation direction**: Use OpusClip for Phase 2 MVP. Build custom model in Phase 4 when we have enough labeled "high engagement dental video" data.

### Meta Graph API — Per-Practice Account Architecture

**Question**: Each of 260+ practices needs its own Meta Business Account for Instagram + Facebook distribution. How to manage OAuth tokens, access permission, and revocation at scale?

**Decision required**:
- Meta Graph API access requires each practice to explicitly authorize an SGA-managed app
- Solution: Build a "Connect your Instagram" onboarding flow in Phase 2 mobile app / Practice Concierge
- Token storage: encrypted in RDS `social_accounts` table per practice
- Token refresh: Meta long-lived tokens (60-day expiry) auto-refresh 5 days before expiry via cron job

**Action**: Register SGA Meta Business Developer app. Submit for advanced access (Pages API, Instagram Content Publishing) — approval takes 1–4 weeks. Start in Phase 1 Week 2.

### Content Review Workflow Migration (Airtable → HQ Dashboard)

**Phase 2 decision**: Build the HQ review queue in Next.js (same admin app or separate production app?)

**Recommendation**: Keep a single Next.js app. Add `/review` section to existing admin dashboard. No separate deployment. Same NextAuth session, same API backend.

---

## 4. Infrastructure as Code

All AWS infrastructure defined in Terraform (Phase 1). Structure:

```
infrastructure/
  terraform/
    main.tf              # Provider config, backend (S3 state)
    ecs.tf               # ECS cluster, services, task definitions
    rds.tf               # PostgreSQL instance, security group
    s3.tf                # Media buckets, lifecycle rules, encryption
    networking.tf        # VPC, subnets, security groups, ALB
    iam.tf               # Task execution roles, S3 access policies
    secrets.tf           # Secrets Manager resources
    cloudwatch.tf        # Log groups, alarms
    variables.tf         # Environment-specific variables
    outputs.tf           # Service URLs, RDS endpoint, etc.
  environments/
    phase1/
      terraform.tfvars   # Phase 1 sizing (t3.medium RDS, 256 CPU tasks)
    phase2/
      terraform.tfvars   # Phase 2 sizing (larger instances, Multi-AZ)
```

---

## 5. Monorepo Structure

```
content-concierge/
  apps/
    scheduler/           # Nightly sync + daily brief (Node.js)
    capture/             # Twilio webhook service (Node.js)
    admin/               # Admin REST API (Node.js/Express)
    dashboard/           # Admin web app (Next.js 14)
  packages/
    db/                  # Shared PostgreSQL client + migrations (drizzle-orm)
    types/               # Shared TypeScript types
    twilio/              # Twilio client wrapper
    sikka/               # Sikka ONE API client
    s3/                  # S3 upload utilities
    airtable/            # Airtable API client
  infrastructure/
    terraform/           # AWS infrastructure
  package.json           # Turborepo config
  turbo.json
```

**Package manager**: pnpm with pnpm workspaces.
**Monorepo tool**: Turborepo (build caching, parallel task execution).
**ORM**: Drizzle ORM (TypeScript-first, lightweight, excellent PostgreSQL support, no code generation step).

---

## 6. CI/CD

### GitHub Actions Pipelines

**PR Pipeline** (on every PR to `main`):
```
1. pnpm install
2. Type check (tsc --noEmit) across all packages
3. Unit tests (vitest) for db, twilio, sikka, s3, airtable packages
4. Build all apps (turbo build)
5. Preview deploy: dashboard to Vercel preview URL
```

**Merge to main** (production deploy):
```
1. Run PR pipeline checks
2. Deploy dashboard to Vercel (production)
3. Build Docker images for scheduler, capture, admin
4. Push to ECR
5. Update ECS service (rolling deployment, zero downtime)
6. Run smoke tests: health check endpoints, test Twilio webhook, Sikka API ping
```

---

## 7. Development Environment

**Local setup**:
```bash
# Infrastructure: run local equivalents
docker-compose up   # PostgreSQL, localstack (S3 emulation)

# Apps
pnpm install
pnpm dev  # runs all services via turborepo

# Twilio local webhooks
npx twilio-cli plugins:install @twilio-labs/plugin-serverless
npx ngrok http 8002  # expose capture service to Twilio
# Register ngrok URL in Twilio console for dev
```

**Seed data**: `packages/db/seeds/phase1-seed.ts` — creates 3 test practices, 5 test champions, sample playbooks.

---

## 8. Phase 1 Cost Estimate

| Item | Cost | Notes |
|------|------|-------|
| AWS ECS (3 small tasks) | ~$80/month | 256–512 CPU, 512MB–1GB memory |
| AWS RDS (db.t3.medium) | ~$70/month | Phase 1 sizing |
| AWS S3 + CloudFront | ~$20/month | ~500 captures × 5 MB avg = 2.5 GB/month |
| Twilio SMS | ~$50/month | ~6,000 SMS/month (15 practices, 40 messages/day) |
| Twilio WhatsApp | ~$30/month | WhatsApp session + message fees |
| Airtable Enterprise | ~$240/month | Required for HIPAA BAA; team license |
| Vercel Pro | $20/month | Admin dashboard hosting |
| Sikka ONE API | TBD | Check with Sikka for partner pricing |
| **Total** | **~$510/month** | Plus 2 human editor salaries (~$4,000/month combined) |

Note: Architecture doc cited ~$5K/month Phase 1 cost. Difference reflects human editor cost ($4K/month) + buffer for additional tooling (Sprout Social/Later for manual distribution). Infrastructure-only cost is ~$510/month.
