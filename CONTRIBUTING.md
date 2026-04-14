# Contributing to gt-wiki

This wiki follows a structured schema for maintaining accuracy and navigability. Contributions are welcome, but please follow these guidelines to keep the wiki consistent.

## Page Structure

Every page starts with YAML frontmatter:

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

Frontmatter is required from day one so Dataview queries work across the whole wiki.

## Page Types

- `binary` — executables produced by the project
- `command` — CLI subcommands
- `package` — source code packages / modules (Go packages, directories)
- `file` — notable single files (Makefile, Dockerfile, entrypoint scripts)
- `data-type` — structs, schemas, data shapes
- `config-file` — shapes of config files
- `env-var` — environment variables
- `build-target` — Makefile targets, Docker build stages
- `concept` — abstract ideas
- `role` — personified agents or positions
- `service` — long-running processes
- `workflow` — multi-step flows
- `dependency` — external tools the project relies on
- `experiment` — captured runs, build attempts, test results
- `drift` — cross-entity drift themes
- `decision` — design or process decisions made during investigation
- `note` — freeform synthesis filed back from queries

## Linking Rules

- **Wiki-internal links** use relative markdown links: `[gt completion](../commands/completion.md)`.
- Do NOT use Obsidian wikilinks (`[[...]]`) — they can't disambiguate when two topics have a page with the same basename.
- **External source refs** use absolute paths in backticks, optionally with line numbers: `/home/kimberly/repos/gastown/internal/cmd/root.go:96-106`.

## Content Guidelines

- **Code references anchor truth.** Every "what it actually does" claim should cite `file:line` in the raw source.
- **Drift annotations live inline on the entity page**, not as separate files. Truth and lie stay adjacent.
- **Keep pages short and linkable.** Split long pages into sub-pages that reference each other.
- **Drift is information, not embarrassment.** Record it plainly. The goal is an accurate map.
- **Don't redo work.** If something has been captured, trust it unless evidence says otherwise.

## Workflow

1. Read the raw source in full.
2. Update or create the entity page with code-grounded descriptions.
3. If the source is a doc making claims, add those to the entity's "Docs claim" section with a source reference.
4. Update `index.md` if new pages were created.
5. Append an entry to `log.md` with the appropriate verb (`ingest`, `query`, `experiment`, `lint`, `decision`, `drift-found`).

## Phase 2 (Mapping) vs Phase 3 (Drift Analysis)

- **Phase 2**: Focus on mapping code to pages. No drift-hunting in PRs — that's Phase 3.
- **Phase 3**: Compare docs/ claims to mapped code-grounded pages to identify discrepancies.

## Task Tracking

Use beads for agent work, Obsidian Tasks plugin for human curation. See `AGENTS.md` and `CLAUDE.md` for details.

## Questions?

Open an issue or discussion on this repo, or reach out to the maintainer.