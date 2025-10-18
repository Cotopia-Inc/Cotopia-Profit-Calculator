README.md
=========

Last updated: October 2025  
Project: Cotopia Profit Calculator  
Repository: cotopia/profit-calculator (example)  
Contact: support@cotopia.org

Table of contents
-----------------
- Project overview
- Key features
- Audience & use cases
- Repository structure
- Quickstart — run locally
- Installation & prerequisites
- Configuration
- Development workflow
- API (HTTP) reference — summary
- Frontend widget / SDK
- Embedding & integration examples
- Deployment & hosting
- Data model & persistence
- Security, secrets & compliance
- Testing strategy
- Observability & monitoring
- Billing & payments integration
- Licensing & distribution
- Contributing
- Release & changelog policy
- Support & contacts
- Appendix: sample env file

Project overview
----------------
Cotopia Profit Calculator is a web-based tool for building pricing, cost and profitability scenarios, exporting results (CSV/XLSX/PDF), embedding calculators into other sites, and integrating with ecommerce and accounting systems. It supports self-serve SaaS, developer SDK/widget distribution, and enterprise/private deployments.

Key goals
- Rapidly compute pricing scenarios and margins with clear breakdowns.  
- Export and share scenarios (CSV, XLSX, PDF).  
- Provide embeddable widget and SDK for developers and agencies.  
- Offer multi-tenant SaaS and options for on-prem/private instances for enterprise customers.  
- Instrument product usage and hooks for billing/integrations.

Key features
------------
- Scenario builder: inputs for costs, variable & fixed expenses, taxes, fees, discounts, currencies.  
- Batch CSV import/export and bulk scenario processing.  
- Charts & visualizations (Chart.js or similar) for margin, break-even and sensitivity analysis.  
- Embeddable widget (JS) + npm SDK for programmatic integration.  
- Account management, multi-seat/team billing and roles.  
- Audit logs, versioned scenarios and export history.  
- Webhooks and integrations (Stripe, Shopify, Zapier, QuickBooks/Xero).  
- Admin dashboard for metrics, billing and customer management.

Audience & use cases
--------------------
- Founders, ecommerce sellers and product managers calculating price vs margin.  
- Agencies integrating calculators into client dashboards and white-labeling tools.  
- Finance teams requiring exports and audit trails.  
- Investors or analysts wanting quick scenario modeling.

Repository structure
--------------------
(Suggested layout — adapt to your stack)
- /apps
  - /web — React/Vue/Next.js frontend (SPA or SSR)
  - /api — REST/GraphQL backend (Node/Express, Nest, or Python/Flask/FastAPI)
  - /worker — background jobs (e.g., exports, email, PDF generation)
- /packages
  - /widget — embeddable JS widget (UMD + ESM)
  - /sdk-js — npm SDK for client integrations
  - /sdk-python — Python SDK (optional)
- /infrastructure — IaC (Terraform / CloudFormation) and deployment configs
- /docs — markdown docs, openapi, dpa and legal templates
- /scripts — helper scripts (migrations, seeding)
- /tests — end-to-end and integration test suites
- /config — shared config templates
- /LICENSE.md, /README.md (this file)

Quickstart — run locally
------------------------
0. Prereqs: Node 18+, pnpm/yarn/npm, Docker (optional), PostgreSQL, Redis.
1. Clone repository:
   git clone git@github.com:cotopia/profit-calculator.git
   cd profit-calculator
2. Copy and edit environment:
   cp .env.example .env
   # fill values (DB URL, REDIS URL, STRIPE keys, JWT secret)
3. Start dev services (recommended):
   docker-compose up -d   # sets up postgres, redis, localstack if used
4. Install and run backend:
   cd apps/api
   pnpm install
   pnpm dev           # or `npm run dev`
5. Run frontend:
   cd ../../apps/web
   pnpm install
   pnpm dev           # open http://localhost:3000
6. Optional: start worker
   cd ../../apps/worker
   pnpm install
   pnpm dev

Installation & prerequisites
----------------------------
- Node.js (LTS recommended), package manager (pnpm/yarn/npm).  
- PostgreSQL (13+ recommended) for primary DB. Use managed RDS/Aurora in production.  
- Redis for caching, rate limiting and job queue.  
- S3-compatible object storage for attachments/exports.  
- Stripe account for billing; configure webhook endpoints.  
- Optional: SMTP provider (SendGrid/Mailgun) for transactional emails.  
- Optional: Sentry/Datadog/Prometheus for monitoring.

