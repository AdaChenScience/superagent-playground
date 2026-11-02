# Marin 研究与视频选题工作台

基于 DeepSeek Harness（DSH）和 Super-agent 的一次真实工作流实践：从 Marin 项目的公开讨论中筛选进展，核查证据，再整理成视频选题与教学材料。

**当前状态：私有展示整理版；完成一期试刊，尚未配置周期运行。** 试刊事实截点为 **2026-11-02 05:30 UTC（北京时间 13:30）**，不是实时项目状态。

## 我为什么做这件事

我关注 Marin 的大模型训练，也需要定期制作相关教学视频。GitHub 的更新多而分散，直接汇总容易堆满工程细节；我需要的是值得讲的变化、能理解的背景，以及支持结论的原始证据。

这次用 Super-agent 跑了一轮从 GitHub 请求、子 Agent 分工、证据核查到 PR 交付的流程。重点是减少我逐条读资料、查错和改稿的负担，保留选题方向与发布决策。

## 从哪里开始看

| 你想看什么 | 阅读入口 |
| --- | --- |
| Marin 最近有什么值得讲 | [试刊 001：三个排序选题](docs/pilot-001/brief.md) |
| 推荐选题怎样讲清楚 | [教学材料：长上下文切换成功，离能力证明还有多远](docs/pilot-001/teaching.md) |
| 这次 Agent 实践有什么设计与得失 | [案例复盘](docs/case-study.md) |
| 想按相同要求运行一次 | [任务规范](docs/task-contract.md) → [运行说明](docs/runbook.md) |

## 这次实际怎样运行

```text
GitHub Issue 请求 / 后续评论
           ↓
Super-agent 关联原任务与执行会话
           ↓
执行 Agent 委派两个研究子 Agent（训练主线 / 评测）
           ↓
核查子 Agent 回读原始证据；写作子 Agent 接收材料及修正
           ↓
教学改写 → Humanizer 润色 → 执行 Agent 整合与检查
           ↓
更新同一 PR，回到原 Issue 交付
```

研究分工并行进行；核查和写作有交接依赖，写作期间也接收核查修正。它不是四个 Agent 各写一遍，再把答案拼起来。

## 已验证的结果与限制

- 四个子 Agent 实际执行；[运行记录](docs/pilot-001/run-record.md)保留职责、会话 ID 和交接说明。
- 交付三个排序选题、一份教学材料、[证据表](docs/pilot-001/evidence.md)和[跟踪快照](docs/pilot-001/tracking.json)。
- 核查纠正了重试次数、异常归因范围、评测类型等表述。正文经过两次 Codex 验收反馈后，才减少缩写并把技术细节移到附录。
- 本轮有 **2 次 Codex 验收指导、0 次用户追加决策**，不能称为无人验收或零返工。
- 未运行 Marin 训练或 GPU 实验；运行数字来自公开记录。未验证持续跟踪的稳定性，也没有多 Agent 对单 Agent 的速度、成本或准确率对照。

## 谁负责什么

| 参与方 | 责任 |
| --- | --- |
| 仓库所有者 | 提出视频制作需求，确定读者、范围、交付和验收目标；决定是否采用及发布 |
| Codex | 协助环境配置、任务派发、日志核对、两轮验收纠偏；完成本次展示文档整理 |
| Super-agent | 请求与任务关联、执行会话管理、反馈续接、PR 与原讨论交付 |
| DSH 与模型 | 提供执行工具、子 Agent 会话及模型推理；完成研究、判断、核查和写作 |

本仓库是**使用、配置和评估开源框架的实践案例**，不是自研多 Agent 框架，也不是 Marin 官方发布渠道。上游将 Super-agent 定位为研究原型，非正式产品；本案例不改变这一定位。

## 证据与历史

- [原任务 Issue #3](https://github.com/AdaChenScience/superagent-playground/issues/3) / [交付 PR #4](https://github.com/AdaChenScience/superagent-playground/pull/4)：派发、追加反馈与交付历史；仓库私有期间需要访问权限。
- [案例复盘](docs/case-study.md)：选题筛选、核查发现、返工与能力边界。
- [早期长上下文专题](docs/marin-long-context-evaluation.md)：试刊前的学习产物，保留原始快照，不作为最新状态。
- [公开前检查](docs/publication-review.md)：已查范围、待处理事项和未覆盖内容。当前没有公开或自动合并操作。

## 上游与来源

- [DeepSeek Super-agent 研究原型](https://github.com/deepseek-ai/dsh-superagent-prototype)、[DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)。本轮版本及本地适配见[运行说明](docs/runbook.md)。
- [Marin](https://github.com/marin-community/marin)：事实来源，具体评论和 PR 链接保留在证据表中。
- [Humanizer-zh](https://github.com/op7418/Humanizer-zh)：本轮使用本机修订版编辑规范，通过读取文件执行，不是训练了新的润色模型；不在本仓库重新分发该技能全文。

当前没有为本仓库新增统一开源许可；上游项目和来源内容仍遵循各自许可。展示与第三方再分发许可是两件事。
