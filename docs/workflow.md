# Super-agent 怎样完成这次研究

[返回首页](../README.md) · [结果摘要](results.md)

## 1. 把研究需求交给一个持续任务

用户需要从 Marin 的大量更新里找出值得学习和讲解的进展。Codex 将需求写入 Issue #3：限定研究窗口，要求筛选选题、回读来源核查，再整理成适合非预训练背景读者的材料。

任务允许读取 Marin，向本成果仓库提交文件。研究、核查和写作的分工要求写在派发消息中；具体研究拆分由执行 Agent 决定。

[实际派发消息](https://github.com/AdaChenScience/superagent-playground/issues/3#issuecomment-6569792751)

## 2. Super-agent 管任务，DSH 执行工具调用

GitHub Source 将消息转成事件。Super-agent 根据讨论与任务的关联，将这次要求交给已有 Ticket 的执行会话。Ticket 保存任务关系；执行会话持有上下文，并在对应工作目录里处理资料和文件。

这里有两层分工：Super-agent 管理长期任务对应的执行 Agent；执行 Agent 再调用 DSH 的 `subagent` 工具，临时安排研究、核查和写作工作。这四个子 Agent 属于同一次研究，不是四个独立的 GitHub 任务。

## 3. 执行 Agent 组织研究与核查

| 子 Agent | 收到的任务 | 产出如何被使用 |
| --- | --- | --- |
| 训练主线研究 | 查主训练、长上下文切换、梯度异常与运行事故 | 提供候选进展、来源和背景 |
| 评测研究 | 查评测长度、指标口径与相关修复 | 补充候选进展及指标解读 |
| 独立核查 | 回读原始评论、PR 和提交，核对研究中的主张 | 给出修正意见，传回写作与主执行 Agent |
| 写作整理 | 根据研究材料、证据和读者要求组织内容 | 形成简报与专题材料，由主执行 Agent 整合 |

两路研究并行；写作在拿到材料后开始，随后接收核查修正。主执行 Agent 负责合并文件和提交。一次具体修正是把“没有长上下文评测”拆清楚：常规预测误差评测和专门的长文能力测试回答不同的问题。

[子 Agent 调用与交接记录](../archive/docs/pilot-001/run-record.md)

## 4. 反馈为什么能继续原任务

Codex 在原 Issue 追加两轮试刊验收意见，要求解释缩写、修正事实范围，并将过细的配置移到附录。Super-agent 将新消息续接给原执行 Agent，已有材料继续修改，结果更新到同一 PR。

代码中的 `Intake.receive` 按事件 ID 去重，沿讨论订阅找到 Ticket，并把消息保存为 Input。有归属的人工后续消息交给对应任务处理，不必每次重新创建任务。`Engine` 负责状态更新，模型负责理解反馈和决定怎样修改。

这个设计让“任务还在等待审阅”与“模型这一轮已经回答完”可以分开处理。

[第一次反馈](https://github.com/AdaChenScience/superagent-playground/issues/3#issuecomment-6570006606) · [第二次反馈](https://github.com/AdaChenScience/superagent-playground/issues/3#issuecomment-6570091548) · [固定版本实现](https://github.com/deepseek-ai/dsh-superagent-prototype/blob/6337fc823ce20251c8de8085948165c81aaa701e/src/core/intake.ts)

## 5. 将结果交到可审阅的位置

主执行 Agent 把简报、专题材料、证据和跟踪状态提交到 PR #4，并回到原 Issue 回复交付链接。后续又在同一任务中完成长上下文导读和口播样稿，Codex 做了最终局部修订。

GitHub 在这里同时承担任务入口、反馈入口和版本化交付。研究结果、修改要求与文件差异可以沿同一条记录查看。

[交付回复](https://github.com/AdaChenScience/superagent-playground/issues/3#issuecomment-6570165496) · [PR #4](https://github.com/AdaChenScience/superagent-playground/pull/4)
