---
title: "Investigating: daemon infrastructure failures"
type: investigation
status: verified
topic: gastown
created: 2026-04-17
updated: 2026-04-17
sources:
  - gastown/commands/daemon.md
  - gastown/commands/boot.md
  - gastown/commands/deacon.md
  - gastown/commands/up.md
  - gastown/commands/down.md
  - gastown/packages/daemon.md
  - gastown/packages/boot.md
  - gastown/packages/deacon.md
  - gastown/packages/tmux.md
  - /home/kimberly/repos/gastown/internal/daemon/daemon.go
  - /home/kimberly/repos/gastown/internal/cmd/daemon.go
  - /home/kimberly/repos/gastown/internal/cmd/up.go
  - /home/kimberly/repos/gastown/internal/cmd/down.go
  - /home/kimberly/repos/gastown/internal/cmd/boot.go
  - /home/kimberly/repos/gastown/internal/deacon/manager.go
  - /home/kimberly/repos/gastown/internal/boot/boot.go
tags: [investigation, daemon-infrastructure, diagnostic, daemon, boot, deacon]
---

# Investigating: daemon infrastructure failures

**Symptom:** The daemon won't start or keeps crashing. Boot fails to
triage the Deacon. The Deacon is missing or unresponsive. `gt up`
partially succeeds. `gt down` leaves processes behind. The shutdown
sentinel persists after a crash and blocks `gt up`.

Gas Town's infrastructure layer has three components:

- **Daemon** — a Go process (not a Claude agent) that runs as a
  singleton per town. Holds an exclusive flock, fires recovery
  heartbeats, manages Dolt, restarts dead agents. See
  [daemon](../../packages/daemon.md).
- **Boot** — a short-lived agent spawned fresh on each daemon tick to
  triage the Deacon. NOT a kennel dog despite the filesystem path
  `~/gt/deacon/dogs/boot/`. See [boot](../../packages/boot.md) and
  [gt boot](../../commands/boot.md) ## Drift.
- **Deacon** — a Claude agent in a persistent tmux session that
  patrols rigs, health-checks agents, and handles recovery. See
  [deacon](../../packages/deacon.md).

The supervision chain: daemon ticks -> Boot triage -> Deacon patrol ->
agent health checks. Each layer depends on the one below it.

## Decision tree

### 0. Is the tmux server alive?

**Shared prefix with [agent-lifecycle](agent-lifecycle.md).**

**Check:**

```bash
tmux -L $(gt town socket-name 2>/dev/null || echo "gt") list-sessions 2>&1
```

