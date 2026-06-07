# Scheduler Protocol

## Scheduler Self-Check

Before creating workers, state the scheduler facts:

```text
Role: scheduler.
Top Goal: <global user objective>
Completion: <merge/readback/closeout conditions, not only implementation done>
Constraints: <scope, forbidden paths/actions, gate owner, external-action limits>
Scheduler duties: split streams, create/resume workers, maintain dispatch table, send cross-thread instructions, classify blockers, run or authorize gates, consume closeout.
```

The scheduler normally should not create a long-lived active goal for itself. Use heartbeat plus the dispatch table for liveness unless the user explicitly asks for a scheduler goal or the scheduler action is short and self-contained.

## Dispatch Table

Maintain a scheduler-owned table:

```text
Top Goal:
- completion criteria:
- global constraints:

T1:
- title:
- thread_id:
- pending_worktree_id:
- objective:
- goal_status_reported_by_worker: unknown | active | blocked | complete
- actual_worksite:
- branch:
- units:
- issue:
- pr:
- head_sha:
- base_sha:
- gate_owner: scheduler | worker-authorized
- state: planned | confirming | active | waiting-hosted | waiting-scheduler-gate | waiting-scheduler | waiting-on-worker | blocked | complete
- blocker:
- next_worker_action:
- next_scheduler_action:
```

Update it after every worker report, hosted check readback, gate result, merge, blocker triage, and closeout event.

## Stream Planning

Before creating worker/worktree/branch resources, register:

```text
Streams:
- worker id:
- units:
- exact objective:
- scope:
- dependencies:
- branch name:
- branch type/prefix:
- allowed write paths:
- title:
- expected artifacts:
- expected conflict surface:
- completion condition:
- gate owner:
- initial state: planned | ready-to-start | dependency-blocked
```

Create the first batch only from dependency-ready streams. Keep dependency-blocked, closeout-only, repair-only, or high-conflict carrier/status/evidence streams as planned until their trigger is true.

Recommended phases:

```text
Phase 0: Planning
- Top Goal
- stream list
- dependency graph
- branch naming
- worker titles
- exact objectives
- first batch selection

Phase 1: First Batch
- create dependency-ready branches/worktrees/workers only
- inject scheduler thread id, worker id/thread id, title, exact objective
- require worksite confirmation plus create_goal/get_goal self-check

Phase 2: Active Scheduling
- update table through heartbeat and worker reports
- create/resume the next worker only when dependencies are satisfied
- create repair streams only after blocker classification

Phase 3: Merge And Closeout
- implementation merge/readback first
- create closeout-only branch after merge readback if versioned closeout is needed
- merge closeout-only through the controlled protocol
```

## Branch And Worktree Rules

Branch prefix guidance:

- Docs, specs, governance, or skill text only: prefer `docs/...`.
- Runtime code, generated artifacts, or shared implementation: use `feat/...` or `fix/...`.
- Test fixtures, regression cases, or baselines: use `test/...` unless the repo has a stronger convention.
- If scope may expand from docs to code, freeze the scope first. If you cannot freeze it, do not use `docs/...`.

Worker objectives must include branch prefix, allowed write paths, and forbidden paths. A worker that finds branch type and actual diff inconsistent must report and wait for correction.

If the thread creation tool uses an existing branch ref, create or verify the ref first:

```bash
git fetch origin --prune
git show-ref --verify --quiet "refs/heads/<branch-name>" || git branch "<branch-name>" <base-ref>
git rev-parse "<branch-name>"
```

Do not assume a thread creation tool both creates a branch and checks it out. After creation, read back the worker's real `cwd`; a Codex-managed worktree path is the worker worksite, while `project/root` is only the repository identity.

Detached HEAD at base/main can be a normal worktree initialization state. If the worker is in its own Codex-managed worktree, the scheduler may explicitly authorize `git switch <assigned-branch>` there before goal creation. Do not ask the worker to switch the project/root main worktree.

## Worker Creation

New workers should receive:

