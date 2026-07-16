# Production hardening (Phase 6)

Run this AFTER the MVP works and the core flow is validated — not before. Each
item below is cheap insurance once there are real users and real money, and
wasted effort before that. Work through in the order listed; earlier items
protect against worse failures. For each item: implement, verify, commit.

This stack (Vercel + Supabase + Stripe) gives several of these nearly for free
— the notes say which. Verify current platform features against live docs;
tiers and defaults change.

---

## 1. Error handling and states (user-facing quality)

- Every screen has explicit **empty, loading, and error states** — no blank
  divs, no infinite spinners, no raw error dumps. This is the single biggest
  visible difference between vibe-coded and professional UI.
- A global error boundary with a friendly message and a retry path; stack
  traces visible in dev only.
- Server errors return a consistent JSON shape; the client maps known errors
  to human messages.

## 2. Abuse protection

- **Rate-limit** auth endpoints and anything expensive (AI calls, email sends,
  exports). On this stack: Vercel firewall rules, or Upstash Ratelimit in
  middleware. Without this, one hostile script can run up a real bill on any
  endpoint that calls a paid API.
- If an AI call is repeated with identical input, cache the result (a hash of
  the input as key) — cheapest cost cut available once real traffic exists.
- Validate and cap input sizes server-side (body size, file size/type on
  uploads).
- CAPTCHA or equivalent on public signup only if abuse actually appears —
  don't add friction preemptively.

## 3. Database integrity (cheap now, disaster-prevention later)

- **Constraints at the DB level**: NOT NULL where required, UNIQUE where
  logical (email), foreign keys everywhere a relation exists, CHECK
  constraints for enums/ranges. App-level validation can be bypassed; the
  database is the last line.
- **Schema changes go through migration files** in `supabase/migrations/`,
  never ad-hoc dashboard edits against production — an untracked schema
  cannot be reproduced, staged, or rolled back.
- **Audit fields** on every table: `created_at`, `updated_at` (trigger-set).
  Add `deleted_at` only if soft delete is genuinely needed.
- **Transactions** around any multi-step write that must succeed or fail
  together (e.g. create order + decrement credit). In Supabase, use a Postgres
  function (RPC) for atomic multi-table writes.
- Indexes on every column used in WHERE/ORDER BY of hot queries, and on every
  foreign key. Check the Supabase query performance page for slow queries.

## 4. Backups and recovery

- Supabase: daily backups exist on paid plans; **Point-in-Time Recovery** is
  the setting that matters — enable it once real user data exists. Verify
  what the current plan includes; do not assume.
- Know the restore procedure before you need it. Once, actually test restoring
  to a new project.
- Vercel deployment rollback: know that reverting to a previous deployment is
  one action in the dashboard.

## 5. Observability (know when it breaks before users tell you)

- **Error tracking**: add Sentry (or equivalent) to both client and server.
  Vibe-coded apps fail silently; this is how you find out.
- A `/api/health` endpoint that checks DB connectivity — used by uptime
  monitoring (e.g. a free ping service) that alerts you when the site is down.
- Structured logs on key actions (signup, payment, core action) with enough
  context to debug: user id, timestamp, outcome. Never log secrets, tokens, or
  full personal data.
- Basic product analytics on the core flow (even just Vercel Analytics or
  PostHog) so "is anyone using this" has an answer.

## 6. Staging and CI

- **A staging environment**: a separate Supabase project + Vercel preview/
  branch deploys with staging env vars and Stripe test keys. Schema changes
  and risky features go here first. Never test against production data.
- **CI on every push** (GitHub Actions): lint, typecheck, build, tests. A
  10-line workflow that blocks broken code from deploying is worth more than
  any amount of care.
- A **smoke test** script that hits /health, signs in a test user, and runs
  one create/read path — run it in CI and after every deploy.

## 7. Admin capability (you can't run a SaaS without it)

