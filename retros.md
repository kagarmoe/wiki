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

## [2026-04-15 02:00] stage | 3.1.1b — Configuration Sweep 1

**Actor:** general-purpose subagent dispatched by main orchestrator for Phase 3 Batch 1b
**Unit:** 11 `GroupConfig` command pages audited; 6 `cobra drift` findings promoted; 2 `wiki-stale` findings (one page has both, total 7 Drift-section rows); `## Docs claim` + `## Drift` sections added on 5 pages (`account`, `config`, `hooks`, `plugin`, `shell`, `uninstall` — 6 pages, of which `hooks` carries both a cobra-drift and a wiki-stale finding); `directive` was fixed inline via wiki-stale body rewrite (no `## Docs claim`/`## Drift` section, since wiki-stale's fix tier is the wiki itself); 4 pages tagged `phase3_findings: [none]` (`disable`, `enable`, `issue`, `theme`); one commit
**Duration:** one dispatch

**What went well:**

- The dispatch prompt's pre-flagged "expected high-yield pages" list (directive, hooks, shell, plugin, issue, uninstall, config/theme dual writers) was unusually accurate this time — all six named pages except `issue` and the config/theme pair landed actual findings, and `issue`'s dispatch note explicitly said "likely NEUTRAL unless Long text claims beads-integration" which turned out correct. This is the opposite of Batch 1a's `info.md` stale high-yield miss — for 1b the high-yield list actively saved time by pointing the audit at the real findings on the first pass.
- The **sibling-file subcommand-wiring discovery on `directive` and `hooks`** was the load-bearing finding of this sub-batch. Both pages' Phase 2 bodies claimed "subcommands not wired up here" and speculated they might not be implemented at all. A 30-second `ls internal/cmd/directive*.go` and `grep -rn "hooksCmd.AddCommand"` confirmed all subcommands were wired in sibling files, all present at v1.0.0. This is exactly the kind of Phase-2-missed-context that the Phase 3 audit is supposed to catch — the Phase 2 synthesis took `directive.go`/`hooks.go` in isolation and assumed the absent `AddCommand` calls meant unwired subcommands, but Go's per-file `init()` pattern means `AddCommand` registrations are scattered across the tree. The re-read-in-context check caught it.
- The **`hooks init` cobra-drift finding** was a bonus discovery that came directly from reading `hooks_init.go` to verify the wiki-stale claim. The 9th subcommand's existence was not noted in the Phase 2 page at all (Phase 2 only listed the 8 that `hooks.go`'s `Long` names), and it would have been easy to miss if I had stopped at "confirmed all 8 are wired." Reading the full sibling-file directory (via `ls internal/cmd/hooks*.go`) surfaced `hooks_init.go` as a sibling-file not mentioned anywhere in the Phase 2 page. This argues for Batch 1c onward to always `ls` the parent file's sibling set, not just the parent itself.
- **`v1.0.0` release-position verification was zero-friction** — same observation as Batch 1a. For all six cobra-drift findings the v1.0.0 check was one `git show v1.0.0:<file> | sed -n '<lines>p'` and the text was byte-identical every time. Configuration commands are stable across v1.0.0 → HEAD.
- **The `config get dolt.port` self-contradiction** (Long text omits the key but the error message lists it) was a particularly clean finding — the same file at two different lines made opposing claims. Sharp fix-tier call: clearly code (edit the `Long`), clearly in-release, clearly minimal-risk since the error message already names the key as supported.

**What didn't:**

