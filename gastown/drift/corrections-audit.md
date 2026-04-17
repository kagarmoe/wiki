---
title: Phase 7 correction validation
type: drift
status: active
topic: gastown
created: 2026-04-17
updated: 2026-04-17
phase: 7
---

# Phase 7 Batch 1: Correction validation + ambiguity audit

Two read-only audits against gastown HEAD (`9f962c4a`).

---

## Section 1: Corrections spot-checked (18 of 61)

Sampled across all 4 groups, biased toward user-audience commands and
large source files where line numbers are most likely to have shifted.

### Group 1: Hand-maintained enumerations (5 sampled)

| Correction | Result | Notes |
|---|---|---|
| **account.md** — `switch` omitted | **Valid** | `accountSwitchCmd` registered at `account.go:518`. Long text at `:34-38` matches quoted text exactly. |
| **crew.md** — 3 subcommands omitted | **Valid** | `crew.go:412-429` registers 13 AddCommand calls; `status`, `rename`, `pristine` confirmed present. `next`/`prev` hidden. Long text at `:50-58` matches quoted text. |
| **hooks.md** — `init` omitted | **Valid** | `hooks_init.go:14` defines `hooksInitCmd`; registered at `:32`. Long text at `hooks.go:17-25` matches. |
| **mail.md** — 4 listed, 22 registered | **Valid with note** | `mail.go:524-541` registers 17 subcommands. Sibling files add 5 more: `mail_channel.go:149`, `mail_directory.go:42`, `mail_group.go:115`, `mail_hook.go:45`, `mail_queue.go:548`. Total = 22. However, `mail_identity.go` exists but does NOT register via `mailCmd.AddCommand`, so it's not a subcommand — correction count is accurate. |
| **quota.md** — `watch` omitted | **Valid** | `quotaWatchCmd` at `quota.go:812`; registered at `:978`. Long text at `:47-50` matches. |

### Group 2: Stale Long text claims (6 sampled)

| Correction | Result | Notes |
|---|---|---|
| **activity.md** — wrong event names | **Valid** | `events.go:67-70`: `TypeMerged = "merged"`, `TypeMergeSkipped = "merge_skipped"`. Long text at `activity.go:52-56` says `merge_complete` and `queue_processed` — both wrong. |
| **config.md** — `dolt.port` omitted | **Valid** | `config.go:896` has `case "dolt.port":`. The error message at `:911` lists `dolt.port` in Supported keys. But the Long text Supported keys block (`:695-718`) omits it. |
| **down.md** — says `gt start` | **Valid** | Long text at `down.go:67`: `"use 'gt start' to bring everything back up."` — should be `gt up`. `gt start` does not start the daemon. |
| **install.md (1)** — HQ contents list | **Valid** | `install.go:307-318` creates `deacon/`; `:327-334` creates `plugins/`. Long text at `:55-58` omits both. |
| **install.md (2)** — `docs/hq.md` ref | **Valid** | `install.go:61-62` references `docs/hq.md`. File does not exist in gastown repo at HEAD. |
| **status.md** — `--fast` understated | **Valid** | Flag description at `status.go:57`: "Skip mail lookups for faster execution". Code skips: overseer mail (`:776`), per-agent mail, hooks (`:891`), and MQ (`:907`). Description is incomplete. |

### Group 3: Docs file corrections (4 sampled)

| Correction | Result | Notes |
|---|---|---|
| **done.md / CLEANUP.md** — self-nuke claim | **Valid** | `CLEANUP.md:27` says "self-nukes worktree, kills own session". `done.go:1209-1263` transitions to IDLE. `selfNukePolecat` at `:1673-1680` has no call sites. |
| **install.md / INSTALLING.md** — `rigs/` dir | **Valid** | `INSTALLING.md:115` shows `├── rigs/`. `runInstall` does not create `rigs/`. Rigs are top-level dirs. |
| **identity.md** — GIT_AUTHOR_NAME path | **Valid** | `env.go:118` (polecat): `env["GIT_AUTHOR_NAME"] = cfg.AgentName`. `:130` (crew): same. Docs say full slash path — wrong. |
| **go-mod.md** — Go version mismatch | **Valid** | `go.mod:3` = `go 1.25.8`. `Dockerfile` ARG = `1.25.6`. `Dockerfile.e2e` = `golang:1.26-alpine`. Three different versions confirmed. |

### Group 4: Implementation-status (3 sampled)

