---
name: closing-sessions
description: Use at the end of any work session in the gt-wiki, when about to say "done" / "complete" / "ready to push" / "all set" / "wrapping up" / "that's it", after finishing a batch, after the last commit of a change set, or when handing off to Kimberly. Enforces the mandatory 7-step close protocol — work is NOT done until `git push` succeeds and `git status` says "up to date with origin." Do not skip, do not short-circuit, do not say "ready for you to push."
---

# Closing Sessions

## Overview

A session is not closed when the code is good. It is not closed when the commit lands. It is closed when `git push` succeeds and `git status` reports "up to date with origin/main." Anything short of that leaves work stranded on a local disk that a compact, crash, or forgotten session will silently lose.

**Core principle: work is not done until it is pushed.**

Corollary: **never ask Kimberly to push.** The LLM pushes. "Ready to push when you are" is a protocol violation — it delegates the last-mile discipline to the human and leaves a trap for the next session.

## When to use

Triggers — any one is sufficient:

- You are about to output the word "done," "complete," "finished," "ready," "all set," "wrapped up," or "that's it."
- You just finished the last commit of a batch / sub-batch / task / change set.
- Kimberly said "close it out," "finish up," "wrap this up," or "we're done for the day."
- You are about to hand off to Kimberly with "let me know what's next."
- You just got a compact warning and want to leave a clean resume state.
- You are about to run `bd close` on an issue (closing the bead is not the same as closing the session).
- You closed all open todos / tasks for the current change set.

When NOT to use:

- Mid-batch with more commits coming — keep working.
- Review gate pauses where Kimberly is reviewing drafts — not a close, just a hand-off.
- Non-wiki work (e.g. a question-answering session with no file changes).

## The checklist (mandatory, in order)

The 7 steps from [CLAUDE.md](../../../CLAUDE.md)'s Session Completion section, with wiki-specific annotations. **Execute all seven. Do not skip any. Do not reorder.**

### 1. File issues for remaining work

Anything discovered during the session that needs follow-up gets a bead BEFORE close:

```bash
BEADS_ACTOR=wiki-curator bd create \
  --title="<short title>" \
  --description="<why this exists and what needs to be done>" \
  --type=task --priority=2
```

Don't let findings die in the conversation tail. If it's worth mentioning, it's worth filing.

### 2. Run quality gates (if code changed)

For the wiki, "quality gates" means:

- **Lint pass** over any touched entity pages if this session authored or edited them — broken links, missing frontmatter, orphaned pages. See [maintaining-wiki-schema](../maintaining-wiki-schema/SKILL.md) for the full lint workflow.
- **Backlink check** on any page whose title, filename, or category changed.
- **Schema check** — if frontmatter was added or restructured, grep to confirm YAML parses.

Wiki sessions often skip this because "it's just markdown." It's not. A broken link is a bug.

### 3. Update issue status

Close finished beads. Update in-progress items with the final state. Do not leave `in_progress` beads dangling across a close — either close them or move them to `open` with a note.

```bash
BEADS_ACTOR=wiki-curator bd close <id1> <id2> ...
```

Close multiple at once — it's more efficient.

### 4. Push to remote (MANDATORY)

```bash
cd /home/kimberly/repos/gt-wiki
git pull --rebase
BEADS_ACTOR=wiki-curator bd dolt push    # see "bd dolt push in gt-wiki" below
git push
git status    # MUST show "up to date with origin/main"
```

**Wiki-specific: `bd dolt push` will fail in gt-wiki with "no remote configured."** This is expected — the wiki's embedded bd is operationally independent from the gastown dolt server by design. The failure is informational, not a blocker. Proceed to `git push` regardless.

**`git pull --rebase` first** — if somebody else (or a parallel session) landed commits on origin since your local HEAD diverged, rebasing now prevents the push from being rejected. Don't skip this.

**`git push` MUST succeed.** If it fails:
- Merge conflict from the rebase → resolve, don't `git stash`, don't `git reset --hard`, don't `--force`. Resolve the conflict and continue.
- Auth failure → fix credentials and retry.
- Hook failure → the hook is telling you something is wrong. Fix the underlying issue, make a NEW commit if needed, push again. Never `--no-verify`.

**`git status` verification is mandatory.** Read the output. If it does not say "Your branch is up to date with 'origin/main'," you are not closed. Investigate and retry.

### 5. Clean up

- Clear any `git stash` entries you created during the session.
- Prune stale remote branches: `git fetch --prune` if you created any local branches.
- Don't leave debug files, scratch notes, or experimental checkouts lying around.

### 6. Verify

Final sanity check. All of these must be true:

- [ ] `git status` reports working tree clean AND "up to date with origin/main"
- [ ] `git log --oneline origin/main..HEAD | wc -l` returns `0`
- [ ] No in-progress beads you didn't intend to leave in-progress
- [ ] No uncommitted changes in tracked files
- [ ] No untracked files that should have been committed or filed as beads

