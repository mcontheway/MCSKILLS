# Follow Goal Spec

Use this reference to write or validate Codex goals against OpenAI's "Follow a goal" guidance and the current skill policy.

Source: https://developers.openai.com/codex/use-cases/follow-goals

## Core Rule

Use a goal when a task needs Codex to keep working across turns toward a verifiable stopping condition.

The starter pattern is:

```text
/goal Complete [objective] without stopping until [verifiable end state].
```

## Output Modes

There are two supported output modes:

- `copyable_goal_command`: for human users copying a command into Codex Desktop, CLI, or IDE. Output exactly one physical line starting with `/goal `. Put the complete contract in the first natural paragraph because slash-command parsing may only set that paragraph as the objective.
- `active_goal_api`: for agents/tools that can create an active goal directly. The API objective may be multi-paragraph, but it must be self-contained. Do not rely on assistant prose outside the objective to carry required instructions.

## Good Fit

Use a goal for:

- Long-running coding work with a clear success condition and validation loop.
- Code migrations, large refactors, deployment retry loops, experiments, prototypes, and side projects where Codex can keep making scoped progress.
- Research, data analysis, cleanup, or triage work when evidence can prove completion.

Avoid a goal for:

- One-line edits or simple explanations.
- Loose lists of unrelated work.
- Vague "make it better" tasks that lack a verification surface.
- Decisions that mostly depend on human judgment unless the goal is to prepare evidence or options.

For unclear goals, ask one concise question or make a conservative assumption.

## Required Contract

A good goal is bigger than one prompt but smaller than an open-ended backlog. It should define:

- Outcome: the final state Codex should achieve.
- Verification surface: commands, artifacts, reports, screenshots, external statuses, or other evidence that proves progress and completion.
- Constraints: behavior, business, safety, data, permission, API, or quality properties Codex must not break.
- Boundaries: allowed and forbidden files, modules, systems, accounts, actions, or write scopes.
- Iteration policy: how Codex should make attempts, rerun checks, report evidence, and avoid repeating failed approaches.
- Blocked stop condition: when Codex should pause instead of guessing.

Add inputs/context only when they clarify the starting point. Say "read first" only when source order matters.

## Action Policy

External or high-impact actions are not automatically forbidden. If the user's objective requires actions such as sending messages, closing issues, merging PRs, deploying, deleting, changing permissions, or writing external systems, include an action policy:

- What actions are allowed.
- Which objects or systems are in scope.
- What remains forbidden.
- Which actions require confirmation.
- What evidence Codex should report for actions taken, drafted, skipped, or blocked.

Require confirmation for irreversible, production, security, privacy, payment, permission, bulk, or ambiguous actions unless the user's authorization is explicit and scoped.

## Contract Templates

Chinese copyable command:

```text
/goal 完成[Outcome]。输入/上下文：[Inputs]。验证：[Verification surface]。约束：[Constraints]。边界：[Boundaries]。迭代：[Iteration policy]。阻塞：[Blocked stop condition]。完成：[Completion condition]。
```

English copyable command:

```text
/goal Achieve [Outcome]. Inputs/context: [Inputs]. Verification: [Verification surface]. Constraints: [Constraints]. Boundaries: [Boundaries]. Iteration: [Iteration policy]. Blocked: [Blocked stop condition]. Done: [Completion condition].
```

Do not mechanically fill every label if a shorter command is clearer. The six core elements still need to be present in substance.

## Status And Completion

Ask for compact progress reports that name:

- Current attempt or checkpoint.
- What was verified.
- What remains.
- Whether Codex is blocked.

Before completion, the goal must have evidence that matches the scope of the objective. Tests, reports, external statuses, or command outputs count only when they actually cover the requirement being claimed.

## Failure Controls

Avoid vague stop conditions such as "until everything is done" unless paired with concrete checks.

Useful blocked conditions include:

- Required access, credentials, files, data, or services are missing.
- The next step needs a product, legal, security, privacy, or business decision.
- A dangerous or irreversible action is needed outside the authorized action policy.
- The same focused approach fails repeatedly without new evidence.
- Verification is blocked by infrastructure or external systems.
