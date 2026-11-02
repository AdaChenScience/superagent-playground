# 运行说明

[返回首页](../README.md) · [任务规范](task-contract.md)

本仓库保存任务约定、成果与案例，不包含 DSH 服务、模型凭据或完整部署配置。**克隆本仓库不会自动启动 Agent。** 本说明记录已用环境和运行步骤；没有在一台全新机器上验证一键复现。

## 本轮版本与来源

| 项目 | 本轮环境 |
| --- | --- |
| DSH CLI | `@deepseek-ai/dsh@0.2.1-rc.3` |
| Super-agent | [上游提交 6337fc8](https://github.com/deepseek-ai/dsh-superagent-prototype/tree/6337fc823ce20251c8de8085948165c81aaa701e)，另有下述本机 Windows 适配 |
| 系统与 shell | Windows / PowerShell |
| 模型服务 | DeepSeek 官方 API；本文不推断具体模型档位 |
| 任务入口 | GitHub；允许的用户请求，专用 Agent 账号执行 |
| 交付 | 每任务工作目录、Git 分支、PR、原 Issue 回复 |
| 润色规范 | 本机 Humanizer-zh 修订版（元数据 revision 2026-09-23），通过文件读取；不是 DSH 注册技能 |

上游定位为研究原型，可能与设计文章不同。本轮版本是历史记录，不是“当前最新版本”的声明。

## 第一次搭建

按固定版本的 [SETUP.md](https://github.com/deepseek-ai/dsh-superagent-prototype/blob/6337fc823ce20251c8de8085948165c81aaa701e/SETUP.md) 与 [部署说明](https://github.com/deepseek-ai/dsh-superagent-prototype/blob/6337fc823ce20251c8de8085948165c81aaa701e/docs/006-deployment.md) 配置 DSH、模型服务和 Super-agent bundle。它们包含依赖要求、profile 创建、来源、授权和执行目录设置。

实际要准备的是：

1. 独立的 DSH profile 与状态目录，能成功调用模型。
2. 用户自己的成果仓库，以及对它有写权限的专用 Agent GitHub 账号；模型密钥和 GitHub 登录只在本机配置。
3. GitHub Source 指向成果仓库，Policy 只允许指定用户发起任务；本案例不配置自动合并。
4. Execution 能创建每任务独立 checkout/worktree；模型能使用文件、shell、GitHub CLI 和子 Agent 工具。
5. 将可读取的 Humanizer-zh 文件路径明确传给写作任务。不要把本机的路径直接当作他人的安装方式。

现有 profile 不要当作新安装重新创建；上游说明指出某些初始化命令会覆盖配置。官方脚本使用 tmux，本轮 Windows 通过本地 PowerShell 启动包装执行 `dsh --profile superagent --no-open`，不是原样运行 Unix 启动脚本。

DSH_HOME、GitHub CLI 配置目录和 PATH 由本机启动包装设置。官方部署说明指出执行进程使用操作者的系统权限；专用 GitHub 账号不提供文件系统隔离。首次部署可使用单独 OS 账号或隔离环境，不能把提示词边界当成安全沙箱。

## Windows 适配边界

本轮上游 checkout 有两个文件的三处小改动（不在本成果仓库内）：

- `src/deployment.ts`：绝对路径校验接受 Windows 盘符路径。
- `src/app/sessions.ts`：Windows 调度会话允许 `pwsh`；等待时长保护同时处理 `pwsh`。

这些是 Codex 协助完成的本地兼容适配，不是新增调度框架。迁移机器时应核对所用上游版本是否仍需要这些修改并重新构建；本仓库未打包补丁，也不声称原版在 Windows 上无需调整即可运行。Agent 的 shell 中曾无法直接调用 GitHub 包装命令，本轮显式使用配置好的 `gh.exe`，仍使用专用账号的登录环境。

## 发起一轮任务

1. 在面板确认服务、GitHub Source 和执行器可用，没有待处理的认证错误。
2. 让执行器可读取本仓库的任务规范与上一期文件。展示分支尚未合并时，要给固定提交/分支，不能假设文件已在 `main`。
3. 复制[任务规范中的派发正文](task-contract.md)，填写窗口、输出目录、编辑规范路径，在成果仓库 Issue 中提及 Agent 账号。
4. 在面板确认请求关联到正确任务，观察是否真的创建子 Agent。提交前检查研究、核查、写作产物，而不只看最后一条“完成”。
5. 查看 PR 的五份交付文件。保留来源与时间；确认没有修改 Marin 上游或自动发布。

本轮是在 Issue #3 追加任务，沿用原会话与 PR #4。复用机制依赖任务状态和订阅关联；不要保证一个已停止、归档或合并的任务永远继续同一 PR。

## 后续更新

原任务和 PR 仍开放时，可以在原 Issue 留言：“从上一期截点更新，只检查变化；沿用任务规范，更新文件并说明新增证据。”

若旧 PR 已合并，下一期建立新的输出目录和 PR，明确引用上一期快照。每轮仍要检索新的主线关联讨论，避免只检查固定的五个链接。

**当前没有定时触发或上游自动订阅。** 记录了来源链接，不等于在监控它们。是否设置周期运行属于下一阶段。

## 轻量验收

在任务 checkout 中运行（将目录替换为本轮目录）：

```powershell
Get-Content docs/pilot-001/tracking.json -Raw | ConvertFrom-Json | Out-Null
git diff --check
git status --short
```

然后人工或验收 Agent 抽查：关键结论能否回到原始来源；测试与合并状态是否准确；正文是否能不查缩写就读懂；子 Agent ID 是否对应真实会话；内部修正和外部反馈是否分别记录。

JSON 能解析、链接能打开和仓库 0/0 CI 都不能证明内容正确。不要为了显示质量而把这些检查称作全面测试。

## 日志与归档

- 对外阅读：`brief.md`、`teaching.md`。
- 对外查证：`evidence.md`、`tracking.json`、`run-record.md`、Issue 与 PR。
- 本机原始证据：DSH_HOME 下的 `sessions/<workspace>/<session-id>/session.v4.jsonl.zstd`。它是压缩事件日志，包含工具调用，不能当普通 Markdown 打开。
- 子 Agent 的临时研究文件本轮未提交；临时目录可能被清理。需要长期保留原始证据时，另做经过脱敏的归档，不把整个 DSH_HOME 上传。

## 发生问题时

| 现象 | 先检查什么 |
| --- | --- |
| 评论后没有任务 | 服务、账号通知、Source 和允许发起任务的账号 |
| 查资料失败 | CLI 登录环境、网络、来源是否公开；不要让模型补造内容 |
| 没有子 Agent | 运行时工具与模型支持；记录失败，不用多角色文本冒充 |
| 润色规范不可读 | 路径与执行权限；报告实际读取方式，不声称技能已安装 |
| PR 更新到错误地方 | 任务目录、分支、关联 PR 和原讨论 |
| 文稿仍难读 | 回到问题与读者要求重写，把配置移到附录；不要只再跑一次润色 |
