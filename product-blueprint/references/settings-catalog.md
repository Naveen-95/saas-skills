# Settings catalog (Pass 5)

The spine below is what every SaaS accumulates eventually. The blueprint's
job is marking each item **v1**, **later**, or **n/a** — and keeping v1
brutally thin. A v1 settings screen with three items is a good sign.

## The spine

### Account
| Item | Default disposition |
|------|---------------------|
| Display name, avatar | v1 (name only; avatar later unless the product shows it to others) |
| Email change | later (support-handled at v1 scale) |
| Password change | v1 if password auth exists; n/a for magic-link/OAuth-only |
| Connected sign-in methods | later |
| Delete account | v1 — a working delete (with confirm + grace note), not a mailto link. This is a compliance surface, not a feature. |
| Export my data | later, UNLESS the product is ledger/record archetype — then v1 (CSV export doubles as this) |

### Security
| Item | Default disposition |
|------|---------------------|
| Active sessions / sign out everywhere | later |
| Two-factor auth | later (hardening phase) |
| API keys | n/a unless the product IS an API |

### Billing (paid products only)
| Item | Default disposition |
|------|---------------------|
| Current plan + usage vs. limits | v1 |
| Upgrade / downgrade | v1 (Stripe-hosted portal counts — don't build a custom one) |
| Payment method, invoices | v1 via Stripe customer portal link |
| Cancel | v1, self-serve, no retention maze. Hidden cancel is the fastest trust-killer in reviews. |

### Notifications
| Item | Default disposition |
|------|---------------------|
| Per-event email toggles | v1 ONLY for events the product actually sends (usually 2–3); one toggle per event, not a channel × event matrix |
| Digest frequency | later |
| Push/SMS preferences | n/a until those channels exist |

Default posture: transactional events on and not disableable; everything
promotional off by default.

### Workspace / team (multi-user products only)
| Item | Default disposition |
|------|---------------------|
| Workspace name | v1 |
| Members list + invite | v1 if teams are in the core loop; later if teams are a growth feature |
| Roles | v1 = owner/member only. Custom roles are an enterprise-ladder feature. |
| Remove member / transfer ownership | v1 if invites are v1 |

### Appearance & locale
| Item | Default disposition |
|------|---------------------|
| Dark mode | later unless the reference product demands it (mirrors design-ux.md) |
| Language | n/a for v1 (ship one language well) |
| Timezone / currency / date format | v1 ONLY when the product displays times or money to other parties (invoices, scheduling) — then it's correctness, not preference. Detect and confirm, don't ask cold. |

## Product-specific settings: the delete-the-setting test

For every candidate setting a feature-map item suggests, write this line in
the blueprint:

> **Setting:** <name> — **Default that would replace it:** <value> —
> **Who diverges and why:** <specific user + reason>

Then apply the test: if "who diverges" is hypothetical ("some users
might..."), delete the setting and ship the default. If it names a real
segment with a real forcing reason (a freelancer's invoice needs THEIR tax
rate; a scheduler user is in a different timezone than their client), keep
it — and put it where the divergence bites: inline at the point of use
first, in the settings screen second. Settings screens are for revisiting
decisions, not making them the first time.

Common product-specific settings that PASS the test: tax rate / currency on
money products, business name + logo on documents sent to third parties,
booking availability windows, notification recipient for team inboxes.

Common ones that FAIL it in v1: items-per-page, default sort order, theme
accent color, editable pipeline stages, custom statuses, toggle-off for a
core feature ("disable analytics tab"). Ship the opinion.

## Placement rules

- Settings live behind the avatar/account menu — never a top-level nav tab
  competing with the product's real surfaces.
- One settings page with sections beats a settings sidebar until there are
  ~10+ items; don't scaffold six empty categories.
- Destructive zone (delete account, cancel plan) sits last, visually
  separated, each action behind its own confirm.
- Billing gets its own page even in v1 — money questions shouldn't share a
  screen with avatar uploads, and support will link users straight to it.
