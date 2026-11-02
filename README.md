# Marin 研究与视频材料

用 DeepSeek Super-agent 和 DSH 跟踪 Marin 的公开训练进展，经过研究、核查、教学改写和 Codex 验收，整理成没有预训练背景也能开始阅读的视频材料。

**先看成果：[长上下文导读](docs/video-001/learn.md) → [约 5–6 分钟口播稿与分镜](docs/video-001/script.md)。** 内容是截至 **2026-11-02 05:30 UTC（北京时间 13:30）** 的历史快照，不代表 Marin 实时状态。尚未录制试讲，也未完成真实初学者理解测试。

## 这一期讲什么

Marin 将训练时一次处理的文本从约 1.6 万 token 延长到约 6.5 万。这里最值得解释的不是参数，而是三个问题：为什么要读更长的文本？怎么判断训练能继续跑？为什么“能跑”还不能证明“更会读长文”？

导读用同一份通知在不同阅读范围下的例子解释长上下文；口播稿给出讲述顺序和画面建议；核查表保留原始讨论、证据范围以及不能下的结论。

| 你想看什么 | 阅读入口 |
| --- | --- |
| 先把问题弄懂 | [零基础导读，含自测问题](docs/video-001/learn.md) |
| 准备试讲 | [口播稿与分镜](docs/video-001/script.md) |
| 核对数字、来源和限制 | [事实核查与制作记录](docs/video-001/fact-check.md) |
| 了解 AI 工具怎样参与、哪里仍需返工 | [案例复盘](docs/case-study.md) |
| 尝试相同工作流 | [任务规范](docs/task-contract.md) → [运行说明](docs/runbook.md) |

## 这个工作流实际做到了什么

任务由 GitHub Issue 发起，Super-agent 将后续评论续接到原任务。执行 Agent 使用 DSH 的子 Agent 工具安排研究、核查和写作，更新同一 PR。Codex 检查内容和运行记录，把验收意见交回原任务，并做了明确记录的最终修订。

第一轮试刊完成了三个排序选题，但材料仍偏难。第二轮聚焦长上下文，把工程记录改成导读、口播和核查表。核查纠正了“常规预测误差评测等于长文能力测试”“训练没出异常就等于能力提升”等表述风险。

这是一次可追溯的工具使用实践：能看到初稿、反馈、返工与交付。尚未验证周期跟踪的稳定性，没有单 Agent 对照、成本统计或准确率测量，不能据此证明多 Agent 更快、更便宜或更准确。

## 谁负责什么

| 参与方 | 实际责任 |
| --- | --- |
| 仓库所有者 | 提出 Marin 视频需求，确定受众与交付目标，判断材料是否能讲懂并决定发布 |
| Codex | 配置、派发任务、核对日志、验收纠偏、最终局部修订与仓库整理 |
| Super-agent | 将请求关联到任务、管理执行会话、续接反馈、组织 PR 交付 |
| 执行模型与子 Agent | 理解任务、选择具体分工、研究、核查、改写与整合 |
| DSH | 提供模型执行环境、工具和子 Agent 会话能力 |

初始试刊的“研究、核查、写作”要求由 Codex 按用户需求写入任务；执行 Agent 决定了具体拆分并实际调用工具。不是用户亲手指派每个子 Agent，也不是 DSH 自行决定研究方向。详见[复盘](docs/case-study.md)。

本仓库展示**配置、使用和评估开源 Agent 框架的实践**，不是自研框架，也不是 Marin 或 DeepSeek 官方发布渠道。Super-agent 上游是研究原型。本仓库主要交付文档，不包含可直接启动的完整运行环境。

## 历史与证据

- [Issue #3](https://github.com/AdaChenScience/superagent-playground/issues/3) / [PR #4](https://github.com/AdaChenScience/superagent-playground/pull/4)：原任务、追加反馈和交付历史。
- [试刊 001 选题简报](docs/pilot-001/brief.md) / [试刊运行记录](docs/pilot-001/run-record.md)：第一轮探索，教学稿已由 video-001 继续改写。
- [早期长上下文专题](docs/marin-long-context-evaluation.md)：更早的研究快照，不作为最新状态。
- [公开检查记录](docs/publication-review.md)：检查范围、脱敏处理及未覆盖部分。

公开材料保留过程摘要和来源，不上传本机登录配置、完整会话日志、数据库或临时研究文件。外部读者不能仅凭会话 ID 重放完整运行。

## 上游与内容许可

- [DeepSeek Super-agent](https://github.com/deepseek-ai/dsh-superagent-prototype)、[DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)：版本和本机适配见[运行说明](docs/runbook.md)。
- [Marin](https://github.com/marin-community/marin)：事实来源，关键链接保留在核查表。
- [Humanizer-zh](https://github.com/op7418/Humanizer-zh)：使用本机修订版编辑规范；不重新分发技能全文。

本仓库未设置统一再分发许可；公开浏览不等于授予任意复用许可，上游和引用来源仍遵循各自许可。AI 参与生成和编辑，引用内容应以原始来源为准。
