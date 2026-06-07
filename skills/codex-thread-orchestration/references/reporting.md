# Reporting Protocol

Scheduler-readable reporting is mandatory. A worker message visible only inside the worker thread is not enough unless the scheduler can consume it through thread readback.

## Delivery Priority

Use the strongest available delivery path:

1. If `send_message_to_thread` is available in the worker thread, send the report to the concrete `scheduler_thread_id`.
2. If cross-thread sending is not available, output a `<codex_delegation>` envelope in a key milestone or final response.
3. If neither is available, output a machine-readable block beginning with `Scheduler Report:`.

The scheduler must provide a concrete `scheduler_thread_id`. If missing, the worker reports the gap after worksite/goal self-check and waits for correction.

## Required Report Nodes

Workers must report at these points:

- worksite and goal self-check complete.
- PR/task created or updated, with head/body/payload metadata readback.
- hosted checks pending or in progress.
- hosted checks pass.
- hosted checks fail, after root-cause classification.
- entering `waiting-scheduler-gate`.
- blocker needs scheduler decision.
- before marking goal `blocked`.
- before marking goal `complete`.
- final scope completion.

Schedulers must read and consume reports before changing table state, running high-cost gates, resuming blocked/complete workers, closing out a batch, or declaring the Top Goal complete.

## Minimal Report Schema

Use this shape in `send_message_to_thread`, delegation envelopes, or fallback reports:

```text
Worker: <worker_id>
Thread: <worker_thread_id>
Title: <standard title>
Unit: <issue / PR / task / N/A>
State: <active | waiting-hosted | waiting-scheduler-gate | waiting-scheduler | waiting-on-worker | blocked | complete>
Objective: <exact objective or short id>
Worksite: <path>
Branch: <branch>
Head: <head_sha>
Base: <base_sha>
PR/Task: <url or N/A>
Validation: <commands and pass/fail summary>
Hosted checks: <pending/pass/fail with run ids if available>
Gate owner: <scheduler | worker-authorized>
Gate status: <not-ready | ready-for-scheduler | authorized | passed | failed | N/A>
Blocker: <none or classified root cause>
Next scheduler action: <exact action needed>
Next worker action: <exact action or waiting>
Risks: <remaining risk or none>
```

## Delegation Fallback

```xml
<codex_delegation>
  <source_thread_id><worker_thread_id or worker_id></source_thread_id>
  <input>
  Worker: <worker_id>
  Unit: <issue / PR / task>
  State: <active | waiting-hosted | waiting-scheduler-gate | blocked | complete>
  PR: <url or N/A>
  Head: <head_sha>
  Base: <base_sha>
  Validation: <commands and pass/fail summary>
  Hosted checks: <pending/pass/fail with run ids if available>
  Gate owner: <scheduler | worker-authorized>
  Blocker: <none or root cause>
  Next scheduler action: <exact action needed>
  Next worker action: <exact action or waiting>
  </input>
</codex_delegation>
```

## Scheduler Decision Request

When a worker needs scheduler judgment, report:

```text
Scheduler decision needed:
- blocker:
- evidence:
- why outside my scope:
- proposed next action:
- current goal status:
```

If the worker cannot continue meaningfully without the decision, block the current goal after sending the report.

## Complete Before Report Is Invalid

A worker must not mark a goal complete before producing a scheduler-readable report that includes:

- PR/task URL or equivalent carrier.
- head/base.
- validation commands and result.
- hosted checks and run ids if available.
- gate owner and gate status.
- issue/task state.
- final worktree status.
- remaining risks or explicit none.

The scheduler must not treat a worker as complete until this report is consumed.
