---
name: writing-entity-pages
description: Use when creating a new wiki entity page under gastown/<category>/ or expanding an existing one in ~/repos/gt-wiki. Covers the YAML frontmatter template (required fields + Phase 3 optional audit fields), the 17 page types, naming and cross-linking conventions, the 5-section entity-page scaffold (What it actually does / Docs claim / Drift / Implementation status / Notes), drift taxonomy with fix-tier classification, the Code → Cobra → Docs authority hierarchy, cross-link discipline rules, and the non-retroactive application rule for Phase 3 sections.
---

# Writing Entity Pages

## Overview

A wiki entity page is a code-grounded markdown document describing one entity (a binary, command, package, file, role, concept, workflow, etc.) in the gastown codebase. Pages follow a fixed scaffold: frontmatter → "What it actually does" → optional Phase 3 audit sections → "Notes / open questions." Every factual claim is cited to source `file:line` in `/home/kimberly/repos/gastown/`.

**Core principle: code is the only source of truth.** Wiki pages are synthesis — they record what the code actually does, with citations. Docs, Cobra help text, and README claims are compared against code during Phase 3+ audits but never treated as authoritative.

## When to use

Triggers:
- Creating a new entity page under `gastown/binaries/`, `gastown/commands/`, `gastown/packages/`, `gastown/files/`, `gastown/roles/`, `gastown/concepts/`, `gastown/workflows/`, `gastown/plugins/`, `gastown/services/`, `gastown/config/`, or `gastown/drift/`.
- Expanding an existing entity page (adding sections, updating `file:line` citations, bumping `updated:`, adding Phase 3 audit annotations).
- Filing a drift finding or implementation-status finding during a Phase 3+ audit batch.
- Promoting a `## Notes / open questions` bullet to a `## Drift` or `## Implementation status` section.

When NOT to use:
- Writing an inventory index page under `gastown/inventory/` — those are A-level enumeration, not C-level entity pages, and have a different shape.
- Writing a topic README (`gastown/README.md`) — those are sub-indexes, not entity pages.
- Writing a plan file under `.claude/plans/` — use the `superpowers:writing-plans` skill instead.
- Editing the `syncing-gastown-updates` skill — that's a project-local skill, not an entity page.

## YAML frontmatter (required)

Every entity page starts with frontmatter:

```yaml
---
title: gt
type: binary                  # see Page types below
status: verified              # verified | partial | stub
topic: gastown
created: 2026-04-11
updated: 2026-04-14
sources:                      # raw source paths this page is grounded in
  - /home/kimberly/repos/gastown/cmd/gt/
  - /home/kimberly/repos/gastown/internal/cmd/root.go
tags: [cli, build]            # free-form, lowercase, kebab-case

# Optional Phase 3+ audit-tracking fields (non-retroactive — see below)
phase3_audited: 2026-04-15                # ISO date, set when the Phase 3 audit reaches this page
phase3_findings: [none]                   # category tags from the drift taxonomy (see below)
phase3_severities: []                     # severity tags orthogonal to categories (see below)
phase3_findings_post_release: false       # true iff ≥1 finding on this page is post-release
---
```

**Required fields:** `title`, `type`, `status`, `topic`, `created`, `updated`, `sources`, `tags`.

