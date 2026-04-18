---
title: Final validation after outgoing-calls sweep
type: drift
status: active
topic: gastown
created: 2026-04-18
updated: 2026-04-18
---

# Final validation after outgoing-calls sweep

Re-scored the same 20 open gastown issues tested in validation rounds 1-4,
the Phase 8 retest, and the cross-page inference retest against the current
wiki state, which now includes `## Outgoing calls` sections on 111 entity
pages documenting exact subprocess invocations, SQL mutations, env vars,
and file writes with `file:line` references.

## Full score history

| # | Issue | Original | Phase 8 | Cross-page | Final | Delta | Gap type |
|---|---|---|---|---|---|---|---|
| 3665 | Centralize cross-platform sleep strings | miss | miss | miss | miss | unchanged | -- |
| 3661 | `bd` `--rig` vs `--repo` | partial | partial | partial | partial | unchanged | detail-gap |
| 3658 | polecat remove zombie sessions | partial | full | full | full | unchanged | -- |
| 3653 | Dead code: HandlePolecatDone | partial | partial | partial | partial | unchanged | cross-page-inference |
| 3652 | witness/refinery Start() no agent beads | partial | partial | full | full | unchanged | -- |
| 3651 | Incorrect install instructions | partial | partial | partial | partial | unchanged | detail-gap |
| 3648 | doctor claude-settings convergence | full | full | full | full | unchanged | -- |
| 3626 | `go install` Linux macOS complaint | full | full | full | full | unchanged | -- |
| 3623 | Dolt connection exhaustion | partial | partial | partial | partial | unchanged | detail-gap |
| 3614 | Dashboard pi/omp detection | partial | partial | partial | partial | unchanged | detail-gap |
| 3604 | Refinery stall branch consolidation | partial | full | full | full | unchanged | -- |
| 3570 | Daemon legacy tmux sockets | partial | partial | partial | partial | unchanged | failure-mode-gap |
| 3565 | tmux keybindings stale prefix | full | full | full | full | unchanged | -- |
| 3563 | Nudge stale pane ID | partial | partial | partial | partial | unchanged | failure-mode-gap |
| 3562 | Rig init duplicate Dolt databases | full | full | full | full | unchanged | -- |
| 3554 | Daemon wisp config warning spam | partial | partial | partial | partial | unchanged | failure-mode-gap |
| 3539 | Sling bd update type rejection | full | full | full | full | unchanged | -- |
| 3538 | Windows not viable: tmux dependency | full | full | full | full | unchanged | -- |
| 3537 | Crew start outside tmux | partial | partial | partial | partial | unchanged | failure-mode-gap |
| 3516 | Rig names underscores, dolt prereq | full | full | full | full | unchanged | -- |

## Aggregate progression

| Score | Original | Phase 8 | Cross-page | Final |
|---|---|---|---|---|
| Full | 7 (35%) | 9 (45%) | 10 (50%) | 10 (50%) |
| Partial | 12 (60%) | 10 (50%) | 9 (45%) | 9 (45%) |
| Miss | 1 (5%) | 1 (5%) | 1 (5%) | 1 (5%) |

**Result: 10 full (50%), 9 partial (45%), 1 miss (5%).**

**No change from cross-page retest.** The outgoing-calls sweep did not
convert any additional partials to full.

---

## Per-issue scoring rationale

### #3665 -- Centralize cross-platform sleep command strings

**All rounds: miss.**

The issue proposes a new `internal/shellcmd` package that does not exist.
The outgoing-calls sections document existing subprocess invocations but
cannot document a package that has not been written. The tmux package
outgoing calls show `kill -TERM/-KILL` and `pgrep` subprocess calls but
no sleep command generation. Score unchanged.

---

### #3661 -- `bd` only has `--repo` flag, not `--rig`

**All rounds: partial. Gap: detail-gap.**

The outgoing-calls section on `gastown/packages/beads.md` now documents
13 `bd` subprocess invocations with exact `file:line` references. However,
the dynamic call at `beads.go:436` (where the `--rig` flag is passed) is
listed as `(dynamic) | fullArgs... | caller-provided`. The specific flag
name `--rig` vs `--repo` is not visible because the outgoing-calls
methodology documents the subprocess table at the call site level, and
the flag is constructed by the caller, not hardcoded at the call site.

Similarly, the `gt done` command page's outgoing calls show only
`gh pr create` -- the `bd` call that creates the MR bead happens through
the beads package, not directly from the done command.

The outgoing-calls sweep documents the right subprocess infrastructure
but the `--rig` vs `--repo` flag mismatch is a caller-provided argument
detail, not a call-site-level observable. Score unchanged.

---

### #3658 -- gt polecat remove leaves zombie tmux sessions

**Phase 8+: full.** Score was already full. The polecat command page
outgoing calls do not add new information beyond what the entity page
already covered. Score unchanged.

