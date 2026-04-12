# Wiki Log

Chronological event log. Append-only. Each entry:
`## [YYYY-MM-DD] <verb> | <subject>`.

Verbs: `ingest`, `query`, `experiment`, `lint`, `decision`, `drift-found`.

---

## [2026-04-11] decision | wiki scaffolded

- Created `~/repos/wiki/` per Karpathy's "LLM Wiki" pattern.
- Purpose: compounding knowledge base, starting with the gastown drift investigation.
- Schema in `CLAUDE.md` (v1). Topic namespacing, YAML frontmatter required, flexible entity-page skeleton, 3-level nesting cap, relative markdown links (not wikilinks), 17 page types (open for extension), closed log-verb starter set.
- Permission scoping via `.claude/settings.local.json` (gitignored): full Read/Write/Edit/Bash within `~/repos/wiki/`, read-only outside.
- First topic: gastown.

## [2026-04-11] experiment | docker compose build with golang:1.26-bookworm

- Context: building `~/gt/Dockerfile` so `gt` can run inside a container.
- Switched base image from `debian:bookworm` (+ Go tarball) back to `golang:1.26-bookworm`, unpinning Go patch version.
- `docker compose build` succeeded through apt + dolt install steps.
- `go install github.com/steveyegge/gastown/cmd/gt@latest` inside the running container installed to `/go/bin/gt` (GOPATH in the `golang:` image is `/go`, not `/root/go` as the original Dockerfile assumed).
- `gt completion bash` build step failed: binary self-killed with "ERROR: This binary was built with 'go build' directly. macOS will SIGKILL unsigned binaries. Use 'make build' instead."
- Root cause: `/home/kimberly/repos/gastown/internal/cmd/root.go:96-106` — check fires when `BuiltProperly == "" && Build == "dev"`. `go install` does not set the `BuiltProperly` ldflag.
- → [gastown/binaries/gt.md](gastown/binaries/gt.md), [gastown/files/makefile.md](gastown/files/makefile.md)

## [2026-04-11] ingest | /home/kimberly/repos/gastown/internal/cmd/root.go

- Read the self-kill check at lines 96-106.
- Bypass: `-ldflags "-X github.com/steveyegge/gastown/internal/cmd.BuiltProperly=1"`.
- Stale comment on line 97 claims "Warning only - doesn't block execution" — but code on line 106 calls `os.Exit(1)`.
- Error message attributes the issue to macOS SIGKILL of unsigned binaries, but the check fires on Linux too.
- → [gastown/binaries/gt.md](gastown/binaries/gt.md)

## [2026-04-11] ingest | /home/kimberly/repos/gastown/Makefile

- Captured canonical LDFLAGS recipe, including `-X ...BuiltProperly=1` (the self-kill bypass).
- Makefile `build` target produces three binaries: `gt`, `gt-proxy-server`, `gt-proxy-client`. README only documents `gt`; the two proxy binaries are undocumented.
- Install dir: `$(HOME)/.local/bin`.
- → [gastown/files/makefile.md](gastown/files/makefile.md), [gastown/binaries/gt.md](gastown/binaries/gt.md)
