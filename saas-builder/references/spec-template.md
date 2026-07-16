# Spec template (Phase 1)

Fill this in and show it to the user for a yes before writing code. Keep the
whole thing to one screen. If a section is getting long, the scope is too big
for a v1 — cut it.

```markdown
# <Product name> — v1 spec

## Pitch
<One sentence: who it's for and the one thing it does.>

## Core flow (the only one that matters for v1)
<Numbered steps of the single path a paying user takes, start to value.
Example:
1. Sign up with email
2. Create a <thing>
3. <core action they pay for>
4. See the result>

## Data model
<Tables, key columns, and — critically — who can see each row.>

| Table | Key columns | Visibility |
|-------|-------------|------------|
| profiles | id, email | owner only |
| <things> | id, user_id, ... | owner only |

## Auth
<magic link / email+password / Google / none-for-v1>

## Payments in v1?
<Yes: one-time / subscription, ~price. Or: No, add later.>

## AI features (include ONLY if the core flow calls an LLM / paid API)
<- Model + one-line why (default to the cheapest model that passes the spike).
- Rough cost per core action vs. price charged — if unit economics look upside
  down, flag it now, not after building. Real numbers come from the Phase 2
  spike; refine this section after it runs.
- At least one IF/THEN behaviour below must cover the AI call failing or
  returning garbage.>

## Core behaviours (EARS notation — 5 to 10 max)
<Write the key behaviours so they cannot be misread. Patterns:
- The system SHALL <always-true rule>. (e.g. store all prices in paise as integers)
- WHEN <trigger>, the system SHALL <response>.
- WHILE <state>, the system SHALL <behaviour>. (e.g. WHILE unsubscribed, limit scans to 3/day)
- IF <bad thing>, THEN the system SHALL <response>. (error handling — include at least 2)
Only the behaviours where ambiguity would produce the wrong build.>

## Acceptance criteria (defines "done" per feature)
<For each MUST-HAVE feature, one or two checks in this form:
GIVEN <starting state> WHEN <user action> THEN <exact observable result>.
Every THEN must be something you can SEE or MEASURE. Include one
failure-case check per feature. These become the verify step in the
build loop — a feature is done when its criteria pass, not when the
code compiles.>

## NOT in v1 (the scope fence)
<Explicit list of everything tempting that is deliberately excluded. Team
features, admin dashboard, mobile app, extra integrations, settings pages,
etc. Everything here goes on a "later" list, not into the build.>

## Success check
<How we'll know the MVP works: e.g. "a second test user cannot see the first
user's rows, and a test-mode payment unlocks the core action.">
```

Guidance:

- The "who can see each row" column is not optional — it defines the RLS model
  and is the thing most likely to be a silent security hole later.
- If the user resists naming a NOT-in-v1 list, push gently. A build with no
  scope fence is a build that never ships.