| Correction | Result | Notes |
|---|---|---|
| **HOOKS.md** — Known Gap #2 | **Valid** | Gap claims `gt tap guard dangerous-command` doesn't exist. `tap_guard_dangerous.go` and `tap_guard_dangerous_test.go` both exist at HEAD. Gap should be removed. |
| **architecture.md (2)** — dead watchdog-chain link | **Valid** | `docs/design/watchdog-chain.md` does not exist at HEAD. Link is dead. `dog-infrastructure.md` exists and is the closest equivalent. Also found 3 more dead `watchdog-chain.md` references in `overview.md:31`, `convoy-lifecycle.md:381`, `property-layers.md:355`. |
| **polecat-self-managed-completion.md** — stale status | **Valid** | Correction says status should be "Implemented". `done.go:1209-1263` confirms persistent polecat model is shipped. |

### Summary

**18 of 61 corrections spot-checked. All 18 valid.** No issues found. The
corrections were drafted from Phase 3 wiki annotations which cited exact
file:line locations; the gastown source at HEAD still matches all cited
locations (no line-number drift from recent commits, which were all in
`internal/cmd/reaper.go` and `internal/daemon/`).

---

## Section 2: Ambiguity findings (12 findings)

Claims too vague to verify against code. Per Phase 3 Option 3 ruling,
these get `severity: ambiguous`.

### Docs file ambiguity

#### Finding 1: `docs/overview.md:109` — "Future dogs might handle" undefined

**Verbatim:** `Future dogs might handle: log rotation, health checks, etc.`

**Why ambiguous:** The "etc." makes this an open-ended claim. No code path
defines what future dogs will handle. "Health checks" is vague — `gt doctor`
already runs health checks; it's unclear what a dog-based health check would
do differently.

**Suggested clarification:** Remove the speculative line entirely, or replace
with: "Additional dog types can be added as Deacon task scripts."

#### Finding 2: `docs/overview.md:176` — "Gas Town only tracks state"

**Verbatim:** `Run individual runtime instances manually. Gas Town only tracks state.`

**Why ambiguous:** "Tracks state" is vague. What state? Beads issues? Session
status? Agent identities? All of these? The reader can't determine what
minimal-mode Gas Town actually does without the daemon.

**Suggested clarification:** "Gas Town tracks issue state (beads), agent identity
(BD_ACTOR), and work attribution (git commits) but does not manage agent
lifecycle."

#### Finding 3: `docs/CLEANUP.md:58` — shutdown "aggressiveness" flags unnamed

**Verbatim:** `Flags control aggressiveness (--graceful, --force, --nuclear, --polecats-only, etc.)`

**Why ambiguous:** The "etc." hides additional flags. What does each
aggressiveness level do? The table cell doesn't distinguish their behaviors.

**Suggested clarification:** List each flag with its behavior, or link to
`gt shutdown --help`.

#### Finding 4: `docs/reference.md:258-259` — "gt done nukes sandbox and exits"

**Verbatim:** `3. gt done nukes sandbox and exits` / `4. Witness removes worktree + branch`

**Why ambiguous:** This contradicts the persistent polecat model shipped in
v1.0.0. `done.go:1209-1263` transitions to IDLE, not nuke. This is not merely
ambiguous — it is stale (already covered by the done.md correction in
corrections.md). However, reference.md was not included in that correction's
scope. The reference.md lifecycle section needs updating too.

**Suggested clarification:** Replace with: "3. gt done syncs worktree to main,
transitions polecat to IDLE. 4. Polecat is ready for new work assignment."

#### Finding 5: `docs/reference.md:284` — GIT_AUTHOR_NAME "same as BD_ACTOR"

**Verbatim:** `GIT_AUTHOR_NAME | Commit attribution (same as BD_ACTOR) | gastown/polecats/toast`

**Why ambiguous:** This is factually wrong for polecats and crew.
`env.go:118,130` sets `GIT_AUTHOR_NAME = cfg.AgentName` (just "toast", not
"gastown/polecats/toast"). It IS the full path for refinery (`:111`) and
deacon dogs (`:139`). The claim is not merely ambiguous — it is incorrect
for the most common roles.

**Suggested clarification:** Replace with: `GIT_AUTHOR_NAME | Commit
attribution (agent name for polecat/crew; full path for refinery/dog) |
toast`

#### Finding 6: `docs/agent-provider-integration.md:36-37` — "15 minutes of work"

**Verbatim:** `Most agent teams should target Tier 1 first (15 minutes of work)`

**Why ambiguous:** The effort estimate is untestable. For a team unfamiliar
with Gas Town's JSON schema, creating a correct preset with all the fields
(30+ fields in the schema table) could take much longer. "15 minutes" is
marketing copy, not a verifiable claim.

