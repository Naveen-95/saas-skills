---
name: skill-harvest
description: >-
  Use when the user asks to harvest skill learnings, improve the skills from
  recent builds, or process skill notes — "/skill-harvest", "update the
  skills from what we learned", "process the skill notes from <project>".
  Runs in its own short session, never inside a build.
---

# Skill Harvest

Turn field friction into skill commits at minimum token cost. Capture was
already done during builds (one-line entries in each project's
`docs/skill-notes.md`); this skill batches the analysis.

## Inputs — read ONLY these

1. `docs/skill-notes.md` from the project(s) the user names.
2. The specific skill sections the notes implicate — not whole skill files
   unless the edit requires surrounding context.

Do not read build transcripts, whole repos, or unrelated references. If a
note is too vague to act on, ask the user one question or drop it — do not
excavate.

## Triage each note with the improvisation filter

- **Real gap** — a decision got improvised mid-build because no file
  answered it → fix the owning file with the smallest edit that closes it.
- **Binding failure** — the guidance existed but didn't fire → don't add
  words; anchor the rule to an executable moment (a numbered step, a
  checkpoint message, a wave boundary), and add a user gate if the decision
  is taste or irreversible. Prose that isn't a step gets skipped.
- **Routing failure** — the wrong skill ran → sharpen the trigger
  descriptions or add an explicit handoff rule.
- **One-off / project-specific / "a serious company would have this"** →
  drop it. Note why in the commit message only if it was borderline.
- **User preference, not a skill gap** → save it to persistent memory
  instead of editing a skill.

## Apply

- One owner per topic — edit the file that owns it; other files point.
- Keep descriptions trigger-only; keep SKILL.md bodies lean (details go to
  references); keep every rule proportionate to the risk it manages.
- One commit per theme, with the field evidence in the message.
- Sync the private skills repo to the public one and push both.
- Delete processed notes from `docs/skill-notes.md` so nothing is
  double-harvested.

## Rules

- Never run inside a build session; harvest context stays small on purpose.
- Never batch-create rules for hypothetical failures — one field
  observation, one fix.
- If two harvests in a row touch the same section, the section is wrong at
  a deeper level — redesign it instead of patching a third time.
