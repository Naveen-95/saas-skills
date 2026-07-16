# Copywriting and positioning

The landing page skeleton lives in `design-ux.md`; this file is what goes IN
it. The goal is copy a competitor would want to steal — which comes from
positioning, not wordplay. Write the positioning first, then the words fall
out.

## Positioning before words (10 minutes, before any headline)

Answer these from the spec and the Phase 0/0b research, in writing:

1. **Who exactly** is the buyer, and what were they doing about this problem
   yesterday? (The real competitor is usually a spreadsheet, WhatsApp, or
   doing nothing — not the named rival.)
2. **What outcome** do they buy — not the feature, the after-state. "Send
   invoices" is a feature; "get paid two weeks faster" is an outcome.
3. **Onlyness**: complete "We're the only ____ that ____ for ____." If a
   competitor could paste your answer onto their site unchanged, it is not
   positioning yet — sharpen the wedge from Phase 0b (the niche, geography,
   channel, or pricing angle that made this v1 worth building).
4. **The enemy**: the old way, named concretely. Great copy gives the reader
   something to leave, not just something to join.

Save the four answers at the top of `docs/design.md` under "Positioning" —
every future page (pricing, ads, emails, App Store) reuses them.

## Research before writing (when the evidence isn't already on disk)

Clone builds inherit research from Phase 0b; greenfield builds often have
none — and that vacuum is where invented positioning comes from. Before
drafting public-facing copy, check: are the buyer's own words, their real
objections, the alternatives they use today, and the differentiation
already evidenced in the spec, blueprint, or intake? If yes, say so and
write. If not, research first: 3–5 credible sources — competitor homepages
and pricing pages, customer reviews of the alternatives (1-star reviews of
the incumbent especially), public discussions where the buyer complains in
their own words, the phrases they would type into search. Save findings to
`docs/copy-research.md` (vocabulary, objections, claims worth making, the
shared phrases now banned) and draw the headline angles and FAQ from it.
Never fill the gap by inventing — no fabricated proof, testimonials, or
outcomes, under any deadline.

## Use competitor research as a repellent, not a template

From the Phase 0b feature map, list the phrases the competitors all use
("all-in-one", "seamless", "supercharge your workflow"). That list is now
banned — shared language makes you a cheaper copy of the incumbent. The
differentiated claim is, by definition, the one nobody else on that list can
make.

## Voice of customer beats voice of founder

Write the pain in the words a real user would type into a search box or a
complaint — from the research, reviews of competitors (1-star reviews of the
incumbent are a goldmine), or the user's own description of their customers.
If the headline sounds like the spec, translate it: spec language describes
the system, buyer language describes their Tuesday.

## Page copy, section by section

- **Headline**: the outcome plus one concrete specific. "Invoices that chase
  themselves — get paid 2 weeks faster" beats "Effortless invoicing for
  modern teams". The specificity is the credibility.
- **Subhead**: who it's for and the onlyness, one sentence.
- **CTA**: the outcome, not the mechanism — "Get my first invoice out" beats
  "Sign up". One CTA repeated; competing CTAs split the click.
- **Benefits (3)**: each one is outcome + proof-shaped specific ("set up in
  under 2 minutes", "works with the bank you already use"). Features go in a
  quiet list lower down if at all.
- **Proof**: real numbers, real names, real screenshots. Zero users yet? Use
  the founder's own result, a concrete demo ("watch a 45-second invoice"), or
  nothing — invented testimonials and fake counters poison trust and are out
  of scope, always.
- **FAQ**: the 3–5 real objections from research (price, migration effort,
  trust, "how is this different from <incumbent>"), answered plainly. The
  differentiation question gets answered head-on, naming the trade-off.

## Rules that keep it honest and sharp

- Specific beats superlative: numbers, timeframes, and named integrations
  beat "powerful", "seamless", "revolutionary" — cut those words on sight.
- One big idea per page. If the headline needs "and", the page is trying to
  sell two things; pick the one the wedge is for.
- Clarity beats cleverness: a reader who skims only the headline, subhead,
  and CTA must still know what this is, who it's for, and why it's different.
- Claims must be true today. No "trusted by thousands" pre-launch, no
  features the v1 doesn't have. Aspirational copy is a bug, not marketing.
- Read it aloud once; delete every sentence that sounds like a press release.

## The taste test

Show the user two or three headline/subhead options built from DIFFERENT
angles (outcome angle, enemy angle, onlyness angle) — not three phrasings of
one angle. The user picks with their gut; they know their customer's voice
better than any database.
