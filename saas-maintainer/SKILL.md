---
name: saas-maintainer
description: >-
  Debug, fix, or add features to an EXISTING app that already runs, on a
  Next.js + React + Supabase + Stripe stack. Use this skill WHENEVER the user
  reports a bug, error, or broken behavior in an app that already exists, or
  asks to change/extend/add to a codebase that's already built — phrases like
  "there's a bug", "X isn't working", "fix the...", "why does...", "add X to my
  app", "the webhook stopped firing", "users are seeing...", "the build broke".
  This is the counterpart to saas-builder: builder is for brand-new projects
  from an idea; this skill is for changing code that already exists and may
  have live users and real data.
---

# SaaS Maintainer

For working on an app that already exists. The failure modes here are the
opposite of greenfield: the risk is not *forgetting* to add something, it is
*breaking* something that already works — editing code you never read, fixing a
symptom instead of the cause, or running a destructive migration on live data.

Two core principles:

1. **Understand before you change.** You are a guest in a codebase you didn't
   write this session. Read the relevant code and reproduce the problem before
   touching anything.
2. **Smallest safe change.** Change the minimum needed. A three-line fix you
   understand beats a fifty-line refactor you hope works.

Default stack assumption: Next.js (App Router) + React + TypeScript + Tailwind +
Supabase + Stripe + Vercel. Adjust if the repo says otherwise.

---

## Phase 0 — Orient (always, both paths)

Before deciding anything:

- Read the project instruction file — `CLAUDE.md` or `AGENTS.md`, whichever
  exists (both, if both) — it carries the conventions and the "Do NOT" rules.
- For feature work, also read the product decision docs when present:
  `docs/spec.md` (scope + NOT-in-v1 fence), `docs/blueprint.md`
  (features/screens/flows), `docs/design.md` (visual direction). A feature
  added against stale or unread docs is how a product loses its shape.
- Get a quick map of the area involved: which routes, which tables, which
  files. Skim, don't rewrite.
- Classify the request: **bug fix** or **feature add**? They take different
  paths below. If it's genuinely both ("this is broken and while you're there
  add X"), do the fix first, ship it, then treat the addition as a separate
  feature.

Never start editing blind. If you can't yet point to the file and line that
matters, you are not ready to change it.

---

## Incidents — when users are being harmed RIGHT NOW

If production is down, data is leaking, or payments are failing, the
reproduce-first rule inverts: **mitigate first, diagnose second.**

1. **Stop the harm:** revert to the previous deployment (one action in
   Vercel), or disable the offending feature/route. Do not debug a live
   fire.
2. **Preserve evidence before it rots:** error tracker snapshot, relevant
   logs, the deploy that broke it, timestamps.
3. **Tell the user the state plainly:** what's affected, what was done,
   what's next.
4. Once stable, run Path A properly on the preserved evidence — the
   mitigation was not the fix.
5. **Close with a prevention, not just a note:** a regression test, an
   alert, or a rule in the instruction file. An incident that leaves only
   a memory will repeat.

The security rules hold even mid-incident: never disable RLS, skip webhook
verification, or expose a secret to stop the bleeding — that trades an
outage for a breach.

## Path A — Fixing a bug (reproduce first)

### 1. Reproduce
Get the exact symptom: steps to trigger, the full error text, and expected vs
actual behavior. Reproduce it yourself if at all possible — locally, with a test
request, or by tracing the exact code path for the reported input. **If you
cannot reproduce it, you cannot know your fix worked.** Say so and work with the
user to get a reliable repro before changing code.

### 2. Locate and read
Find the real code path the failing behavior runs through. Read it — and the
code immediately around it — before forming a fix. Many "bugs" on this stack are
misread behavior; see `references/debugging-playbook.md` for the common
signatures (an empty result that's actually RLS doing its job, a stale value
that's actually Next.js caching, a webhook that's actually a signature mismatch).

### 3. Root cause, not symptom
State the actual cause in one sentence before fixing. If you're about to add a
`try/catch` that swallows the error, or a `?? []` that hides an empty result,
stop — that's papering over the symptom. Fix the cause.

### 4. Smallest fix, then verify
Make the minimal change. Then:
- Re-run the reproduction steps — the bug is gone.
- **Regression check**: find what else uses the code you changed and confirm you
  didn't break a neighbor (see blast-radius guidance in
  `references/change-safety.md`).
- Commit with a message that names the actual cause.

---

## Path B — Adding a feature to an existing app