- Minimum viable admin: a way for YOU to look up a user, see their records,
  fix a broken state, and process a refund — even if v1 of this is a protected
  page plus Supabase dashboard access and the Stripe dashboard.
- Admin routes gated by a real role check (an `is_admin` flag checked
  server-side, or Supabase custom claims) — never a hardcoded email in client
  code, never an unlinked-but-guessable URL.
- An audit trail for admin actions once more than one person has access.

## 8. Payments hardening (beyond the v1 gates)

- Handle the full Stripe lifecycle, not just checkout: failed payments,
  subscription cancellation, plan changes, refunds. Every one of these arrives
  as a webhook event the app must process.
- Idempotent webhook handling: Stripe retries events; processing the same
  event twice must not grant double credit. Store processed event IDs.
- Reconciliation: a periodic check (even manual) that Stripe's view of who is
  paid matches the database's view.

## 9. Compliance and trust surface

- Privacy policy and terms pages — required by Stripe, ad platforms, and most
  jurisdictions once collecting personal data. For India-targeted products,
  check current DPDP Act obligations; for EU users, GDPR basics (data export
  and deletion on request).
- A working support contact (email is fine) visible in the app.
- User data deletion: a way to actually delete a user and their data on
  request — much easier to build when the schema is small.

## 10. Transactional email (users cannot recover accounts without it)

- Password reset and email verification need a real sender. Supabase's
  built-in mailer is heavily rate-limited (a few emails/hour) and intended for
  development only — configure custom SMTP in Supabase auth settings via a
  provider like Resend, Postmark, or SES, with a verified sending domain.
- Verify the reset flow end-to-end on the deployed URL: request reset, receive
  the email, land back in the app, set a new password.
- Receipts and invoices: enable Stripe's built-in email receipts instead of
  building your own.
- Product email (digests, notifications) comes later, only when the core loop
  demands it.

## 11. Background work and scheduled jobs

- Anything slower than a request/response — bulk exports, digest emails, long
  AI runs, cleanup — moves out of the request path. On this stack: Vercel cron
  hitting a route handler, Supabase `pg_cron` or Edge Functions, or a managed
  queue (Inngest, QStash) once retries and observability matter.
- Every job must be idempotent (re-running it is safe) and logged with an
  outcome; add a stuck-job check to monitoring.

## 12. Teams, roles, and the enterprise ladder

- **Multi-seat**: the moment a second seat per account is real, introduce an
  `orgs` + `memberships` model — RLS keyed to membership, roles
  (owner/admin/member) checked server-side. Retrofitting an org layer onto
  user-owned tables is one of the most painful migrations on this stack, so do
  it early even if every org has one member at first.
- **Enterprise asks** — SSO/SAML (WorkOS or Supabase SSO), audit log export,
  data residency, SOC 2 — are sales-gated, not architecture-gated: sell first,
  build when a signed contract demands the item, in that order.

## 13. Pre-launch release checklist

- [ ] Full `security-checklist.md` passes on production (RLS second-user test
      against the prod database).
- [ ] Staging tested: core flow, mobile viewport, wrong inputs, empty states.
- [ ] Stripe in live mode: real card test, webhook pointing at prod URL,
      idempotency verified.
- [ ] Error tracking receiving events; uptime monitor active; health check
      green.
- [ ] Backups/PITR enabled and restore procedure written down.
- [ ] Password reset works end-to-end on the production URL with a real
      email provider (not Supabase's dev mailer).
- [ ] Rollback known: previous Vercel deployment restorable in one step.
- [ ] Terms, privacy, support contact live.
- [ ] The one metric that defines success is being tracked.

---

## What deliberately stays out

Kubernetes, microservices, multi-region, queues, and load testing are not
solo-founder v1 problems on this stack. Vercel and Supabase handle scaling
concerns well past the first thousands of users. Revisit only when a specific
measured bottleneck appears — not because an architecture article said so.
