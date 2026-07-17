---
name: saas-builder
description: >-
  Use when the user describes a product, app, tool, or SaaS idea they want
  built, or says things like "build me a...", "I have an idea for...",
  "let's start a new project", "spin up a SaaS for...", "scaffold an app
  that...", "vibe code a...", or "clone [existing product]" / "build a
  [product] alternative". Trigger even when the user does not say "skill" or
  "SaaS" — any request to turn a product idea into a working app. Also use
  when the user asks to "make it production ready" or "harden" an app built
  with this skill. Default stack: Next.js + React + Supabase + Stripe.
---

# SaaS Builder

A phased workflow for turning a one-line idea into a shippable MVP without the
usual vibe-coding failure modes (leaked secrets, missing row-level security,
hallucinated packages, and 4000-line "just build it" sessions that can't be
reviewed).

The core principle: **go fast on scaffolding, go slow on the things that are
expensive to get wrong.** Speed on UI and CRUD. Discipline on auth, data access,
payments, and secrets.

Default stack (override only if the user asks): Next.js (App Router) + React +
TypeScript + Tailwind + Supabase (Postgres, Auth, Storage) + Stripe + Vercel.
Details in `references/stack.md`.

**Existing repository? Adapt before imposing.** If the work lands in a codebase
that already exists, first identify its package manager, test commands,
deployment path, and conventions — and follow them. Do not replace an
established stack, tooling, or style with the default stack casually; the
default is for greenfield projects.

---

## The loop

Work through these phases in order. Do not skip ahead to code. Announce the
current phase so the user always knows where they are, and stop at the two
checkpoints (end of Phase 1 and Phase 2) for a yes before continuing.

### Phase 0 — Intake (5 minutes, conversational)

The user gives you an idea. Before anything else, pin down what actually needs
to exist for a v1. Ask a **tight** batch of questions — no more than 5, in one
message — covering only what changes the build:

1. **Who pays and for what?** The one core action a user pays to do. This is the
   MVP; everything else is later.
2. **Auth model** — email/password, magic link, Google, or none for v1?
3. **Does money change hands in v1?** If yes, one-time or subscription, and
   roughly what price? (Drives whether Stripe goes in now or later.)
4. **Any hard data-privacy needs** — is user data seen only by its owner, or
   shared/team-visible? (This decides the RLS model, so it matters early.)
   And does the product touch a regulated domain — health, finance, minors,
   or similar? (That changes the compliance surface and is cheaper to know
   now than after launch.)
5. **One reference product** they want it to feel like.

Do not ask about tech stack, hosting, or frameworks unless the user raises it —
assume the default stack. Do not brainstorm the business model; that is not this
skill's job. If the user already answered any of these in their idea, do not
re-ask — reflect it back instead.

If the idea depends on a third-party platform's API (Meta, WhatsApp,
Instagram, a marketplace), verify against current docs that the required API
access actually exists and what it gates — app review, business verification,
messaging windows, rate limits — before writing the spec. A permissions
blocker kills faster than a technical one, and no spike can code around it.

### Phase 0b — Clone mode (when the idea is "clone <existing product>")

If the user names an existing product to clone or copy ("clone Stan Store",
"build a Besmi alternative", "make me a Linktree competitor"), do NOT jump
straight to intake questions. Research the target first:

1. **Research the target** using web search / web fetch: its core feature set,
   pricing tiers and what gates them, the primary user flow (what a new user
   does in the first 10 minutes), and who it's for. Check the product's own
   site plus a review/comparison source or two. Summarize findings in a short
   feature map: feature -> which tier -> how central it is.
2. **Pick the wedge, not the whole.** Never clone the full mature product —
   that is years of accumulated scope and the number-one clone failure mode.
   Propose a v1 that clones the ONE core paid workflow plus whatever the
   user's differentiation angle is (a niche, a geography, a channel, a pricing
   model). Everything else from the feature map goes on the NOT-in-v1 list.
3. **Clone functionality, not identity.** Rebuild the workflow and feature
   logic; do not copy the target's name, logo, brand assets, marketing copy,
   or pixel-for-pixel design. Same job-to-be-done, original expression.
4. Then run the normal intake (Phase 0) — most questions will already be
   answered by the research plus the user's angle; ask only what remains.

