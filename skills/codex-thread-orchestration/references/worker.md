# Worker Protocol

## Worksite Confirmation

Start read-only. Confirm and report:

- worker id, title, objective, concrete `scheduler_thread_id`.
- `pwd`, repo root, and whether this is the assigned worker worksite.
- branch, HEAD, base branch or equivalent target.
- `git status` and dirty diff.
- assigned branch and allowed write paths.
- PR/task head/body metadata and issue/task state when applicable.

If the current `cwd` is a Codex-managed worktree and is detached at base/main, that can be normal initialization. Only switch to the assigned branch inside the worker worksite when the scheduler authorized it. Never switch or write the project/root main worktree unless explicitly authorized.

If any locator is missing or inconsistent, produce a scheduler-readable report and wait for correction. If the inconsistency prevents meaningful progress, end the current goal as `blocked` if one exists.

## Goal Start

After worksite confirmation is clean, create the worker goal with the exact delegated objective. Do not paraphrase or expand it.

Immediately run `get_goal` and report:

- confirmed worksite and head.
- `get_goal.objective`.
- `get_goal.status`.
- whether `create_goal` failed or an old goal remained.
- whether work can continue.

Load `goal-lifecycle.md` before blocking, completing, or recovering a goal.

## Scope Boundaries

Do:

- execute only the assigned units and allowed write paths.
- update PR/task metadata for the assigned scope.
- run local and targeted validation appropriate to the diff.
- classify hosted checks, tool failures, and findings before retrying.
- report every key milestone through the reporting protocol.

Do not:

- change another worker's unit, branch, PR/task, carrier/state, or blocker.
- expand scope to fix unrelated failures.
- weaken policy, parser, review, head-binding, approval, merge, release, or gate semantics.
- run high-cost guardian/formal review/semantic review/controlled merge/release unless the scheduler explicitly authorizes the current head.
- use raw host commands to bypass controlled wrappers.
- mark complete without a scheduler-readable final report.

## Worker States

Use these table/report states:

- `confirming`: checking worksite and goal.
- `active`: doing scoped work.
- `waiting-hosted`: waiting for the same hosted run or bounded transient retry; usually not goal-blocked.
- `waiting-scheduler-gate`: local validation, metadata readback, hosted checks, and finding disposition are clean; scheduler must run or authorize the next high-cost gate. This is not a worker blocker.
- `waiting-scheduler`: scheduler decision is needed; if meaningful progress must pause, block the goal.
- `waiting-on-worker`: another worker owns the needed unblock; block the goal and wait for scheduler resume.
- `blocked`: goal is formally blocked or no meaningful progress is possible without scheduler/external change.
- `complete`: scoped objective is complete and final evidence has been reported.

These states are scheduler table states, not always goal API states. Use the goal API only according to `goal-lifecycle.md`.

## Required Reports

Report to the scheduler when:

- worksite plus goal self-check completes.
- local validation passes.
- PR/task is created or updated and head/body/payload metadata readback aligns.
- hosted checks are pending, in progress, passing, or failing after classification.
- transient API/transport issues are classified and bounded.
- guardian/review finding is fixed, including root cause, fix scope, same-class search, targeted validation, new head, and PR body status.
- entering `waiting-scheduler-gate`.
- needing a scheduler decision.
- about to mark the goal `blocked` or `complete`.

If `gate_owner=scheduler`, the normal local completion state is `waiting-scheduler-gate`, not merge or final complete.

## Read-Only Explorers

A worker may use read-only explorer subagents when the environment supports them, for code entry discovery, implementation survey, tests/logs review, fixture/schema lookup, or local risk review.

Read-only explorer boundaries:

- no file changes, commits, pushes, PR/issue edits, merge, release, or closeout.
- no scope/objective/gate change.
- no independent delivery responsibility.
- scheduler tracks only the worker; the worker summarizes explorer findings.

Report explorer use at the next milestone:

```text
Used read-only explorer:
- purpose:
- conclusion:
- effect on implementation/validation:
- scope changed: no
```

Ask the scheduler before using a nested agent that writes files, changes host state, owns an implementation slice, changes scope/gate strategy, or may run long enough to obscure worker status.
