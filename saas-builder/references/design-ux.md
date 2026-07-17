# Design & UX flow

Two jobs: (1) make the product look deliberately designed, not AI-default;
(2) make the flow so simple a distracted user on a phone reaches value without
thinking. If the official `frontend-design` skill is installed, use it for
visual craft; this file adds the product-flow layer it doesn't cover.

---

## Visual direction (set once, in Phase 3, before any screen)

- **Derive a personality from the reference product** named in intake (Phase 0,
  question 5). One adjective pair: "warm + trustworthy", "sharp + technical",
  "playful + fast". Every visual choice serves it.
- **When the user provides screenshots** (a competitor, a product they admire,
  a design they like): read them and extract the SYSTEM, not the pixels —
  the personality pair, density, dominant neutral + accent direction, type
  scale contrast, radius/shadow character, and what specifically makes it
  feel premium. Write that extraction into `docs/design.md` as the working
  reference. Never trace a screenshot pixel-for-pixel or lift its brand
  assets — recreate the feel with original expression, same rule as clone
  mode. A competitor screenshot is also copy-research input: what they
  claim, and which of their phrases go on the banned list.
- **Source the choices, don't invent them.** If the `ui-ux-pro-max` skill is
  installed, query its database for this product type: a matching style, a
  color palette, and a font pairing (its style/color/typography/product
  domains). Pick from curated options, then adapt to the personality — this
  beats improvising a palette from memory.
- **Overrides are the user's call.** Deviating from what a design source
  recommended — a motion level, a pattern, a palette — may be right (a
  conversion page for mid-range phones has different needs than a
  portfolio), but the decision is not yours to make alone: present the
  recommendation, your concern, and the trade-off, get a yes, THEN record
  the override and its reason in `docs/design.md`. A silently overridden
  recommendation is an improvised design decision wearing a justification.
- **Persist the direction to `docs/design.md`** alongside the spec and plan:
  the personality pair, the chosen style and palette names, the font pairing,
  the token values (including motion), the layout direction (density:
  spacious/standard/dense; navigation style; card/table/form conventions),
  icon style, component sources (shadcn base, MagicUI for landing
  expressiveness), and the reference product. Every later screen — and every
  later session — styles against this file, not against memory. A restyle
  request starts by updating `docs/design.md`, then propagating to config.
- **Set design tokens before the first screen and never improvise after:**
  one font pairing (one display, one body — not defaults), a 4/8px spacing
  scale, ONE accent color plus neutrals (not three accents), one corner-radius
  value, one shadow style. Put them in Tailwind config / CSS vars so every
  component inherits.
- **Kill the AI-default look:** no unmodified shadcn gray-on-white, no
  centered-everything, no purple-gradient hero because it's the model's habit.
  Pick a real background tone, real type scale contrast (big headings, quiet
  body), generous whitespace. One visual style per product — never mix — and
  gradients, glass, blur, or decorative effects appear only where they serve
  the personality recorded in `docs/design.md`, never as garnish.
- Dark mode is NOT v1 unless the reference product demands it.

## Motion (part of the tokens, not a garnish)

Motion is a token set decided in Phase 3 with the rest — never improvised
per screen. If `ui-ux-pro-max` is installed, query its motion guidance for
the product type. Record in `docs/design.md`:

- **Personality and level:** calm / crisp / premium / playful. Low for dense
  work tools, medium for polished SaaS, high only for marketing or creative
  surfaces.
- **Tokens:** one duration scale (fast ~150ms for hover/press feedback,
  medium ~200–250ms for menus/modals/drawers/list updates, slow ~300ms+ for
  landing reveals) and one easing. Every transition uses a token value.
- **The split:** the app stays fast and calm — motion earns its place only
  by explaining a change, confirming an action, preserving spatial context,
  or making waiting feel understood. Expressive choreography (scroll
  reveals, hero staggers) belongs on the landing page, where visitors are
  judging whether the product feels premium — never inside the work. A
  premium dashboard is still a calm dashboard: its premium feel is carried
  by typography, spacing, and depth, with motion as garnish only — the
  sanctioned garnish list lives in `stack.md` (MagicUI's quiet end).
- **Hard rules:** animate `transform` and `opacity` only — never layout
  properties (width/height/top/left), which cause jank; skeletons or
  progress over indefinite spinners; honor `prefers-reduced-motion` (drop
  movement, keep subtle opacity).
- **Library:** CSS transitions for simple feedback; a React motion library
  for app interactions; GSAP only for landing-page choreography.
- **Craft:** apply Emil Kowalski's animation skills (`emil-design-eng` for
  building, `review-animations` for auditing) to every animation you write —
  they govern execution quality (easing choice, physicality,
  interruptibility) while this file governs where motion may exist and at
  what level. If they are not installed, ask the user once at Phase 3:
  offer `npx skills@latest add emilkowalski/skills` with one line on what
  it adds; on a no, proceed with this file's rules alone and don't re-ask.

## Flow rules (apply to every screen in Phase 4)

- **Click budget: 3.** From opening the app to completing the core action —
  three taps/clicks for a returning user. If a flow needs more, cut steps, not
  add instructions.
