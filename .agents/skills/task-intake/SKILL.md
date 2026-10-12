---
name: task-intake
description: Load before resolving a request's project, routing work to a secondmate, resolving a ship's delivery mode, yolo posture, branch prefix, or base branch, writing a brief, spawning, steering a worker, or relaying a mid-task ask.
user-invocable: false
metadata:
  internal: true
---

# Task intake and dispatch

`AGENTS.md` section 7 keeps the always-loaded contract: the scout-versus-ship classification, merge authority, and the isolated-worktree rule.
This skill owns intake, delivery-mode resolution, concurrency, spawn handoff, and steering.
Referenced scripts own exact commands, flags, and data mechanics.

## Intake

Resolve the project independently for every request.
An explicit project wins.
A clear follow-up inherits its referent.
Otherwise, match the request against the registry, work under way, and project code or README.
Proceed on one confident match and name the project in plain language.
Ask one concise question when multiple projects or no project plausibly match.

Route by the nature of the work against each registered secondmate scope, not by a non-exclusive clone list.
Keep `local-only` work in the main home.
Send in-scope work to the fitting secondmate unless it is blocked or the captain explicitly redirects it.
Do not read the secondmate's chat: marked routed replies return through its status or a referenced document.
If no secondmate scope fits, use the main home or discuss creating an appropriate persistent secondmate.

For one-off or infrequent operational work, start with the simplest direct end-to-end path.
Do not build wrappers, control planes, policy layers, custom verifiers, or automation unless the direct path exposes a concrete blocker or a repeated need that justifies them.

Before commissioning an investigation, consult existing reports and established evidence.

## Delivery mode, yolo, and branch

Resolve every ship task's concrete delivery mode and `yolo` merge posture at intake.
Pass the mode explicitly to the brief.
Pass both values explicitly to the spawn and to any scout promotion.
Each command refuses to guess the values it consumes.

A current explicit captain instruction wins.
Otherwise the project's registry entry is the captain's standing posture, and dropping below its rigor needs a reason that can be stated.
Resolve the project's registered ship-branch prefix the same way with `bin/fm-project-mode.sh --branch-prefix <project>`.
Pass it explicitly to the brief, the ship spawn, and any scout promotion as `--branch-prefix`.
The default `fm/` needs no flag.

When the work must start from and target a branch other than the project's default, such as a named feature or release branch, pass `--base-branch <branch>` to the ship or scout brief and spawn.
Any promotion reads it from task meta.

On a `no-mistakes-prod-only` project, classify the task's surface:

- Internal-only tooling, automation, contributor or operator process, and release or submission work ships `direct-PR`.
- Product-facing, mixed, and uncertain work ships `no-mistakes`.
- Never infer internal-only from file location or project name.

An unregistered project or an absent registry resolves to `no-mistakes` with yolo off.
Send the registration gap to the captain.
Record the resulting mode, the `yolo` merge posture, and the one-line reason for any deviation in the backlog item note.

## Concurrency

Treat file or subsystem overlap as a risk signal, not as an automatic reason to wait.
Dispatch isolated work immediately, with no concurrency cap, when each change can be independently implemented and validated and the selected delivery path can reconcile ordinary rebases or conflicts.
A project's declared machine capacity (`config/project-capacity`) still bounds that dispatch.
A spawn beyond it exits 75 without launching, and its item stays queued rather than blocked.
Serialize only for a true semantic dependency, shared mutable external state, an incompatible concurrent migration, or another concrete condition that makes independent progress or reconciliation unsafe.
Same-file editing alone is not enough, and genuine blockers remain durable.

## Brief and spawn

Write the task-specific brief under `AGENTS.md` section 11 before spawning.
Spawn only through `bin/fm-spawn.sh`, after the profile and backend checks in `AGENTS.md` section 4.
The spawn must resolve a genuine isolated task worktree distinct from the primary checkout.
A failed isolation assertion stops the task.

When the configured tasks-axi backlog gate applies, the spawn itself moves the work item to In flight and refuses to dispatch work for which this home has no item.
Recording the dispatch is never a separate step.
A manual-backend home keeps the hand-editing contract in `docs/configuration.md`.

After spawning, confirm that the worker is processing the brief.
Handle any trust dialog through `harness-adapters`.
A persistent secondmate is recorded in the secondmate registry and runtime state, never as a backlog work item.

## Steering

Steer a worker with ordinary text through `fm-send`, which stops safely when its target is not explicit.
The message becomes a durable record in the task's steering inbox, and multi-line text is legal for local and remote workers.
The worker's terminal receives only a constant doorbell line.
The watcher re-rings an unacknowledged local message and escalates a stuck one (`bin/fm-task-inbox-lib.sh`; `bin/fm-send.sh` owns the typed-plane carve-outs).

A remote secondmate steer uses the same durable-inbox model through the remote transport.
After an unconfirmed delivery, the only safe resend is the exact `FM_PENDING_REPLY_EXISTING_CORR=<id>` command that `fm-send` prints, because it preserves the request body for remote enqueue deduplication (`bin/fm-send.sh` header).
`bin/fm-pending-reply-lib.sh` owns the parent-side correlation, recovery, and escalation contract for marked secondmate requests.

When a steer answers an open keyed decision or blocker, pass `fm-send`'s `--resolve-key`.
The answer then closes that decision record at answer time, for local and remote workers alike (contract: `bin/fm-send.sh` header).

`fm-send` is the data plane for text that the worker should read.
Never use its key or text paths for interrupt, exit, or other lifecycle control, because routing-marked lifecycle text becomes chat that the worker reasons about instead of executing.
Drive a worker's lifecycle through `bin/fm-control.sh <task-id> interrupt|exit|relaunch`.
It owns the per-runtime mechanics, verifies each action, and never tears down or discards anything ([`docs/agent-control.md`](../../../docs/agent-control.md)).

A secondmate's routed reply returns through status or a document pointer, not through a look at its chat.

## Mid-task asks

When the captain adds or changes an ask mid-task, append the captain's words to that brief's `## Captain's intent`.
Add no speaker labels and no direct address.
Relay the same words to the worker.
Firstmate build constraints stay in `## Firstmate spec` or in the steer.
