---
title: internal/events
type: package
status: verified
topic: gastown
created: 2026-04-11
updated: 2026-04-17
sources:
  - /home/kimberly/repos/gastown/internal/events/events.go
tags: [package, data-layer, events, activity-feed, jsonl, flock]
phase3_audited: 2026-04-15
phase3_findings: [wiki-stale]
phase3_severities: [incomplete]
phase3_findings_post_release: false
phase8_audited: 2026-04-17
phase8_findings: [silent-suppression]
---

# internal/events

Append-only JSONL activity-feed writer. Callers invoke `events.Log(...)` (or
`LogFeed` / `LogAudit` wrappers) when something interesting happens in gt —
sling, hook, handoff, done, mail, spawn, session-death, merge-queue
transitions, witness patrols, scheduler activity — and the package serializes
the event and appends one JSON line to `<townRoot>/.events.jsonl`. A
`flock`-based cross-process lock guards the file so multiple gt processes
never tear each other's writes. This is the **raw audit log**; the separate
feed daemon curates a subset into `~/.feed.jsonl` for user display.

> **Phase 3 correction (2026-04-15):** Phase 2 said "21 event types + 21
> payload constructors." There are actually **30 event-type constants** and
> 21 payload constructors. The per-category breakdown below correctly lists
> all 30; the opening count was wrong.

**Go package path:** `github.com/steveyegge/gastown/internal/events`
**File count:** 1 go file, 1 test file
**Imports (notable):**

- `github.com/gofrs/flock` — cross-process file locks.
- [`internal/workspace`](workspace.md) — `FindFromCwd` to locate the town root.
- stdlib: `encoding/json`, `os`, `path/filepath`, `time`.

**Imported by (notable):** gt command code that wants to surface activity in
the feed — sling, hook/unhook, handoff, done, mail, nudge, spawn/kill,
boot/halt, session lifecycle, the witness patrol loop, the refinery's merge
queue, and the scheduler's deferred-dispatch logic. Also consumed by the
feed daemon that tails `.events.jsonl` and produces the curated user feed.

## What it actually does

### Event shape

`Event` (`events.go:19-26`) has six fields:

- `Timestamp` — RFC3339 UTC, stamped at write time.
- `Source` — hard-coded `"gt"` (`events.go:88`).
- `Type` — one of the `Type*` constants (see below).
- `Actor` — caller-provided, typically a role or polecat identity.
- `Payload` — arbitrary `map[string]interface{}`; helper constructors
  (`*Payload`) define the canonical shapes.
- `Visibility` — `"audit"`, `"feed"`, or `"both"`.

### Event type constants (`events.go:36-77`)

All defined as string constants so callers never pass raw strings:

- **Command events:** `sling`, `hook`, `unhook`, `handoff`, `done`, `mail`,
  `spawn`, `kill`, `nudge`, `boot`, `halt`.
- **Session events:** `session_start`, `session_end` (for seance discovery),
  `session_death`, `mass_death` (for crash investigation).
- **Witness patrol events:** `patrol_started`, `polecat_checked`,
  `polecat_nudged`, `escalation_sent`, `escalation_acked`,
  `escalation_closed`, `patrol_complete`.
- **Merge queue events (refinery):** `merge_started`, `merged`,
  `merge_failed`, `merge_skipped`.
- **Scheduler events:** `scheduler_enqueue`, `scheduler_dispatch`,
  `scheduler_dispatch_failed`, `scheduler_close_retry`.

### Visibility levels (`events.go:30-33`)

- `VisibilityAudit` — only stored in the raw log; never shown to users.
- `VisibilityFeed` — intended for the curated feed.
- `VisibilityBoth` — feed daemon decides.

### Public API

- `Log(eventType, actor string, payload map[string]interface{}, visibility string) error`
  (`events.go:85-95`) — the core entry point.
- `LogFeed(eventType, actor string, payload map[string]interface{}) error`
  (`events.go:98-100`) — convenience for `VisibilityFeed`.
- `LogAudit(eventType, actor string, payload map[string]interface{}) error`
  (`events.go:103-105`) — convenience for `VisibilityAudit`.
- `EventsFile = ".events.jsonl"` (`events.go:80`) — the filename at the
  town root.