Configuration
-------------
Key env vars 
- DATABASE_URL=postgres://user:pass@localhost:5432/cotopia
- REDIS_URL=redis://localhost:6379
- APP_ENV=development
- PORT=3000
- JWT_SECRET=supersecret
- STRIPE_SECRET_KEY=sk_test_xxx
- S3_BUCKET=cotopia-exports
- SMTP_HOST=smtp.sendgrid.net
- SENTRY_DSN=
- ANALYTICS_WRITE_KEY=
Store secrets in a secrets manager (AWS Secrets Manager, Vault) in production.

Development workflow
--------------------
- Feature branches: feature/<short-desc> (rebase on main, open PR)  
- Commit message style: Conventional Commits (feat/, fix/, chore/, docs/)  
- CI: run linters, typechecks, unit tests, and build checks on PRs (GitHub Actions recommended)  
- Main branch: protected; require PR review + passing CI.  
- Releases: semantic versioning; tag releases and publish changelog.

API (HTTP) reference — summary
------------------------------
Provide OpenAPI spec in /docs/openapi.yaml. Summary endpoints:

Auth
- POST /api/v1/auth/register — register account (email/password or SSO)
- POST /api/v1/auth/login — returns JWT
- POST /api/v1/auth/refresh — refresh token

Users & Accounts
- GET /api/v1/me
- PATCH /api/v1/me
- GET /api/v1/accounts/:id (enterprise/admin)

Scenarios
- GET /api/v1/scenarios — list (with filters)
- POST /api/v1/scenarios — create
- GET /api/v1/scenarios/:id — read
- PATCH /api/v1/scenarios/:id — update
- DELETE /api/v1/scenarios/:id — delete
- POST /api/v1/scenarios/:id/export — request export (CSV/XLSX/PDF)

Batch & Import
- POST /api/v1/imports — upload CSV for batch scenarios

Billing & Subscriptions
- POST /api/v1/billing/checkout-session — Stripe checkout session
- GET /api/v1/billing/invoices
- POST /api/v1/billing/webhook — Stripe webhook endpoint

Webhooks & Integrations
- POST /api/v1/webhooks — register webhook (user)  
- POST /api/v1/webhooks/:id/test — test delivery

Admin & Metrics
- GET /api/v1/admin/metrics — access-controlled metrics

Frontend widget / SDK
---------------------
- Widget (UMD+ESM) supports initialization via:
  <script src="https://cdn.cotopia.org/widget/vX.Y.Z/cotopia-widget.min.js"></script>
  <script>
    CotopiaWidget.init({
      container: '#widget',
      publicKey: 'pk_live_xxx',
      prefill: { price: 29.99, currency: 'USD' },
      onExport: (data) => console.log('exported', data),
    })
  </script>

- SDK (npm) usage (JS example):
  import Cotopia from '@cotopia/sdk';
  const client = new Cotopia({ apiKey: process.env.COTOPIA_API_KEY });
  const scenario = await client.scenarios.create({...});

Embedding & integration examples
--------------------------------
- Shopify: use app proxy or embedded app SDK; push SKU/costs to calculator via SDK.  
- Wordpress: iframe or script tag embed with postMessage for data sync.  
- API-first: use server-side SDK to generate prefilled scenarios for authenticated users.

Deployment & hosting
--------------------
Production recommendations:
- Frontend: build and serve from CDN (Vercel, Netlify, S3+CloudFront).  
- Backend: containerized services behind autoscaling group (ECS/Fargate, GKE) or serverless (e.g., AWS Lambda with API Gateway) depending on request patterns.  
- DB: managed PostgreSQL (RDS/Aurora) with automated backups and read replicas.  
- Cache & queues: managed Redis (Elasticache) and worker autoscaling for exports.  
- Storage: S3 for exports/uploads with signed URLs, lifecycle rules.  
- CI/CD: GitHub Actions -> deploy to environments (staging, prod). Blue/green or canary deploys recommended.  
- Secrets: AWS Secrets Manager / Vault.  
- Backups: daily DB backups with 30–90 day retention; test restore runbooks.