**Suggested clarification:** Remove the time estimate, or qualify: "Tier 1
requires creating a single JSON config file with ~5 required fields."

### Cobra Long text ambiguity

#### Finding 7: `internal/cmd/rig.go:127` — "Reset various rig state"

**Verbatim:** `Reset various rig state.`

**Why ambiguous:** "Various" is undefined. The subsequent lines clarify
(handoff, mail, stale issues), but the opening line alone tells users nothing.
The `Short` field at `:126` is better: "Reset rig state (handoff content, mail,
stale issues)".

**Suggested clarification:** Replace opening line with: "Reset rig state:
handoff content, stale mail, and orphaned in_progress issues."

#### Finding 8: `internal/cmd/witness.go:35` — "Nukes sandboxes when polecats complete"

**Verbatim:** `Nukes sandboxes when polecats complete via 'gt done'`

**Why ambiguous:** Contradicts the persistent polecat model. `done.go:1209`
transitions to IDLE; sandboxes are preserved. This is stale, not merely
ambiguous. The witness.go correction in corrections.md addresses `--foreground`
but not this stale "nukes sandboxes" claim in the parent Long text.

**Suggested clarification:** Replace with: "Monitors polecats after completion
(IDLE state), handles crash recovery."

#### Finding 9: `internal/cmd/witness.go:55-57` — "Self-Cleaning Model" description

**Verbatim:** `Self-Cleaning Model: Polecats nuke themselves after work. The
Witness handles crash recovery (restart with hooked work) and orphan cleanup
(nuke abandoned sandboxes). There is no "idle" state - polecats either have
work or don't exist.`

**Why ambiguous:** Directly contradicts the persistent polecat model. There IS
an idle state (`done.go:1263`: "Polecat transitioned to IDLE"). Polecats do
NOT nuke themselves. This entire paragraph is stale.

**Suggested clarification:** Replace with: "Persistent Polecat Model: Polecats
transition to IDLE after completion (sandbox preserved). The Witness handles
crash recovery and reassignment of idle polecats."

#### Finding 10: `internal/cmd/config.go:28` — "agent aliases and defaults"

**Verbatim:** `This command allows you to view and modify configuration settings
for your Gas Town workspace, including agent aliases and defaults.`

**Why ambiguous:** "Configuration settings" is vague — what settings beyond
agents? The `config get` subcommand handles `convoy.*`, `cli_theme`,
`default_agent`, `scheduler.*`, `maintenance.*`, `lifecycle.*`, `dolt.port`
(18+ keys). The Long text suggests only agent config.

**Suggested clarification:** "Manage Gas Town configuration including agent
presets, scheduler settings, lifecycle policies, and town defaults. Use
`gt config get --help` for the full key list."

#### Finding 11: `internal/cmd/refinery.go:57-58` — "appropriate target branches"

**Verbatim:** `monitors for polecat work branches and merges them to the
appropriate target branches.`

**Why ambiguous:** "Appropriate target branches" is undefined. Is it always
`main`? Integration branches? The reader can't determine the merge target
without reading the merge queue config.

**Suggested clarification:** "monitors for polecat work branches and merges
them to the rig's default branch (or active integration branch if configured)."

#### Finding 12: `docs/overview.md:31` — dead link to watchdog-chain.md

**Verbatim:** `Background supervisor daemon ([watchdog chain](design/watchdog-chain.md))`

**Why ambiguous:** The link target does not exist. The reader clicks through
to a 404. `design/dog-infrastructure.md` is the closest equivalent. (Note:
this is also a dead-link finding, but the ambiguity is that "watchdog chain"
as a concept is not defined anywhere accessible.)

**Suggested clarification:** Link to `design/dog-infrastructure.md` and use
text "dog infrastructure" instead of "watchdog chain".

### Ambiguity summary

| Source type | Count | Notes |
|---|---|---|
| Docs files | 6 | reference.md (2), overview.md (2), CLEANUP.md (1), agent-provider-integration.md (1) |
| Cobra Long text | 6 | witness.go (2), config.go (1), refinery.go (1), rig.go (1), reference.md+witness.go overlap (1) |
| **Total** | **12** | 4 are also stale (not just ambiguous); 2 are dead links |

**Notable pattern:** 4 of 12 ambiguity findings stem from the same root cause:
the persistent polecat model (gt-hdf8) shipped in v1.0.0 but several docs and
Cobra Long text blocks still describe the old self-nuke model. The corrections
file addresses `done.md` and `witness.go --foreground` but misses reference.md
(lifecycle section) and the witness.go parent Long text.
