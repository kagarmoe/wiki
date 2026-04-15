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

## [2026-04-15 00:00] stage | 3.1.1a — Diagnostics Sweep 1

**Actor:** general-purpose subagent dispatched by main orchestrator for Phase 3 Batch 1a
**Unit:** 22 `GroupDiag` command pages audited; 4 `cobra drift` findings promoted to `## Docs claim` + `## Drift` sections; 18 pages tagged `phase3_findings: [none]`; drift index stub created at `gastown/drift/README.md`; one commit
**Duration:** one dispatch

**What went well:**

- The plan file's "Batch entry format" section and the writing-entity-pages skill's `## Drift` / `## Docs claim` templates were detailed enough to emit correctly-shaped sections without back-and-forth on structure. The only structural question I had to resolve myself was whether to place `## Docs claim` before or after `## Related` — I followed the writing-entity-pages scaffold (Docs claim before Related) rather than some pages' inherited Phase 2 ordering.
- Re-reading `activity.go`, `repair.go`, and `status.go` in full against current HEAD surfaced the same findings Phase 2 had noted but with fresh line refs — the code-first verification discipline produced no line-ref corrections in this sub-batch because Diagnostics commands haven't churned since Phase 2. (See "what to change" #3 below — this is itself a finding.)
- The `v1.0.0` release-position check via `git show v1.0.0:<file>` worked cleanly for all four finding pages. Every cited code block was byte-identical to v1.0.0, so every finding is `in-release`. No post-release surprises in Diagnostics.
- Phase 2's Notes sections were mostly high-quality — 18 of 22 were genuinely neutral observations (implementation curiosities, architecture notes, open questions about helper functions), not mis-filed drift. The 4 findings I did promote were all obvious once the source was re-read.

**What didn't:**

- **The plan's "expected high-yield" list for Batch 1 is stale in one case.** The plan's Batch 1 section names `info` alongside `directive`, `hooks`, `tap`, `warrant`, `repair` as parent-only stubs whose `Long` text advertises subcommands that aren't wired. Current `info.go` is a terminal command with two flags and no subcommands, and its `Long` text is accurate. Either `info` was fixed between Phase 2 and now, or the plan's categorization was wrong from the start. I did not adjust the plan — that's a main-orchestrator call — but future sub-batches should not spend effort on `info` expecting a cobra-drift yield.
- **Ambiguity about whether to promote drift observations that live in the Phase 2 page body rather than in `## Notes`.** Sweep 1's charter says "promote Phase 2 notes bullets to proper sections." Doctor's drift observation was already written in the page body ("The Long description categories are documentation-only — the actual registration order in `runDoctor` above is what ships"); repair's was also in the body as a "Drift risk: ... This is a neutral observation — not filed as drift per phase scoping" paragraph. I promoted both anyway because the charter's intent is clearly "formalize all drift observations regardless of Phase 2 parking spot," but the plan could be explicit about this. A stricter reading would leave body drift observations alone in Sweep 1 and let Sweep 2 or Batch 13 pick them up.
- **The dispatch prompt's "Classify" step lists 8 categories including `neutral`, but the writing-entity-pages SKILL.md taxonomy table lists 9 (the same 8 plus `neutral`).** The skill and the dispatch prompt are consistent, but the plan's `phase3_findings` enum legal-values list explicitly names 8 category tags including `none` but NOT `neutral`. That's semantically fine — neutral observations stay in Notes and don't produce a finding tag — but the terminology confused me briefly: is `neutral` a category that takes up a tag, or a non-tag category that means "stays in Notes, no tag"? It's the latter. A one-line clarification in the skill's taxonomy table would prevent the same pause for the next subagent.
- **The `→ promoted to ## Drift` redirect format in the dispatch prompt is ambiguous about what to do when the drift observation lives in the page body, not in a bullet.** I left the body observation in place (rewriting it to point forward to `## Drift`) for doctor; for repair I replaced the "drift risk ... neutral observation" paragraph with a forward reference. Both are reasonable but inconsistent. The skill could standardize: "when promoting a Phase 2 body observation, replace the original with a one-line forward link; when promoting a Notes bullet, either remove it or add a `→ promoted` redirect suffix."

**What to change next time:**

- **Plan file:** remove `info` from the Batch 1 "expected high-yield" list in `.claude/plans/2026-04-14-phase3-drift.md` at the `Batch 1 — Sweep 1: commands/` section (around line 395). It's noise for Batches 1b–1h dispatchers who see it in their reading list.
- **Plan or dispatch template:** add one sentence to the Sweep 1 charter clarifying that drift observations already written into the Phase 2 page body (rather than in `## Notes / open questions`) should be promoted to `## Drift` sections too — with the original body text rewritten to forward to the new section. I wouldn't make this a separate "pre-existing body drift" category; it's just a clarification that "drift observation in body" is equivalent to "drift observation in notes" for promotion purposes.
- **Skill edit (`.claude/skills/writing-entity-pages/SKILL.md`):** add a one-line clarification to the drift taxonomy table that `neutral` is not a tag that appears in `phase3_findings:` frontmatter — it means "stays in Notes, the page's `phase3_findings` is `[none]` or omits this observation." The current wording ("Stays in `## Notes / open questions`. No promotion.") is accurate but doesn't answer "so what's in the frontmatter list."
- **Dispatch template for Batch 1b onward:** add a one-line note that `v1.0.0` release-position verification is cheap (one `git show v1.0.0:<file>` per finding, byte-compare to HEAD) and should always be done inline when a finding is being written, not deferred. I did it inline for all four findings this sub-batch and it was zero-friction.

**Follow-ups filed:**

- none (bd beads) — all follow-ups are documentation/skill/plan edits that the main orchestrator should apply before Batch 1b dispatches.
- skill edit suggestion: `.claude/skills/writing-entity-pages/SKILL.md` drift taxonomy table | clarify that `neutral` doesn't appear in `phase3_findings:` frontmatter
- plan edit suggestion: `.claude/plans/2026-04-14-phase3-drift.md` Batch 1 section | remove `info` from "expected high-yield" list; add one sentence clarifying body-drift promotion semantics

**For Kimberly retro discussion:** nothing blocking, but the "body drift vs notes drift" promotion question is a potentially recurring pattern — if other Phase 2 categories (packages, files, concepts) have more body-parked drift observations than notes-parked ones, the Sweep 1 charter will need a real rewrite rather than a one-line clarification.
