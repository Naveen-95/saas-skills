# Screen archetypes (Pass 3)

Pick ONE archetype. It decides the main surface, the navigation model, and
the screen a logged-in user lands on. Mixing archetypes at the top level is
the most common IA mistake — a product can contain a board *view* inside an
object-first app, but it has one primary shape.

## The six archetypes

### 1. Object-first (default for most SaaS)

The product manages a collection of one central object (invoices, forms,
projects, links, listings).

- **Main surface:** list/grid of the object, newest or most-relevant first.
- **Landing screen:** the list. If the list is empty → the teaching empty
  state with the create button.
- **Navigation:** object list is home; sidebar/tabs only when a SECOND
  first-class object genuinely exists (not for settings — settings live
  behind the avatar menu).
- **Detail screen:** one object, its status, and the actions its lifecycle
  allows. Create/edit as a focused page or slide-over — full-page for ≥5
  fields, slide-over below that.
- **Trap:** adding a dashboard homepage. A returning user wants their
  objects, not a chart about their objects. Dashboards are Later, earned by
  real usage data.

### 2. Board / pipeline (CRM, ATS, order management, kanban)

The object is the same as object-first, but its POSITION in a staged process
is the information.

- **Main surface:** columns = stages, cards = objects, drag between stages
  as the core interaction (with a tap-based move menu on mobile — drag alone
  fails the touch test).
- **Landing screen:** the board.
- **Also needed:** a list view of the same data (boards fail past ~50 cards
  per column) — but the list VIEW is v1 polish, not a second archetype.
- **Trap:** configurable stages in v1. Ship an opinionated default pipeline;
  custom stages go to Later.

### 3. Feed-first (social, content, community, review streams)

Value is consuming a stream produced by others (or by automation).

- **Main surface:** a single vertically-scrolled feed; the create action is
  a persistent affordance (FAB / composer at top), not a separate section.
- **Landing screen:** the feed, pre-populated — a feed product with an empty
  feed at first run is dead on arrival, so onboarding MUST seed content
  (follow suggestions, starter subscriptions, backfilled data).
- **Navigation:** bottom tabs (feed / create / profile-or-activity), max 5.
- **Trap:** building profile pages and social graphs before the feed itself
  retains anyone.

### 4. Canvas / editor-first (builders, design tools, doc editors, site builders)

The product IS a workspace where the user makes a thing.

- **Main surface:** the editor, full-bleed; everything else (file list,
  settings) is chrome around it.
- **Landing screen:** returning user → most recent document, one click from
  the rest; new user → template gallery or "start blank", never an empty
  file list.
- **Navigation:** minimal top bar in the editor (name, save state, share,
  exit); the document list is a secondary screen, not home.
- **Trap:** over-investing in the document-management layer (folders, tags,
  search) before the editor produces something worth managing.

### 5. Ledger / record-first (finance, invoicing backends, analytics, admin panels)

Truth lives in rows; users filter, reconcile, and export.

- **Main surface:** a dense, sortable, filterable table. Density is correct
  here — this is the one archetype where "spacious" is wrong.
- **Landing screen:** the table, default-filtered to the current period.
- **Also needed in v1:** column sort, one date-range filter, CSV export.
  Ledger users assume export exists; its absence reads as data hostage-taking.
- **Trap:** charts before the table is trustworthy. Summaries are earned.

### 6. Conversation-first (chat products, AI assistants, support inboxes)

The unit of work is a thread of turns.

- **Main surface:** thread list + active thread (split view on desktop,
  stacked navigation on mobile).
- **Landing screen:** new users → a new empty thread with the composer
  focused and 3–4 example prompts/actions; returning users → most recent
  thread.
- **Also needed in v1:** streaming/typing feedback, an error state per
  message (retry on the message, not a global toast), thread rename/delete.
- **Trap:** folders, tags, and search over threads before multi-thread usage
  exists.

## Baseline v1 screen set (start here, delete what doesn't apply)

Every SaaS v1 is roughly this; the archetype fills in rows 4–5.

| # | Screen | Job | Primary action |
|---|--------|-----|----------------|
| 1 | Landing page (public) | Convince + convert | Sign up |
| 2 | Signup / Login / Reset | Get in with minimum friction | Submit |
| 3 | First-run sequence | Reach the activation moment | Do the core action |
| 4 | Main surface | The archetype's home | The core action |
| 5 | Detail / create-edit | One object's lifecycle | Save / advance status |
| 6 | Settings | The v1 spine (see settings-catalog.md) | — |
| 7 | Paywall / plan picker | Convert at the value moment (paid products only) | Choose plan |

Rules of thumb:

- Every screen row gets its empty, loading, and error state named at
  blueprint time — they are design decisions, not implementation details.
- If the inventory exceeds ~10 screens for a v1, the feature map is
  over-scoped; go back to Pass 2 and demote to Later.
- Mobile check per screen: what is the ONE thing this screen does at 390px?
  If the answer needs a horizontal scroll or a second primary button, the
  screen is overloaded.
