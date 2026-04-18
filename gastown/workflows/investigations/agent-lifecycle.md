---
title: "Investigating: agent lifecycle failures"
type: investigation
status: verified
topic: gastown
created: 2026-04-17
updated: 2026-04-17
sources:
  - gastown/commands/polecat.md
  - gastown/commands/witness.md
  - gastown/commands/refinery.md
  - gastown/commands/crew.md
  - gastown/packages/polecat.md
  - gastown/packages/witness.md
  - gastown/packages/refinery.md
  - gastown/packages/crew.md
  - gastown/packages/session.md
  - gastown/packages/tmux.md
  - /home/kimberly/repos/gastown/internal/polecat/session_manager.go
  - /home/kimberly/repos/gastown/internal/witness/manager.go
  - /home/kimberly/repos/gastown/internal/refinery/manager.go
  - /home/kimberly/repos/gastown/internal/crew/manager.go
  - /home/kimberly/repos/gastown/internal/tmux/tmux.go
  - /home/kimberly/repos/gastown/internal/session/lifecycle.go
tags: [investigation, agent-lifecycle, diagnostic, polecat, witness, refinery, crew]
---

# Investigating: agent lifecycle failures

**Symptom:** An agent (polecat, witness, refinery, or crew) fails to
start, stops unexpectedly, or appears stuck. `gt agents list` shows
a session as missing or unhealthy. `gt witness status`, `gt refinery
status`, or `gt crew status` reports "not running" when it should be
running.

Gas Town has four agent types with distinct lifecycle shapes:

- **Polecat** (ephemeral): worktree-backed, witness-managed, auto-nuked
  after work. Start path: [polecat](../../packages/polecat.md)
  `SessionManager.Start` at `session_manager.go:222`.
- **Witness** (per-rig): health monitor for polecats. Start path:
  [witness](../../packages/witness.md) `Manager.Start` at
  `manager.go:110`.
- **Refinery** (per-rig): merge queue processor. Start path:
  [refinery](../../packages/refinery.md) `Manager.Start` at
  `manager.go:113`.
- **Crew** (persistent): user-managed full git clones. Start path:
  [crew](../../packages/crew.md) `Manager.Start` at
  `manager.go:182+`.

All four depend on tmux for session hosting and use the shared
[session](../../packages/session.md) substrate for naming, identity
parsing, and startup sequences.

## Decision tree

### 0. Is the tmux server alive?

**Shared prefix with [daemon-infrastructure](daemon-infrastructure.md).**

**Check:**

```bash
tmux -L $(gt town socket-name 2>/dev/null || echo "gt") list-sessions 2>&1
```

- **"no server running"** or **"error connecting"** -> the per-town
  tmux server is down. All agent sessions are dead. Run `gt up` to
  restart everything. See
  [daemon-infrastructure](daemon-infrastructure.md) Step 1.
- **Sessions listed** -> tmux server is alive. Continue.

### 1. Which agent type is failing?

**Check:** What agent are you investigating?

