# Unit Providers / 调度单元来源

## Provider Selection / 来源选择

不要假设项目一定有 milestone。先选择最小可证明的 unit provider。

```text
provider_selection:
- available_sources:
- chosen_provider:
- why_this_provider:
- source_locator:
- completion_predicate_source:
- known_gaps:
```

## Provider Types / 来源类型

### github_milestone

适用于 GitHub milestone 明确承载一批 issues/PRs 的项目。

默认 completion predicate：

- milestone open issues 为 0，或全部目标 issue 已 CLOSED/COMPLETED。
- milestone 相关 open PR 为 0，或所有相关 PR 已 merged/closed 且被解释。
- scheduler final report 显示 merge/readback/closeout consumed。
- repo carrier 或项目状态已消费完成事实。

### parent_issue

适用于 parent issue 拥有 child issues、blockedBy/blocks 或 task list 的项目。

默认 completion predicate：

- child issues 全部 terminal。
- parent issue 的 acceptance / closeout 被消费。
- 无 linked open PR 或 unresolved scheduler gate。
- scheduler final report 证明 parent closeout/readback。

### issue_set

适用于 label、query、project status 或手工列表定义的一组 issues。

默认 completion predicate：

- issue set 中所有 required issues terminal。
- optional/deferred issues 被明确标记 deferred，不算 complete。
- issue set 不再有 open merge/gate/closeout PR。
- scheduler final report 与 live readback 一致。

### project_view

适用于 GitHub Project、Linear、Jira 或等价项目视图是主要 truth 的项目。

默认 completion predicate：

- project view 中目标 items 到 terminal/status done。
- linked PR/issue 状态与 project view 一致。
- scheduler final report 解释任何 project drift。

### release

适用于 release train、version、deployment 或 launch checklist。

默认 completion predicate：

- release checklist terminal。
- release/no-release evidence 记录完整。
- deployment/approval gate 由 scheduler 消费。
- post-release readback 或 no-release rationale 已记录。

### repo_carrier

适用于仓库内状态文件是权威事实，如 `.loom/status/current.md`、roadmap、plan、progress。

默认 completion predicate：

- carrier 中 current unit terminal。
- related PR/issue/live host 与 carrier 一致。
- shadow/status/review/evidence 等派生载体同步。
- scheduler final report 证明 carrier consumption。

### manual_goal

适用于用户只给了自然语言目标，尚无正式 issue/milestone。

默认 completion predicate 必须由 watcher 创建前写明，不得使用“用户说完成了”作为唯一证明。至少包含：

- expected scheduler output。
- expected host/repo readback。
- expected closeout 或 not_applicable rationale。

## Provider Fallback / 来源不足时的兜底

如果 provider 事实不足：

- 不创建多 scheduler。
- 可以创建一个 planning scheduler，objective 是建立 unit graph 和 completion predicate。
- watcher prompt 必须记录 `provider_gap`，并说明需要补充哪些 host/repo facts。
