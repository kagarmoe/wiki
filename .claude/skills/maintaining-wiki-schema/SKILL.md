---
name: maintaining-wiki-schema
description: Use when proposing a schema change to the gt-wiki (new page type, new frontmatter field, new category folder, version bump, new discipline rule), running a lint pass over wiki pages to find orphans/broken-links/stale-claims/mention-gaps, or reviewing whether an existing page needs updating against new evidence. Covers schema evolution policy, maintenance principles, lint workflow, and how schema decisions get recorded in log.md.
---

# Maintaining the Wiki Schema

## Overview

The wiki schema is evolvable, not frozen. It's expected to change as the wiki grows and as new patterns emerge. This skill governs how schema changes land: via append-only `decision` entries in `log.md`, with optional updates to CLAUDE.md (for coordination-surface changes) and the other project-local skills (for reference-surface changes).

**Core principle: schema decisions are append-only.** Every change to the schema (even "trivial" ones like adding a page type) gets a `decision` entry in `log.md`. CLAUDE.md's schema version is the summary; `log.md` is the full history of how the schema got where it is.

## When to use

Triggers:
- **Proposing a new page type** (e.g. adding `library` or `protocol` to the 17 existing types). The set is open, but additions are recorded.
- **Proposing a new frontmatter field** (e.g. `phase4_audited` for the Coverage/Completeness phase, `phase5_audience` for audience classification, or a topic-scoping field).
- **Proposing a new category folder** (e.g. `gastown/services/`, `gastown/drift/`, `gastown/config/`). Folders are lazy — created when the first page lands — but the folder name and purpose get recorded at decision time.
- **Proposing a schema version bump** (typically part of one of the above).
- **Proposing a new discipline rule** (e.g. a cross-link rule, a naming convention).
- **Running a lint pass** over wiki pages to find orphans, broken links, contradicted claims, stale dates, or entity-mention gaps.
- **Reviewing whether an existing page needs updating** against new evidence (e.g. after a gastown release sync surfaces a wiki-stale finding).

When NOT to use:
- Writing or editing an entity page — use the `writing-entity-pages` skill.
- Filing a bd bead or handling the beads/Tasks cross-tool handoff — use the `tracking-wiki-work` skill.
- Running a release sync after a new gastown release — use the `syncing-gastown-updates` skill.
- Executing a phase plan — use `superpowers:executing-plans` or the active plan's own task-level steps.

## Schema evolution policy

The schema (CLAUDE.md + the three `.claude/skills/` project-local skills) evolves under these rules:

1. **Every change is recorded as a `decision` entry in `log.md`.** Format: `## [YYYY-MM-DD] decision | <subject>`. The entry explains what changed, why, and what artifacts (CLAUDE.md, skills, entity pages) were touched.

2. **Schema version bump semantics:**
   - **Major bump (X → X+1.0)** — restructuring of where content lives, or a breaking change to the audit trail. Example: the 1.1 → 1.2 restructure that moved schema reference content from CLAUDE.md into project-local skills.
   - **Minor bump (X.Y → X.Y+1)** — adding a page type, frontmatter field, or discipline rule without restructuring.
   - **No bump needed** — typo fixes, clarifications, wording improvements to existing rules.
   Schema version lives in CLAUDE.md's header line: `**Schema version:** X.Y (YYYY-MM-DD)`.

3. **Old pages don't need to be retrofitted unless a lint pass says so.** Schema additions are non-retroactive by default. Existing entity pages remain valid under older schema versions; they pick up new fields/sections only when a sweep explicitly re-audits them. Example: Phase 3's optional `phase3_audited` frontmatter field doesn't require retrofitting 206 Phase 2 pages — the absence of the field means "not yet audited," not "in violation of v1.2."

4. **New page types extend the writing-entity-pages skill.** When a new page type is added (e.g. `library`), the decision entry in `log.md` records the type name + definition, and the writing-entity-pages skill gets updated to include the new type in its Page Types section.

5. **New frontmatter fields extend the writing-entity-pages skill.** Same pattern: decision entry + skill update. Required vs optional status is explicit; legal values (if enumerated) are listed.

6. **New category folders are lazy.** The folder is mentioned in CLAUDE.md's Directory layout tree and its purpose is described in the decision entry, but the folder itself is created only when the first page lands. Don't scaffold empty folders.

7. **Cross-skill coherence.** If a schema change affects multiple skills (e.g. a new drift taxonomy category affects `writing-entity-pages` and any phase plan that references the taxonomy), update all affected skills in the same commit. The decision entry enumerates every skill touched.

8. **CLAUDE.md only changes for coordination-surface changes.** CLAUDE.md is the slim coordinator. It changes when the Directory layout tree changes, when a new skill lands and needs a pointer, when a hard rule changes, or when the schema version bumps. It does NOT change for reference-content changes (those live in the skills).

## Maintenance principles

Six one-liners that apply to every schema update and every entity-page expansion:

1. **Don't redo work.** If something has been captured, trust it unless evidence says otherwise. Re-deriving existing content wastes time and risks inconsistency.