---

### #3653 -- Dead code: HandlePolecatDone / HandlePolecatDoneFromBead

**All rounds: partial. Gap: cross-page-inference.**

The witness package outgoing calls section documents only one subprocess
invocation: `gt mail send` at `handlers.go:733`. This does NOT help
identify `HandlePolecatDone` as dead code. The outgoing-calls methodology
documents what the code *does* (sends mail), not what code paths are
*never called*.

The witness page still lists `HandlePolecatDone` (`handlers.go:118+`) and
`HandlePolecatDoneFromBead` (`handlers.go:182+`) as live entry points
without dead-code annotations. An investigator must still cross-reference
callers across multiple pages to discover these are never invoked from
production code. Score unchanged.

---

### #3652 -- witness and refinery Start() never create agent beads

**Cross-page+: full.** Score was already full from the lifecycle
comparison table. The outgoing-calls sections do not add further
information. Score unchanged.

---

### #3651 -- Incorrect install instructions on 1.0 release

**All rounds: partial. Gap: detail-gap.**

The outgoing-calls sweep does not affect this issue. The gap is between
the wiki's documented npm scope (`@gastown/gt` from `goreleaser-yml.md`
and `inventory/auxiliary.md`) and the GitHub release notes' incorrect
scope (`@anthropic/gastown`). Outgoing-calls sections document code-level
subprocess invocations, not release documentation errors. Score unchanged.

---

### #3648 -- doctor claude-settings convergence

**All rounds: full.** The doctor and hooks command pages' outgoing calls
add subprocess detail but the issue was already fully diagnosable from
entity page content. Score unchanged.

---

### #3626 -- `go install` Linux macOS complaint

**All rounds: full.** The `BuiltProperly` ldflag gate remains thoroughly
documented. The Makefile outgoing calls reinforce the build flags but
the issue was already fully diagnosable. Score unchanged.

---

### #3623 -- Dolt connection exhaustion

**All rounds: partial. Gap: detail-gap.**

The doltserver outgoing calls document `dolt sql-server` startup at
`doltserver.go:1594` with "startup args | config" and
`DefaultMaxConnections = 1000` in the detail tables. The daemon outgoing
calls document `BEADS_DOLT_PORT` and `GT_DOLT_PORT` env vars propagated
to subprocess calls. The beads package outgoing calls show 13 `bd`
subprocess invocations.

However, the core gap remains: the wiki documents the Gas Town side
(`DefaultReadTimeoutMs = 300000`, `DefaultWriteTimeoutMs = 300000`,
`DefaultMaxConnections = 1000`) but NOT Dolt's own `wait_timeout`
default (28800s / 8 hours). The outgoing-calls sweep documents the
`dolt sql-server` invocation but not the MySQL session variable defaults
that Dolt inherits. The connection death spiral under concurrent agent
load (short-lived `bd` processes leaving connections open for 8 hours)
is still not documented.

The outgoing-calls sections bring the subprocess invocation count and
env-var propagation into sharp focus, which slightly improves navigation,
but the root cause (`wait_timeout`) is a MySQL server variable, not a
subprocess flag or env var. Score unchanged.

---

### #3614 -- Dashboard doesn't show pi/omp polecats

**All rounds: partial. Gap: detail-gap.**

The web package outgoing calls now document `tmux display-message -t
<session>:0.0 -p #{pane_current_command}` at `api.go:1817`. This is the
exact tmux command the dashboard uses to detect the current process in
each pane. Combined with the monitoring investigation workflow's
documentation of `GT_PROCESS_NAMES` and `IsAgentAlive`, an investigator
can now see more of the detection chain:

1. Dashboard calls `tmux display-message -p #{pane_current_command}`
   to get the process name
2. The process name is compared against expected names
3. Non-Claude agents (pi, omp) have different process names

However, the link between step 1 (in `web.md` outgoing calls) and
step 2-3 (in `tmux.md` and the monitoring investigation workflow) is
still not explicit. The web package Notes section says "Fetch errors
are never surfaced to HTTP clients -- they result in empty panels,"
which explains the silent failure mode but not the agent-type filtering
logic specifically.

The outgoing-calls section brings this closer -- the `#{pane_current_command}`
mechanism is now visible -- but the full detection chain requires
connecting three separate pages. Score unchanged, though the gap has
narrowed.

---

### #3604 -- Refinery stall branch consolidation

**Phase 8+: full.** Score was already full. Score unchanged.

---

### #3570 -- Daemon legacy tmux sockets

**All rounds: partial. Gap: failure-mode-gap.**

The daemon outgoing calls document `gt sling`, `gt convoy check`,
`gt scheduler run`, and various `dolt` subprocess invocations. The
`gt down` command outgoing calls document `ps -eo pid,comm,args` for
orphan Claude process detection at `down.go:756`.

