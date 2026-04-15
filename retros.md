# Retrospectives

Append-only retrospective log for the gt-wiki project. Complements [log.md](log.md): log.md records **what happened**, retros.md records **how the work felt and what to change next time**.

## Who writes here

- **Subagents** append a `stage` retro at the end of every unit of dispatched work (one sub-batch, one task, one focused investigation). Written as the last step of the subagent's run, before the subagent reports back to the main session.
- **Main orchestrator (me)** appends a `phase` retro at the end of every phase, consolidating patterns across the phase's stage retros and surfacing recommendations for the next phase.
- **Scheduled retros** are human-LLM discussions Kimberly and I hold when enough friction or success patterns accumulate to warrant it. I flag when I think it's time ("we should schedule a retro"); Kimberly decides when. Notes from scheduled retros land here as `scheduled` entries.

## Rules

1. **Append-only.** Historical entries are preserved verbatim. Corrections land as new entries that reference the old one, not as in-place edits. Same rule as log.md.
2. **Honesty over flattery.** Retros are information, not performance. If a stage went badly, say so plainly. A retro that only records what went well is a broken retro.
3. **Every "what to change" item becomes a concrete follow-up.** Filed as a bead (`bd create`), a skill edit, a plan edit, or a note for the next scheduled retro. Vague aspirations ("do better next time") are not actionable and don't belong here.
4. **Every retro entry tags its origin.** `stage` entries name the subagent role + the phase/batch/sub-batch ID. `phase` entries name the phase number. `scheduled` entries name the participants and the trigger.
5. **Timeline scannable via grep.** `grep "^## \[" retros.md` gives the full retro timeline, same discipline as log.md.

## Entry formats

### Stage retro (subagent — end of sub-batch / task)

```markdown
## [YYYY-MM-DD HH:MM] stage | <phase>.<batch>.<sub-batch> — <short label>

**Actor:** <subagent role / prompt type>
**Unit:** <what this stage covered — pages audited, files read, commits landed>
**Duration:** <approximate wall-clock or "one dispatch">

**What went well:**
- <specific thing, not generic praise>

**What didn't:**
- <specific friction, confusion, or wrong turn>

**What to change next time:**
- <concrete suggestion — skill edit, plan edit, prompt adjustment, bead filing>

**Follow-ups filed:**
- bd-<id> | <title>  (or: "none — lessons are purely informational")
- skill edit suggestion: <skill path> | <change>

**For Kimberly retro discussion:** <optional — only when the lesson is too big for a one-line fix>
```

### Phase retro (main orchestrator — end of phase)

```markdown
## [YYYY-MM-DD] phase | Phase <N> — <name>

**Scope:** <what the phase produced>
**Duration:** <start date → end date>
**Stages logged:** <count of stage retros contributing>

**Patterns across stages:**
- <cross-cutting observations from the stage retros>

**What went well:**
- <project-level, not stage-level>

**What didn't:**
- <project-level friction>

**Schema / skill changes surfaced:**
- <what the phase's experience suggests we should bake in before the next phase>

**Recommendations for next phase:**
- <actionable>

**Outstanding follow-ups:**
- bd-<id> list
- skill edits not yet landed
- plan items deferred

**For Kimberly retro discussion:** <always — phase retros are a natural trigger for a scheduled conversation>
```

### Scheduled retro (human + LLM — triggered discussion)

```markdown
## [YYYY-MM-DD] scheduled | <trigger / topic>

**Participants:** Kimberly + <LLM role>
**Trigger:** <what prompted the scheduling — phase close, friction threshold, specific incident>

**Discussed:**
- <topic> — <conclusion>

**Decided:**
- <decision> → landed as <log.md entry | skill edit | plan edit | bead>

**Deferred:**
- <topic> — <why deferred + when to revisit>
```

## When to flag "we should schedule a retro"

I surface the suggestion to Kimberly when any of these triggers fire:

- A phase just closed (automatic trigger — phase retros warrant discussion).
- Three or more stage retros in a row surface the same "what didn't" pattern.
- A stage retro contains a "For Kimberly retro discussion" note that's too big for a one-line fix.
- A subagent dispatch went badly enough that the pattern needs rethinking before the next dispatch.
- Kimberly has expressed frustration about a recurring friction that I haven't surfaced yet.
- It's been more than one full phase since the last scheduled retro and the backlog of discussion items is non-empty.

Flagging is cheap; Kimberly decides when to actually schedule.

---

## Timeline

_(empty — first retro lands at the close of Phase 3 Batch 1, the first sub-batch after this file is created)_
