# 技术复盘：一次研究怎样变成可继续的任务

[返回首页](../README.md) · [运行证据](evidence-index.md)

## 需求为什么适合用来检验 Super-agent

Marin 跟踪需要记住上一轮看到哪里、哪些结论待确认，以及新消息属于哪个问题。一次总结只需读取资料；持续跟踪还要管理任务、输入、交付和等待状态。Super-agent 的价值要在这些后续环节验证。

当前完成的是“请求与反馈可续接”的实验，还没有完成“上游更新自动进入跟踪”的验证。两者不能混为一谈。

## 原任务如何接到新消息

本次在 Issue #3 追加研究要求和验收意见，执行会话继续处理并更新 PR #4。对应实现中，外部事件叫 Trigger，讨论串叫 Thread，持续任务叫 Ticket，一次执行叫 Assignment。

`Engine.receive` 在事务中调用 `Intake.receive`。后者先按 Trigger ID 去重，再按 Thread 的订阅关系把事件转成已有 Ticket 的 Input；有归属的人工后续消息不必重新创建研究任务。需要另行判断的事件才进入 Eval。模型判断内容怎么处理，代码负责记录它属于谁、有没有投递。

这比“把新评论再塞进一段提示词”多了任务归属和状态，但不保证每条消息都适合原任务，也不保证模型正确使用旧上下文。[实现：Engine](https://github.com/deepseek-ai/dsh-superagent-prototype/blob/6337fc823ce20251c8de8085948165c81aaa701e/src/core/engine.ts)、[Intake](https://github.com/deepseek-ai/dsh-superagent-prototype/blob/6337fc823ce20251c8de8085948165c81aaa701e/src/core/intake.ts)。

## 两层 Agent 分工不能混在一起

Super-agent 管理 Ticket 对应的执行会话与目录。研究 A、研究 B、核查和写作，则是这次执行模型在任务内部调用 DSH `subagent` 工具创建的会话，不是四个独立的 Super-agent 业务任务。

初始派发已要求研究、核查和写作；执行模型把研究拆成训练主线与评测两条线。研究产物交给核查和写作，核查修正再传回，主执行 Agent 统一整合并提交。DSH 执行工具调用，不替模型决定研究方向。

分工的理由是让核查者回读一手材料，而非仅复述研究摘要。本次确实纠正了评测类型、重试次数和异常归因范围，但没有单 Agent 对照，不能归因出多 Agent 的准确率增益。[运行记录](pilot-001/run-record.md)

## 为什么执行完成不等于任务结束

Worker 提示词要求在任务目录完成工作，用 `orchestrator_report` 报告，并区分已完成工作与未完成草稿。框架还保留讨论及 PR 的关联；`ready` 并不总等于 Ticket 结束，生命周期也要考虑它跟踪的讨论是否关闭和后续消息是否处理完。

这适合 PR 审阅期间继续接收修改意见。本次已合并 PR #4，下一期应引用旧快照并创建新的交付 PR，不能默认永远更新旧 PR。[Worker 提示词](https://github.com/deepseek-ai/dsh-superagent-prototype/blob/6337fc823ce20251c8de8085948165c81aaa701e/prompts/worker.md)、[架构说明](https://github.com/deepseek-ai/dsh-superagent-prototype/blob/6337fc823ce20251c8de8085948165c81aaa701e/docs/007-architecture.md)

## 可靠性由代码承担，内容质量仍需验收

Engine 统一事务更新核心状态，外部操作在提交后执行。例如 `Outbox.send` 在同一事务中保存消息回执并排队发送；发送失败可以重试。这样不会把“模型想回复了”直接当成“对方收到回复了”。去重、事务和待发送记录也不能直接推出外部系统端到端恰好执行一次。

这是本次核对过的源码设计，不是我们已经做过重启、断网或重复事件测试。原型还实现中断恢复，但本案例没有对应故障注入证据。[Outbox 实现](https://github.com/deepseek-ai/dsh-superagent-prototype/blob/6337fc823ce20251c8de8085948165c81aaa701e/src/app/outbox.ts)

内容质量是另一条线：独立核查后仍需要 Codex 返工，Humanizer 润色也没让教学稿达到使用者要求。框架保证工作能继续，并不保证总结值得读。

## 对这次实践的判断

已验证：任务续接、真实子 Agent 调用、核查反馈交接、同一 PR 迭代。

尚未验证：持续发现 Marin 新进展、过滤重复和琐碎变更、减少人的阅读负担。下一轮应交付相对上一期的变化及筛选理由，用一次真正的增量运行检验这些缺口，而不是继续增加文稿数量。[任务与验收](task-contract.md)
