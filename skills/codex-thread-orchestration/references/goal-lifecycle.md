# Goal Lifecycle

## API Limits

Goal tools operate only in the current thread.

Supported:

- create a goal in the current thread.
- read the current thread goal.
- end the current thread goal as `complete` or `blocked`.

Not supported:

- edit a goal objective.
- pause a goal.
- resume an active goal.
- unblock a blocked goal.
- create or read another thread's goal.
- convert `blocked` or `complete` back to active.

## Worker Rules

Workers must create their own goal after confirming the worksite. The objective must exactly match the delegated objective. Immediately run `get_goal` and report objective/status to the scheduler.

If `create_goal` fails because an old goal exists, report the old goal state. If it is `blocked` or `complete`, the scheduler must send a new exact objective for a new goal before work continues.

Do not mark a goal complete without a scheduler-readable final report. If `gate_owner=scheduler`, completion is usually not local merge; the worker should report `waiting-scheduler-gate`.

## Waiting Is Not Always Blocked

Do not block the goal for:

- hosted checks pending on the same run.
- bounded waiting for a hosted run already in progress.
- transient API/transport issue under bounded read-only retry.
- waiting for the scheduler to consume a `waiting-scheduler-gate` report when the worker can simply stop after reporting.

Use table/report states such as `waiting-hosted` and `waiting-scheduler-gate` for these cases.

## When To Block

Block the current goal after reporting when:

- worksite, branch, head, status, or metadata does not match the assignment.
- `create_goal` or `get_goal` is abnormal.
- progress requires another worker's scope, another unit, or the main/project worktree.
- the scheduler explicitly asks the worker to pause, wait for decision, or wait for another worker.
- a stable semantic failure requires changing policy, parser, review, head binding, approval, merge, release, or gate rules.
- bounded retries cannot prove host enforcement, branch protection, ruleset, or required checks.
- a controlled wrapper fails for a non-transient reason.
- the only remaining path would use a raw command to bypass wrapper policy.
- the objective is inconsistent or impossible.

After `blocked`, the scheduler must resume with a new exact objective and the worker must create a new goal.

## Recovery Prompt

```text
Your previous goal is blocked/complete. The API cannot resume or edit it.
Create a new goal with this exact objective:
"<new exact objective>"

After creation, run get_goal and report objective/status. Do not treat the old goal as active.
```

## Scheduler Implications

The scheduler audits worker self-reports; it cannot directly inspect worker goals through goal APIs. `read_thread` is useful for status readback, but it is not a goal API.

Do not ask a blocked or complete worker to "continue" without a new objective. Do not consider the global objective blocked just because one worker goal is blocked; classify the blocker and schedule the next actionable owner.