2. **When evidence contradicts a page, update the page** — do not file a parallel contradiction. Then note the correction in `log.md` with the `lint` or `drift-found` verb depending on the source of the contradiction.

3. **Code references anchor truth.** Every "what it actually does" claim should cite `file:line` in the raw source. Uncited assertions about code behavior are a lint finding.

4. **Keep pages short and linkable.** Split long pages into sub-pages that reference each other. A single entity page that grows past ~500 lines is usually a sign that the entity should be decomposed into parent + children (e.g. a command with many subcommands → parent command page + per-subcommand child pages).

5. **Drift is information, not embarrassment.** Record it plainly. The goal is an accurate map, not a flattering one. Mis-classifying drift as "neutral" or hiding vestigial code is worse than documenting them.

6. **Schema co-evolves with wiki content.** When a new kind of thing appears in the wiki (a new entity category, a new finding type, a new kind of cross-reference), the schema gets extended to accommodate it. Don't force-fit content into ill-matched existing types.

## Lint workflow

Periodically or on request, scan the wiki for:

### Lint targets

1. **Orphan pages.** Entity pages not linked from `index.md` or from any other page. Run:
   ```bash
   cd ~/repos/gt-wiki
   for p in $(find gastown -name '*.md' -type f); do
     basename=$(basename "$p")
     links=$(grep -rl "$basename" gastown/ index.md 2>/dev/null | grep -v "^$p$" | wc -l)
     if [ "$links" = "0" ]; then echo "ORPHAN: $p"; fi
   done
   ```
   Orphans are either (a) recently added and not yet indexed, (b) legitimately standalone (rare), or (c) schema violations to fix.

2. **Broken cross-links.** Relative markdown links in entity pages pointing at files that don't exist. Run:
   ```bash
   cd ~/repos/gt-wiki
   for p in $(find gastown -name '*.md' -type f); do
     grep -oE '\]\(\.\./[^)]+\.md\)' "$p" | sed -E 's/\]\((.*)\)/\1/' | while read -r link; do
       dir=$(dirname "$p")
       target="$dir/$link"
       if [ ! -f "$target" ]; then echo "BROKEN: $p -> $link"; fi
     done
   done
   ```

