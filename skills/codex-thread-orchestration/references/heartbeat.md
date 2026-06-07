# Scheduler Heartbeat

Use heartbeat automation to keep the scheduler alive without a long-lived scheduler goal.

## When To Use

Create or update a heartbeat when the scheduler is coordinating workers across hosted checks, gate waits, merges, dependencies, or closeout. Delete it when the Top Goal is complete or no scheduled wakeup is useful.

Do not store the full dispatch table in the heartbeat. Store only the facts needed for the next wakeup, and keep the full table in the scheduler thread, issue/PR comment, project carrier, or other authoritative record.

## Compact Prompt Skeleton

```text
You are the scheduler thread. Do not create a scheduler active goal.

Top Goal:
<completion criteria, including merge/readback/closeout; not only implementation done>

Global Gate:
- gate_owner:
- high-cost gate owner:
- live / integration / release gate default:
- forbidden scope expansion:

Batch Plan:
- current batch:
- later batches:
- closeout conditions:

Dependency / Ownership:
- shared shape owner:
- blocker dependencies:
- downstream consumers:
- schema / contract / metadata that must not be duplicated:

Current Workers:
<only active / waiting-hosted / waiting-scheduler-gate / waiting-scheduler / waiting-on-worker / blocked>
- worker_id:
- thread_id:
- issue / PR / task:
- branch / worksite:
- head / base:
- state:
- blocker:
- next_scheduler_action:
- next_worker_action:

Planned But Not Started:
<only unstarted items and start conditions>

Completed Readback:
<merged / closed / readback facts, one compact line each>

Heartbeat Action:
1. Read worker reports / PR / issue / task / base state.
2. If a worker is waiting-scheduler-gate, scheduler runs or authorizes the exact next gate.
3. If the current batch is complete and readback is clean, create the next dependency-ready worker.
4. If a worker is blocked, classify root cause and send a precise correction or new objective.
5. If this prompt is stale, update the automation before further scheduling.
6. Final readback issue / PR / main or equivalent target state.
```

## Update Rules

Rewrite the heartbeat prompt after every batch merge, closeout, worker retirement, major blocker classification, or dependency unlock.

Keep:

- active/waiting/blocked workers, usually 1-4.
- next scheduler action for each current worker.
- next worker action only when the worker needs an actual message.
- shared contract/schema/metadata ownership.
- gate owner and forbidden scope expansion.

Move completed workers to `Completed Readback`; do not leave them as current scheduling subjects.

If the heartbeat prompt is stale, update or delete the automation before continuing. Do not let an old prompt keep waking obsolete batches or worker ids.

## Heartbeat Action Rules

On each wakeup:

1. Decide whether the Top Goal is complete. If complete, produce final summary and delete unnecessary heartbeat.
2. If incomplete, decide whether at least one worker is effectively progressing.
3. If a worker is waiting on the same hosted run, record that scheduler judgment and avoid fake messages or unnecessary reruns.
4. If a worker is `waiting-scheduler-gate`, the scheduler must run or authorize the next gate.
5. If all workers are idle/blocked/waiting and the Top Goal is incomplete, pick the highest-priority blocker and act.
6. If a worker must act, use cross-thread messaging.
7. If resuming a blocked/complete worker, send a new exact objective and require `create_goal` plus `get_goal` self-check.
8. Do not write a scheduler-thread reply that sounds like the worker has received it.