Neither outgoing-calls section documents the tmux socket path
selection or the `cleanupLegacyDefaultSocket` /
`cleanupLegacyBaseSocket` functions. The outgoing-calls methodology
captures subprocess invocations, but the legacy socket gap is about
which tmux socket the daemon *connects to* (a library call to
`tmux.NewClient`), not a subprocess invocation. The tmux package
outgoing calls show the dynamic `tmux` calls at `tmux.go:177,231`
but the socket selection is embedded in the client constructor, not
in the call arguments.

The gap persists: daemon startup does not call legacy socket cleanup.
Score unchanged.

---

### #3565 -- tmux keybindings stale prefix

**All rounds: full.** Score unchanged.

---

### #3563 -- Nudge stale pane ID

**All rounds: partial. Gap: failure-mode-gap.**

The nudge package outgoing calls document only `gt nudge-poller <session>`
at `poller.go:93` and two file writes. The session package outgoing calls
document `(agent exe)` spawn at `agent_logging_unix.go:58` and `ps -o lstart=`
at `pidtrack.go:183`. Neither documents the `GT_PANE_ID` lifecycle.

The `GT_PANE_ID` env var is set during session creation (documented in the
session package detail tables) but the outgoing-calls section does not
cover it because `GT_PANE_ID` is set via tmux `set-environment`, which is
a library call through the tmux package, not a direct subprocess invocation
from the session package. The specific failure mode -- `GT_PANE_ID` is set
once at session creation and never refreshed on restart -- is still not
explicitly documented. Score unchanged.

---

### #3562 -- Rig init duplicate Dolt databases

**All rounds: full.** The doltserver outgoing calls now document
`CREATE DATABASE <rigName>` at `doltserver.go:2276` and
`DROP DATABASE IF EXISTS <dbName>` at `doltserver.go:2773`, which
reinforce the existing full coverage. Score unchanged.

---

### #3554 -- Daemon wisp config warning spam

**All rounds: partial. Gap: failure-mode-gap.**

The daemon outgoing calls document 26 subprocess invocations across
`convoy_manager.go`, `daemon.go`, `dolt_backup.go`, `dolt.go`,
`lifecycle.go`, and others. None of these are related to wisp config
reading. The wisp config warning is emitted from the daemon's patrol
loop (the heartbeat tick), which reads a file and logs a warning --
it does not invoke a subprocess or write to a file. The outgoing-calls
methodology does not capture log-emission patterns.

The daemon package page's existing content mentions "skipping parked
rigs" in the heartbeat loop at 5-second intervals, but the wisp config
check and its warning message are not documented. Score unchanged.

---

### #3539 -- Sling bd update type rejection

**All rounds: full.** The sling command outgoing calls now document
the exact `bd update` calls:
- `bd update <beadID> --status=open --assignee=` at `sling.go:750`
- `bd update <beadID> --status=pinned --assignee=<assignee>` at `sling.go:1083`

These document the `--status` flag usage but not the `--type=agent` call
that causes the issue. The `--type=agent` flag is passed from the polecat
spawn path (via the beads package), not from the sling command directly.
However, the issue was already fully diagnosable from the existing entity
page coverage of the custom type registration system. Score unchanged.

---

### #3538 -- Windows not viable: tmux dependency

**All rounds: full.** The outgoing-calls sections reinforce the
cross-platform concerns with explicit tmux subprocess calls throughout.
Score unchanged.

---

### #3537 -- Crew start outside tmux

**All rounds: partial. Gap: failure-mode-gap.**

The crew command outgoing calls document 5 `bd` and 1 `git` subprocess
invocations for the `remove` subcommand. The `start` subcommand's
outgoing calls are NOT documented because `crew start` delegates to the
crew package's `Start()` method, which in turn calls the session package's
`NewSessionWithCommandAndEnv` -- a chain of library calls, not direct
subprocess invocations from the command layer.

The session package outgoing calls document `(agent exe)` spawn at
`agent_logging_unix.go:58` but not the tmux session creation itself
(which is a library call to `tmux.NewSession` via the tmux package).
The tmux package outgoing calls show `tmux (dynamic)` at `tmux.go:177,231`
but the socket selection logic is embedded in the tmux client constructor.

The specific failure mode -- crew `Start()` creating a session on a
different tmux socket than `crew at` / `tmux list-sessions` queries --
remains undocumented because the socket is selected at the library level,
not passed as a subprocess argument. Score unchanged.

---

## Gap type distribution (remaining partials)

| Gap type | Count | Issues |
|---|---|---|
| failure-mode-gap | 4 | #3570, #3563, #3554, #3537 |
| detail-gap | 4 | #3661, #3651, #3623, #3614 |
| cross-page-inference | 1 | #3653 |

