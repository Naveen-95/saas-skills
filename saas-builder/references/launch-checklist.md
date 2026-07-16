# Launch checklist

The single gate between "deployed" and "announced". Run it after Phase 5's
ship check passes, before telling the world. Items marked with a file name
are enforced in detail there — this list confirms they happened; it does not
replace them. Report every unchecked box plainly; an honest "not done" beats
a quiet gap.

## 1. Product complete

- [ ] Core flow solves the spec's problem end-to-end; every must-have
      acceptance criterion passes (`docs/spec.md`).
- [ ] Sign-up, login, and password reset work on the production URL with a
      real email provider (`production-hardening.md` §10).
- [ ] A minimal account/profile page exists: change email/password, delete
      account. Users must be able to manage themselves without support.
- [ ] First-run onboarding points at the core action; empty, loading, and
      error states on every screen; mobile-width pass done (`design-ux.md`).
- [ ] Smoke test on production: fresh signup -> core action -> logout ->
      login -> data still there.

## 2. Infrastructure and security

- [ ] Full `security-checklist.md` passes against production (RLS second-user
      test, secrets, webhook signature, input validation).
- [ ] Custom domain connected, HTTPS live, www/apex both resolve.
- [ ] Backups/PITR on; rollback known; uptime monitor and error tracking
      receiving events (`production-hardening.md` §§4–5, 13).

## 3. Payments (if paid at launch)

- [ ] Live-mode end-to-end: real card, entitlement granted from the verified
      webhook, cancel/failure paths handled (`security-checklist.md` §4).
- [ ] Pricing page states price, what's included, and refund terms.
- [ ] Stripe receipt emails enabled.

## 4. Analytics — instrument BEFORE launch, not after

Launch traffic is unrepeatable; uninstrumented launch days are lost forever.

- [ ] Website analytics on the landing page (visitors, referrers).
- [ ] Product events for the funnel: signup -> activation (first core action
      completed) -> repeat usage. Define "activated" in one sentence and
      track exactly that.
- [ ] The success signal from `docs/spec.md` is visible on a dashboard the
      user can open.

## 5. Support surface

- [ ] Visible way to reach a human: support email or contact form, linked in
      the app and on the landing page.
- [ ] A feedback/bug channel that fits the stage — a simple form, a mailto,
      or a chat widget; heavyweight tooling can wait.
- [ ] FAQ covers the real objections (from `copywriting.md` research).
- [ ] Founder/user is reachable during launch week; error tracker alerts are
      actually going somewhere someone reads.

## 6. Marketing surface

- [ ] Landing page copy per `copywriting.md`; pricing page; privacy, terms,
      and a short about/contact page.
- [ ] Real screenshots (current UI, not mockups). A short demo video or GIF
      of the core action if the product benefits from being seen moving.
- [ ] OG/meta tags verified by pasting the URL into WhatsApp/X/LinkedIn and
      checking the preview.
- [ ] SEO basics on public pages: unique title + description per page,
      `robots.txt` present, sitemap generated. App (authed) routes excluded
      from indexing.

## 7. Launch motion

Sequence, not simultaneity — each step catches what the previous missed:

1. [ ] Internal pass: the user (and a friend) run the whole flow on
       production, on a phone.
2. [ ] Soft launch: 5–20 beta users (or the waitlist, if one exists)
       onboarded personally; watch the analytics funnel and error tracker
       for a few days; fix what breaks.
3. [ ] Public launch: launch email to any list, social announcement,
       communities where the buyer actually is. Product Hunt optional —
       worth it only with assets ready and a launch-day plan.
4. [ ] Post-launch watch: first 48 hours, check funnel numbers, error
       tracker, and support inbox daily; log findings in `PROGRESS.md`.

## 8. First success review (one week after launch)

- [ ] Visitors -> signups -> activated -> paying: write the four numbers
      down, however small. Small-but-real beats vanity.
- [ ] Read every piece of user feedback and every error; pick the next
      week's work from evidence, not the backlog's oldest wish.
