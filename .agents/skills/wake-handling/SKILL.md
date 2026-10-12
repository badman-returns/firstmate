---
name: wake-handling
description: Load before acting on a signal, stale, check, or heartbeat wake, and when any wake reports a merged PR or a Relay-linked milestone.
user-invocable: false
metadata:
  internal: true
---

# Wake handling

`AGENTS.md` section 8 keeps the always-loaded supervision contract: one live cycle, drain before acting, acknowledge only after handling, and no turn that ends blind.
This skill owns what to do for each wake type after the drain.

## Wake types

1. For `signal:`, read the listed event lines first.
   Then reconcile current state only where the action depends on it.
2. For `stale:`, inspect the recorded endpoint.
   Load `stuck-crewmate-recovery` for a stopped, looping, confused, or unresponsive worker.
   A deep-inspection reason also requires current-state and validation-log inspection.
3. For `check:`, act on the named poll result.
   Poll results include merges, contribution signals, Relay events, process-to-event source results, and captain inbox notes.
   - Acknowledge a handled inbox note with `bin/fm-inbox.sh drain --ack <id>`, or it stays counted as still waiting for firstmate.
   - When the note needs a durable answer that the submitter can read, publish it with `bin/fm-inbox.sh reply <id>` (the script header owns the reply contract).
     Do not leave the answer only in the transcript.
   - A `check: secondmate <id> auto-relaunched` wake records a recovery that already completed.
     Reconcile the mate's current state and do not relaunch again.
     A repeat or a paused-bound wake means the mate keeps exiting, so investigate why.
4. For `heartbeat:`, review the whole fleet from the structured fleet view.
   Reconcile suspicious tasks and PR state, and update the backlog.
   Never report an unchanged fleet as progress.

Load `bearings` on a contributions check wake or when filing work linked to an upstream issue.
Its contribution-follow-up section owns triage and exact signal acknowledgement.

## Merged PRs and Relay

When any wake reports a merged PR for a project cloned in this home, refresh that clone through the guarded fleet-sync path.

When Relay-linked work reaches a milestone or terminal state, load `fmx-respond`.
Before terminal teardown, use its promised-final reconciliation when a typed public commitment exists.
Otherwise, post the final completion follow-up so the link clears even if earlier follow-ups were spent.
