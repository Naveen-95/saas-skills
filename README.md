# saas-skills

Three composable [Agent Skills](https://agentskills.io) that take a SaaS product
from a one-line idea to a launched, maintained app — built for Claude Code, and
portable to any harness that supports the skill format (OpenAI Codex, Google
Antigravity, and others).

The default stack is **Next.js (App Router) + React + TypeScript + Tailwind +
Supabase + Stripe + Vercel**, with an adapt-before-imposing rule for existing
repositories.

## The pipeline

Each skill's output is a file on disk that the next stage consumes — the state
lives in your repo, not in any chat session.

| Stage | Skill | Output |
|---|---|---|
| 1. Define | **product-blueprint** | `docs/blueprint.md` — feature map with an explicit NOT-in-v1 fence, screen inventory from an IA archetype, onboarding sequence aimed at a named activation moment, settings inventory |
| 2. Build & launch | **saas-builder** | `docs/spec.md`, `docs/plan.md`, `docs/design.md`, `PROGRESS.md`, and the app itself — through spec, plan, scaffold, build loop, ship check, launch checklist, and post-validation hardening |
| 3. Maintain | **saas-maintainer** | Safe changes to a live app — reproduce-before-fix, smallest safe change, migration safety on live data, never fixing by weakening security |

Use them independently or as a chain. saas-builder consults `docs/blueprint.md`
when it exists; saas-maintainer holds changes to the same evidence standards
saas-builder set.

## What makes these different from "just prompt the agent"

- **Security gates that vibe-coding skips.** Supabase RLS enabled and tested
  with a second user, Stripe webhook signature verification with idempotency,
  secrets never in tracked files, server-only privileged keys — enforced at
  every commit that touches those surfaces, and again before ship.
- **Risk-proportionate verification.** A UI tweak needs a build check and a
  visual look; auth, data access, payments, and schema changes need the
  heavyweight evidence. Maximum rigor where it counts, no ceremony where it
  doesn't.
- **Scope discipline.** An approved spec with a NOT-in-v1 list, mid-build
  ideas deferred instead of absorbed, and a launch checklist that separates
  "deployed" from "announced".
- **Honest reporting.** Nothing is "done" without recorded evidence from this
  session; what's stubbed, skipped, or unverified is said plainly.
- **Session continuity.** `PROGRESS.md` plus the project instruction file let
  a fresh session — or a different harness — pick up mid-build without
  re-explaining.

## Install

### Claude Code

Copy the three skill folders into your skills directory:

```bash
git clone https://github.com/Naveen-95/saas-skills.git
cp -r saas-skills/saas-builder saas-skills/product-blueprint saas-skills/saas-maintainer ~/.claude/skills/
```

Skills trigger automatically from your request ("build me a...", "there's a
bug in...", "what features should this have?"), or invoke them explicitly.

### Codex / Antigravity / other harnesses

Copy the same folders into that harness's skills directory (e.g.
`~/.codex/skills` for Codex). The workflows are harness-portable: plan-mode
steps degrade to propose-in-chat-before-touching-files, and the project
instruction file is written as `CLAUDE.md` or `AGENTS.md` to match the
harness.

## Optional companions

The skills reference two optional companions and degrade gracefully without
them: `ui-ux-pro-max` (a community skill — a searchable style, palette, and
typography database consulted when setting design direction) and Anthropic's
`frontend-design` skill.

## Structure

```
saas-builder/
  SKILL.md                     # the phased workflow
  references/                  # loaded on demand, not up front
    stack.md                   # setup commands, secret boundaries, churn-prone APIs
    spec-template.md           # the one-screen spec format
    claude-md-template.md      # project instruction file template
    design-ux.md               # tokens, flow rules, first-run, landing page
    copywriting.md             # positioning and landing-page copy
    security-checklist.md      # RLS / secrets / auth / Stripe gates
    hooks.md                   # deterministic secret-blocking git hooks
    launch-checklist.md        # the deployed -> announced gate
    production-hardening.md    # post-validation ladder to production grade
product-blueprint/
  SKILL.md                     # the five passes: anchor, features, screens, onboarding, settings
  references/
    screen-archetypes.md       # six IA archetypes with their traps
    settings-catalog.md        # the settings spine + delete-the-setting test
saas-maintainer/
  SKILL.md                     # bug-fix and feature paths with safety gates
  references/
    debugging-playbook.md      # reproduce-first method + stack failure signatures
    change-safety.md           # live-data migrations, blast radius, security rules
```

## License

MIT — see [LICENSE](LICENSE).
