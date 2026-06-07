# Gates And Closeout

## Scheduler-Owned Gate Protocol

High-cost gates are scheduler-owned by default:

- guardian.
- formal review.
- semantic review.
- controlled merge.
- release/deployment approval.
- post-merge readback.
- closeout consumption.

A worker may run one only when the scheduler explicitly authorizes the exact PR/task and head.

Workers enter `waiting-scheduler-gate` when all are true:

- scope diff matches the objective and allowed write paths.
- local validation passes.
- PR/task body metadata is updated and read back.
- hosted checks are green for the current head.
- head/base are explicit.
- branch type matches actual diff.
- findings are resolved or dispositioned.
- same-class drift search is complete when a guardian/review finding was fixed.

`waiting-scheduler-gate` is a scheduler action queue. The scheduler must run or authorize the next gate; do not ask the worker to repeat the same missing-review report.

## Review Entry Checklist

Before high-cost gate execution, verify:

- PR/task head, base, and body metadata were read back.
- hosted checks are green for the current head, not an old head.
- scope diff matches objective, branch type, and allowed write paths.
- machine-readable metadata is placed where the parser consumes it, with legal enum values.
- recent guardian/review findings are dispositioned.
- same-class search covered relevant surfaces: public API, formal spec, generated output, metadata parser, head binding, integration/live gate, or equivalent risk surfaces.
- previous root-cause drift has been corrected before rerunning guardian.

If a similar finding, test failure, metadata gap, or gate gap appears twice, stop high-cost retries and issue a narrow root-cause correction objective.

## Guardian Finding Root Cause

When a finding is fixed, the worker report should include:

- finding id/source.
- root cause.
- changed files and why the scope is sufficient.
- same-class search command/results.
- targeted validation.
- PR/task body update and readback status.
- new head/base.
- remaining risk.

The scheduler consumes this before another high-cost run.

## Controlled Merge / Release

Use the repository's controlled wrapper for merge, approval, release, or deployment.

Proceed only when:

- local gate/check passes.
- hosted required checks pass.
- head/body/payload metadata align.
- mergeability/readiness is clean.
- host enforcement and required checks are proven.
- controlled wrapper check passes.

Stop and request scheduler/root-cause action when:

- host enforcement/readback remains unavailable after bounded retry.
- required checks cannot be proven enforced.
- wrapper fails for a non-transient reason.
- the proposed path uses a raw host command to bypass wrapper policy.

## Post-Merge Readback

Implementation merge is usually not full completion. Read back:

- merged state and merge/release commit.
- target branch/base state.
- related issue/task closure or completion.
- reconciliation/closeout checks.
- terminal carrier/status/shadow/evidence state.
- final validation evidence and remaining risk.

If the target branch still records pre-merge state, create a closeout-only branch/PR/task based on latest base/main.

## Closeout-Only Work

Closeout-only work should change only carrier/status/evidence files needed to consume completion facts. It must not include implementation changes unless the scheduler explicitly changes the objective.

Closeout-only PR/task still requires:

- fresh base/main.
- narrow branch and allowed write paths.
- metadata readback.
- hosted checks.
- controlled merge.
- final readback.

Post-merge evidence must be labeled as post-merge. Do not present post-merge review comments or checks as if they were pre-merge gates.
