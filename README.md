# 用 Super-agent 研究 Marin 大模型训练进展

我定期制作 Marin 项目的宣传教学视频，也希望通过这个项目理解大模型从零训练的过程。Marin 的更新分散在大量 Issue、PR 和讨论里，逐条阅读会花掉很多时间。

这次实践的目标是：让 Super-agent 完成资料搜集、重要性筛选、事实核查和整理，把值得深入了解的进展交给我，用于后续学习和视频选题。

## 我怎样使用它

1. **我提出需求。** 在与 Codex 的对话中说明要关注 Marin 的哪些进展、材料给谁看。Codex 协助配置 DSH、Super-agent 和 DeepSeek 官方 API，并将研究要求写入 GitHub Issue #3。
2. **Super-agent 把消息交给对应任务。** 它接收 GitHub 消息，关联已有任务和执行会话，让执行 Agent 带着之前的上下文继续工作。
3. **执行 Agent 安排研究。** 它通过 DSH 创建两路研究子 Agent，分别调查训练主线和评测；再安排核查与写作，接收结果和修正意见，统一整合文件。
4. **执行 Agent 提交，反馈继续回到原任务。** 结果提交到 PR #4。Codex 检查后将修改要求写回 Issue #3，Super-agent 再投递给原执行 Agent，由它修改文件并更新同一 PR。

![从研究需求到 PR 交付与反馈续接](assets/task-flow.svg)

[查看任务如何执行](docs/workflow.md)

## 最终结果

本次完成一轮 Marin 535B-A23B 研究，整理出三个视频候选方向：长上下文训练切换、评测口径变化、梯度尖峰与训练稳定性。推荐深入研究长上下文切换，并整理了背景解释、来源和后续观察点。

交付包括选题简报、专题材料、事实核查表和跟踪记录，集中保存在同一 PR。研究范围为 2026 年 10 月 19 日至 11 月 2 日。

[直接看结果摘要](docs/results.md) · [查看交付 PR #4](https://github.com/AdaChenScience/superagent-playground/pull/4)

## 工具分工

| 参与方 | 本次负责的工作 |
| --- | --- |
| 我 | 提出真实需求、确定研究范围与读者、判断结果是否适合学习和视频制作 |
| Codex | 协助部署、把需求转成任务、检查运行记录、提出验收反馈并修订交付内容 |
| Super-agent | 接收 GitHub 请求、关联任务、管理执行会话、续接后续消息与交付 |
| 执行 Agent | 决定具体研究拆分，委派子 Agent，整合结果、提交 PR |
| DSH | 提供模型会话、文件与命令工具、子 Agent 创建和消息传递 |

[运行记录](evidence/run-record.md) · [原始选题简报](evidence/brief.md)

工具来源：[Super-agent](https://github.com/deepseek-ai/dsh-superagent-prototype) · [DSH](https://github.com/deepseek-ai/deepseek-harness)
