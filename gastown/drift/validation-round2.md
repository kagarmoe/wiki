---
title: Wiki validation round 2
type: drift
status: active
topic: gastown
created: 2026-04-17
updated: 2026-04-17
phase: 7
---

# Wiki validation round 2

Five more open gastown issues scored against the wiki to measure how well
it supports issue investigation without reading gastown source.

## Summary

| Issue | Title | Score | Gap type |
|---|---|---|---|
| #3651 | Incorrect install instructions on 1.0 release | partial | npm package name documented but install command not verified against release notes |
| #3648 | doctor: claude-settings never converges for polecats | full | wiki documents both sides of the conflict: doctor checks for `costs record` in Stop hook, hooks package sets polecats to `gt tap polecat-stop-check` |
| #3626 | `go install` method on Linux complains about macOS | full | wiki documents the exact self-kill check, BuiltProperly ldflag, and error message verbatim |
| #3623 | Dolt connection exhaustion and slow mail/beads queries | partial | wiki documents DefaultMaxConnections=1000, read/write timeouts, CLOSE_WAIT prevention, but not wait_timeout or connection lifecycle under load |
| #3614 | Dashboard doesn't show pi/omp polecats | partial | wiki documents the fetcher's subprocess-based detection but doesn't detail agent-type-specific detection logic |

**Scores: 2 full, 3 partial, 0 miss.**

---

### #3651 -- Incorrect install instructions on 1.0 release

**Issue summary:** The v1.0.0 release notes reference `npm install -g
@anthropic/gastown` as the install command. The reporter flags this as
incorrect.

**Wiki pages checked:**
- [.goreleaser.yml](../files/goreleaser-yml.md) -- release pipeline config
- [gt install](../commands/install.md) -- workspace setup command
- [gt](../binaries/gt.md) -- main CLI binary
- [Makefile](../files/makefile.md) -- canonical build recipe
- [flake.nix](../files/flake-nix.md) -- Nix flake
- [inventory/auxiliary.md](../inventory/auxiliary.md) -- npm-package directory