3. **Claims contradicted by newer sources.** Every Phase 3+ sub-batch produces `wiki-stale` findings when Phase 2 synthesis disagrees with current source. The lint pass scans entity pages for `file:line` citations that no longer exist at the cited location:
   ```bash
   # For each citation, verify it still resolves:
   grep -oE '`/home/kimberly/repos/gastown/[^`]+:[0-9]+(-[0-9]+)?`' <page.md>
   # Then for each match, git show <base>:<file> | sed -n '<line>p'
   ```

4. **Stale `updated` dates.** Pages whose `updated:` frontmatter is older than the last commit touching them, OR older than the last commit touching the source file they cite:
   ```bash
   # Pages with updated dates older than their git blame last-touched:
   for p in $(find gastown -name '*.md' -type f); do
     yaml_updated=$(awk '/^updated:/{print $2; exit}' "$p")
     git_updated=$(git log -1 --format=%cs "$p")
     # compare; flag if yaml_updated < git_updated
   done
   ```

5. **Entity mentions that should have their own page but don't.** When a wiki page mentions an entity by name (e.g. `internal/bitbucket`) without linking to a wiki page, it's either (a) a gap finding (the page should exist), (b) a false positive (the mention is incidental), or (c) a link-needed finding (the page exists but wasn't linked). Requires hand review.

6. **Gaps that could be filled by reading a specific file.** Phase 2's `## Notes / open questions` sections often flag "this file is 1291 lines but only the top 400 is mapped" — these are coverage gaps, filable as `wants-wiki-entry` beads for follow-up.

### Lint discipline

**Lint produces findings, not fixes.** The goal of a lint pass is to surface problems, not silently fix them. Fixes are decisions that need review. Hand findings to Kimberly (or file beads with the `wiki-investigation` label) before acting.

**Log the lint pass** with the `lint` verb in `log.md`:

```markdown
## [YYYY-MM-DD] lint | <subject>

**Scope:** <what was scanned>

**Findings:**
- Orphan: <path> — <reason to keep or fix>
- Broken link: <source> → <broken target>
- Wiki-stale: <page> — cited `file:line` no longer matches source at <commit>
- Stale updated date: <page> — updated:<date>, last git touch <date>
- Mention gap: <page> mentions <entity> without link; wiki page <exists/missing>

**Next action:** <handoff to Kimberly | filed as beads | direct fix approved>

→ <pages referenced>
```

## Common patterns

**Adding a new page type.** Example: adding `protocol` for wire-protocol entities.

1. Decide the type name and definition. Add it to the writing-entity-pages skill's Page Types section under the appropriate group (here: "Config & runtime surface" or a new group).
2. Bump CLAUDE.md's schema version if the type addition is a minor bump (X.Y → X.Y+1).
3. Append `## [YYYY-MM-DD] decision | Schema vX.Y+1: add page type 'protocol'` to `log.md`.
4. Commit all three changes together.

**Adding a new frontmatter field.** Example: adding `last_audited_by` to track who last audited a page.

1. Update writing-entity-pages skill's YAML frontmatter section with the new field, its type, and its required/optional status.
2. Decide if this is retroactive (every page needs it) or non-retroactive (opt-in).
3. Bump schema version.
4. Append a `decision` entry to `log.md`.
5. Commit together.

**Adding a new category folder.** Example: adding `gastown/services/` for long-running processes.

1. Update CLAUDE.md's Directory layout tree with the new folder + one-line purpose description.
2. Update writing-entity-pages skill if the new category has type-specific conventions.
3. Bump schema version.
4. Append a `decision` entry.
5. Do NOT create the folder on disk until the first page lands there.

**Cleaning up a stale convention.** Example: the Phase 2 walkback at `log.md:158` removed premature `## Docs claim` / `## Drift` sections because Phase 2 was mapping-only.

1. Identify every affected page (by grep or by wiki-stale lint finding).
2. Apply the cleanup inline on each page.
3. Append a `decision` entry to `log.md` that explains *why* the convention is changing and what future phases will re-enable it. This is critical — future readers need to understand that the removal wasn't a regression but a scope decision.
4. Bump schema version if the removal changes the recommended page template.

## Common mistakes

| Mistake | Fix |
|---|---|
| Bumping schema version without a `decision` entry | Always pair version bumps with `log.md` decision entries. The version number is metadata; the decision is the record. |
| Editing CLAUDE.md without updating the relevant skill | CLAUDE.md changes for coordination-surface changes only. Reference content lives in skills. If you're editing CLAUDE.md because content moved, the skill update is the primary change. |
| Retroactively applying a new frontmatter field to 200 pages | Unless the field is mandatory AND the decision entry explicitly says so, new fields are opt-in. Pages acquire them as their category gets re-audited. |
| Fixing lint findings silently during the lint pass | Lint produces findings, not fixes. Hand them to Kimberly (or file beads) for review. |
| Adding a page type without logging the decision | Every schema addition gets a `decision` entry. "Trivial" additions are still schema changes. |
| Deleting an orphan page instead of investigating | An orphan might be a genuine standalone or a recently-added-but-not-yet-indexed page. Investigate before deleting. |
| Splitting a long page without updating cross-links | When splitting an entity page (e.g. a command with many subcommands), update every existing backlink to point at the right child page. Run the backlink check from the writing-entity-pages skill before committing. |
| Writing a schema decision in a commit message instead of log.md | Commit messages are terse; `log.md` decision entries are the durable record. The commit message can reference `log.md` line numbers, but the decision lives in log.md. |

## Self-check

Run after any schema change or lint pass.

### Coverage checklist

- [ ] Schema change has a `decision` entry in `log.md`
- [ ] Schema version bumped in CLAUDE.md header (if applicable; minor wording refinements don't bump)
- [ ] All affected skills updated in the same commit (cross-skill coherence)
- [ ] Lint pass produced FINDINGS, not silent fixes
- [ ] Lint findings handed to Kimberly (or filed as beads) before acting
- [ ] Retroactive vs non-retroactive distinction honored (new fields are opt-in by default)
- [ ] CLAUDE.md only changed for coordination-surface changes (reference content lives in skills)

### Self-check questions

1. **"Is there a `decision` entry in `log.md` for this change?"** — `grep "decision |" log.md | tail -3` should include your change. If not, you forgot the entry.
2. **"Did I update every skill this change touches?"** — A drift taxonomy change affects `writing-entity-pages`; a new label affects `tracking-wiki-work`. List the affected skills explicitly.
3. **"Is this retroactive or opt-in?"** — Default is opt-in. If you're requiring a retrofit of existing pages, that needs explicit Kimberly approval and a decision entry saying so.

### Verification commands

```bash
# Recent decisions should include this change
grep "decision |" ~/repos/gt-wiki/log.md | tail -3

# Schema version matches the decision entry
grep "Schema version:" ~/repos/gt-wiki/CLAUDE.md

# Cross-skill coherence: all skills committed together
git diff --cached --name-only | grep "skills/"
```

### Example: good schema change vs bad schema change

**Bad:** Add a new page type `protocol` to `writing-entity-pages/SKILL.md`, commit, no `log.md` entry, no version bump, `maintaining-wiki-schema` not updated.

**Good:** Add `protocol` to `writing-entity-pages/SKILL.md` Page Types section, bump schema version in CLAUDE.md, append `## [YYYY-MM-DD] decision | Schema vX.Y+1: add page type 'protocol'` to `log.md`, commit all three files together.

## Reference

- **Wiki schema and coordination:** [CLAUDE.md](../../../CLAUDE.md)
- **Writing entity pages (frontmatter, types, scaffold, drift taxonomy):** `.claude/skills/writing-entity-pages/SKILL.md`
- **Task tracking for wiki work:** `.claude/skills/tracking-wiki-work/SKILL.md`
- **Release sync after upstream gastown release:** `.claude/skills/syncing-gastown-updates/SKILL.md`
- **Event log (all schema decisions):** [log.md](../../../log.md) — `grep "decision |" log.md` to see the history
- **Active phase plan:** `.claude/plans/` (gitignored; grep for the most recent file)