- **Polecat** -> [Step 2: Polecat start failures](#2-polecat-start-failures)
- **Witness** -> [Step 5: Witness start failures](#5-witness-start-failures)
- **Refinery** -> [Step 7: Refinery start failures](#7-refinery-start-failures)
- **Crew** -> [Step 9: Crew start failures](#9-crew-start-failures)
- **Agent appears running but is stuck** -> [Step 11: Zombie/hung detection](#11-zombiehung-detection)

### 2. Polecat start failures

`SessionManager.Start(polecat, opts)` at `session_manager.go:222-400`
runs a long sequence. Common failure points:

**a. Polecat identity not found:**
`Start` checks `m.hasPolecat(polecat)` at `session_manager.go:223`.
If the polecat name doesn't exist in the rig's `polecats/` directory,
returns `ErrPolecatNotFound`. See [polecat](../../packages/polecat.md)
for identity management.

**Fix:** Create the identity first: `gt polecat identity add <rig> <name>`.

**b. Session already running:**
`session_manager.go:236-243` checks for an existing session. If the
session exists and is NOT stale (agent process alive), returns
`ErrSessionRunning`. If the session IS stale (`isSessionStale` returns
true), it kills the zombie and proceeds.

**Check:** `gt polecat status <rig> <name>` or
`tmux list-sessions | grep <polecat-session-name>`.

**c. Issue validation failure:**
If `opts.Issue` is set, `validateIssue` at `session_manager.go:254-258`
checks the issue exists and isn't tombstoned BEFORE creating the session.
This prevents CPU spin loops from agents retrying invalid work.

**Check:** `bd show <issue-id>` — does the issue exist? Is it tombstoned?

**d. Agent config resolution failure:**
`session_manager.go:268-278` resolves the runtime config for the
specified agent. If `--agent` specifies an unknown agent preset,
`ResolveAgentConfigWithOverride` errors.

**Check:** `gt config agents list` — is the specified agent preset
registered?

-> Continue to [Step 3: Session creation failures](#3-session-creation-failures)

### 3. Session creation failures (shared across agent types)

After passing agent-specific prechecks, all four agent types call into
the [session](../../packages/session.md) substrate to create a tmux
session. Common failures at this stage:

**a. Runtime settings creation failure:**
`runtime.EnsureSettingsForRole` writes `.claude/settings.json` in the
agent's working directory. If the directory is read-only or the disk
is full, this fails.

**Check:** `ls -la <agent-workdir>/.claude/` — do settings exist?
Are they writable?

**b. Tmux session creation failure:**
`session.NewSessionWithCommand` at `lifecycle.go` creates the tmux
session with the agent's startup command. Failures here mean tmux
itself can't create the session.

**Check:** Look at the error message. Common causes:
- tmux server not running (see Step 0)
- session name collision (another process created a session with the
  same name between the existence check and creation — TOCTOU race)
- tmux socket permissions (the per-town socket may have wrong ownership)

**c. Agent startup timeout:**
After creating the session, `session.WaitForRuntimeReady` polls for
the agent process to appear. If the agent fails to start within the
timeout, the session exists but the agent is dead inside it.

**Check:** `tmux capture-pane -t <session> -p` — what's in the pane?
Look for error messages, permission prompts, or trust dialogs.

### 4. Post-start failures (agent started then died)

If the agent started successfully but died shortly after:

**a. `gt prime` failure:** The agent's `SessionStart` hook runs
`gt prime --hook`, which sets up role identity and context. If prime
fails (e.g., workspace not found, Dolt unreachable for beads-dependent
roles), the agent may crash on its first action.

**Check:** `gt daemon logs | grep -i prime` or examine the agent's
tmux pane for prime error output.

**b. Startup dialog not accepted:** Claude Code shows trust/permission
dialogs on first run. `session.AcceptStartupDialogs` at `startup.go`
sends keystrokes to dismiss them. If the dialog changed or the timing
is off, the agent hangs at a prompt.

**Check:** Attach to the session: `gt <agent> attach <rig>` and look
for a dialog.

**c. Nudge poller failure:** For agents that need nudge delivery,
`nudge.StartPoller` spawns a background poller. If the poller fails
to start, the agent won't receive nudges (but will otherwise function).
See [message-delivery](message-delivery.md) Step 4.

### 5. Witness start failures

`Manager.Start(foreground, agentOverride, envOverrides)` at
`witness/manager.go:110-278`.

**a. Foreground mode error:**
`manager.go:114-117` — foreground mode is deprecated. Passing
`--foreground` returns an error immediately. See
[gt witness](../../commands/witness.md) ## Drift for the cobra drift
finding.

**b. Zombie session detection:**
`manager.go:119-148` — if a witness session exists but the agent is
dead inside it, the manager enters a TOCTOU-safe zombie kill sequence:
record creation time, wait `constants.ZombieKillGracePeriod`, re-verify.
If the zombie persists, kill it and proceed. If the session was replaced
between checks, return `ErrAlreadyRunning`.

**c. Rig parked or docked:**
The CLI's `runWitnessStart` at `witness.go:164` calls
`checkRigNotParkedOrDocked(rigName)` before getting the manager.
Parked/docked rigs refuse agent starts.

**Check:** `gt rig status <rig>` — is the rig parked or docked?
`gt rig unpark <rig>` or `gt rig undock <rig>` to restore.

**d. Agent bead creation asymmetry:**
Note that witness `Manager.Start` does **not** create an agent bead.
Only polecat `SessionManager.Start` calls `createAgentBeadWithRetry`
(`polecat/manager.go:308-326`) during startup. Witnesses, refineries,
and crew members do not have agent beads — they rely on tmux session
existence for liveness. See the
[polecat lifecycle comparison table](../../workflows/polecat-lifecycle.md)
(row: "Creates agent bead") for the full asymmetry.

-> Continue to [Step 3](#3-session-creation-failures) for session
creation failures.

### 6. Witness auto-restart by daemon

If the witness keeps dying and restarting, the daemon's heartbeat
loop detects dead witness sessions and restarts them. The
[daemon](../../packages/daemon.md) `RestartTracker` applies
exponential backoff (30s initial, 10m max, 5 crashes before
crash-loop state).

**Check:**

```bash
gt daemon logs | grep -i witness
cat ~/gt/daemon/restart_state.json  # Per-agent backoff counters
```

- **Crash loop detected:** `gt daemon clear-backoff <witness-session>`
  resets the counter. Then investigate why the witness is crashing
  (attach to it immediately after restart).

### 7. Refinery start failures

`Manager.Start(foreground, agentOverride)` at
`refinery/manager.go:113-200+`.

The refinery start path is structurally similar to the witness:

**a. Foreground mode:** Unlike the witness, the refinery's
`--foreground` is still live — it blocks inside `mgr.Start` running
the merge loop in-process.

**b. Already running:** `refinery.ErrAlreadyRunning` is converted to
a warning + nil at the CLI level (`refinery.go:313-317`).

**c. Rig parked or docked:** Same guard as witness. Check rig status.

**d. Agent bead creation asymmetry:**
Like the witness, refinery `Manager.Start` does **not** create an
agent bead during startup. See Step 5d above and the
[polecat lifecycle comparison table](../../workflows/polecat-lifecycle.md)
for the full contrast.

-> Continue to [Step 3](#3-session-creation-failures) for session
creation failures.

### 8. Refinery-specific operational failures

**a. Status shows stopped but Dolt is the real problem:**
`runRefineryStatus` at `refinery.go:374-378` silently discards errors
from `mgr.IsRunning()`, `mgr.Status()`, and `mgr.Queue()`. If Dolt is
down, status reports "stopped" with queue length 0 rather than
indicating data is unavailable. **This is an absent failure mode** —
the status display may be misleading. See
[refinery](../../commands/refinery.md) ## Failure modes -> Silent
suppression.

**Check:** `gt dolt status` to verify Dolt is reachable before
trusting refinery status output.

### 9. Crew start failures

`Manager.Start` in `crew/manager.go` handles both `gt crew at` (with
auto-start) and `gt crew start`.

**a. Crew workspace not found:**
`ErrCrewNotFound` — the crew name doesn't exist under `<rig>/crew/`.

**Fix:** `gt crew add <name>` to create the workspace first.

**b. Session already running:**
`ErrSessionRunning` — the crew session is already active.

**Fix:** `gt crew at <name>` to attach, or `gt crew restart <name>`.

**c. Invalid crew name:**
`validateCrewName` at `manager.go:106-126` rejects names with hyphens,
dots, spaces, path separators, or traversal sequences. These break
agent ID parsing.

**Check:** Does the crew name contain any of these characters?
Use underscores instead.

**d. Clone failed (during `gt crew add`):**
`crew add` performs a full git clone. If the remote is unreachable or
the clone is interrupted, the workspace is left in a partial state.

**Check:** `ls -la <rig>/crew/<name>/` — does a `.git` directory
exist? Is the clone complete?

-> Continue to [Step 3](#3-session-creation-failures) for session
creation failures.

### 10. Crew resume failures

**a. Resume flag unsupported:**
`buildResumeArgs` at `manager.go:86-102` checks the agent preset for
a `ResumeFlag`. Agents without one (subcommand-style agents) can't
use `--resume`.

**b. Invalid session ID:**
`validateSessionID` at `manager.go:70-80` restricts IDs to
alphanumeric + `-_.`. This prevents shell injection when the ID is
interpolated into the resume command string.

### 11. Zombie/hung detection

An agent appears in `tmux list-sessions` but isn't responsive.

**Check:** `CheckSessionHealth` at `tmux.go:2155` performs three
levels:

1. **Session existence:** `HasSession` — does tmux know about it?
2. **Agent process liveness:** `IsAgentAlive` at `tmux.go:2644` —
   checks process tree inside the pane. Uses `GT_PROCESS_NAMES` env
   var (set at startup) to know which processes to look for.
3. **Activity staleness:** `GetSessionActivity` — checks tmux output
   timestamp. If no activity for longer than `maxInactivity`, returns
   `AgentHung`.

Four possible states (`ZombieStatus` at `tmux.go:2108-2120`):
- `SessionHealthy` — session exists, agent alive, activity recent.
- `SessionDead` — tmux session gone entirely.
- `AgentDead` — tmux session exists but agent process died. The shell
  prompt is visible but no agent is running.
- `AgentHung` — agent process exists but hasn't produced output.
  May be in a long tool call, or genuinely stuck.

**Fix for AgentDead:** Kill the zombie session and restart:

```bash
gt <agent-type> restart <rig>
```

All restart commands (`witness`, `refinery`, `crew`) do a best-effort
stop (ignoring `ErrNotRunning`) then start. Note: restart always
starts in background mode, ignoring any previous foreground state.
See [witness](../../commands/witness.md) ## Failure modes ->
Partial completion for the error suppression behavior.

**Fix for AgentHung:** The agent may be in a legitimate long operation
(e.g., git clone, large tool call). Wait and re-check. If it persists,
use `gt deacon force-kill <agent-address>` to terminate with an audit
trail.

## Cycles and shared dependencies

- **Witness watches polecats:** If the witness is down, crashed
  polecats aren't detected or recovered. The daemon eventually notices
  the dead witness and restarts it (see Step 6), but there's a window
  where polecats can accumulate as zombies.
- **Deacon watches witnesses:** If the deacon is down, dead witnesses
  aren't detected. Boot triages the deacon on each daemon tick. See
  [daemon-infrastructure](daemon-infrastructure.md).
- **All agents depend on tmux:** If the per-town tmux server dies,
  all agents die simultaneously. `gt up` is the recovery path.
- **Restart suppresses stop errors:** Both
  `runWitnessRestart` (`witness.go:349`) and `runRefineryRestart`
  (`refinery.go:546`) discard `mgr.Stop()` errors with `_ = mgr.Stop()`.
  If stop partially completes (kills tmux but fails to update state),
  the fresh start may see stale state. See
  [witness](../../commands/witness.md) ## Failure modes -> Partial
  completion.

## Related investigation workflows

- [Investigating: daemon infrastructure](daemon-infrastructure.md) --
  daemon, boot, deacon startup and shutdown failures. Shares the
  "is tmux alive?" prefix with this workflow.
- [Investigating: message delivery](message-delivery.md) -- nudge/mail
  delivery failures that may cause agents to miss work assignments.
- [Investigating: data-plane failures](data-plane.md) -- Dolt
  connectivity issues that affect beads-dependent agent operations.
