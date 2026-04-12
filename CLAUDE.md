# CLAUDE.md — Wiki Schema

**Schema version:** 1 (2026-04-11)

## Purpose

This is an LLM-maintained knowledge wiki following Karpathy's "LLM Wiki"
pattern. It's a compounding artifact: findings, cross-references, and
syntheses accumulate here rather than being re-derived every session.

**Scope (current):** the `gastown` project — its source code, documentation,
build system, runtime behavior, and the drift between docs and code.

**Scope (planned):** will grow to cover other repos in `~/repos/`, and may
eventually absorb content from the personal Obsidian vault at
`~/Documents/obsidian/`.

**Ownership:** the LLM writes and maintains this wiki. Kimberly curates
sources, directs investigation, asks questions, and reviews changes. Raw
sources are never modified — only read.

## Layers

1. **Raw sources (immutable):** source repos under `~/repos/<name>/`,
   documents Kimberly drops in, command outputs captured during
   investigations. Read-only. Source of truth.
2. **Wiki (this directory):** LLM-owned markdown. Pages, index, log. Fully
   rewriteable.
3. **Schema (this file):** conventions and workflows. Co-evolves with the
   wiki as patterns emerge.

## Directory layout

```
~/repos/wiki/
  CLAUDE.md          # this file
  index.md           # global catalog of all pages
  log.md             # chronological event log
  .claude/
    settings.local.json   # permission scoping; gitignored
  <topic>/           # one folder per topic namespace (gastown, ...)
    README.md        # topic overview + sub-index
    binaries/        # entity pages: one per binary
    commands/        # entity pages: one per CLI subcommand
    packages/        # entity pages: Go packages / source modules
    files/           # entity pages: notable single files
    config/          # entity pages: config file shapes, env vars
    concepts/        # abstract ideas
    roles/           # personified agents / positions
    services/        # long-running processes
    workflows/       # multi-step flows
    ...              # create additional categories as content arrives
```

Sub-folders are created lazily. Don't scaffold empty folders.

**Nesting is capped at 3 levels:** `wiki/<topic>/<category>/<entity>.md`. If
a page feels like it needs deeper nesting, split the category into sibling
categories instead.

## Page conventions

**Naming:** kebab-case filenames matching the entity (`gt.md`,
`mayor-attach.md`, `gt-proxy-server.md`).

**Wiki-internal links** use relative markdown links:
`[gt completion](../commands/completion.md)`. Do NOT use Obsidian wikilinks
(`[[...]]`) — they can't disambiguate when two topics have a page with the
same basename.

**External source refs** (to files outside the wiki) use absolute paths in
backticks, optionally with line numbers:
`/home/kimberly/repos/gastown/internal/cmd/root.go:96-106`. Obsidian cannot
follow out-of-vault links, so treat these as grep/copy-paste targets rather
than clickable navigation.

### YAML frontmatter (required)

Every page starts with frontmatter:

```yaml
---
title: gt
type: binary            # see type list below
status: verified        # verified | partial | stub
topic: gastown
created: 2026-04-11
updated: 2026-04-11
sources:                # raw source paths this page is grounded in
  - /home/kimberly/repos/gastown/cmd/gt/
  - /home/kimberly/repos/gastown/internal/cmd/root.go
tags: [cli, build]      # free-form, lowercase, kebab-case
---
```

Frontmatter is required from day one so Dataview queries work across the
whole wiki without needing a retrofit pass later.

### Page types (starter set — open for extension)

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

**This set is open.** When a new kind of thing doesn't fit any existing
type, add a type to this schema and log the decision in `log.md`. Don't
force-fit.

### Entity-page starter template (flexible, not a contract)

Required: frontmatter. Recommended: the structure below as a scaffold.
Allowed: any additional structure the page needs.

```markdown
# <Title>

## What it actually does
<code-grounded description with absolute-path file:line references>

## Docs claim
<what README, docs/, help text say — with source references>

## Drift
<contradictions between "actually does" and "docs claim">

## Notes / open questions
```

Drift annotations live **inline on the entity page**, not as separate files.
Truth and lie stay adjacent. A separate `<topic>/drift/` folder may hold
cross-entity drift themes ("the whole install story is wrong") without
double-maintaining per-entity details.

## Ingest workflow

When a new source is read (a file, a command output, a doc page):

1. Note the source path / identifier.
2. Read it in full.
3. Discuss key takeaways with Kimberly before writing (unless she has
   delegated).
4. For each entity the source touches, update or create the entity page.
5. If the source is a doc making claims, add those to the entity's "Docs
   claim" section with a source reference.
6. Update `index.md` if new pages were created.
7. Append an entry to `log.md` with the `ingest` verb.

## Query workflow

When Kimberly asks a question:

1. Read `index.md` to find candidate pages.
2. Read candidate pages in full (not chunks).
3. Synthesize an answer grounded in wiki content, with links to the pages
   used.
4. **If the answer is a valuable synthesis, file it back as a new page**
   under `<topic>/concepts/` or `<topic>/notes/`, update `index.md`, and log
   with the `query` verb.

## Lint workflow

Periodically or on request:

- Orphan pages (not linked from `index.md` or any other page)
- Broken cross-links
- Claims contradicted by newer sources
- Stale `updated` dates on pages that should be revisited
- Entity mentions that should have their own page but don't
- Gaps that could be filled by reading a specific file

**Lint produces findings, not fixes.** Discuss with Kimberly before acting.
Log with the `lint` verb.

## index.md format

One section per topic. Within each section, pages grouped by category, one
line each with a brief descriptor.

## log.md format

Append-only. Each entry starts with `## [YYYY-MM-DD] <verb> | <subject>` so
`grep "^## \[" log.md` gives a timeline.

**Verb starter set** (closed for now; loosen later if needed):
`ingest`, `query`, `experiment`, `lint`, `decision`, `drift-found`.

## Schema evolution

This schema is **version 1**. It will change as the wiki grows. Update this
file when new page types emerge, when conventions start hurting rather than
helping, or when a better workflow appears. Note schema changes in `log.md`
with the `decision` verb. Old pages don't need to be retrofitted unless a
lint pass says so.

## Maintenance principles

- **Don't redo work.** If something has been captured, trust it unless
  evidence says otherwise.
- **When evidence contradicts a page, update the page** and note the
  correction in `log.md`.
- **Code references anchor truth.** Every "what it actually does" claim
  should cite `file:line` in the raw source.
- **Keep pages short and linkable.** Split long pages into sub-pages that
  reference each other.
- **Drift is information, not embarrassment.** Record it plainly. The goal
  is an accurate map.

## Permission scoping

`.claude/settings.local.json` (gitignored) grants this wiki full
Read/Write/Edit/Bash permissions within `~/repos/wiki/` and read-only
access everywhere else. The LLM owns the wiki and can mutate it freely, but
treats everything outside `~/repos/wiki/` as reference only.