### 1. Mini-spec
One short paragraph: what the feature does, the user flow, and — explicitly —
**what it touches**: new tables? a migration? changes to shared components or
existing API routes? Payments? This "what it touches" answer decides how careful
the rest of the work is. Two rules from the build phase still bind here:

- **The write side implies a read side.** If the feature creates or changes
  data, name where each affected role sees and manages it (list/detail,
  lifecycle states, who may act on it) — that surface is part of the same
  feature, not a follow-up.
- **Docs stay true.** If the change alters roles, tenancy, billing, or a
  core workflow, update `docs/spec.md` (and `docs/blueprint.md` if present)
  BEFORE coding, so the documents keep describing the real product.

### 2. Find the seam
Locate where the feature plugs in. Read the neighboring code and match its
existing patterns (file naming, data-access style, component structure) rather
than importing a new convention. Consistency keeps the codebase reviewable.

### 3. Build in small steps
Implement in individually-committable pieces, verifying each in the browser or
with a test. For each piece that touches data, auth, or payments, apply the
matching gate below.

### 4. Verify feature AND neighbors
Confirm the new feature works, then confirm you didn't break the flows next to
it. New code that breaks old behavior is a net loss. Any new or changed screen
ships with its empty, loading, and error states and gets a mobile-width check —
the same bar the saas-builder skill sets for new screens.

---

## Gates that apply to every change

Run the relevant one before committing. These are the things that quietly go
wrong when changing a live app.

- **Never fix by weakening security.** Do not disable RLS, loosen a policy to
  "get past" an empty result, skip webhook signature verification, or move a
  secret client-side to make something work. A security control that's "in the
  way" is usually surfacing the real bug, not causing it. If you believe a
  control must change, flag it to the user explicitly and explain the tradeoff —
  never do it silently.
- **New user-data tables get RLS.** Any table you add gets RLS enabled and
  owner-scoped policies, tested with a second user — same standard as a new
  build. The greenfield hole reappears every time a table is added later.
- **Migrations on live data are dangerous.** Prefer additive, reversible
  changes (add a nullable column, add a table). Renaming or dropping columns,
  changing types, or deleting rows can destroy real user data and often can't be
  undone. For anything destructive: confirm with the user first, ensure a
  backup/point-in-time recovery exists, and prefer a safe multi-step migration
  (add new -> backfill -> switch -> remove old later). See
  `references/change-safety.md`.
- **Dependency changes are not free.** Don't blind-upgrade to "fix" something.
  Check the changelog for breaking changes, change one thing at a time, and
  confirm the app still builds. And never add a package without confirming it
  actually exists on npm.
- **Blast radius.** If a fix forces you to touch shared code, check every caller
  before committing.
- **Same evidence standard as a new build.** When a change touches auth, data
  access, payments, or schema, run the matching section of the saas-builder
  skill's `references/security-checklist.md` and meet its minimum-evidence bar:
  two-user test for data access, signature + idempotency + test-mode run for
  webhooks, migration review + rollback plan for schema. Maintenance is where
  greenfield standards quietly erode — don't let them.

---

## Operating rules

- **Evidence before "done".** Never claim a fix works without re-running the
  reproduction and seeing it pass this session. "Should be fixed" is not a
  state. If unverified, say so explicitly.
- If the project keeps a `PROGRESS.md`, read it at the start (it carries
  decisions and state from prior sessions) and update it after meaningful
  changes — fixes made, causes found, anything deferred.
- If a fix reveals a durable project rule — a convention, a gotcha, a "never
  do X in this codebase" — add one line to the project's `CLAUDE.md` (its
  Do-NOT or conventions section) so future sessions don't repeat the mistake.
  CLAUDE.md carries rules, PROGRESS.md carries history; keep both concise.
- Small commits, each reviewable and reversible. Easy rollback is a feature.
- Say what you did NOT do — what's stubbed, what you couldn't reproduce, what you
  suspect but didn't touch.
- Secrets discipline still applies: nothing sensitive in tracked files or logs.
- Don't expand scope. A bug fix is not a refactor; log tempting cleanups for
  later instead of bundling them in.
- When unsure whether a change is safe on live data, ask before running it, not
  after.

## Reference files

Read as needed:

- `references/debugging-playbook.md` — the reproduce-first method plus concrete
  Next.js / Supabase / Stripe failure signatures (what looks like a bug but
  isn't, and where to actually look).
- `references/change-safety.md` — safe database migrations on live data, the
  don't-weaken-security rule in detail, blast-radius discipline, and dependency
  changes.