The spec in Phase 1 should name the reference product and state the wedge
explicitly: "v1 clones <target>'s <core workflow> for <user's angle>."

### Phase 1 — Spec (write it down, get a yes)

If the `product-blueprint` skill is installed and the feature set, screen
list, or onboarding path is not already obvious from intake, run it before
drafting the spec: its `docs/blueprint.md` supplies the core flow, the
table-stakes features, the screen scope, and the Later list that becomes
NOT-in-v1. Skip it when the user arrived with a well-defined feature set.

Produce a short spec using the template in `references/spec-template.md`. Keep it
tight. It must contain: the one-sentence pitch, the single core user
flow for v1, the data model (tables + who-can-see-what), the auth model, whether
Stripe is in v1, core behaviours in EARS notation, acceptance criteria per
must-have feature, and an explicit **"NOT in v1"** list to kill scope creep.

**When the user arrives with a feature list**, treat it as candidates, not
committed scope: sort each into must-have-v1 or NOT-in-v1 with a one-line
reason, and present the split at the checkpoint. The user's insistence
always wins — but it wins as a visible decision with the schedule impact
stated, not as silent scope absorption. (If the list is long or the
screens implied by it are fuzzy, that is the signal to run
product-blueprint first.)

**Self-audit before showing it.** Attack your own spec before the user sees it:
(a) any requirement interpretable two ways — rewrite it; (b) missing error
cases — what happens on bad input, failed external call, payment failure, empty
state; (c) anything that works at 10 users but breaks at 10,000 (cost, data
growth); (d) hidden assumptions about the user that may be false. Fix what you
find; mention the one or two most important fixes when presenting.

**Checkpoint:** show the spec and ask the user to confirm or correct it before
you write any code. Scope creep is the number-one reason these builds rot — the
NOT-in-v1 list is how you fight it. Hold the line politely if the user tries to
add features mid-build: note them under "later" and keep going.