- worker id and concrete `worker_thread_id` if known after creation.
- concrete `scheduler_thread_id`; never "current scheduler thread".
- standardized title: `[<Project/Round>][T3][<unit-range>][PR#123] short task name`.
- exact objective.
- project/root and actual worker worksite.
- assigned branch, base, related units, allowed write paths, forbidden paths.
- gate owner and high-cost gate authorization status.
- first action: read-only worksite check, then `create_goal` and `get_goal` self-check.

Model/reasoning defaults:

- Complex scheduling, shared contracts, cross-module work, release/merge risk, high-cost gates, security/data/policy risk, or repeated blockers: use a high-capability model such as `gpt-5.5` with `high` reasoning.
- Standard single-module implementation, test analysis, or local review: use a capable standard model with `medium` reasoning; raise to `high` when uncertainty or blast radius grows.
- Mechanical lookup, formatting, inventory, or low-risk read-only checks: use a fast lower-cost model with `low` reasoning.
- If the environment cannot choose model or reasoning explicitly, record the limitation in the dispatch table and compensate with tighter review/validation.

Register the thread id immediately after creation. If title tools exist, set/rename the thread immediately; otherwise keep the standard title in the prompt and dispatch table.

## Tool Availability And Fallbacks

Prefer purpose-built thread and automation tools when available:

- `create_thread`: create dependency-ready workers and bind the intended branch/worktree when the tool supports it.
- `send_message_to_thread`: deliver any instruction that requires worker action.
- `read_thread`: read worker reports or status summaries; do not treat it as a goal API.
- `handoff_thread`: recover or migrate an existing worker when creation is not the right operation.
- heartbeat automation: keep scheduler liveness across hosted checks, gate waits, and closeout.

Fallback rules:

- If `create_thread` is unavailable, keep the stream `planned` and report the exact worker prompt, branch, worksite requirement, and blocker; do not pretend a worker exists.
- If branch/worktree binding is unavailable, create or verify the branch manually where allowed, then include the required worker-side worksite confirmation and branch switch authorization in the prompt.
- If `send_message_to_thread` is unavailable, use the reporting fallback in `reporting.md`: require the worker to emit a `<codex_delegation>` envelope or `Scheduler Report:` block that the scheduler can consume through `read_thread`.
- If `read_thread` is unavailable, require explicit worker reports in the current reachable channel before changing scheduler state.
- If heartbeat automation is unavailable, maintain the compact heartbeat prompt as a scheduler-local checklist and report the liveness limitation; do not create a long-lived scheduler goal just to poll.
- If a high-cost gate tool is unavailable, classify the failure as tool/permission/environment, preserve `waiting-scheduler-gate`, and either authorize an equivalent controlled path or report a scheduler blocker.

## Cross-Thread Commands

Use `send_message_to_thread` whenever a worker must act. A scheduler note in the scheduler thread is not a worker instruction.

Good scheduler-local note:

```text
Scheduler judgment:
T2 is waiting on the same hosted run. No worker message needed.
Next: wait for the hosted run; do not rerun.
```

Good worker message:

```text
T2, scheduler decision:
Continue triage with these boundaries:
- Preserve fail-closed semantics.
- Do not weaken stale review or head binding.
- Confirm payload/body/head readback before metadata fixes.
- If it is a transient API race, rerun only the failed lightweight gate.
```

Use the user's current language for scheduler-worker messages unless the scheduler explicitly changes language. Field names, commands, logs, and error text may stay in their source language.

## Blocker And Dependency Triage

Worker block does not equal global block. Classify each blocker:

- Worker scope blocker: send a precise correction or recovery objective to the same worker.
- Another worker owns the blocker: mark the blocked worker `waiting-on-worker`, activate or create the owner worker, and resume the blocked worker only after readback.
- Shared contract/schema/policy blocker: appoint one owner, prohibit duplicate definitions, and gate downstream workers on the owner artifact.
- Environment/tool/host transient: bound retry/readback; do not mark global failure until classified.
- Gate/root-cause failure: stop high-cost retries and issue a narrow root-cause correction objective.

If every worker is idle, blocked, or waiting while the Top Goal is incomplete, the scheduler must intervene: unblock, create the next dependency-ready worker, authorize a gate, or report a real global blocker.
