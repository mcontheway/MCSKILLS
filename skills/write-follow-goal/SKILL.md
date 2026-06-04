---
name: write-follow-goal
description: Draft, refine, or set Codex goals that follow OpenAI's "Follow a goal" guidance. Before drafting, resolve the output language from the latest user-facing request. For copyable slash-command output, generate one single-line /goal command. For active goal creation through a goal tool/API, create a self-contained objective that may be multi-paragraph. If the user writes Chinese, mixes languages, or language is unclear, output Chinese. Preserve only commands, paths, code identifiers, URLs, issue IDs, and exact quoted literals.
---

# Write Follow Goal

## Overview

Create a concise, durable Codex goal contract from the user's context. Prefer one objective, a verifiable stopping condition, explicit constraints and boundaries, and a practical validation loop.

Use `references/follow-goal-spec.md` as the source of truth for the goal contract. Read it when the user asks for official compliance, asks to validate an existing goal, or the goal is high stakes, ambiguous, or long-running enough that exact structure matters.

## Language Policy

Reply in the user's language by default. If the user writes in Chinese or the language is mixed or unclear, use Chinese. Keep commands, file names, code identifiers, URLs, issue IDs, and exact quoted literals unchanged.

## Language Gate

Before any draft, explanation, question, or tool-created goal, set `output_language` from the latest user-facing request, not from this skill file, examples, file paths, code, logs, or quoted text.

Rules:

- If the latest user message contains Chinese characters, set `output_language = Chinese`.
- If the message mixes Chinese and English, set `output_language = Chinese`.
- If the language is unclear, set `output_language = Chinese`.
- Use `output_language` for every generated sentence, including the `/goal` body, explanations, headings, assumptions, self-checks, and questions.
- Keep only shell commands, file paths, code identifiers, URLs, issue IDs, and exact quoted source text unchanged.

## Output Mode Gate

Before drafting or setting a goal, choose exactly one output mode:

- `copyable_goal_command`: Use when the user asks for goal wording, a `/goal` command, a copyable command, a draft/refinement/analysis that should produce a goal, or does not explicitly ask Codex to set an active goal. Return a single physical line starting with `/goal ` as the executable artifact.
- `active_goal_api`: Use when the user explicitly asks to set/create/start an active goal and a goal tool/API is available. Create the goal through the tool/API. The objective may be multi-paragraph, but it must be self-contained and include the full goal contract.

When no goal tool/API is available, fall back to `copyable_goal_command` and say that the user can copy the command.

## Command Shape Contract

For `copyable_goal_command`, the final executable goal must be one physical line that starts with `/goal ` and contains the complete goal contract after that space. Do not put blank lines, Markdown paragraphs, bullets, or line breaks inside the command.

Desktop/CLI/IDE slash-command parsing may only set the first natural paragraph after `/goal ` as the goal body, so every required instruction for a copyable command must be in that same paragraph. If you include any explanation, analysis, or self-check, place it outside the command and never rely on it to carry required goal details.

For `active_goal_api`, do not force the objective into one line. Multi-paragraph objectives are allowed when they improve clarity, but every required instruction must be inside the objective passed to the goal tool/API. Do not rely on assistant prose outside the objective to carry outcome, verification, constraints, boundaries, iteration policy, blocked conditions, or completion criteria.

## Goal Contract Model

Every finalized command or API objective must cover the six core elements, compressed as needed:

- Outcome: the final state to achieve, not only an activity to perform.
- Verification surface: commands, artifacts, reports, screenshots, external statuses, or other evidence that proves progress and completion.
- Constraints: business, safety, data, permission, API, behavior, or quality properties that must not be broken.
- Boundaries: allowed and forbidden files, modules, systems, accounts, actions, or write scopes.
- Iteration policy: how Codex should proceed between attempts, rerun checks, record evidence, and avoid repeating failed approaches.
- Blocked stop condition: when Codex must pause instead of guessing, including missing access, required human decisions, repeated failures, or unsafe ambiguity.

Add two supporting fields when useful:

- Inputs/context: source files, issues, logs, reports, URLs, data exports, screenshots, or external systems that define the starting point. Only say "read first" when the order materially matters.
- Completion condition: the exact evidence state that permits Codex to stop or mark the goal complete.

## Action Policy

Do not blindly forbid external or high-impact actions. If the user's goal requires actions such as sending, closing, merging, deploying, deleting, changing permissions, or writing to an external system, include an action policy in the goal contract:

- Allowed actions: what Codex may do, on which objects, and within which scope.
- Forbidden actions: actions outside the user's authorization or outside the goal.
- Confirmation triggers: irreversible, production, security, privacy, payment, permission, bulk, or ambiguous actions that still require explicit approval.
- Evidence: what proof Codex should report for actions taken, actions drafted, and actions skipped or blocked.

## Workflow