Once confirmed, write the spec to `docs/spec.md` (create the folder if the
project doesn't exist yet). The spec must live on disk, not only in chat —
planning sessions run long, and compaction eats chat.

### Phase 2 — Plan (propose before touching files)

If the harness has a plan/read-only mode, use it (in Claude Code: plan mode,
Shift+Tab; `/model opusplan` pairs Opus planning with Sonnet execution if
available). In a harness without one (Codex, Antigravity), present the full
plan in chat and touch no files until it's approved — the discipline is the
point, the mode is a convenience.

Model economics follow the risk table, and model choice is the user's
lever, not yours: the thinking phases (blueprint/spec, this plan, the
schema and risk sketches, the Phase 5 fresh-context review) deserve the
strongest available model; bulk UI and CRUD execution runs fine on a
faster one. `/model opusplan` automates exactly that split in Claude Code.
The moments to surface this are the plan checkpoint and every wave start
below — not "session start", which has already passed by the time this
skill is read. Do not build per-task subagent orchestration to simulate
it — cold subagents re-derive context on every task and cost more than
they save.

Produce an ordered task list of small, individually-committable steps. A
good step is one PR-sized change you could review in a couple of minutes,
with a stated way to verify it on its own (a command, a screenshot, a spec
criterion it advances) — not "testable once three later steps land".

For each step, choose the verification by what the step risks, not by habit:

| Risk of the step | Minimum evidence before it counts as done |
| --- | --- |
| Pure UI/content | build/lint passes plus a visual check at mobile width |
| Business logic | focused automated test, including a failure case |
| Auth/data access/storage | two-user test: user B cannot reach user A's data |
| Payments/webhooks | signature verified, idempotent, test-mode end-to-end run |
| Schema/destructive change | migration reviewed, backup/rollback plan stated |

When the product has more than one role or more than a few tables, the
plan's first artifact is a schema sketch: entities, relationships, state
fields (a lead's `status`, an order's lifecycle), and which role can do what
— reviewed against the spec's data model before any task is sliced. This is
the MVP-appropriate architecture record; relationships invented mid-build
are the ones that need painful migrations later.

**Conditional risk sketch.** If the product introduces a trust boundary the
stack's standard gates don't already cover — regulated data, user file
uploads, AI that takes actions or shows one user's model output to another,
inbound integrations/webhooks beyond Stripe, or any cross-tenant platform
access — add a half-page risk sketch to the plan: what a hostile user can do
through that boundary, the worst credible outcome, and which task carries
the check that closes each path. `references/security-checklist.md` covers
the stack's standard surfaces; product-specific abuse paths live in the
plan or nowhere.

Order the work so a thin slice works end-to-end early: auth -> one real table
with RLS -> the single core action -> then polish. Do not scaffold 20 tables
up front. **Risk-first exception:** if the product's core value depends on
something genuinely uncertain (an AI call's quality, a third-party API, a
scraping target), prove THAT first with a throwaway spike — before auth,
before anything — so a dead idea dies in an hour, not a week.

For anything involving current API shapes (Supabase auth helpers, Stripe API
version, Next.js App Router patterns), plan to **verify against live docs** — use
Context7 MCP or fetch the official docs rather than trusting memory, because
these APIs churn and stale patterns are a top source of broken builds. See
`references/stack.md` for the specific churn-prone spots.

**Group the tasks into waves.** A wave is the unit of execution,
verification, and session — not of commit:

- Merge 2–6 tasks into one wave when they share a risk tier and surface
  (three UI screens over the same data, a batch of CRUD endpoints) and have
  no interleaved dependencies — one explore pass, one combined verify pass,
  far fewer tool calls than task-by-task ceremony.
- Any task touching auth, RLS, payments, or schema/migrations — and any
  spike — is a wave of ONE.
- Each wave's entry in `docs/plan.md` states: goal, its tasks, expected
  files, the combined verification checklist (the union of the risk-table
  evidence for its tasks), a one-line "done means", and a **model
  recommendation** — the strongest available model for solo/high-risk and
  design-defining waves, a faster model for UI/CRUD waves.

**Checkpoint:** show the plan — waves, their tasks, and each wave's model
recommendation — and get a yes before building. In the SAME message, state
the handoff for Wave 1: "this session is on <model>; Wave 1 recommends
<model> — switch with /model, or say continue." You cannot switch models
yourself; wave boundaries are the user's moments to. Once confirmed, write
the plan to `docs/plan.md`.

General rule for Phases 0–2: any large artifact the user pastes or you
generate — a JSON schema, an API payload, competitor research, a data model
draft — that will be referenced again gets saved to a file immediately and
referenced by path from then on. Chat is not storage; disk survives
compaction and `/clear`.

### Phase 3 — Scaffold

1. Create the project (`references/stack.md` has exact commands).
2. **Immediately** write the project's instruction file from
   `references/claude-md-template.md`, filled in with this project's
   specifics — named for the harness in use: `CLAUDE.md` in Claude Code,
   `AGENTS.md` in Codex/Antigravity and most other agents (same content,
   different filename; if the harness reads both, write CLAUDE.md and make
   AGENTS.md a copy or pointer). This is what keeps every later session
   on-rails — do it before feature code, not after.
3. Set up `.env.local` and confirm `.gitignore` covers it **before** any key is
   ever pasted. Verify with `git status` that no env file is tracked. This one
   check prevents the most common and most damaging vibe-coding leak. Then
   offer to install the deterministic gates from `references/hooks.md` — the
   secret-blocking commit hook especially — so this protection stops depending
   on anyone remembering it.
4. Wire up Supabase client (server + browser), auth, and a health-check page.
5. **Install the component base** — this step is why builds look premium
   instead of hand-rolled: `npx shadcn@latest init` (see `references/stack.md`),
   and propose connecting the shadcn and MagicUI MCP servers (user approves)
   so components are pulled as real code, not guessed. From here, product
   screens are COMPOSED from shadcn primitives and landing/dashboard
   expressiveness comes from MagicUI within the motion budget — hand-writing
   a primitive or effect these libraries already ship is a defect, not a
   style choice.
6. Set the visual direction per `references/design-ux.md`: derive the design
   personality from the reference product, define the design tokens (font
   pairing, spacing, one accent, radius) in config BEFORE building any screen,
   and scaffold the public landing page as the root route. If the
   `ui-ux-pro-max` skill is installed, query it for the style, palette, font
   pairing, and motion level matched to this product type instead of
   inventing them. Write the chosen direction to `docs/design.md` —
   personality, tokens (including motion tokens), style and palette names,
   and the reference product — so every later screen and session styles
   against the same record. If the official `frontend-design`
   skill is installed, apply it here too.
7. Commit. From here, commit after every working step.

### Phase 4 — Build loop (one wave at a time)

Work wave by wave from `docs/plan.md`. Never start a wave without the
user's go.

**At every wave start**, announce the wave (goal, tasks) and surface its
model recommendation from the plan for user input: "Wave N recommends
<model>; this session is on <model> — switch with /model, or say
continue." WAIT for the answer. A wave start is also the natural fresh
session point: resuming costs only the instruction file, PROGRESS.md, and
the wave's plan entry.

**Within the wave:**

- **Explore once** for the whole wave: read the files all its tasks touch
  before editing anything.
- **Code** one task at a time — commits stay small and reviewable even
  though the wave batches everything else. Compose screens from the
  component sources recorded in `docs/design.md`: shadcn primitives for
  forms, buttons, dialogs, tables, accordions; MagicUI where the motion
  budget sanctions it. Search them via their MCP servers when connected.
  Hand-write a component only when neither library ships it. Every screen
  ships with its empty, loading, and error states — a screen without them
  is not done.
- **Verify cheaply as you go** (build passes, the screen renders), but run
  the full evidence pass ONCE at wave end against the wave's combined
  checklist: acceptance criteria as GIVEN/WHEN/THEN with observable
  results including the failure cases, screenshots or Playwright at mobile
  width, the design-ux flow rules (one dominant primary action, 3-click
  budget). One browser pass over four screens costs a fraction of four
  separate cycles. **Security gates are never batched:** the moment a task
  touches auth, data access, or payments, the matching section of
  `references/security-checklist.md` runs before that task's commit.
- **Commit** per working step with a clear message.

**At wave end — hard pause.** Update PROGRESS.md; update any other doc the
wave moved (spec/design/blueprint if scope or direction changed); report
in one short message — what shipped, the evidence, what the next wave is —
and ASK whether to start it. Do not roll into the next wave on your own,
and recommend starting it in a fresh session.

Keep a **PROGRESS.md** at the project root and update it at every commit:
what's done (one line each), key decisions made (and why, one line), and
what's next. This file is the session handoff — a fresh session reads
the project instruction file for the rules and PROGRESS.md for the state,
and can continue without re-explaining. Keep it under a screen; prune
completed noise.

A full build is NOT one unbroken session. PROGRESS.md is current at every
commit, which makes a reset safe at any commit — use that. Reset (`/clear`
or compact in Claude Code, a fresh session elsewhere) at these concrete
points, not "when it feels degraded":

- After the Phase 3 scaffold commit, before the first wave.
- At every wave boundary — the hard pause IS the reset point; start the
  next wave fresh.
- Immediately on any of: quality drifting, instructions being forgotten, a
  second failed fix, or context usage past roughly half if the harness
  shows it — finish the current task, commit, and pause the wave early.

Large-context models make an unbroken session survivable, not optimal —
quality drift arrives well before the context limit does.

**Every time you touch data access, auth, or payments, run the matching section
of `references/security-checklist.md` before committing.** This is
non-negotiable and is the main thing this skill exists to enforce.

### Phase 5 — Deploy and ship check

1. Run the full `references/security-checklist.md` top to bottom BEFORE
   deploying. Fix anything that fails.
2. **Independent review, fresh eyes.** Dispatch ONE fresh-context reviewer (a
   subagent, or a new session pointed at the repo) with no memory of building
   this code, scoped to the high-stakes surface only: auth, data access/RLS
   policies, payment/webhook handling, and secret usage. The builder must not
   be the only grader of its own work — a fresh context catches what the
   building context reasoned itself into. Fix real findings; skip stylistic
   ones. One review at ship, not per task.
3. Deploy: push to GitHub, connect the repo to Vercel (or `vercel` CLI), and set
   every env var from `.env.local` in Vercel's project settings — production
   values, not placeholders. If Stripe is live, point the webhook endpoint at
   the deployed URL and use the production webhook secret.
4. Verify ON the deployed URL, not just localhost: sign up as a fresh user, run
   the core flow end to end, and confirm the RLS second-user test passes in
   production too.
5. Report results honestly — confirm env vars set in the host, RLS on for every
   user-data table and actually tested with a second user, Stripe webhook
   signature verified, no secret in any client-side bundle, and flag what is
   not done rather than declaring victory.
6. When the user is ready to announce the product publicly, run
   `references/launch-checklist.md` — the consolidated gate covering product
   completeness, analytics instrumentation, support surface, marketing pages,
   and the launch sequence itself. Deployed and launched are different states.

### Phase 6 — Production hardening (after the MVP is validated)

The MVP shipping is not the end state. Once the core flow works and there is
any signal worth keeping (users, payments, feedback), work through
`references/production-hardening.md` in order: error/empty/loading states,
rate limiting, DB constraints and transactions, backups, error tracking and
uptime monitoring, staging + CI, a minimal admin capability, full Stripe
lifecycle handling, the compliance surface, real transactional email,
background jobs, and the teams/enterprise ladder when seats demand it. Do NOT front-load these before
validation — they are cheap insurance for a product with users and wasted
effort for one without. When the user says things like "make it production
ready", "harden it", or "we're getting real users", this phase is what they
mean. If the app was built elsewhere (another agent, another tool) and has
no instruction file or docs, first run the saas-maintainer skill's
"adopting an app built elsewhere" step — baseline docs plus an immediate
security triage — before climbing this ladder; hardening an unaudited
codebase polishes over its holes.

---

## Operating rules

- **Evidence before "done".** Never claim something works without having run
  it and seen the result this session — a passing test, a rendered screenshot,
  a successful request. "It should work" is not a completion state. If it
  wasn't verified, say "written but not yet verified" instead.
- **Simplest thing that works.** No speculative abstractions, no config
  options nobody asked for, no "flexible architecture" for futures that may
  never come. Three similar lines beat one premature helper. Add complexity
  only when the current task forces it.
- **Two strikes, then reset.** If the same fix or approach has failed twice,
  do NOT try it a third time — repeated retries pollute context and burn
  tokens guessing. Instead: revert to the last working commit, clear context
  (`/clear` or a fresh session), and restart from PROGRESS.md with a 3-line
  brief (goal, exact sticking point, last working state), explicitly
  avoiding the approach that already failed.
- **Stall means stop, not push.** If a task is spinning — attempts mounting,
  no measurable progress, or the work drifting beyond the planned step — stop,
  report status honestly (done / stuck / suspected cause), and hand the
  decision to the user. Ten minutes of silent thrashing costs more than one
  honest "I'm stuck here."
- **Say when unsure.** If uncertain about an API, a pattern, or whether an
  approach is right — say so and verify, don't produce confident-sounding
  guesses. An honest "I need to check this" beats a plausible hallucination.
- **Never invent packages.** If you reference a library, it must be one you know
  exists or have just verified on npm. Roughly a fifth of AI-suggested packages
  are hallucinated, and attackers squat those names. When unsure, check.
- **Never paste a real secret into a file that git tracks**, and never print a
  full secret back to the user. Use `.env.local` + host env vars.
- **Server-side for anything privileged.** The Supabase service-role key and
  Stripe secret key touch server code only — never a client component, never
  `NEXT_PUBLIC_*`.
- **Small commits, reviewable diffs.** If a change is too big to review, it is
  too big to trust.
- **Say what you did NOT do.** End substantial steps with a one-line note on
  what is stubbed, skipped, or needs the user's attention.
- **Hold scope.** New ideas mid-build go on the "later" list, not into v1.
- **Some decisions are never the agent's.** Pricing and refund policy,
  legal/compliance signoff, data retention and deletion policy, production
  accounts/domains/vendor contracts, launch approval, and any irreversible
  data migration belong to the user (or a qualified professional). Surface
  each with a concrete recommendation and wait — do not decide by default.

## Reference files

Read as needed — never all up front. Each is introduced at its point of use
in the phases above; this list is only the index:

- `references/stack.md` — setup commands, secret boundaries, churn-prone APIs, MCP servers (Phase 3).
- `references/spec-template.md` — the spec format (Phase 1).
- `references/claude-md-template.md` — project instruction file (Phase 3).
- `references/security-checklist.md` — sections in Phase 4, full run in Phase 5.
- `references/production-hardening.md` — the Phase 6 ladder.
- `references/design-ux.md` — visual/motion direction and per-screen rules (Phases 3–4).
- `references/copywriting.md` — positioning and landing copy (Phase 3).
- `references/launch-checklist.md` — the deployed→announced gate (end of Phase 5).
- `references/hooks.md` — deterministic secret/typecheck gates (offer in Phase 3).