- **The wiki-stale vs drift-found log-entry category question.** Per the skill taxonomy table, wiki-stale findings are "logged under `lint` verb, not `drift-found`." But Batch 1b's audit pass produced wiki-stale findings interleaved with cobra-drift findings on the same pages, and splitting them into two separate log entries would (a) lose the audit-trail continuity of "this is what I found in one pass of re-reading the Configuration group" and (b) double the log-entry count for no clear reader benefit. I chose to bundle wiki-stale findings into the `drift-found` batch entry with a taxonomy note acknowledging the convention; Batch 1a set no precedent here because it had zero wiki-stale findings. This is a decision the orchestrator should ratify before Batch 1c. Candidate resolutions: (a) ratify the bundle-into-drift-found pattern for Sweep 1 sub-batches because Sweep 1 is "promotion from Phase 2 notes" and wiki-stale is a flavor of that; (b) require a separate `lint` entry per sub-batch that has wiki-stale findings, accepting the double-entry overhead; (c) distinguish "wiki-stale against churn" (real lint) vs "wiki-stale against Phase-2-time truth" (Phase 2 was wrong at the time, so arguably a drift-found because it's still a drift-against-code finding). My intuition says (a) is right for Sweep 1 and (c) is the analytically correct framing — Batches 1b's directive/hooks findings are **"Phase 2 was wrong at Phase 2 time against code that existed,"** not **"Phase 2 was right then and churn made it wrong,"** which is a meaningfully different thing.
- **The body-parked drift observation clarification from Batch 1a** was useful in principle but the 1b pages that had body-parked observations mostly already ALSO had them mirrored in Notes (plugin's "Gate semantics beyond cooldown" lived in Notes; shell's "install→enable coupling" lived in Notes; uninstall's "narrow workspace heuristic" lived in Notes). The charter clarification was aimed at "drift observation lives in body, not Notes" — that pattern didn't actually surface much in Batch 1b. The pages where the body was wrong (directive, hooks) were not body-parked drift observations; they were body-parked *claims* that turned out to be false, which is a different thing — wiki-stale, not promoted drift. The charter clarification didn't speak to this case. A separate charter sentence is needed: "when the body makes a factual claim about code behavior that is contradicted by re-reading source, the claim is `wiki-stale` regardless of whether Phase 2 framed it as a claim or an observation."
- **The `plugin` and `theme` dual-writer-of-`CLITheme` observations both stayed neutral**, but the reason is slightly weak. Neither command's `Long` claims to be the sole writer, so there's no docs claim to contradict. But a user reading `gt theme cli --help` would reasonably assume the theme command is the place to set the CLI theme, and a user reading `gt config set cli_theme --help` would reasonably assume the config command is the place. Both are true simultaneously, which is a maintenance hazard (future drift between the two validators) but not a current-state docs drift. Classified as neutral, consistent with Batch 1a's `--since` parser asymmetry neutral call. If a future phase catches one of the two writers diverging from the other's validator, this becomes a proper drift finding retroactively. Low-priority lint-style issue.
- **The `account` Commands-list omission is a borderline call**. Hand-maintained summary lists in `Long` text are common across the gastown CLI; treating every omission as a `cobra drift` finding will inflate the Phase 6 work-list. Batch 1a made the same call on `doctor` (the `Long` catalog enumerates a curated subset of checks). The consistent framing is "if the `Long` text is the only user-facing description of the subcommand/check/option set, an omission in that list is a drift claim." But the opposite framing — "hand-maintained summary lists are informational; cobra's own `Available Commands:` block is authoritative" — is also defensible and would demote these. I followed Batch 1a's precedent (promote) but flagged it as a judgment call in the log entry for orchestrator review.
- **The batch entry's cross-link discipline section grew verbose.** I found myself writing five distinct clauses about which pages got which sections, plus a parenthetical about taxonomy convention for wiki-stale. Batch 1a's entry was tighter. The extra length isn't wrong but it's a smell that the entry format could use a more structured template (e.g. "Sections added: N x Docs claim, M x Drift, P x redirects" rather than prose).

**What to change next time:**

- **Dispatch template for Batch 1c onward: add a mandatory `ls <parent-file>*.go` step.** Before re-reading a parent-looking `internal/cmd/<command>.go`, list the sibling-file set. If there are siblings named `<command>_*.go`, read each one's `init()` block for `<command>Cmd.AddCommand(...)` calls. This catches (a) wiki-stale "subcommands not wired" claims when Phase 2 missed the siblings, and (b) `cobra drift` claims where the parent's hand-maintained subcommand list is incomplete (as on `hooks init`). Cost: 10 seconds per parent page. Benefit: catches the exact class of findings that made Batch 1b's yield. This pattern would have caught `hooks init` much earlier if I had started there instead of finding it mid-audit.
- **Plan file clarification on wiki-stale log-entry placement.** The skill's "log under `lint` verb, not `drift-found`" rule should be qualified: "Sub-batch pass may consolidate wiki-stale findings into the `drift-found` entry if they surface interleaved with drift findings; standalone wiki-stale findings (e.g. a lint pass between audit batches) get their own `lint` entry." The distinction matters because Sweep 1 sub-batches are structurally audit passes, not lint passes. Batches 2-4 will hit this too.
- **Dispatch template: add a Phase-2-time-vs-churn clause for wiki-stale.** When a wiki-stale finding surfaces, the subagent should decide whether the Phase 2 claim was wrong at Phase 2 time (code existed at 2026-04-11 contradicting the wiki page on day one) vs stale against churn (code moved after Phase 2 landed). The two have different Phase 6 implications: "wrong at Phase 2 time" is a lint find about Phase 2's rigor; "stale against churn" is a release-sync find. Batch 1b's directive + hooks findings are both "wrong at Phase 2 time" per `git show v1.0.0:<file>`. Worth capturing that distinction in the log entry so Phase 6 and release-sync can scope differently.
- **Skill edit suggestion (non-blocking):** the `writing-entity-pages` skill's drift taxonomy row for `wiki-stale` should explicitly note that wiki-stale findings do NOT get `## Docs claim` / `## Drift` sections — they get inline edits to `## What it actually does` and a one-line note in `## Notes / open questions` pointing forward to the inline fix. Currently the skill says "Fix inline in `## What it actually does`. Log under `lint` verb, not `drift-found`." but doesn't explicitly say "no `## Drift` section." I followed this reading for `directive` (no Drift section) but for `hooks` I put the wiki-stale row inside the existing `## Drift` section alongside the cobra-drift finding because that page has both. If the skill rule is read strictly, the wiki-stale row should NOT live in a `## Drift` section even when it co-habits with a cobra-drift. A clarification would prevent the next subagent from making the same call.
- **Plan file update to the Batch 1 tracker table:** Batch 1b's actual numbers are 6 cobra-drift + 2 wiki-stale + 4 none out of 11 pages (82% yield if we count every non-none tag as yield, or 55% if we count pages-with-findings as yield). Compared to Batch 1a's 4-cobra-drift-out-of-22 (18% yield), Configuration is the highest-yield sub-batch so far. This changes the calibration expectation for Batches 1c-1h. Update the tracker table's "Findings count" column and add a "Yield surprise" note if the orchestrator wants to re-scope remaining sub-batches.

**Follow-ups filed:**

- none (bd beads) — all follow-ups are documentation/skill/plan edits for the main orchestrator to apply before Batch 1c dispatches or at the Sweep 1 retrospective.
- plan edit suggestion: `.claude/plans/2026-04-14-phase3-drift.md` Batch 1 tracker table | update 1b row with 6 cobra-drift + 2 wiki-stale + 4 none; note yield surprise
- plan edit suggestion: `.claude/plans/2026-04-14-phase3-drift.md` Sweep 1 charter | clarify wiki-stale log-entry placement (bundle vs separate lint entry) and Phase-2-time-vs-churn distinction for wiki-stale findings
- skill edit suggestion: `.claude/skills/writing-entity-pages/SKILL.md` drift taxonomy wiki-stale row | add explicit "no `## Drift` section — fix inline, note the correction in `## Notes / open questions`" clarification
- dispatch template enhancement: add mandatory `ls <parent-file>*.go` + sibling-file `init()` read step before any parent-looking command audit

**For Kimberly retro discussion:**

- The wiki-stale-at-Phase-2-time finding class is genuinely interesting. Two of 11 pages had wiki bodies that were wrong at Phase 2 time — not stale against churn, but wrong on day one, because Phase 2 took parent `.go` files in isolation without checking sibling-file registrations. This suggests Phase 2's coverage was patchier than the 213-page count implies — some pages were filed as `status: partial` or "parent-only stub" when the subcommand tree was actually fully wired. The Phase 3 audit is now doubling as a Phase 2 rigor check. If the pattern repeats in Batches 1c-1h and Batch 2 (packages/), it's worth re-scoping Phase 3 to include "re-verify Phase 2 'partial' status flags against current source" as an explicit sub-goal, rather than discovering them opportunistically. Could also be handled as a Phase-7 validation pass, but surfacing it now is cheaper than rediscovering later.
- The `account` and `doctor`-style "hand-maintained summary list is incomplete" drift class (now three findings across 1a + 1b) may be a single meta-finding worth documenting: "Gas Town's CLI `Long` texts include hand-maintained enumerations of subcommands/targets/keys that reliably drift from the code's actual registration/switch/enumeration lists." Phase 6 could address this class with a single pattern (e.g. auto-generate subcommand lists via cobra introspection instead of hand-maintaining them) rather than fixing each finding individually. Not Phase 3's call, but worth noting for the Phase 6 planning input.

## [2026-04-15 17:00] stage | 3.1.1c — Work Management Sweep 1

**Actor:** general-purpose subagent dispatched by main orchestrator for Phase 3 Batch 1c
**Unit:** 26 `GroupWork` command pages audited; 3 cobra-drift findings (`assign`, `changelog`, `molecule`), 3 wiki-stale findings all tagged `phase-2-incomplete` (`molecule`, `mq`, `wl`), 1 drift finding (`done` — reformat of pre-plan f143813 section to v1.2 schema), 20 pages tagged `phase3_findings: [none]`. `molecule` carries BOTH a cobra-drift AND a wiki-stale finding. 6 distinct finding pages total. One commit.
**Duration:** one dispatch

**What went well:**

- **The mandatory `ls <parent>*.go` sibling-file check from the Batch 1b retro paid off immediately on `molecule`, `mq`, and `wl`.** Batch 1b's retro recommended always listing sibling files before re-reading a parent `.go` and grepping each one's `init()` for AddCommand registrations. Applying that rule to the 26 Work Management pages surfaced three distinct phase-2-incomplete wiki-stale findings that Phase 2 had missed by reading parent files in isolation: `molecule.go` missing 4 step-group children + 1 shortcut in sibling files, `mq.go` missing `mq_next` in `mq_next.go`, and `wl.go` missing TEN subcommands in ten sibling files. Without the sibling-file check, this sub-batch would have reported 3 cobra-drift findings total and moved on; with the check, it reported 3 cobra-drift + 3 wiki-stale + 1 drift (done carveover) = 7 findings, a 133% yield bump over the naive read. This validates the 1b retro's recommendation strongly and should become canonical for all remaining Batch 1 sub-batches.
- **Release-position verification stayed zero-friction for most findings.** `git -C /home/kimberly/repos/gastown show v1.0.0:<file>` + byte-compare was the canonical check; it worked for every finding in ≤2 seconds. Only the `wl` case involved broader verification — `git -C /home/kimberly/repos/gastown ls-tree -r --name-only v1.0.0 | grep internal/cmd/wl_` to enumerate which sibling files existed at the tag, then per-file `show` to confirm their AddCommand lines — but that's still ~30 seconds of work for a load-bearing finding. Two findings in Batch 1c are `in-release` by direct byte-identity check against v1.0.0; four are `in-release` by file-presence + AddCommand check. Zero post-release.
- **The `6e1fddc` schema clarification held up cleanly in practice.** The new `**Phase 2 root cause:** phase-2-incomplete` line was easy to apply to all three wiki-stale findings. The heuristic fallback (documented in the skill clarification) was used for all three — I didn't run the full archaeology (`git -C ~/repos/gt-wiki log --diff-filter=AM -1 --format=%H -- gastown/commands/<page>.md` → corresponding gastown commit → show that commit). Instead I used the heuristic: "sibling-file `AddCommand` registration was already in place at `v1.0.0`, so Phase 2 running on 2026-04-11 had access to it, so Phase 2 was wrong at Phase 2 time." This is exactly the `phase-2-incomplete` fallback case the skill clarification describes ("missed sibling-file `init()` registration") and noted inline in each finding body. Took about 10 seconds per finding instead of ~2 minutes.
- **The `wl` case (the biggest wiki-stale finding in Batch 1) resolved unambiguously once I checked v1.0.0.** The dispatch prompt described `wl_*` subcommands as "post-v1.0.0" citing the plan's PR-delta scoping draft. Two-minute verification showed they're all pre-v1.0.0 — the plan's draft is wrong. This is a load-bearing retro finding because the plan's PR-delta scoping is what the "Release position" column of every Sweep 1 finding depends on, and if it's wrong for `wl` it may be wrong for other entries. Flagged in the log.md Judgment Calls section as a Sweep 1 retrospective item.
- **Phase 2's Work Management notes were overwhelmingly neutral (20/26 = 77%).** Most pages in this sub-batch (`cleanup`, `close`, `compact`, `formula`, `handoff`, `hook`, `mountain`, `orphans`, `prune-branches`, `ready`, `release`, `resume`, `scheduler`, `sling`, `synthesis`, `trail`, `unsling`, etc.) were well-mapped in Phase 2 and their Notes sections were genuine implementation-curiosity observations, not mis-filed drift candidates. The 6 pages with findings were concentrated in the parent-only-looking files (`assign`'s orphan `--force`, `changelog`'s orphan `--week`, `molecule`/`mq`/`wl` hand-maintained lists and sibling-file misses, `done`'s pre-plan drift-found carveover). Yield: 6/26 = 23%, between 1a's 18% and 1b's 55%. Calibration continues to drop as the commands/ sub-batches progress through higher-surface-area commands with better Phase 2 coverage.

