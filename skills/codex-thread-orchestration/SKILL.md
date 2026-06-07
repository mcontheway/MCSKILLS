---
name: codex-thread-orchestration
description: Scheduler and worker collaboration protocol for Codex thread orchestration. Use when Codex acts as a scheduler to split a global objective, create or resume worker threads, manage branches/worktrees, dispatch messages, run gates, and close out work; or when Codex acts as a worker to confirm its assigned worksite, create and verify a scoped goal, execute only its assigned scope, report scheduler-readable milestones, and wait for scheduler-owned gates.
---

# Codex Thread Orchestration

## Overview

Use this skill as a shared protocol between a scheduler thread and worker threads.

The scheduler owns the global objective: decomposition, worker creation, dependency order, gate ownership, merge/readback, and closeout. A worker owns scoped execution: confirm the assigned worksite, create its own goal, do only its assigned work, validate, and report milestones in a scheduler-readable form.

This is project-agnostic. PR, issue, Work Item, release, review, and closeout mean the equivalent carrier in the active repository or host system.

Language policy: keep protocol fields, states, and templates in concise English for machine readability; write scheduler-worker decisions, status summaries, and user-facing reports in the user's current language unless the scheduler explicitly switches language. Commands, logs, identifiers, and host field names may stay in their source language.

## Role Detection

First decide your role in the current thread:

- Scheduler: you were asked to coordinate multiple workers, branches, PRs, issues, release gates, merge readiness, or closeout.
- Worker: you received a delegated objective, thread/worksite/branch identifiers, or instructions to report back to a scheduler.
- Mixed/unclear: default to worker only if the prompt contains a concrete scoped objective and a scheduler identity; otherwise act as scheduler and build the dispatch plan before creating workers.

Do not let a worker infer the global plan from scratch. Do not let the scheduler pretend that a message written only in the scheduler thread has reached a worker.

## Worker Quick Start

1. Read `references/worker.md` and `references/reporting.md`.
2. Confirm the assigned `scheduler_thread_id`, worker id/title, objective, branch, and worksite.
3. Read-only check `pwd`, repo root, branch, HEAD, base, status, PR/task metadata, and issue/task state when applicable.
4. If the worksite is consistent, create a goal in the worker thread using the exact delegated objective, then immediately run `get_goal` and report objective/status.
5. Execute only the assigned scope. Report key milestones to the scheduler.
6. If `gate_owner=scheduler`, stop at `waiting-scheduler-gate` after local validation, metadata readback, hosted checks, and finding disposition are clean. Do not run guardian, formal review, controlled merge, or closeout unless the scheduler explicitly authorizes the exact head.

## Scheduler Quick Start

1. Read `references/scheduler.md`, `references/reporting.md`, and `references/heartbeat.md`.
2. Define the Top Goal, completion criteria, constraints, streams, dependencies, branch names, allowed write paths, expected artifacts, gate owner, and first batch.
3. Create only dependency-ready workers. Create or verify branch refs before using thread creation tools that require an existing branch.
4. Register each worker's thread id, title, actual worksite, branch, head/base, state, blocker, next worker action, and next scheduler action.
5. Use cross-thread messaging for worker actions. Keep a compact heartbeat for scheduler liveness.
6. Own high-cost gates by default: guardian, formal review, semantic review, controlled merge, post-merge readback, and closeout consumption.

## Core Invariants

- Scheduler owns global goal, dependency graph, gate policy, merge/readback, and closeout.
- Worker owns scoped execution, local validation, PR/task metadata, and scheduler-readable reporting.
- No scheduler-readable report, no complete.
- The scheduler cannot create, read, edit, pause, resume, or unblock a worker goal. The worker must create and inspect its own goal.
- Once a goal is `blocked` or `complete`, continuing requires a new goal with a new exact objective.
- `waiting-hosted` is not blocked when the worker is waiting on the same hosted run or a bounded transient retry.
- `waiting-scheduler-gate` is a scheduler action queue, not a worker blocker.
- When `gate_owner=scheduler`, the worker stops at `waiting-scheduler-gate` and waits for scheduler gate execution or explicit authorization.
- Do not write or checkout the main/project worktree from a worker unless the scheduler explicitly authorizes it.
- Do not use raw host commands to bypass controlled merge, release, review, or approval wrappers.

## Reference Routing

- `references/scheduler.md`: load when acting as scheduler, creating/resuming workers, choosing model/reasoning level, handling unavailable orchestration tools, maintaining the dispatch table, naming branches, triaging blockers, or planning next batches.
- `references/worker.md`: load when acting as worker, confirming a delegated worksite, creating a scoped goal, handling state transitions, or checking scope boundaries.
- `references/reporting.md`: load for both roles before any milestone, blocker, gate wait, final completion, or cross-thread message.
- `references/goal-lifecycle.md`: load when creating, completing, blocking, recovering, or explaining goals.
- `references/heartbeat.md`: load when creating or updating scheduler heartbeat automations or compact scheduler prompts.
- `references/gates-and-closeout.md`: load before guardian/review/merge/release gates, post-merge readback, closeout-only work, or repeated gate failures.
- `references/templates.md`: load when drafting worker initial prompts, scheduler corrections, recovery prompts, fallback delegation envelopes, `waiting-scheduler-gate` reports, or heartbeat prompts.