**Optional Phase 3+ fields** (non-retroactive — see below):
- `phase3_audited` — ISO date (YYYY-MM-DD) when the Phase 3 audit reached this page. Absence means "not yet audited," not "in violation."
- `phase3_findings` — list of **category** tags from the drift taxonomy. Legal values: `drift`, `cobra-drift`, `implementation-status-unbuilt`, `implementation-status-partial`, `implementation-status-vestigial`, `wiki-stale`, `gap`, `none`. A page with zero findings still gets `phase3_findings: [none]` when audited, so it appears in the audit roll-call.
- `phase3_severities` — list of **severity** tags, orthogonal to categories. Legal values per the docs convention (least → most severe): `ambiguous`, `incomplete`, `wrong`, `missing`. Phase 3 findings are overwhelmingly `wrong`; `incomplete` applies to `implementation-status: partial`; `ambiguous` applies to the rare docs-wording finding. `missing` is NOT a Phase 3 severity (it's Phase 4). A page with no findings gets `phase3_severities: []` alongside `phase3_findings: [none]`.
- `phase3_findings_post_release` — boolean. `true` iff any finding on this page has release-position `post-release` (introduced after the last stable gastown release tag).

Frontmatter is required from day one so Dataview queries work across the whole wiki without needing a retrofit pass later.

**Dataview query patterns:**
- `phase3_audited is undefined` — unfinished-work list (pages not yet audited)
- `phase3_findings contains "drift"` — all pages with drift findings
- `phase3_findings contains "cobra-drift"` — Cobra help text contradictions
- `phase3_severities contains "wrong"` — all pages with wrong-severity findings (Phase 3 primary scope)
- `phase3_severities contains "ambiguous"` — all pages with ambiguous findings (Phase 7 input)
- `phase3_findings_post_release = true` — Phase-6-blocked-on-release work-list

## Page types (starter set — open for extension)

**Code surface:**
- `binary` — executables produced by the project
- `command` — CLI subcommands
- `package` — source code packages / modules (Go packages, directories)
- `file` — notable single files (Makefile, Dockerfile, entrypoint scripts)
- `data-type` — structs, schemas, data shapes

**Config & runtime surface:**
- `config-file` — shapes of config files
- `env-var` — environment variables
- `build-target` — Makefile targets, Docker build stages

**Design / mental model:**
- `concept` — abstract ideas
- `role` — personified agents or positions
- `service` — long-running processes
- `workflow` — multi-step flows

**External:**
- `dependency` — external tools the project relies on

**Meta (investigation artifacts):**
- `experiment` — captured runs, build attempts, test results
- `drift` — cross-entity drift themes (when docs are broadly wrong)
- `decision` — design or process decisions made during investigation
- `note` — freeform synthesis filed back from queries

**The set is open.** When a new kind of thing doesn't fit any existing type, add a type (update this skill + the maintaining-wiki-schema skill, log the decision in `log.md` with the `decision` verb). Don't force-fit.

## Naming and cross-linking conventions

**Filenames:** kebab-case matching the entity. Examples: `gt.md`, `mayor-attach.md`, `gt-proxy-server.md`, `polecat-lifecycle.md`.

**Wiki-internal links** use relative markdown links:

```markdown
[gt completion](../commands/completion.md)
[polecat role](../../gastown/roles/polecat.md)
```

**Do NOT use Obsidian wikilinks** (`[[...]]`). They can't disambiguate when two topics have a page with the same basename, and they break outside Obsidian.

**External source refs** (to files outside the wiki) use absolute paths in backticks, optionally with line numbers:

```markdown
`/home/kimberly/repos/gastown/internal/cmd/root.go:96-106`
`/home/kimberly/repos/gastown/Makefile:18-22`
```

Obsidian cannot follow out-of-vault links, so treat these as grep/copy-paste targets rather than clickable navigation.

## Entity-page starter template

Required: frontmatter. Recommended: the structure below as a scaffold. Allowed: any additional structure the page needs.

```markdown
---
title: <entity-canonical-name>
type: <binary|command|package|file|role|concept|workflow|...>
status: <verified|partial|stub>
topic: gastown
created: 2026-04-14
updated: 2026-04-14
sources:
  - /home/kimberly/repos/gastown/<path>
tags: [<tag1>, <tag2>]
---

# <Title>

**Also known as:** <aliases, if any>

## What it actually does

<code-grounded body with file:line references. This is the only mandatory
content section. Every factual claim cites source code at a specific
file and line range.>

## Docs claim                        [optional, Phase 3+]

<verbatim quotes from upstream docs + Cobra Long text + README, each with
 its source reference. Added during the Phase 3 audit. Absent on pages not
 yet audited — that is NOT a schema violation, just "not audited yet."
 When present, every quote is verbatim (no paraphrasing) and cites its
 source with absolute path + line range.>

### Source
- `/home/kimberly/repos/gastown/docs/<path>` (lines NN-NN)
- `internal/cmd/<file>.go:NN` — Cobra Long text (if applicable)
- `/home/kimberly/repos/gastown/README.md` (lines NN-NN, if applicable)

### Verbatim
> [quoted text here]

## Drift                             [optional, Phase 3+]

<only when docs and code disagree. See the drift taxonomy below for
 category selection. Structured as Claim → Code → Resolution with
 file:line on both sides.>

### <One-line drift title>
- **Claim source:** [where the wrong claim lives — docs/<file>.md, Cobra
  Long text at <file>:NN, README, etc.]
- **Docs claim:** [verbatim or tight paraphrase + source ref]
- **Code does:** [summary + `file:line`]
- **Category:** `drift` | `cobra drift` | `compound drift`
- **Severity:** `wrong` (default for drift findings) | `ambiguous` (rare;
  claim too vague to verify, recommend rewrite for specificity)
- **Fix tier:** `code` | `docs` | `both` (derived from authority hierarchy:
  code for cobra, docs for drift, both for compound)
- **Release position:** `in-release` | `post-release`
- (optional, populated by Phase 6 or release-sync) **PR reference:**
  `gastown#<N> (open)` | `gastown#<N> (merged in v<X>)`

## Implementation status             [optional, Phase 3+]

<only when docs describe aspirational / partial / vestigial behavior.
 Explicitly distinct from drift — aspirational docs aren't wrong, they
 describe a future state that hasn't been built.>

- **Status:** `unbuilt` | `partial` | `vestigial`
- **Severity:** `wrong` (for `unbuilt` / `vestigial`) | `incomplete`
  (for `partial`)
- **Docs describe:** [summary + source]
- **Code provides:** [summary + `file:line`, or "none"]
- **Release position:** `in-release` | `post-release`
- **Fix tier:** `preserve-as-vision` (for `unbuilt` / `partial`) | `code`
  or `docs` per finding (for `vestigial`)
- **Resolution:** preserve as vision doc | reframe with status callout | delete
- (optional, populated by Phase 6 or release-sync) **PR reference:**
  `gastown#<N> (open)` | `gastown#<N> (merged in v<X>)`

## Notes / open questions

<neutral observations that are not drift or implementation-status findings.
 Present on most Phase 2 entity pages; bullets get PROMOTED to Drift /
 Implementation status sections during Phase 3 audits if they turn out
 to be drift candidates.>
```

**No section is mandatory on every page.** An entity page with no drift, no aspirational docs, no vestigial code gets no `## Drift` or `## Implementation status` section — but every entity page that has *any* Phase 3 annotation gets a `## Docs claim` section at minimum (even if the claim matches the code).

Drift annotations live **inline on the entity page**, not as separate files. Truth and lie stay adjacent. A separate `<topic>/drift/` folder holds the consolidated Phase 3 drift index + cross-entity drift themes, without double-maintaining per-entity details.

## Authority hierarchy (Code → Cobra → Docs)

**The load-bearing rule for drift classification:** code is the only source of truth. The three tiers, in authority order:

- **Code** — authoritative. `/home/kimberly/repos/gastown/internal/...`, `cmd/...`. Source of truth for every behavior claim.
- **Cobra** — derived. Cobra command `Long` strings, annotations, error messages embedded in `.go` files. Should agree with code.
- **Docs** — derived. `/home/kimberly/repos/gastown/docs/*.md`, README. Should agree with Cobra + code.

Docs are NOT authoritative by themselves. Wiki pages (Phase 2 synthesis) are ALSO not authoritative — they must be verified against source during every Phase 3+ audit.

**The hierarchy determines fix direction.** When a finding surfaces, the fix always starts at the highest tier that's wrong, because fixes there cascade downward.

**Operational classification procedure:**

1. **Check Cobra vs code.** Read the cobra `Long` string at the cited `file:line` and compare to the code at the same file/function. If Cobra disagrees with code → `cobra drift`. Fix tier: **code** (edit the Long text + any other in-source docstrings).
2. **Check docs vs correct code.** If Cobra was correct in step 1, compare docs to Cobra/code. If Cobra was wrong in step 1, compare docs to what code *actually* says (ignoring the wrong Cobra). If docs disagrees → `drift`. Fix tier: **docs** (upstream docs PR).
3. **If both Cobra AND docs are wrong** → `compound drift`. Tag both `cobra drift` and `drift` on the same finding. Fix order: code first, then docs (fixing Cobra may make the docs fix obvious or even automatic).

## Drift taxonomy (9 categories)

Every Phase 3+ finding fits exactly one primary category (with `compound drift` being the exception that carries two tags).

| Category | Definition | Where it goes | Fix tier |
|---|---|---|---|
| `cobra drift` | Cobra `Long` text (or in-code docstring) contradicts code in the same source tree. | `## Drift` section, tagged `cobra drift` | **Code** — edit Cobra text + any related in-source docs |
| `drift` | `docs/<file>.md` or README contradicts correct Cobra/code. | `## Drift` section, tagged `drift` | **Docs** — upstream docs PR |
| `compound drift` | Both Cobra AND docs are wrong independently. Tag both `cobra drift` AND `drift` on the same finding. | `## Drift` section with both tags | **Code first, then docs** |
| `implementation-status: unbuilt` | Docs describe behavior clearly labeled "vision" / "planned" with zero code support. | `## Implementation status` tagged `unbuilt` | **None** (preserve + status callout) |
| `implementation-status: partial` | Docs describe partially-implemented behavior. | `## Implementation status` tagged `partial` | **None** (preserve + status callout) |
| `implementation-status: vestigial` | Code exists but is explicitly documented or commented as superseded / dead / "no longer runs". | `## Implementation status` tagged `vestigial` | **Code** (remove dead code) OR **Docs** (document the vestige) — per finding |
| `gap` | Docs describe something real that has no wiki page yet (Phase 2 missed it or deferred it). | New `wants-wiki-entry` bead + `## Notes / open questions` entry pointing at the gap. | **Wiki** (Phase 2 coverage extension) |
| `wiki-stale` | Phase 2 wiki page body disagrees with current source — our own synthesis has drifted. | Fix inline in `## What it actually does`. Log under `lint` verb, not `drift-found`. | **Wiki** (our synthesis) |
| `neutral` | Notes that on review turn out NOT to be drift (naming observations, curiosities, future considerations). | Stays in `## Notes / open questions`. No promotion. | **None** |

**The critical distinction:** `drift` / `cobra drift` / `compound drift` mean *something needs to be corrected*. `implementation-status` means *docs are aspirationally correct and the reader should be told the feature isn't built yet*. Mixing these up is the single biggest risk during audits. Phase 6 (Implementation) treats them differently: drift gets factually rewritten; implementation-status gets a "not yet implemented" callout preserved as-is.

**Release position (orthogonal):** every finding carries a release position alongside its category.

- **`in-release`** — the cited code existed at the last stable release tag. Phase 6 fixes can upstream immediately.
- **`post-release`** — the cited code was introduced after the last release. Phase 6 fixes must wait for the next release before upstreaming.

To determine release position: check whether the cited `file:line` exists at the last stable release tag. Example:

```bash
last_tag=$(cd ~/repos/gastown && git tag --sort=-creatordate | head -1)
cd ~/repos/gastown
git show "$last_tag:<cited-file>" 2>/dev/null | grep -n "<symbol>"
# If symbol present at tag → in-release
# If absent (file didn't exist or symbol not yet introduced) → post-release
```

The release position is recorded inline in the finding body. Frontmatter summarizes via `phase3_findings_post_release: true` iff the page has any post-release finding.

## Severity (orthogonal to category, docs convention)

Alongside category and release position, every finding carries a **severity** tag. Legal values follow the docs convention, least → most severe:

- **`ambiguous`** (least severe) — the claim is too vague to verify against code; reader has to guess what's meant. Phase 3 Sweep 2 flags this opportunistically when reading docs files, but the dedicated audit is Phase 7 (Correctness). Rare in Phase 3.
- **`incomplete`** — the claim covers the topic but leaves out important details. `implementation-status: partial` findings default to this. Phase 4 (Coverage/Completeness) owns the full audit.
- **`wrong`** — the claim actively contradicts code. **Phase 3's primary severity.** Every `cobra drift`, `drift`, `compound drift`, and `implementation-status: unbuilt` / `vestigial` finding defaults to `severity: wrong`.
- **`missing`** (most severe) — the code has a feature that no claim exists for anywhere. NOT Phase 3 scope; Phase 4 owns this entirely. If you're about to write a `missing` severity in Phase 3, you're in the wrong phase — file a `wants-wiki-entry` bead or escalate to Kimberly.

**Why severity is separate from category:** category tells you WHAT the finding looks like (drift vs implementation-status vs wiki-stale vs gap); severity tells you HOW BAD it is for docs consumers. A `cobra drift` finding and a `drift` finding are both `wrong` — same severity, different fix tier. A `partial` implementation-status finding is `incomplete` — less severe than `wrong` but more severe than `ambiguous`.

**Phase 7 feedback loop:** Phase 7 (Correctness) re-evaluates severity tags after Phase 6 ships. Findings that Phase 3 tagged `wrong` but Phase 6 couldn't cleanly fix may get re-tagged `ambiguous` — the root cause turned out to be "code does something unclear, not something measurably different from the docs." This is how the audit adapts when implementation reveals audit errors.

## PR reference (optional, populated over time)

Alongside the release-position and severity tags, each finding can carry an optional **PR reference** field pointing at the upstream gastown PR that addresses it. The field is absent by default — **Phase 3 does not populate it**. Phase 6 (Implementation) adds it when it files or discovers a PR; release-sync updates it when the PR merges.

States:

- **Absent (default)** — no upstream PR has been filed or discovered yet. The finding body simply omits the `**PR reference:**` line.
- **`gastown#<N> (open)`** — an upstream PR exists, either filed by Phase 6 or discovered during release-sync. The PR is open, not yet merged.
- **`gastown#<N> (merged in v<X>)`** — the upstream PR has merged in release `v<X>`. The finding is resolved; it moves to an archived section of `gastown/drift/README.md` (or gets removed with a `lint` log entry).

**Why this matters:** without the PR reference field, the corrections list goes stale as PRs land upstream between Phase 3 and Phase 6. With it, the list becomes a living work-tracker that survives multiple release-syncs — findings archive themselves as they resolve, and Phase 6 can see which findings already have open PRs before filing duplicates.

**Phase 3's responsibility is just to create findings with a slot where a PR ref can later live.** Do NOT populate `**PR reference:**` during a Phase 3 audit. That's Phase 6's job.

## Cross-link discipline (10 rules)

This wiki is an Obsidian vault. Cross-links are not decoration — they are the substrate that lets a future session answer "is X correct?" with "no, it's Y" by following backlinks from X to the authoritative page that code-grounds what X actually is. **Every entity page MUST satisfy these rules.**

1. **Source anchoring.** Frontmatter `sources:` lists every source file read when writing the page. Inline claims in the body cite `file:line` refs to the specific place in the code that supports the claim. No uncited assertions about code behavior.

2. **Forward links to adjacent entities.** Every entity page links to every other wiki page it depends on, refers to, or is referred to by. Specifically:
   - `files/` → files it references + binaries it produces + packages whose build it drives.
   - `packages/` → packages it imports (as wiki links where the target page exists, `internal/foo` as a plain path otherwise) + commands, roles, concepts, services that consume it.
   - `commands/` → parent binary + packages it calls + the domain entity it represents (e.g. `commands/mayor.md` → `roles/mayor.md`).
   - `binaries/` → entry-point `cmd/` files + top-level commands registered + key imported platform packages.
   - `roles/` → every `packages/`, `commands/`, `workflows/`, `concepts/` that describes the role.
   - `concepts/` → every code or role page that implements or instantiates the concept + any `docs/concepts/<name>.md` source file in the gastown repo that names the concept upstream.
   - `services/` → daemon/process source package + packages that connect to it + lifecycle workflows.
   - `workflows/` → roles and packages participating, in sequence, with `file:line` refs at each step.

3. **Bidirectional.** When page A references page B, open B and add the back-reference explicitly. Obsidian's backlinks panel surfaces one-way links automatically, but two-way pairs preserve the *relationship type* (A depends on B is not the same as B depends on A), which is what multi-hop reasoning needs.

4. **Alias coverage.** If an entity has multiple names (e.g. `mayor`, `mayorCmd`, "the Mayor process", `internal/mayor`), list them in an **"Also known as"** line directly under the `# Title` heading. Canonical name goes in the title. Makes grep and Obsidian search find the right page regardless of which variant the reader types.

5. **Authoritative assertion pattern.** Every factual claim about what the code does follows this shape: (a) state the claim plainly; (b) cite `file:line` in the source that supports it; (c) link to any entity pages the claim depends on.

6. **Relative markdown links only.** `[text](path/to/page.md)`, never `[[wikilinks]]`. Required for basename collision safety across topics.

7. **Backlink check before commit.** For every new or expanded entity page, run `grep -rn "<canonical-name>" ~/repos/gt-wiki/` to find existing pages that mention the entity. Add forward links from any existing mention that doesn't already link. This is the single biggest driver of map coherence.

8. **Docs-claim source verbatim** (Phase 3+). Every `## Docs claim` section quotes the docs text exactly, never paraphrased. The reader should be able to diff the quote against the source file and find it unchanged.

9. **Re-read cited source at audit time** (Phase 3+). Phase 2 notes are days old and gastown is a moving repo. Before promoting any Phase 2 bullet or annotating any docs claim, **read the cited source file at current HEAD** (or at the Phase 3 audit base commit, per the active plan's convention) and search for the claimed behavior by symbol/string, not just by line number. The cited `file:line` in the new annotation is the CURRENT line, not the stale Phase 2 line.

10. **Drift index bidirectional** (Phase 3+). Every entity page with a `## Drift` or `## Implementation status` section has a forward link to `gastown/drift/README.md`; every row in `gastown/drift/README.md` links back to the entity page. Obsidian backlinks alone are not enough — the relationship type matters.

## Non-retroactive application (Phase 3+)

**The `## Docs claim` / `## Drift` / `## Implementation status` sections are NOT a retroactive schema requirement on Phase 2 pages.** Existing pages written during Phase 2 remain valid as-is; they acquire Phase 3 sections only when the Phase 3 audit sweep reaches them. A future session reading v1.2 should NOT interpret it as "every page needs a `## Docs claim` section retrofit."

Equivalently: `phase3_audited` is the opt-in marker. A page without `phase3_audited` is NOT in violation of v1.2. It's simply "not yet audited." Phase 2's output stands; Phase 3 annotates it incrementally.

## Common mistakes

| Mistake | Fix |
|---|---|
| Copying `file:line` citations from the existing wiki page | Re-read source at the active audit base commit; line numbers move. |
| Paraphrasing the docs claim in the `## Docs claim` section | Quote verbatim. Paraphrasing loses the ability to diff against source. |
| Tagging all drift as `drift` without checking Cobra | Run the authority-hierarchy check first: Cobra vs code, then docs vs correct code. |
| Deleting a `## Notes / open questions` bullet without promoting | Promoted bullets leave a `→ promoted to ## Drift` redirect or are removed cleanly; neutral bullets stay. |
| Inventing a new page type without logging the decision | Add the type to this skill + `maintaining-wiki-schema` skill, log a `decision` entry in `log.md`, bump schema version. |
| Skipping the backlink check before commit | Run `grep -rn "<canonical-name>" ~/repos/gt-wiki/` and add forward links from every existing mention that isn't already a link. |
| Applying Phase 3 sections to a Phase 2 page during non-audit work | Respect non-retroactive application. Phase 3 sections land only during the audit sweep, not as drive-by updates. |
| Using Obsidian wikilinks (`[[...]]`) | Use relative markdown links `[text](path/to/page.md)`. |
| Absolute path inside a wiki link | Absolute paths are for external source refs (backticked code, not links). Wiki links use relative markdown. |

## Reference

- **Schema version and project coordination:** [CLAUDE.md](../../../CLAUDE.md)
- **Schema evolution + lint + maintenance principles:** `.claude/skills/maintaining-wiki-schema/SKILL.md`
- **Task tracking for wiki work:** `.claude/skills/tracking-wiki-work/SKILL.md`
- **Release sync after upstream gastown release:** `.claude/skills/syncing-gastown-updates/SKILL.md`
- **Active phase plan:** `.claude/plans/` (gitignored; grep for the most recent file)
- **Schema v1.2 decision entry** (drift taxonomy + authority hierarchy + non-retroactive rule rationale): [log.md](../../../log.md) — grep for `[2026-04-14] decision | Phase 3 schema change`
- **Event log timeline:** [log.md](../../../log.md) — `grep "^## \[" log.md`
- **Page catalog:** [index.md](../../../index.md)