**What didn't:**

- **The plan's PR-delta scoping draft is wrong about `wl`.** Plan line 1479 says "10 new `wl_*.go` command files post-v1.0.0 (wl_browse, wl_charsheet, wl_claim, wl_done, wl_scorekeeper, wl_show, wl_stamp, wl_stamps)." Direct verification at v1.0.0 shows all of those files (plus `wl_post.go`, `wl_sync.go`) existed at the tag with their `AddCommand` lines intact. If the plan's PR-delta draft is wrong on `wl`, it may be wrong on other entries too — and any Phase 3 finding that inherits its release-position from the draft inherits the error. I am not confident how the draft got to the "post-v1.0.0" conclusion; possible sources are (a) an intermediate tag between v1.0.0 and HEAD that introduced commits to `wl_*` files without being where the *files* were introduced, (b) a grep-for-`func init()` that surfaced recent refactors, or (c) the draft was written by reading commit messages rather than by checking `ls-tree v1.0.0`. Fix: the Sweep 1 retrospective gate should either re-derive the draft from scratch (`git diff v1.0.0..HEAD --stat -- internal/cmd/` is the canonical source) or drop the draft and use per-finding v1.0.0 verification as the canonical check. Every Batch 1 sub-batch so far has done per-finding verification anyway, so the draft's utility is mostly as a pre-filter hint, not as ground truth.
- **The `done.md` carveover is scope-bending.** The task explicitly said to reformat the pre-plan `f143813` sections to the v1.2 schema. To produce a compliant `## Docs claim` section I had to read `docs/CLEANUP.md:17-29` for the verbatim quote. This is "reading a docs/ file during Sweep 1," which is structurally out of scope for Sweep 1 but explicitly permitted by the task carveout. I logged the read separately in the `Docs files read:` column of the log entry with a justification, but the precedent feels soft: a future sub-batch might also have pre-plan Phase-3 sections that need reformatting (only if Phase 2 produced any — `done.md` is the only known case). If it does, the same carveout applies. Documented in the log entry's Judgment Calls section as item 3.
- **The `selfNukePolecat` dead-code observation doesn't quite fit any category.** `done.go:1673-1680` declares the function, `:1735` has a doc comment saying it's "Kept for explicit kill scenarios," and `grep -n "selfNukePolecat\b" done.go` shows only the declaration (no call sites in `done.go`). But call sites may exist in `polecat*.go` (which I didn't read because those are out of Batch 1c's scope — they're in Batch 1g Workspace). If call sites exist, it's neutral (the comment is accurate, function is used elsewhere); if they don't, it's `implementation-status: vestigial`. I left it as a neutral sentence inside the `done.md` drift body rather than promoting it. The taxonomy doesn't have a clean "observed-but-unverified-until-cross-batch-read" category. This is a minor gap in the skill but not urgent.
- **`assign --force` borderline: is flag description "doc text"?** The `--force` finding hinges on treating `assignCmd.Flags().BoolVar(&assignForce, "force", false, "Replace existing hooked work")` as a cobra Long-text-equivalent claim. The strict reading — "any docstring on a flag registration is a claim subject to the authority hierarchy" — is what I applied. A narrower reading would say "cobra drift is only about `Long` text, not flag descriptions" and demote this to a neutral observation about a dead flag. The 1b `shell` and `uninstall` findings similarly promoted flag-description claims, so 1c stays consistent, but I notice this is a recurring judgment question and flagged it as Judgment Call 8 in the log.md entry.
- **`molecule`'s `step` group sub-subcommand case is mildly novel.** The four sibling-file additions register on `moleculeStepCmd` (the child group), not on `moleculeCmd` itself. The Batch 1b `hooks` finding was simpler — all sibling-file registrations went on the parent. For `molecule`, the cobra auto-`Available Commands:` block on `gt mol --help` shows the 10 direct parent children (including `step`), but users have to type `gt mol step --help` to see the 4 children of `step`. The drift in `moleculeCmd.Long` is therefore user-visible only at the `step` level; a user scanning `gt mol --help`'s categorised text never sees that the `step` group has more than `done`. This makes the finding slightly harder to reproduce than 1b's `hooks init` case but the same category. Worth noting for Batches 1d-1h if similar patterns surface.
- **The batch-entry log.md section I wrote is ~300 lines long.** Batch 1a was ~110, Batch 1b was ~105. The Work Management sub-batch is the largest in Batch 1 (26 pages vs 22/11) and the "Source files re-read" list alone grew to 40+ lines because of all the sibling-file verification (10 `wl_*` + 3 `molecule_*` + 1 `mq_next`). The "Findings by category" block also grew because three separate wiki-stale findings need per-page explanations. Not wrong but verbose. A future schema refinement might add a more compact per-finding row format, or split the Source files column into "parent files" and "sibling-file verifications." Not blocking; flagged as a "consider a structured template" observation.

**What to change next time:**

- **Plan file: fix the PR-delta scoping draft (plan lines ~1470-1500).** Re-derive from `git -C /home/kimberly/repos/gastown diff v1.0.0..HEAD --stat -- internal/cmd/` or drop the draft and note that Sweep 1 does per-finding verification inline. Either way, the current "10 new wl_* files post-v1.0.0" text is wrong and will mislead future dispatchers. Candidate text for the new section: "The PR-delta between v1.0.0 and HEAD is load-bearing for Phase 6's release-position tagging but is NOT authoritative at the Sweep 1 sub-batch level — every finding is verified inline via `git show v1.0.0:<cited-file>` or file-presence checks at the tag. Treat the draft as a hint for pre-filtering high-yield commands, not as ground truth for finding metadata."
- **Dispatch template for Batches 1d-1h: add an explicit `## sibling-file audit` step before the parent file is re-read.** Current template (implicit from Batch 1b's retro) says "check for sibling files when promoted findings cite them." Batch 1c validates that the sibling check should be eager, not reactive — every parent-looking file benefits from a 10-second `ls <parent>*.go` + grep for AddCommand in each. Templating step: (1) `ls internal/cmd/<command>*.go`; (2) for each sibling file, `grep -n "<command>Cmd.AddCommand\|<command>StepCmd.AddCommand" <file>` (or the equivalent step-group name); (3) if new registrations surface, include them in the `sources:` frontmatter expansion and the Subcommands table rewrite. Cost: ~20 seconds per parent. Benefit: catches phase-2-incomplete wiki-stale findings immediately and catches hand-maintained-list cobra drift via the same pass (since you now know what the full subcommand set actually is).
- **Skill edit suggestion (non-blocking): the `writing-entity-pages` skill could explicitly bless the "hand-maintained subcommand/target list is incomplete" pattern as a recurring cobra-drift class.** Five findings across 1a/1b/1c fit it: `doctor` (curated check catalog), `repair` (6 targets listed, 2 registered), `account` (4 listed, 5 registered), `hooks` (8 listed, 9 wired), `molecule` (categorised list omits 4+ sibling-file subcommands). A dedicated paragraph in the drift taxonomy's `cobra drift` row would help future subagents classify this pattern immediately instead of re-deriving the framing each time. Candidate text: "Pattern: when Cobra `Long` text hand-maintains an enumeration of subcommands, flags, keys, or targets, the enumeration reliably drifts against the code's actual `AddCommand` / `Flag` / `switch` / registration lists. Promote every such omission to `cobra drift`, fix tier `code`. A Phase 6 meta-fix would replace the hand-maintained list with cobra's auto-generated `Available Commands:` or similar introspection, eliminating the drift class entirely."
- **Plan file update to the Batch 1 tracker table:** Batch 1c's actual numbers are 3 cobra-drift + 3 wiki-stale + 1 drift (done carveover) + 20 none out of 26 pages. Yield count depends on how you count `molecule` (2 findings on 1 page): 7 findings across 6 pages = 23% pages-with-findings yield, consistent with 1a (4/22 = 18%) and lower than 1b (6/11 = 55%). The Batch 1 tracker row for 1c should be updated with the commit SHA and the finding counts: "3 cobra-drift / 3 wiki-stale (all phase-2-incomplete) / 1 drift (done reformat) / 20 none; molecule carries both cobra-drift and wiki-stale on the same page (second dual-tag case after 1b hooks); `wl` release-position correction required, plan's PR-delta draft is wrong; sibling-file audit from 1b retro is the load-bearing methodology improvement." Also update the "Notes" column to flag the PR-delta correction for the Sweep 1 retrospective gate.
- **Dispatch template: mention the `done.md` carveout precedent explicitly for Batches 1d-1h.** The `done.md` pre-plan Phase-3 sections are unique to Batch 1c (I checked the other 25 pages — only `done.md` has pre-plan `## Docs claim`/`## Drift` sections). The carveout should not apply to Batches 1d-1h unless the dispatch template names a specific page with pre-plan sections. The dispatch template for 1d should explicitly state "no pages in this sub-batch have pre-plan Phase-3 sections; Sweep 1 reads zero `docs/` files" so a future subagent doesn't assume a general carveout. Prevents scope creep.

**Follow-ups filed:**

- none (bd beads) — wiki-bd is independent from the main Gas Town bd tracker and I did not touch it this sub-batch per dispatch convention.
- plan edit suggestion: `.claude/plans/2026-04-14-phase3-drift.md` lines ~1470-1500 PR-delta scoping draft | correct the "10 new `wl_*.go` files post-v1.0.0" text; all cited sibling files exist at v1.0.0 with their AddCommand lines intact
- plan edit suggestion: `.claude/plans/2026-04-14-phase3-drift.md` Batch 1 tracker table row 1c | update with commit SHA and finding counts once landed
- dispatch template enhancement: formalize the `ls <parent>*.go` + per-sibling AddCommand grep as a mandatory step in the Sweep 1 audit workflow, for Batches 1d-1h and also Batches 2-4
- skill edit suggestion: `.claude/skills/writing-entity-pages/SKILL.md` drift taxonomy `cobra drift` row | add a paragraph documenting the "hand-maintained enumeration drifts against code registration list" pattern as a recurring class with 5+ instances across Batches 1a/1b/1c
- skill edit suggestion (optional): expand the drift taxonomy with a "observed-but-cross-batch-unverified" flag for cases like `selfNukePolecat` where the finding's classification depends on code outside the current sub-batch's scope

**For Kimberly retro discussion:**

- **The sibling-file audit is the single biggest methodology improvement in Phase 3 so far.** Batch 1b surfaced it reactively (Phase 2 was wrong about `directive` and `hooks`, the subagent speculated the fix). Batch 1c made it proactive — every parent-looking file got a sibling-file audit up front — and caught three more wiki-stale findings that would have otherwise shipped as cobra-drift (the hand-maintained list omissions) without realizing the Phase 2 page body was ALSO wrong. Promoting this to a mandatory skill step would make Batches 2-4 (packages, files, concepts, roles) tangibly higher yield, because the same parent-file-in-isolation blindness probably extends beyond commands/. Worth ratifying as schema v1.2.1 or as a skill-only refinement.
- **The Phase 2 `status: partial` flag may be correlated with phase-2-incomplete findings.** `directive`, `hooks`, `molecule`, `wl`, `mq` all had `status: partial` or `status: verified` as Phase 2's self-assessment. The wiki-stale findings cluster hard in the ones Phase 2 self-assessed as "partial" (directive was marked partial; hooks was marked partial; molecule was marked partial; `wl` was surprisingly marked `verified`; `mq` was marked partial). A cheap lint pass in a future Phase 3 batch or at the retrospective gate: "re-verify every Phase 2 `status: partial` page's sibling-file wiring proactively, because the partial status may be a signal that Phase 2 knew its coverage was thin." This would let us estimate the total `phase-2-incomplete` surface area before Phase 4 (Coverage) rather than discovering it incrementally across 111 pages.
- **The `wl_*` family deserves per-subcommand wiki pages.** There are 11 Wasteland subcommands plus `internal/wasteland` helpers — enough to fill a `gastown/commands/wl/` sub-folder or an expanded set of `wl-<name>.md` pages. Consolidating them on `wl.md` works for Batch 1c's wiki-stale fix (one page, one drift row) but obscures the individual subcommands from Obsidian search + cross-links. Filed as a Phase 4 (Coverage) follow-up in the `wl.md` Notes section. If the Phase 4 coverage pass is heavy on inventory expansion, this is a natural candidate. Would mean breaking the "one .go-file = one wiki page" convention, which is already not strict (see `convoy.md` covering 4 `.go` files).
- **The "hand-maintained list drifts" meta-pattern (5+ findings across 1a/1b/1c) is real enough to warrant a named class in the drift index.** It's not a new drift taxonomy category — the category is still `cobra drift` — but at Phase 6 planning time, having these 5+ findings grouped under one fix-pattern ("stop hand-maintaining subcommand lists in `Long` text; use `Available Commands:`") would let a single Phase 6 commit address them all instead of five individual fixes. Skill-level refinement: the drift taxonomy table's `cobra drift` row could carry a "Common sub-patterns" column listing this one as the most-seen class. Optional, not blocking.