Data model & persistence
------------------------
High-level entities:
- users (id, email, name, role, account_id)
- accounts/organizations (id, name, billing_plan, stripe_customer_id)
- scenarios (id, account_id, name, inputs JSONB, outputs JSONB, created_by, version)
- exports (id, scenario_id, format, s3_key, generated_by)
- audit_logs (entity, action, data, user_id, timestamp)
- webhooks (id, account_id, url, events)

Use JSONB columns for flexible scenario inputs/outputs; index common query paths.

Security, secrets & compliance
------------------------------
- Use TLS (1.2+) everywhere.  
- JWT or OAuth2 (access/refresh token) for APIs; use short-lived access tokens + refresh tokens.  
- Hash passwords with Argon2 or bcrypt (salted).  
- Role-based access control (RBAC) for account/team permissions.  
- Input validation & output encoding to prevent injection/XSS.  
- Rate-limiting on public endpoints, especially contact and auth.  
- Vulnerability scanning for dependencies (Dependabot/Snyk).  
- Regular pentesting and third-party audits for enterprise customers.  
- DPA & SCCs available for EU/EEA customers; consider ISO27001 or SOC2 if pursuing enterprise market.

Testing strategy
----------------
- Unit tests for core business logic (scenario calculations).  
- Integration tests for API endpoints (use test DB fixtures).  
- End-to-end tests (Playwright / Cypress) for critical user flows (signup, create scenario, export).  
- Contract tests for SDK and widget.  
- Static analysis, type checks (TypeScript), linting (ESLint/Prettier).  
- CI to run tests on PRs and require coverage thresholds (e.g., 80% on non-frontend glue code).

Observability & monitoring
--------------------------
- Error tracking: Sentry.  
- Metrics: Prometheus / Datadog for request latency, error rates, worker queue depth.  
- Logs: structured logs (JSON) to centralized logging (CloudWatch, Datadog).  
- Alerts: set up SLOs/SLAs and PagerDuty/opsgenie for on-call escalation.  
- Health checks & readiness probes for containers.

Billing & payments integration
------------------------------
- Use Stripe for subscriptions, invoices, coupons, and trials.  
- Use webhooks to react to invoice.payment_succeeded, invoice.payment_failed, customer.subscription.deleted.  
- Store Stripe customer IDs and subscription IDs on account records; do not store raw PANs.  
- Support free, pro, team, agency and enterprise tiers. Provide billing portal (Stripe billing or custom) for management.

Licensing & distribution
------------------------
- Open-source core components may be licensed under GPLv3 (or MIT for SDKs). Decide and document in LICENSE.md.  
- Dual-licensing: offer commercial licenses for white-label/enterprise redistribution. Contact support@cotopia.org for commercial licensing.  
- Include copyright header in source files.

Contributing
------------
  
- Create issues for bugs or feature requests. Fork → branch → PR.  
- Run tests and linters locally. Provide clear PR description and link associated issues.  
- Maintainers will review PRs and merge after CI passes and at least one approving review.

Release & changelog policy
--------------------------
- Follow semantic versioning (MAJOR.MINOR.PATCH).  
- Use a changelog (CHANGELOG.md) with Keep a Changelog style.  
- Tag releases and create GitHub release notes with migration guide if needed.

Support & contacts
------------------
- Support / help: support@cotopia.org  
- Billing: billing@cotopia.org  
- Security: security@cotopia.org (PGP key on request)  
- Press: press@cotopia.org  
- Investors: investors@cotopia.org

Appendix: sample .env.example
-----------------------------
DATABASE_URL=postgres://cotopia:password@localhost:5432/cotopia
REDIS_URL=redis://localhost:6379
APP_ENV=development
PORT=3000
JWT_SECRET=change_me_to_a_strong_secret
STRIPE_SECRET_KEY=sk_test_xxx
STRIPE_WEBHOOK_SECRET=whsec_xxx
S3_BUCKET=cotopia-exports-dev
S3_REGION=us-west-2
SMTP_HOST=smtp.sendgrid.net
SMTP_USER=apikey
SMTP_PASS=sendgrid_api_key
SENTRY_DSN=
ANALYTICS_WRITE_KEY=
FRONTEND_URL=http://localhost:3000
BACKEND_URL=http://localhost:4000