1. Extract the durable objective from the user's context.
2. Identify the verifiable end state before drafting.
3. Choose the output mode: `copyable_goal_command` or `active_goal_api`.
4. Define inputs/context only where they clarify the starting point; avoid default "read first" noise when source order does not matter.
5. Define verification: commands to run, artifacts to inspect, screenshots to compare, tests to pass, scores to reach, external states to confirm, or documents to update.
6. Set constraints and boundaries: what Codex may change or do, what it must preserve, and what requires confirmation.
7. Add action policy for external systems or high-impact operations when relevant.
8. Add iteration behavior: focused attempts, validation cadence, evidence to report, and repeated-failure handling.
9. Add pause or block conditions: missing credentials, ambiguous decisions, unsafe actions, policy calls, unclear requirements, or repeated validation failure.
10. For `copyable_goal_command`, compress the contract into one `/goal ` command line; keep supporting prose optional and nonessential.
11. For `active_goal_api`, create the active goal with the full self-contained objective, then briefly report what was set.
12. Check that every generated sentence follows `output_language`; revise before responding if any generated prose falls back to English by accident.

## Draft Shape

The command or objective should follow the spec starter pattern, translated into `output_language`: complete one objective without stopping until one verifiable end state is true.

For `copyable_goal_command`, use a single-line template in `output_language` unless the user's environment needs a shorter command. For Chinese output, prefer:

```text
/goal 完成[Outcome]。输入/上下文：[Inputs]。验证：[Verification surface]。约束：[Constraints]。边界：[Boundaries]。迭代：[Iteration policy]。阻塞：[Blocked stop condition]。完成：[Completion condition]。
```

For non-Chinese copyable command output, translate the same one-line structure into `output_language`; do not copy the English shape unless `output_language` is English.

For English output, prefer:

```text
/goal Achieve [Outcome]. Inputs/context: [Inputs]. Verification: [Verification surface]. Constraints: [Constraints]. Boundaries: [Boundaries]. Iteration: [Iteration policy]. Blocked: [Blocked stop condition]. Done: [Completion condition].
```

Do not mechanically fill every label if a shorter version is clearer, but do not omit the underlying six core elements. For `copyable_goal_command`, do not output the executable command as a multi-line block. If you use a Markdown code block for a copyable command, the code block must contain exactly one command line.

For `active_goal_api`, you may use multiple paragraphs in the objective. Use labels or compact paragraphs when that makes the contract easier to audit.

## Quality Bar

A good goal must be:

- Bigger than a single normal prompt but smaller than an open-ended backlog.
- Centered on one objective, not a bundle of unrelated requests.
- Clear about what "done" means before Codex starts.
- Concrete about validation artifacts or commands.
- Explicit about constraints, boundaries, and non-goals.
- Specific enough that another Codex instance could continue after compaction.

Avoid vague stopping conditions such as "when it looks good", "until everything is done", or "until no issues remain" unless they are paired with concrete checks.

Do not turn every task into a goal. If the user's request is a one-line edit, simple explanation, loose brainstorming, or a decision that mostly depends on human judgment, say that a normal prompt or planning pass is more appropriate unless the user still wants a goal. For ambiguous goals, ask at most one concise question or make a conservative assumption.

## Conversation Handling

Ask at most one concise question only when a missing detail changes the goal materially and cannot be inferred safely. Otherwise, make conservative assumptions and state them in the goal.

When the user says "set the goal", "create an active goal", "start this goal", or equivalent, create the goal using the available goal tool if present. When they only ask for wording, a draft, analysis, or a copyable command, provide the `/goal` text without activating it.

## Example

用户：帮我写一个 /goal，让 Codex 持续修复 CI，直到 PR 可合并。

输出语言：中文

```text
/goal 修复当前 PR 的 CI 失败。输入/上下文：PR 描述、失败 CI 日志、相关测试和最近 diff。验证：本地复现命令、对应测试、最终 PR checks 全绿。约束：不跳过、不删除、不弱化测试，不做无关重构。边界：只改导致 CI 失败的代码、测试或配置。迭代：按失败项分组修复，每修一类重跑最小相关验证，完成前跑全量必需检查。阻塞：缺少权限、外部凭据、同类失败重复 3 次仍无法定位或需要产品/安全决策。完成：所有必需 checks 通过，PR 无阻塞性 review finding，并在最终摘要中列出修复内容与验证证据。
```

## Validation Pass

Before finalizing, check that the goal answers:

- Output mode: Is this a copyable slash command or an active goal API request?
- Outcome: What final state should Codex achieve?
- Verification surface: What commands, artifacts, external states, or reports prove progress and completion?
- Constraints: What behavior, data, permissions, security, quality, or business rules must not be broken?
- Boundaries: What files, systems, accounts, actions, or write scopes are allowed or forbidden?
- Iteration policy: How should Codex make attempts, validate, record evidence, and avoid repeated failed approaches?
- Blocked stop condition: What should cause Codex to pause and ask?
- Completion condition: What exact evidence lets Codex stop or mark complete?
- Action policy: If external or high-impact actions are involved, what is allowed, forbidden, confirmation-gated, and recorded?
- Command/API shape: For `copyable_goal_command`, is the executable goal exactly one physical line starting with `/goal `? For `active_goal_api`, is the objective self-contained even if multi-paragraph?
- Spec contract: Does the command or objective state what to achieve, what not to change, how to validate progress, and when to stop?
- Language: Does every generated sentence, including the `/goal` body, match `output_language`?

If any answer is missing, revise the goal before returning it.