- **One primary action per screen.** One visually dominant button; everything
  else is secondary/quiet. If two things compete, the screen is two screens or
  one of them is wrong.
- **Defaults over decisions.** Pre-select the sensible option everywhere
  (plan, date, visibility). Every decision you make for the user is a step
  they don't abandon on.
- **Forms: minimum viable fields.** v1 signup is email + password or just
  phone/OTP — never name/company/phone/how-did-you-hear. Ask for data at the
  moment it's needed, not upfront.
- **Progressive disclosure.** Advanced options live behind "More"; the happy
  path shows only what the happy path needs.
- **Mobile-first, literally.** Design at 390px width first, expand to desktop
  second. Tap targets 44px+. Test every flow at mobile width before calling
  it done — for India-market products, mobile IS the product.

## First-run experience (the empty-workspace problem)

A new user lands in an empty app and leaves. Prevent it:

- **The first screen after signup points at the core action** — one clear
  "create your first X" state, not an empty dashboard with a sidebar of
  options.
- Empty states teach: every empty list says what goes here and has the button
  that creates it.
- Time-to-first-win under 2 minutes. If the core action needs setup (connect
  something, upload something), collapse setup into the first run of the
  action itself.
- Seed a demo example where it helps ("here's a sample project — make your
  own") — but make it dismissible.
- **Name the activation moment** in one sentence — the specific action after
  which a new user has felt the product's value ("sent their first invoice",
  "published their first page"). The entire first-run flow is engineered to
  reach it; it is also the exact event the launch checklist's analytics
  funnel tracks. If you can't name it, the onboarding has no target.
- **No product tours.** Overlay walkthroughs, multi-step tooltip sequences,
  and "let us show you around" modals are what products add when the first
  screen failed at its job. A well-aimed first screen plus teaching empty
  states makes the product self-explanatory; if a feature needs a tooltip
  essay, redesign the feature.
- **Plan for the user who stalls.** Some users sign up and don't reach
  activation. v1 response is minimal: the first screen keeps pointing at the
  core action on every return visit (no "welcome back" dashboard that hides
  it). A single reminder email nudging toward the activation moment is a
  Phase 6 addition once transactional email exists — never a drip campaign
  in v1.

## Landing page (part of v1, not an afterthought)

The app needs a front door. Scaffold it in Phase 3 as the public root. Write
its copy per `copywriting.md` — positioning first, buyer language, competitor
phrases banned — not by pasting spec language into a hero:

- Hero: headline and CTA built per `copywriting.md`, a real screenshot or
  product visual — no stock illustrations.
- One page: problem line, 3 concrete benefits (outcomes, not features),
  pricing (even if one plan), FAQ (3–5 real objections), footer with
  terms/privacy/support.
- Meta + OG tags set (title, description, share image) so WhatsApp/social
  shares render properly — for WhatsApp-first distribution this is not
  optional.
- The landing page loads fast: static/server-rendered, optimized images. It
  will be opened from ads on mid-range phones.

## Polish passes (for the screens that matter)

The core screen and the landing page deserve refinement beyond "works". Good
design is passes, not one shot. Run these one at a time, pausing for the
user's reaction after each:

1. **Layout** — structure and spacing only; get the bones right.
2. **Hierarchy** — the most important thing must be the most obvious; size,
   weight, and position guide the eye.
3. **Color + type** — apply the tokens with intention, not decoration.
4. **Interaction** — hover/press feedback, transitions, loading/empty/error
   states feel deliberate.
5. **Polish** — alignment, spacing fine-tuning, the last 5%.

If Emil Kowalski's skills are installed, run the passes WITH
`emil-design-eng` — it encodes the invisible details (spacing rhythm,
state transitions, interaction feel, component proportions) that separate
premium from default — and audit pass 4 with `review-animations`. These
skills are why two screens with identical tokens can read as polished or
as slop.

The taste test after the passes: would this screen look out of place next to
the reference product? If yes, name the specific offending element and run
another pass on it. The user supplies the taste; ask them to look.

## Microcopy rules

- Buttons say the outcome, not the mechanism: "Get my report", not "Submit".
- Errors say what to DO: "That email is already registered — log in instead?"
  not "Error: duplicate key".
- No lorem ipsum ever reaches a commit — write real copy inline, flag weak
  copy for the user to improve.

## Accessibility baseline (every screen, not a later phase)

Not optional and not a hardening item — retrofitting is far costlier:

- Every interactive element reachable and operable by keyboard, with a
  visible focus state (never remove focus rings).
- Text contrast 4.5:1 minimum (3:1 for large text); never color alone to
  convey state — pair it with an icon or text.
- Every form input has a real label (not placeholder-only); errors are
  announced next to the field they belong to.
- Meaningful images get alt text; icon-only buttons get aria-labels.
- Tap targets 44px+ (already in the flow rules — it's also accessibility).

## Verify (add to the Phase 4 check for every screen)

- Screenshot at 390px width — does the primary action dominate? Could your
  least technical user say what to tap?
- Count the clicks to core value — over budget means redesign, not tooltip.
- Tab through the screen once: focus visible, order sensible, everything
  reachable.
- Transitions: each one has a purpose, uses the motion tokens, runs smoothly
  at mobile width, honors reduced motion, and shifts no layout.