If any check fails, you are not closed. Return to the relevant step.

### 7. Hand off

Leave a resume note for the next session. One short message to Kimberly that covers:

- **Where we are** (phase, batch, task) — the `compact-recovery` skill's synthesis format is the template
- **Last commit SHA + one-line summary**
- **In-flight artifacts** (populated plan drafts, in-progress beads, pending decisions)
- **Next action** (specific task + step number, or "awaiting Kimberly's verdict on X")
- **Blockers** (any / none)

The next session (or Kimberly herself) will read this cold. Write so a cold reader can resume without asking questions.

## Red flags — STOP and run the checklist

Any of these phrases in your draft response mean you are about to violate the close protocol. Stop, run the checklist, then respond.

| Phrase | What you must do before saying it |
|---|---|
| "Done." / "Complete." / "Finished." | Verify `git status` shows "up to date with origin." |
| "Ready to push when you are." | NO. You push. Run step 4. |
| "I've committed the changes." | Committed ≠ pushed. Run step 4. |
| "Let me know what's next." | First run the full checklist. Then hand off with a resume note. |
| "All set." / "That's it." | Run the checklist first. |
| "You can push now." | You push. Run step 4. |
| "Work's complete, just needs a push." | No such state. Either pushed = done, or not-pushed = not-done. Run step 4. |
| "Wrapping up." | Wrap up means run the checklist to completion, not announce intent. |

## Common rationalizations (and why they're wrong)

| Rationalization | Reality |
|---|---|
| "Kimberly is right here, she can push" | The discipline is for YOU, not her. "Ready to push" delegates the last-mile check. Next session inherits an unpushed commit and may not notice. |
| "It's just a markdown change, no push needed" | Every commit belongs on origin. There is no "skip push for small changes" exception. |
| "I'll push on the next session" | Next session will have a different context window and may not remember to push. Push now. |
| "git status showed clean so I'm done" | Clean working tree + unpushed commits = NOT done. Check `git log origin/main..HEAD` too. |
| "bd dolt push failed, can't continue" | bd dolt push failing in gt-wiki is expected. It's not a blocker for `git push`. |
| "The commit message already explains everything" | Commit messages live in git. The resume note lives in the conversation so the next session sees it. Both are required. |
| "Compact is about to fire, no time" | Compact-safe state = pushed. Not pushing before compact is the worst possible moment to skip. |
| "I'll just leave it uncommitted for Kimberly to look at" | Uncommitted edits to tracked files get lost on `git checkout`, `git reset`, or a bad merge. If it's worth keeping, commit it. If it's not, discard it. |

**All of these mean: run the checklist in full. No exceptions.**

## Wiki-specific gotchas

- **Plan files under [.claude/plans/](../../plans/) are gitignored.** They never appear in `git status` and never need committing. Do not try to stage them. Populated draft subsections inside them are legitimate in-flight state that lives across sessions without commits.
- **`.claude/settings.local.json` is gitignored.** If you see it in `git status`, something is wrong — investigate, don't commit.
- **[log.md](../../../log.md) is append-only.** Never in-place edit historical entries at close. If correction is needed, append a new entry.
- **`BEADS_ACTOR=wiki-curator` is required for every `bd` command** in this repo — see [tracking-wiki-work](../tracking-wiki-work/SKILL.md). A bd command run without the actor prefix will land under the wrong identity and pollute the audit trail.
- **`bd dolt push` always fails here.** Design decision — the wiki's embedded bd is independent. Do not try to "fix" it by configuring a remote.
- **Batch-entry log discipline is part of closing.** If the session landed a batch, the `log.md` batch entry is mandatory before push. See the phase's active plan or [syncing-gastown-updates/workflow.md](../syncing-gastown-updates/workflow.md) for the batch-entry templates.

## Self-check — did you actually close?

After you think you're done, answer these three questions out loud (in your response, briefly) before handing off:

1. **"What is the current HEAD SHA, and is it on origin/main?"** — Answer from `git log --oneline -1` AND `git log origin/main..HEAD`. If the second command returns anything, you are not closed.
2. **"What does `git status` say in full?"** — Not "clean" — quote the actual line. "On branch main / Your branch is up to date with 'origin/main' / nothing to commit, working tree clean" is the only acceptable state.
3. **"If Kimberly pulled this repo on another machine right now, would she see everything?"** — If no, you are not closed.

If you cannot answer all three affirmatively, return to step 4.

## Reference

- [CLAUDE.md](../../../CLAUDE.md) — Session Completion section (abbreviated version of this checklist)
- `bd prime` — full bd command reference; auto-loaded at SessionStart via hook
- [tracking-wiki-work](../tracking-wiki-work/SKILL.md) — bd actor conventions + cross-tool handoff
- [maintaining-wiki-schema](../maintaining-wiki-schema/SKILL.md) — lint workflow for step 2
- [compact-recovery](../compact-recovery/SKILL.md) — the inverse skill; use at the START of a session to recover from what the previous session's close left behind
