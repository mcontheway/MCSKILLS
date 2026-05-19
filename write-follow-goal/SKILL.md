---
name: write-follow-goal
description: Draft and refine Codex /goal instructions that follow OpenAI's "Follow a goal" guidance. Before drafting, resolve the output language from the latest user-facing request. All assistant prose and generated /goal body must use that language. If the user writes Chinese, mixes languages, or language is unclear, output Chinese. Preserve only commands, paths, code identifiers, URLs, issue IDs, and exact quoted literals.
---

# Write Follow Goal

## Overview

Create a concise, executable `/goal` instruction from the user's context. Prefer one durable objective, a verifiable stopping condition, explicit scope boundaries, and a practical validation loop.

Read `references/follow-goal-spec.md` when the user asks for official compliance, asks to validate an existing goal, or the goal is high stakes, ambiguous, or long-running enough that exact structure matters.

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

## Workflow

1. Extract the durable objective from the user's context.
2. Identify the verifiable end state before drafting.
3. Define what Codex must read first: files, docs, issues, logs, plans, screenshots, or URLs.
4. Set boundaries: what to change, what to preserve, and what requires asking the user.
5. Define validation: commands to run, artifacts to inspect, screenshots to compare, tests to pass, scores to reach, or documents to update.
6. Add checkpoint behavior: work in scoped checkpoints, keep a short progress log, and re-run validation after meaningful changes.
7. Add pause or block conditions: missing credentials, destructive operations, product decisions, policy calls, unclear requirements, or repeated validation failure.
8. Return the final `/goal` plus a short self-check when useful.
9. Check that every generated sentence follows `output_language`; revise before responding if any generated prose falls back to English by accident.

## Draft Shape

Use a template in `output_language` unless the user's environment needs a shorter command. For Chinese output, prefer:

```text
/goal 完成[目标]，持续推进直到[可验证终态]成立。

先阅读[必要上下文]。
范围限定在[边界]。
按检查点推进：[检查点节奏或里程碑]。
用[命令、产物或审查步骤]验证进展。
维护简短进度记录，包含当前检查点、已验证内容、剩余事项和阻塞。
仅在[阻塞条件]出现时暂停。
当[完成条件]成立且[最终验证]通过时停止。
```

For non-Chinese output, translate the same structure into `output_language`; do not copy the English shape unless `output_language` is English.

## Quality Bar

A good goal must be:

- Bigger than a single normal prompt but smaller than an open-ended backlog.
- Centered on one objective, not a bundle of unrelated requests.
- Clear about what "done" means before Codex starts.
- Concrete about validation artifacts or commands.
- Explicit about boundaries and non-goals.
- Specific enough that another Codex instance could continue after compaction.

Avoid vague stopping conditions such as "when it looks good", "until everything is done", or "until no issues remain" unless they are paired with concrete checks.

## Conversation Handling

Ask at most one concise question only when a missing detail changes the goal materially and cannot be inferred safely. Otherwise, make conservative assumptions and state them in the goal.

When the user says "set the goal" or asks you to create an active goal, create the goal using the available goal tool if present. When they only ask for wording, provide the `/goal` text without activating it.

## Example

User: 帮我写一个 /goal，让 Codex 持续修复 CI，直到 PR 可合并。

Output language: Chinese

```text
/goal 修复当前 PR 的 CI 问题，持续推进直到所有必需检查通过且 PR 达到可合并状态。

先阅读 PR 描述、最近失败的 CI 日志、相关测试文件和最近一次修改记录。
范围限定在导致 CI 失败的代码、测试或配置；不要扩大到无关重构。
按失败项逐个检查点推进，每修复一类失败后重新运行对应验证。
用本地测试、CI 日志和 PR 检查状态验证进展。
维护简短进度记录，包含当前失败项、已验证内容、剩余失败和阻塞。
仅在缺少权限、需要外部凭据、发现破坏性操作需求或同类失败重复两次仍无法定位时暂停。
当所有必需 CI 检查通过且 PR 没有阻塞性 review finding 时停止。
```

## Validation Pass

Before finalizing, check that the goal answers:

- Objective: What exactly should Codex accomplish?
- End state: What verifiable condition lets Codex stop?
- Context: What must Codex read first?
- Boundaries: What should Codex avoid changing?
- Loop: How should Codex prove progress?
- Checkpoints: How should Codex report compact status?
- Blocks: What should cause Codex to pause and ask?
- Language: Does every generated sentence, including the `/goal` body, match `output_language`?

If any answer is missing, revise the goal before returning it.
