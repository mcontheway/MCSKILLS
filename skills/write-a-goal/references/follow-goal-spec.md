# 目标规范

本参考用于按照 OpenAI《Follow a goal》指南和当前 skill 策略，编写或校验 Codex goal。

来源：https://developers.openai.com/codex/use-cases/follow-goals

## 核心规则

当任务需要 Codex 跨多轮持续推进，直到达到可验证的停止条件时，才适合使用 goal。

基础模式是：

```text
/goal Complete [objective] without stopping until [verifiable end state].
```

## 输出模式

支持两种输出模式：

- `copyable_goal_command`：面向人类用户复制到 Codex Desktop、CLI 或 IDE 的命令。必须输出一条物理单行，并以 `/goal ` 开头。完整契约必须放在第一个自然段内，因为 slash command 解析可能只把该段设置为 objective。
- `active_goal_api`：面向可以直接创建 active goal 的 agent 或 tool。API objective 可以是多段，但必须自包含。不要依赖 objective 外的 assistant 正文承载必要要求。

## 适用场景

适合使用 goal 的任务：

- 有明确成功条件和验证循环的长周期编码任务。
- 代码迁移、大型重构、部署重试循环、实验、原型和可以持续限定范围推进的 side project。
- 研究、数据分析、清理或分诊任务，前提是有证据能够证明完成。

不适合使用 goal 的任务：

- 一行修改或简单解释。
- 松散且互不相关的任务清单。
- 缺少验证面的模糊“让它更好”类任务。
- 主要依赖人工判断的决策，除非 goal 是为了准备证据或选项。

对于不清晰的 goal，问一个简洁问题，或做保守假设。不要默认建议用户先执行 `/plan`；只有在用户明确需要规划命令时才输出 `/plan`。

## 必要契约

好的 goal 应该大于一次普通 prompt，但小于开放式待办列表。它应定义：

- 结果（Outcome）：Codex 应达成的最终状态。
- 验证面（Verification surface）：用于证明进展和完成的命令、产物、报告、截图、外部状态或其他证据。
- 约束（Constraints）：不得破坏的行为、业务、安全、数据、权限、API 或质量属性。
- 边界（Boundaries）：允许和禁止触碰的文件、模块、系统、账号、动作或写入范围。
- 迭代策略（Iteration policy）：Codex 如何尝试、重跑检查、报告证据，并避免重复失败路径。
- 阻塞停止条件（Blocked stop condition）：Codex 应在什么情况下暂停，而不是继续猜测。

只有当输入/上下文能澄清起点时才加入。只有在资料顺序确实重要时，才写“先阅读”或“先检查”。

## 动作授权策略

外部或高影响动作不应被自动禁止。如果用户目标需要发送消息、关闭 issue、merge PR、部署、删除、修改权限或写入外部系统等动作，应加入动作授权策略：

- 允许执行哪些动作。
- 哪些对象或系统在范围内。
- 哪些动作仍然禁止。
- 哪些动作需要确认。
- Codex 应为已执行、已草拟、已跳过或被阻塞的动作报告什么证据。

除非用户授权明确且范围具体，否则不可逆、生产、安全、隐私、付款、权限、批量或含义模糊的动作都应要求确认。

## 契约模板

中文可复制命令：

```text
/goal 完成[结果]。输入/上下文：[输入和上下文]。验证：[验证面]。约束：[约束]。边界：[边界]。迭代：[迭代策略]。阻塞：[阻塞停止条件]。完成：[完成条件]。
```

英文可复制命令：

```text
/goal Achieve [Outcome]. Inputs/context: [Inputs]. Verification: [Verification surface]. Constraints: [Constraints]. Boundaries: [Boundaries]. Iteration: [Iteration policy]. Blocked: [Blocked stop condition]. Done: [Completion condition].
```

如果更短的命令更清晰，不要机械填满每个标签。但六个核心要素必须在实质上存在。

## 状态与完成

进展报告应简洁说明：

- 当前尝试或检查点。
- 已验证什么。
- 还剩什么。
- Codex 是否被阻塞。

完成前，goal 必须有与 objective 范围匹配的证据。测试、报告、外部状态或命令输出，只有在确实覆盖所声明的要求时才算有效证据。

## 失败控制

避免使用“直到全部完成”这类模糊停止条件，除非同时配有具体检查。

有用的阻塞条件包括：

- 缺少必要访问权限、凭据、文件、数据或服务。
- 下一步需要产品、法务、安全、隐私或业务决策。
- 需要执行动作授权策略之外的危险或不可逆动作。
- 同一个聚焦方案反复失败，且没有新证据。
- 验证被基础设施或外部系统阻塞。