**What the wiki knows:** The wiki documents the npm package at
`npm-package/` under the scope `@gastown/gt` (in
`gastown/inventory/auxiliary.md`: "npm-package/ -- @gastown/gt npm
installer" and `gastown/files/goreleaser-yml.md`: "npm: `npm install -g
@gastown/gt`"). The goreleaser page also notes that `npm-package/`
exists at the repo root but "the release flow here doesn't reference it."
The wiki documents four build/install paths: `make build`/`make install`,
GoReleaser releases, Nix flake, and npm. The `gt install` command page
documents workspace initialization, not binary installation.

**What the wiki doesn't know:** The wiki records the npm scope as
`@gastown/gt`, while the issue reports the release notes say
`@anthropic/gastown`. The wiki doesn't verify which command the GitHub
release page actually displays to users. If the release notes were
manually written with a wrong package name, the wiki wouldn't catch
that -- it documented the correct npm scope from the source tree but
doesn't cross-reference against the published release page.

**Score:** partial

**Remediation:** The wiki has the correct package name (`@gastown/gt`)
documented from source. This is sufficient to identify the issue
reporter's concern (the release page says `@anthropic/gastown` which
doesn't match). A dedicated "installation methods" concept page
cross-referencing all install paths (npm, Homebrew, `go install`,
`make build`, Nix) against their documented locations would make this
easier to validate. No wiki page currently consolidates all install
paths in one place.

---

### #3648 -- doctor: claude-settings never converges for polecats

**Issue summary:** `gt doctor` reports polecat `.claude/settings.json`
files as stale because the Stop hook doesn't contain `costs record`.
But `gt hooks sync` regenerates the file with `gt tap polecat-stop-check`
as the Stop hook (which is the correct polecat override). The two
subsystems disagree on what the correct Stop hook should be, creating
an infinite fix-then-regenerate cycle.

**Wiki pages checked:**
- [internal/doctor](../packages/doctor.md) -- health-check registry
- [internal/hooks](../packages/hooks.md) -- hook configuration management
- [gt hooks](../commands/hooks.md) -- CLI surface for hooks
- [gt doctor](../commands/doctor.md) -- diagnostics command
- [gt tap](../commands/tap.md) -- tap commands including polecat-stop-check

**What the wiki knows:** The wiki documents both sides of this conflict
with enough precision to identify the root cause:

1. **Doctor side:** The `internal/doctor` package page documents
   `ClaudeSettingsCheck` at `claude_settings_check.go:532-573`, stating
   that `checkSettings` "verifies the presence of `enabledPlugins`, a
   `SessionStart` hook containing `prime --hook`, and a **`Stop` hook
   containing `costs record`**."

2. **Hooks side:** The `internal/hooks` package page documents
   `DefaultOverrides()` at `config.go:207-351`, stating: "**`polecats`:
   `Stop -> gt tap polecat-stop-check`** (catches idle polecats who
   forgot to run `gt done`)." The base config's Stop hook is
   `gt costs record &`.

3. The wiki also documents that `ClaudeSettingsCheck` must run before
   `DaemonCheck` (ordering constraint), and that its fix path deletes
   stale files and calls `runtime.EnsureSettingsForRole` to regenerate
   them -- which would produce files matching the hooks package's output,
   not doctor's expectation.

An investigator reading the wiki would see that doctor checks for
`costs record` in Stop, hooks sets polecats' Stop to
`gt tap polecat-stop-check`, and the fix path regenerates using the hooks
system -- a clear convergence failure. The wiki provides all three pieces
needed to diagnose the bug.

**Score:** full

**Remediation:** None needed for diagnosis. Once the fix lands (doctor
should accept role-specific Stop hooks, not just the base `costs record`),
update the `ClaudeSettingsCheck` section in `gastown/packages/doctor.md`
to reflect the corrected template conformance scan.

---

### #3626 -- `go install` method on Linux complains about macOS

**Issue summary:** Running `go install github.com/steveyegge/gastown/cmd/gt@latest`
on Linux produces the error: "ERROR: This binary was built with 'go build'
directly. macOS will SIGKILL unsigned binaries. Use 'make build' instead."
The error is inappropriate for Linux.

**Wiki pages checked:**
- [gt](../binaries/gt.md) -- main CLI binary, self-kill check
- [Makefile](../files/makefile.md) -- canonical build recipe, LDFLAGS
- [.goreleaser.yml](../files/goreleaser-yml.md) -- release pipeline
- [internal/version](../packages/version.md) -- build metadata

**What the wiki knows:** The wiki documents this exact mechanism with
the error message verbatim. From `gastown/binaries/gt.md`:

> `/home/kimberly/repos/gastown/internal/cmd/root.go:94-107` -- the
> `persistentPreRun` function runs before every subcommand. First thing
> it does:
>
> ```go
> if BuiltProperly == "" && Build == "dev" {
>     fmt.Fprintln(os.Stderr, "ERROR: This binary was built with 'go build' directly.")
>     fmt.Fprintln(os.Stderr, "       macOS will SIGKILL unsigned binaries. Use 'make build' instead.")
>     // ...
>     os.Exit(1)
> }
> ```
>
> Any command (including `gt --version` and `gt completion bash`) hits
> this gate. Bypassed only when `BuiltProperly="1"` OR `Build != "dev"`.

The Makefile page confirms that `make build` sets
`-X ...cmd.BuiltProperly=1` via LDFLAGS. The goreleaser page documents
that release binaries use `Build={{.ShortCommit}}` (not `"dev"`), so they
pass the `Build != "dev"` branch. The wiki makes clear that `go install`
(which sets neither `BuiltProperly` nor `Build`) will always trigger
this gate -- and that the error message mentioning macOS is misleading
on Linux because the check is platform-agnostic.

An investigator could fully diagnose this issue from the wiki alone: the
self-kill check doesn't test `runtime.GOOS`, it fires unconditionally
when `BuiltProperly` and `Build` are both at their defaults.

**Score:** full

**Remediation:** None needed for diagnosis. Once the fix lands (either
making the error message platform-aware, or allowing `go install` to
work), update the self-kill check section in `gastown/binaries/gt.md`.

---

### #3623 -- Dolt connection exhaustion and slow mail/beads queries

**Issue summary:** Two related problems: (1) Dolt's default
`wait_timeout` of 28800s (8 hours) causes connections from short-lived
`bd` processes to linger, eventually hitting the 1000-connection limit
in multi-agent setups. (2) Commands like `gt mail inbox` take 23-58
seconds because they spawn multiple sequential `bd` subprocesses, each
with ~1-2s of Go binary startup and connection overhead.

**Wiki pages checked:**
- [internal/doltserver](../packages/doltserver.md) -- Dolt server lifecycle
- [internal/beads](../packages/beads.md) -- beads library
- [internal/mail](../packages/mail.md) -- mail package
- [internal/health](../packages/health.md) -- health primitives
- [gt dolt](../commands/dolt.md) -- dolt CLI surface
- [gt mail](../commands/mail.md) -- mail CLI surface

**What the wiki knows:** The doltserver page documents several relevant
details:

- `DefaultMaxConnections = 1000` (the limit the issue hits).
- `DefaultReadTimeoutMs` and `DefaultWriteTimeoutMs` both = 5 minutes,
  with an explicit note: "they prevent CLOSE_WAIT accumulation from
  abandoned connections during compactor GC."
- `GetActiveConnectionCount` and `HasConnectionCapacity` health methods.
- The `Config` struct includes `MaxConnections`, `ReadTimeoutMs`,
  `WriteTimeoutMs`.
- The `writeServerConfig` function writes `config.yaml` from the Config
  struct, and "CLI flags are ignored by dolt when `--config` is set."

The beads page documents that every `bd` subprocess connects to the
Dolt server via MySQL protocol. The mail page documents the mail query
path.

**What the wiki doesn't know:** The wiki does not document:
- Dolt's default `wait_timeout` (28800s) or its role in connection
  exhaustion -- the read/write timeouts are documented but `wait_timeout`
  (a MySQL server-level idle connection timeout) is a different parameter
  not surfaced in the `Config` struct.
- The subprocess-per-query overhead pattern: the wiki notes that `bd`
  subprocesses connect via MySQL but doesn't quantify the per-invocation
  cost (~1-2s Go startup + connection init) or flag it as a performance
  bottleneck.
- Connection pooling (or lack thereof): the wiki doesn't discuss whether
  connections are pooled or each `bd` invocation opens a fresh connection.
- The specific thundering-herd scenario where many agents' `bd` processes
  accumulate faster than connections expire.

The wiki provides enough to understand the connection limit (1000) and
the architecture (subprocess per bd call), but an investigator would need
source access to understand why connections accumulate (wait_timeout) and
why mail is slow (sequential bd spawns).

**Score:** partial

**Remediation:** Add a "Connection lifecycle" section to
`gastown/packages/doltserver.md` covering `wait_timeout` defaults,
connection accumulation under multi-agent load, and the gap between
per-process read/write timeouts and idle connection lifetime. Add a
performance note to the `internal/beads` or `internal/mail` pages about
the subprocess-per-query overhead pattern. Once a fix lands (e.g.,
`wait_timeout: 30` in config template), document the change.

---

### #3614 -- Dashboard doesn't show pi/omp polecats

**Issue summary:** `gt dashboard` shows "No polecats" and "No active
sessions" when polecats are running through non-Claude agents (pi, omp).
`gt rig status` correctly detects them. The likely cause is that the
dashboard's detection logic relies on Claude Code-specific process names
or session metadata.

**Wiki pages checked:**
- [gt dashboard](../commands/dashboard.md) -- dashboard command
- [internal/web](../packages/web.md) -- HTTP server, fetcher, templates
- [internal/polecat](../packages/polecat.md) -- polecat manager
- [internal/session](../packages/session.md) -- session management
- [internal/tmux](../packages/tmux.md) -- tmux wrapper
- [polecat-lifecycle](../workflows/polecat-lifecycle.md) -- lifecycle flow
- [internal/config](../packages/config.md) -- agent config, role_agents

**What the wiki knows:** The web package page documents the
`LiveConvoyFetcher` and its `FetchSessions` / `FetchWorkers` methods.
It notes that most fetch methods "shell out to `bd list --flat --json`,
`gh pr list --json`, or `tmux list-sessions` and marshal the output
into the presentation row structs." The `SessionRow` struct in
`templates.go` includes status fields, and `DashboardSummary` tracks
`PolecatCount`, `DeadSessions`, and `StuckPolecats`.

The dashboard command page documents the `web.NewLiveConvoyFetcher()`
constructor, which builds a `session.PrefixRegistry` from the town.
The session package page documents prefix-registry parsing for mapping
tmux session names to rig/role components.

The `ZombieSessionCheck` in the doctor package is documented as checking
"whether the pane still has a Claude/node process" -- suggesting
Claude-specific process detection is a known pattern in the codebase.

**What the wiki doesn't know:** The wiki documents the fetcher interface
and its subprocess-based approach but doesn't detail the agent-type
filtering logic inside `FetchWorkers` or `FetchSessions`. Specifically:
- Whether the fetcher filters by process type (Claude vs pi vs omp)
  when enumerating active sessions.
- Whether `tmux list-sessions` output is filtered by session name
  pattern, and whether pi/omp sessions use a different naming convention.
- Whether the `config` package's `role_agents` configuration (which maps
  roles to agent types) is consulted by the dashboard fetcher.

The wiki mentions the `ZombieSessionCheck` probes for "a Claude/node
process" in the pane, hinting that process-type assumptions are embedded
in the detection logic. An investigator would find the right area (web
fetcher + session prefix registry) but would need source access to
confirm the agent-type filtering hypothesis.

**Score:** partial

**Remediation:** Expand the `FetchWorkers` / `FetchSessions` sections
in `gastown/packages/web.md` to document how active sessions are detected
and whether the detection assumes a specific agent type. Add a note about
multi-agent detection gaps if the fetcher is Claude-specific. The
`internal/session` page could also document whether session name patterns
vary by agent type. Once a fix lands, update the wiki to reflect
agent-agnostic detection.