The gap distribution is identical to the cross-page retest.

## What the outgoing-calls sections specifically helped with

### What they accomplished

The outgoing-calls sweep added 111 structured tables documenting exact
subprocess invocations, SQL mutations, env vars, and file writes with
`file:line` references. For the 10 already-full issues, these sections
provide reinforcing evidence:

- **#3562 (rig init duplicate Dolt DBs):** `CREATE DATABASE <rigName>`
  at `doltserver.go:2276` is now visible in the outgoing-calls table,
  making the duplicate-DB creation path explicit.
- **#3539 (sling bd update type):** The `bd update` subprocess calls
  at `sling.go:750,1083` are documented with exact flags.
- **#3626 (go install macOS complaint):** The Makefile build flags are
  now documented alongside the `gt` binary page.
- **#3614 (dashboard pi/omp):** The `tmux display-message -p
  #{pane_current_command}` call at `api.go:1817` is now documented,
  narrowing the gap (though not closing it).

### Why they did not convert any partials

The remaining 9 partials and 1 miss fall outside the outgoing-calls
methodology's capture surface:

1. **Caller-provided dynamic arguments (#3661):** The `bd` subprocess
   call at `beads.go:436` is documented but with `fullArgs... |
   caller-provided`. The specific `--rig` flag is constructed upstream
   by the caller, not visible at the call site.

2. **MySQL server variables (#3623):** `wait_timeout` is a Dolt/MySQL
   server-side default (28800s), not a subprocess flag or env var that
   Gas Town passes. The outgoing-calls sweep captures what Gas Town
   *sends* to Dolt, not what Dolt configures internally.

3. **Library-level behavior (#3570, #3537):** The tmux socket selection
   happens in `tmux.NewClient()` constructor calls, not in subprocess
   arguments. Outgoing-calls capture subprocess invocations, not
   library-level constructor parameters.

4. **Env var lifecycle (#3563):** `GT_PANE_ID` is set via
   `tmux set-environment`, a library call through the tmux package.
   The staleness-across-restart problem is about when the env var is
   *not* refreshed, which is an absence of a call, not a call.

5. **Log emission patterns (#3554):** The wisp config warning is a
   `log.Warn()` call in the daemon heartbeat loop. Outgoing-calls
   document subprocess calls and file writes, not log emissions.

6. **Dead code identification (#3653):** Outgoing-calls document what
   code *does*, not what code is *never called*. Dead code detection
   requires caller analysis, which is outside the methodology.

7. **External documentation errors (#3651):** The npm scope mismatch
   is between the wiki and GitHub release notes, not a code-level issue.

8. **Cross-page detection chains (#3614):** The `#{pane_current_command}`
   tmux call is now documented in `web.md`, but connecting it to
   `GT_PROCESS_NAMES` (in `tmux.md`) and `IsAgentAlive` (in the
   monitoring workflow) requires cross-page inference.

9. **Feature requests (#3665):** The proposed `internal/shellcmd`
   package does not exist.

### Structural insight

The outgoing-calls sweep was designed to surface "what does this code
call, with what arguments, and where?" This is maximally useful for
issues where the bug is in a *wrong argument* passed to a subprocess.
But in the remaining partials, the bugs are at different levels:

| Level | Example issues | Outgoing-calls captures it? |
|---|---|---|
| Wrong subprocess argument | (none remaining) | Yes |
| Caller-provided dynamic argument | #3661 | No -- caller is upstream |
| MySQL server default | #3623 | No -- server-side config |
| Library constructor parameter | #3570, #3537 | No -- not a subprocess |
| Env var lifecycle (absence of refresh) | #3563 | No -- absence of call |
| Log emission pattern | #3554 | No -- not a subprocess/file write |
| Dead code (absence of callers) | #3653 | No -- absence of call |
| External documentation | #3651 | No -- not in code |
| Agent detection chain | #3614 | Partial -- first step documented |
| Feature request | #3665 | No -- code does not exist |

The outgoing-calls sections are a high-value structural asset for future
issues where the root cause is a wrong flag, missing env var, or
incorrect SQL statement. They did not convert these specific 9 partials
because none of those issues have root causes at the subprocess-argument
level.

## Remaining path to 60% target

To reach 12/20 full (60%), two more partials must be converted. The
same two highest-value targets identified in the cross-page retest
remain:

1. **#3653 (cross-page-inference):** Add a dead-code annotation to the
   witness package page flagging `HandlePolecatDone` and
   `HandlePolecatDoneFromBead` as test-only code superseded by
   `DiscoverCompletions`. (+1 full)

2. **#3563 (failure-mode-gap):** Add a note to the session or nudge
   package page about `GT_PANE_ID` staleness across session restarts:
   "set once at session creation, never refreshed on restart." (+1 full)

These two changes would yield 12/20 = 60%.
