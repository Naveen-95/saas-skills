---
name: product-blueprint
description: >-
  Use when a product's feature set or UX structure needs defining — before or
  during a build — and the user asks things like "what features should this
  have?", "how should onboarding look?", "what goes in settings for a product
  like this?", "map out the screens", "define the UX", or gives a product idea
  where the feature set is not obvious. Also use when a saas-builder spec is
  about to be written and the core flow, screen list, or scope fence feels
  under-defined. This skill defines WHAT exists (features, screens, onboarding
  steps, settings); visual styling belongs to ui-ux-pro-max and design
  references.
---

# Product Blueprint

Think like a product manager and product designer before code exists: turn a
product idea into a concrete UX definition — the feature map, the screen
inventory, the onboarding sequence, and the settings contents — so the build
starts from a deliberate product, not an improvised one.

The core principle: **every screen, feature, and setting must earn its place
by serving the core loop.** The blueprint's job is as much deciding what does
NOT exist as what does.

Output is always a written artifact: `docs/blueprint.md` in the project (or
the working directory if no project exists yet). If saas-builder is in play,
the blueprint feeds its Phase 1 spec directly — core flow, NOT-in-v1 list,
and acceptance criteria all draw from it.

---

## The five passes

Work through these in order. Each pass produces a section of `docs/blueprint.md`.

### Pass 1 — Anchor

Three sentences, written down before anything else:

1. **The job:** who hires this product and for what recurring task.
2. **The core loop:** the repeated cycle that delivers value, as a verb chain
   ("log expense → see category totals → export for tax"). Not the signup
   flow — the loop a retained user runs weekly.
3. **The activation moment:** the specific first action after which a new
   user has felt the value ("sent their first invoice"). Every onboarding
   decision later is aimed at this sentence.

### Pass 2 — Feature map

Research first, then sort. Scan 2–3 reference products in the category (web
search/fetch when available; the user's named reference product from intake
counts as one). You are looking for **table stakes** — features users of this
category assume exist and whose absence reads as broken — not for features to
copy wholesale.

Sort every candidate feature into exactly one bucket:

| Bucket | Test | v1? |
|--------|------|-----|
| **Core loop** | Removing it breaks the loop from Pass 1 | Yes |
| **Table stakes** | Category users assume it exists (found in ALL reference products' base tier) | Yes, minimal version |
| **Differentiator** | The ONE reason to pick this over the reference products | Yes |
| **Plumbing** | Auth, billing, notifications, settings — standard machinery | Yes, thinnest version |
| **Later** | Everything else, however good | No — listed explicitly |

Rules: one differentiator, not three. If a "table stakes" claim rests on one
reference product having it, it is Later. The Later list is written out in
full — it becomes saas-builder's NOT-in-v1 fence.

Admin capability is plumbing but almost never v1 plumbing: the v1 default is
running support from the platform dashboards (Supabase for data, Stripe for
payments), with an in-app admin panel arriving via saas-builder's
production-hardening ladder once real users justify it. It enters the v1
buckets only when moderation or support IS part of the core loop (e.g. a
marketplace). Record the decision either way — "Admin: dashboards until
hardening" is a blueprint line, not an assumption.

### Pass 3 — Screen inventory

Pick ONE information-architecture archetype from
`references/screen-archetypes.md` (object-first, board, feed, canvas, ledger,
or conversation) — the archetype dictates the main surface, the navigation,
and what the user lands on. Do not default to a dashboard homepage; v1 almost
always lands on the object list or the core action instead.

Then enumerate every screen as a table:

| Screen | Job (one line) | Primary action | Key elements | Notes |
|--------|----------------|----------------|--------------|-------|

Constraints per row: exactly one primary action; every list screen's empty
state is specified in Key elements (what it teaches + the button it shows); a
screen whose job needs two sentences is two screens. The baseline set for any
SaaS v1 (auth screens, first-run, main surface, detail/create, settings,
paywall if paid, landing) is in the archetypes reference — start from it and
delete, don't invent from zero.

**Then connect the nodes — the inventory is screens; the flow is edges:**

- Every primary action names its destination in its table row ("Save →
  invoice detail", "Sign up → first-run step 1"). An action without a stated
  destination is an undesigned redirect the build will improvise.
- Write the **golden path** as one arrow chain from landing page to the
  activation moment (landing → signup → first-run → main surface → core
  action → result). If the chain busts the click budget, cut screens now,
  not mid-build.
- Default redirects unless a row overrides them: after create → the created
  object's detail; after delete → the list it came from; after payment → the
  thing just paid for (never a bare "success" page); after login → the main
  surface, or the deep link originally requested.

### Pass 4 — Onboarding sequence

A numbered sequence from signup to the activation moment from Pass 1. For
each step: what the screen shows, the ONE input it asks for (if any), and
what skipping it does. Design rules:

- **Value before data.** Ask for personal/config data at the moment it's
  used, never as a signup questionnaire. If a step's input doesn't change
  what the next screen shows, cut the step.
- **Collapse setup into the first run** of the core action — connecting,
  importing, and configuring happen inside "create your first X", not before
  it.
- **Prefill or sample when input is expensive.** Template galleries, sample
  data, a demo object marked as deletable — anything that lets the user see
  the loop's output before fully paying its input cost.
- **Count the steps.** Signup → activation in ≤ 5 screens for a v1. No
  product tours, no tooltip sequences — if a step needs explaining, redesign
  the step.
- For team products: single-player value first; the invite prompt appears
  after activation, not during onboarding.

When used with saas-builder, its `references/design-ux.md` first-run rules
apply on top of this sequence — this pass defines the steps, that file
governs their feel.

### Pass 5 — Settings inventory

Settings are where undecided product questions go to hide. The rule: **a
setting exists only where real users genuinely diverge; everywhere else,
make the decision and ship a default.** Every toggle doubles the test
surface.

Build the inventory from the spine in `references/settings-catalog.md`
(account, security, billing, notifications, workspace, data/privacy), mark
each spine item v1 / later / n-a, then add product-specific settings ONLY
where a feature-map item forces one — and for each, write the default that
would let you delete the setting entirely. If the default survives scrutiny,
delete the setting and keep the default.

---

## Handoff

Write all five sections to `docs/blueprint.md`.

**Checkpoint:** show the blueprint and get a yes before anything downstream
uses it — same rule as saas-builder's spec and plan checkpoints. Corrections
go into the file, not just the chat. Then:

- **Into saas-builder:** the core loop becomes the spec's core flow; the
  Later list becomes NOT-in-v1; the screen inventory scopes the plan's
  tasks; the activation moment becomes the analytics target in the launch
  checklist. Run its Phase 0 intake as normal — most answers now exist.
- **Standalone use** (user only wanted the thinking): present the blueprint
  and stop; do not start scaffolding.

The blueprint is a living scope document: mid-build feature ideas get sorted
into a bucket (usually Later) before any code, same as the spec's scope
fence.

## Common mistakes

- **Feature map by brainstorm instead of by loop.** Symptom: buckets full of
  "nice for the user" items. Fix: re-run the bucket tests; anything justified
  by "users would like" and not by the loop or a reference product is Later.
- **Dashboard-first IA for a product with one object type.** A dashboard
  summarizes activity the user doesn't have yet. Land on the list or the
  action.
- **Onboarding as a form.** Three questions before showing anything is a
  questionnaire, not onboarding. Move each question to the moment its answer
  is used.
- **Settings as a feature dump.** "Make it configurable" is deferred design.
  Apply the delete-the-setting test to every product-specific entry.
- **Blueprint only in chat.** Chat is not storage; if it isn't in
  `docs/blueprint.md`, the next session rebuilds it from memory, wrongly.

## Reference files

- `references/screen-archetypes.md` — the six IA archetypes with their main
  surface, navigation, landing screen, and the baseline v1 screen set.
- `references/settings-catalog.md` — the settings spine (what every SaaS
  eventually needs, and which parts are v1) plus the delete-the-setting test.