- A constructor per event type for payload shape consistency:
  `SlingPayload`, `HookPayload`, `HandoffPayload`, `DonePayload`,
  `MailPayload`, `SpawnPayload`, `BootPayload`, `MergePayload`,
  `PatrolPayload`, `PolecatCheckPayload`, `NudgePayload`,
  `EscalationPayload`, `UnhookPayload`, `KillPayload`, `HaltPayload`,
  `SessionDeathPayload`, `MassDeathPayload`, `SessionPayload`,
  `SchedulerEnqueuePayload`, `SchedulerDispatchPayload`,
  `SchedulerDispatchFailedPayload` (`events.go:154-371`).

### Internals / Notable implementation

- **Best-effort.** `write()` returns `nil` (not an error) when the caller
  isn't inside a Gas Town workspace (`events.go:112-115`): activity logging
  is never allowed to break command flow.
- **Cross-process lock.** Each append acquires an exclusive `flock` on
  `<file>.lock` before opening the file in append mode
  (`events.go:128-132`). The comment at `events.go:108-109` is explicit:
  `sync.Mutex` only protects one process, and multiple gt processes hit
  this file concurrently.
- **Town root resolution.** Uses `workspace.FindFromCwd()` to locate the
  town so the log is always written to the *current* town (not a hard-coded
  `~/gt`).
- **Line-based, not indented JSON.** One event = one line; `json.Marshal`
  (not `MarshalIndent`). This is append-log format for fast tail reads.
- **0644 file mode** — operational, non-sensitive.

### Usage pattern

```go
events.LogFeed(events.TypeSling, actor, events.SlingPayload(beadID, target))
```

The caller picks visibility by helper choice, picks the event type from a
string constant, and constructs the payload with the matching
`*Payload` helper. Errors are rare and usually ignored; when the caller
isn't in a town workspace, `Log` quietly succeeds with no work done.

## Relationship to internal/channelevents

`internal/events` (this package) and
[`internal/channelevents`](channelevents.md) are **two different event
systems**, not layers of the same one. They share no code.

- **`internal/events`** writes a single append-only JSONL file at
  `<townRoot>/.events.jsonl`, ordered globally, with flock-serialized
  writes. It's an *activity feed / audit log*: "this happened, and here's
  when." The feed daemon is the typical consumer.
- **`internal/channelevents`** writes one file per event under
  `<townRoot>/events/<channel>/`, consumed by `await-event` subscribers
  (e.g., the refinery watching a named channel for a `MERGE_READY` event).
  It's a *file-based pub/sub / rendezvous*: "wake me when this shows up."

Both packages derive `townRoot` from `workspace.FindFromCwd()` but they do
not share storage, schema, or delivery semantics. A fact that belongs in
the activity log goes through `events`; a notification that somebody
should wait for goes through `channelevents`.

## Related wiki pages

- [gt](../binaries/gt.md)
- [internal/channelevents](channelevents.md) — the per-channel file-based
  event system; distinct from this package.
- [internal/workspace](workspace.md) — town root resolution.
- [internal/telemetry](telemetry.md) — parallel OTel-based event system;
  `events` is the on-disk audit feed, `telemetry` is the push-based OTel
  waterfall. They cover overlapping events.
- [gt done](../commands/done.md) — emits `TypeDone` events.
- [gt nudge](../commands/nudge.md) — emits `TypeNudge` events.
- [go-packages inventory](../inventory/go-packages.md)
- [go.mod](../files/go-mod.md) — `github.com/gofrs/flock` is the file-lock
  library used here.

## Failure modes

### Silent suppression (what errors are swallowed?)
- **Write failure when not in a workspace:** `events.go:112-116` —
  `write()` returns `nil` when `FindFromCwd()` fails or returns
  empty. Events are silently dropped with no indication that the
  activity feed is not being populated. **Absent** — callers like
  `events.LogFeed` return nil, so commands think the event was logged
  when it was not.

## Notes / open questions

- The package has 30 event-type string constants and 21 payload helpers;
  they're not in a table structure, so adding a new event is a three-step
  chore (constant, helper, caller) but the uniformity is paid back in
  consistent payload shapes downstream. Nine event types lack dedicated
  payload constructors (callers build their own `map[string]interface{}`
  for those).
- Error handling is deliberately permissive: `write` returns `nil` on
  "not in a workspace", but returns an error on flock/write failures.
  Callers typically ignore both.
- There's no rotation or truncation logic in this package — the file grows
  indefinitely. The feed daemon is presumed to own that policy.