- **"no server running"** -> the per-town tmux server is down. The
  Deacon and Boot sessions are dead (they're tmux sessions). The
  daemon may still be alive (it's a Go process, not a tmux session).
  Run `gt up` to restart everything. `gt up` sets
  `t.SetExitEmpty(false)` so the tmux server survives even after all
  sessions die.
- **Sessions listed** -> tmux server is alive. Continue.

### 1. Is the daemon running?

**Check:**

```bash
gt daemon status
# or manually:
cat ~/gt/daemon/daemon.pid 2>/dev/null && \
  kill -0 $(cat ~/gt/daemon/daemon.pid) 2>/dev/null && \
  echo "alive" || echo "dead"
```

`daemon.IsRunning(townRoot)` at `daemon.go` reads the PID file at
`<townRoot>/daemon/daemon.pid` and probes with `signal(0)`.

- **Dead or PID file missing** -> go to
  [Step 2: Why isn't the daemon running?](#2-why-isnt-the-daemon-running)
- **Alive** -> go to
  [Step 4: Is the daemon healthy?](#4-is-the-daemon-healthy)

### 2. Why isn't the daemon running?

**a. Shutdown sentinel blocking restart:**
`gt up`'s `ensureDaemon` at `up.go:504-566` checks for a
`<townRoot>/daemon/shutting-down` sentinel file (GH#2656). If the
sentinel exists and its PID is alive, daemon start is refused
(rightfully — a shutdown is in progress). If the sentinel's PID is
dead (GH#2907), the sentinel is treated as stale and removed.

**Check:**

```bash
cat ~/gt/daemon/shutting-down 2>/dev/null
```

If the file exists and contains a dead PID, `gt up` should
auto-clean it. If it persists after a crash (`down.go:124` defers
removal but abnormal termination skips defers), manually remove it:

```bash
rm ~/gt/daemon/shutting-down
```

See [gt down](../../commands/down.md) ## Failure modes -> Partial
completion.

**b. Lock held by another process:**
`daemon.Run` at `daemon.go:340-352` acquires an exclusive flock on
`<townRoot>/daemon/daemon.lock`. If another daemon instance holds the
lock, `TryLock()` fails and the new daemon exits with "daemon already
running (lock held by another process)."

**Check:**

```bash
fuser ~/gt/daemon/daemon.lock 2>/dev/null
```

If the lock holder's PID is dead but the file descriptor is still
held (can happen with orphaned subprocesses), the lock persists until
the holder is killed. Note: the lock file is **never removed** —
flock works on file descriptors, not paths (see
[gt down](../../commands/down.md) ## Notes).

**c. Rig backend check failed:**
`daemon.go:354-357` runs `checkAllRigsDolt()` before acquiring the
lock. If any rig is not on the Dolt backend, the daemon refuses to
start.

**Check:** `gt dolt status` — are all rigs configured for Dolt?

**d. Startup failure (short-lived crash):**
`gt daemon start` (`cmd/daemon.go:169-235`) re-execs `gt daemon run`
as a detached child, then polls `daemon.IsRunning` every 100ms for up
to 3 seconds. If the daemon crashes during startup, the poll times
out. `readDaemonStartupFailure` at `cmd/daemon.go:320-336` reads the
daemon log for a `"Daemon startup failed (PID N):"` line.

**Check:**

```bash
gt daemon logs -n 50  # or: tail -50 ~/gt/daemon/daemon.log
```

Look for the startup failure line. Common causes: port conflict for
Dolt, corrupt database, metadata repair failure.

### 3. Daemon start via `gt up` — additional nuances

`gt up` starts the daemon in Phase 1 via `ensureDaemon` at
`up.go:504-566`, which is **shorter** than `gt daemon start`'s poll
loop. It sleeps a single `daemonStartupGrace` (300ms on Unix, 2s on
Windows at `up.go:113-118`) then verifies once. If the daemon takes
longer than the grace window, `gt up` may report a transient failure
even though the daemon comes up moments later.

**Fix:** Re-run `gt up` — it's idempotent and will see the daemon as
already running on the second attempt.

### 4. Is the daemon healthy?

The daemon is running. Is it doing its job?

**a. Heartbeat check:**
`gt daemon status` shows the last heartbeat time and counter. The
daemon fires a recovery heartbeat at `d.recoveryHeartbeatInterval()`
(configurable, default varies). If the heartbeat counter hasn't
incremented, the daemon goroutine may be stuck.

**b. Binary version mismatch:**
`gt daemon status` compares the binary's mtime to the daemon's start
time. If the binary is newer, it prints:
`"Binary is newer than process — consider 'gt daemon stop && gt daemon start'"`.

**c. Agent restart tracking:**
The `RestartTracker` at `daemon/restart_tracker.go:14-45` applies
exponential backoff (30s initial, 10m max, 5 crashes before crash-loop
state). If an agent is in crash-loop state, the daemon stops restarting
it.

**Check:**

```bash
cat ~/gt/daemon/restart_state.json
gt daemon clear-backoff <agent>  # Reset if needed
```

### 5. Is Boot running its triage cycle?

Boot is spawned fresh on each daemon tick. It is NOT a persistent
session.

**Check:**

```bash
gt boot status [--json]
```

This shows: `running` (triage in progress), `session_alive` (tmux
session exists), `degraded` (no-tmux mode), last action, last error.

**a. Boot is not running and last action is recent:**
Normal. Boot runs and exits within seconds. The status file at
`<townRoot>/deacon/dogs/boot/.boot-status.json` records each
execution.

**b. Boot last action shows an error:**
`runBootSpawn` at `cmd/boot.go:217` saves the error to the status
file. But the save itself uses `_ = b.SaveStatus(status)` — if the
save fails, the status file has stale data. See
[gt boot](../../commands/boot.md) ## Failure modes -> Silent
suppression.

**c. Boot triage lock stuck:**
`boot.go:241` defers `_ = b.ReleaseLock()`. If the lock release
fails (or the previous Boot was killed mid-triage), the lock file at
`<townRoot>/deacon/dogs/boot/.boot-running` persists. Subsequent
triage runs fail to acquire the lock.

**Fix:** Remove the stale lock file:

```bash
rm ~/gt/deacon/dogs/boot/.boot-running
```

### 6. Is the Deacon alive?

The Deacon is a persistent Claude session. Boot starts it if missing.

**Check:**

```bash
gt deacon status [--json]
```

This shows: pause state, tmux running state, heartbeat state (age,
cycle, last action, health label).

- **Not running, Boot is running** -> Boot should restart it on the
  next daemon tick. Wait for one heartbeat cycle, then recheck.
- **Not running, daemon also down** -> start the whole stack:
  `gt up`.
- **Running but paused** -> `gt deacon resume` to unpause. A paused
  Deacon cannot update its heartbeat, so the daemon will eventually
  detect it as stale and try to poke it. Whether that poke succeeds
  on a paused Deacon is ambiguous. See
  [gt deacon](../../commands/deacon.md) ## Notes.
- **Running but heartbeat stale/very stale** -> the Deacon session
  exists but hasn't called `gt deacon heartbeat` recently. It may be
  stuck in a long tool call, or its patrol molecule may have crashed.
  Attach and check: `gt deacon attach`.

### 7. Deacon start failures

`deacon.NewManager(townRoot).Start(agentOverride)` at
`deacon/manager.go:75-185`.

**a. Session creation failures:** Same patterns as
[agent-lifecycle](agent-lifecycle.md) Step 3 — tmux session creation,
runtime settings, startup command building.

**b. Environment setup silently fails:**
`startDeaconSession` at `cmd/deacon.go:542-549` sets environment
variables (`GT_ROLE`, `GT_TOWN_ROOT`, etc.) with
`_ = t.SetEnvironment(...)`. If tmux env fails, the Deacon runs
without role detection env vars. Agents detect these via cwd fallback,
but the fallback may resolve wrong. See
[gt deacon](../../commands/deacon.md) ## Failure modes -> Silent
suppression.

**c. Startup dialog hang:**
`cmd/deacon.go:573` runs `runtime.RunStartupFallback` with
`_ =` discard. If startup dialog handling fails, the Deacon may hang
at a trust prompt. This is an **absent failure mode** — no error
propagation. See [gt deacon](../../commands/deacon.md) ## Failure
modes -> Silent suppression.

**Fix:** Attach to the Deacon session manually and accept any dialogs.

### 8. Shutdown failures (`gt down` leaves processes behind)

`gt down` at `cmd/down.go:45-471` runs a 6-phase shutdown sequence.
Partial completion can leave processes behind.

**a. Shutdown lock contention:**
`acquireShutdownLock` at `down.go:678-700` takes an exclusive flock
with a 5-second timeout. If another `gt down` holds the lock, the
second one times out.

**b. Sentinel persists after crash:**
The `shutting-down` sentinel is deferred for removal at `down.go:124`.
On abnormal termination (panic, SIGKILL), the defer doesn't fire. The
stale sentinel blocks `ensureDaemon` from restarting the daemon.

**Fix:** Manually remove: `rm ~/gt/daemon/shutting-down`

**c. Imposter Dolt servers respawn:**
`gt down` Phase 4b-i stops `bd dolt idle-monitors` BEFORE stopping
Dolt (Phase 4b-ii). If the order is wrong, idle monitors respawn
per-agent Dolt servers that grab port 3307 before the canonical server
restarts. `gt down` Phase 4b-iii cleans up imposter and orphan Dolt
servers specifically to handle this race.

**d. Verification finds respawned processes:**
`verifyShutdown` at `down.go:704-743` runs 500ms after Phase 5 and
checks for surviving tmux sessions, daemon PID, orphan Claude/node
processes, and respawned idle monitors. If anything survives, `allOK`
flips to false and the output reports "respawned" components.

**Check:** `gt down --all` with `--force` for the most thorough
cleanup. If processes persist after that, check for non-Gas-Town
processes that happen to match the cleanup patterns.

### 9. `gt up` partial completion

`gt up` at `cmd/up.go:160-448` starts 5 services in parallel (Phase 1)
with no rollback. If daemon succeeds but deacon fails, there's no
automatic cleanup of the daemon. This is by design — `gt up` is
idempotent, so re-running it starts the missing services.

**Phase ordering matters:**
- Phase 1: Dolt, daemon, deacon, mayor start in parallel
- Phase 2: Dolt readiness gate (10s timeout) — only gates
  witnesses/refineries, NOT the daemon. If the daemon reads Dolt
  during its first heartbeat, it races the Dolt server startup.
- Phase 3: Witnesses + refineries (gated on Dolt)
- Phase 4: Crew + polecats (opt-in via `--restore`)

**Check:** `gt up --json` for a structured report of which services
succeeded/failed. Re-run `gt up` to start any that failed.

## Cycles and shared dependencies

- **Daemon -> Boot -> Deacon -> agents:** The supervision chain is
  strictly layered. A dead daemon means no Boot triage, which means
  no Deacon restart, which means no agent health checks.
- **Daemon depends on flock, not tmux:** The daemon is a Go process
  with file-based locking. It survives tmux server crashes. The
  Deacon and Boot depend on tmux.
- **Boot is daemon-tick-driven:** Boot runs fresh per tick. There is
  no `boot stop` command because Boot's lifecycle is the daemon's
  heartbeat. Killing the `gt-boot` session is the closest equivalent.
- **Shutdown lock is never removed:** `down.go:114-120` explains the
  flock-on-inode semantics. Removing the lock file while another
  waiter is queued causes a phantom-lock split.
- **`gt down` recommends `gt start` as complement, but `gt up` is the
  actual complement.** The Long text at `down.go:66` says "use
  'gt start' to bring everything back up" — but `gt start` does NOT
  start the daemon. This is a known cobra drift finding. See
  [gt down](../../commands/down.md) ## Drift.

## Related investigation workflows

- [Investigating: agent lifecycle](agent-lifecycle.md) -- agent
  start/stop/crash failures. Shares the "is tmux alive?" prefix.
- [Investigating: data-plane failures](data-plane.md) -- Dolt
  server issues that affect daemon startup and agent operations.
- [Investigating: message delivery](message-delivery.md) -- nudge
  delivery failures that may prevent the Deacon from receiving
  health-check responses.
