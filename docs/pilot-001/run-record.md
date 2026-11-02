# 运行记录：Marin 视频选题与学习工作台「试刊 001」

本文件记录这一轮实际发生了什么、谁做了什么、哪些是框架能力、哪些是本次配置、哪些只是模型产物。目标不是写得漂亮，而是让面试官能从 GitHub、子 Agent ID、原始链接和验证命令复看过程。

## 轮次与范围

- 触发：Issue #3 评论 `65a0783c8fa0b963b0318e76bc9f0fb7`，2026-11-02T05:26:10Z；沿用原 Issue #3、原执行会话与 PR #4。
- 观察窗口：`2026-10-19T05:30Z`–`2026-11-02T05:30Z`（UTC，14 天）。截点后的事件不计入。
- 范围：Marin 535B-A23B 主线；只读公开 GitHub，不用 mtracker 数字，不读 W&B 曲线，不跑 GPU，不在 Marin 写任何东西。
- 交付：`docs/pilot-001/` 下的 brief、teaching、evidence、run-record、tracking.json；更新 PR #4；在原 Issue 回复链接与三个选题。

## 子 Agent 实际调用

四个子 Agent 都由 DSH 的 `subagent` 工具实际创建；ID 是运行时返回的，不是手工编的。`subagent` 的 description 是工具展示名，ID 是会话内真实句柄。完成证据是运行时收到的子 Agent 完成消息和对应输出文件，不是“看起来像子 Agent”的角色扮演。

| 角色 | description | Agent ID | 职责 | 输出 | 完成证据 |
|---|---|---|---|---|---|
| 研究 A（主线） | Research hero run main line | `f3f263d0-1f7a-4930-b79e-0f69fbbf2ef0` | 14 天内的 hero/65K/梯度裁剪/事故主线；跨 issue 搜索；三个视频候选 | `research-run.md`（约 33.6 KB，189 行） | 运行时完成消息；文件可读；只读约束自述 |
| 研究 B（评测） | Research eval and measurement line | `db60204a-87ee-4553-87b3-1617da1634c8` | 4K/16K 打包、262K 零 token、#10711/#10709/#10618、预测与旧材料 | `research-eval.md`（约 34.5 KB，207 行） | 运行时完成消息；文件可读 |
| 独立核查 | Independently verify pilot facts | `219cbce0-205d-4fde-8202-44c39f4ac514` | 自己重读原始 comment/PR/commit，逐条核对 22 条主张 + 14 项专项检查 | `verification.md`（76 行） | 运行时完成消息；报告 20/22 成立、列出修正项 |
| 写作/润色 | Write teaching pack and humanize | `bcd4a486-00af-4b9f-97e9-d87e7bfb4d28` | 写 brief/teaching；读 Humanizer-zh 全文；两阶段改写 | `brief.md`、`teaching.md`、`humanizer-audit.md` | 完成消息与文件（见下节） |

研究 A、B 并行；核查在拿到研究记录与 evidence 草稿后开始；写作在 evidence 初稿后开始，并在核查返回后收到一次修正清单。没有递归创建子 Agent，也没有第五个“监督者”。

## 研究 A：主线

**输入**：窗口、入口（#8435、#8506、#10821、#10820、#10783、#10677，mtracker 仅线索）、证据阶段词表、只读约束。**输出**：`research-run.md`。

关键发现：

- 10-23 两次启动失败与 10-29 OOM 事故是两件事；后者是 176-task gang、attempts 1–15 失败、IMEX 残留显存、human operator 迁移 rack 399、attempt 16 从 352321 恢复。
- #10677 复现了 step-324000 的 spike，并追到单条 16K 序列/单 token/单算子；325710、353060 没有在各自权重上复现。这个“复现范围”的限定后来由核查再次确认。
- #10783 合并、`hero-gradclip-step356k` 重启、50 步 gate GO，但 clip 在 hero 上 0/50、0/197，trial 只证明无回归。
- 65K：step-372000 交接、CP1×EP64、capacity factor 1.5、qk_mult 1.66、#10820 合并、03:25Z 启动、04:03Z 六条 gate PASS。
- 发现官方“−15% tok/s”是预估值，按 16.05 vs 14.38 s 约 −10.4%；提醒不要直接引用 −15%。
- 发现 mtracker 与 GitHub 有两处不一致，建议不用其数字。

**限制**：没有读 W&B、Echo、S3；所有运行数字来自 GitHub 评论；没有 GPU 复跑。研究 A 推荐 65K 选题，理由是唯一全链路闭合。

## 研究 B：评测/测量

**输入**：同一窗口与只读约束；入口为 #10711、#10709、#10618、#7638、#8435 的 10-21~10-26 评论、#10821 的 E4。**输出**：`research-eval.md`。

关键发现：

- 10-22 的 4K/16K 打包测量是外部观察者 bethehand 的独立测量（2.0929/2.0926 vs 2.1234，约 0.031；qk_mult 1.3 约 0.028），不是 Marin 官方 eval；4K 重评只是“跑完后”的承诺，到截点无公开记录。
- 262K 短语料会产生零 token tag；组内 macro 静默变成 14 子集（约 +0.06），顶层 macro 被 0 拉低（约 −0.14）；67B A/B 为 +0.0376。
- 修复链从 65,536 cap 到 padding/mask 再到多长度 + 延迟高于训练长度的 eval；#10711 最终 CPU 3,578、CI 绿、被批准，但 open/dirty、无 GPU 演练，且 ClassicLarry 批准后 8 分钟提出未答的 1-rack partial-chunk 问题。
- #10709 是单机 65K CP4 的显存证据（+7.4 GiB vs 5.8 GiB 余量，90.49 GiB 分配失败），11 机架未测；#10618 已批准但 dirty，承诺的 65K CP4 rack 验证没有出现。
- #7638 被 stale bot 关闭；65K 截点前没有 eval 结果；HERO_FIXED_EVAL_SEQ_LENS 在 65K 会丢掉 16K 档（从 patch 读出，作者未讨论）。

**限制**：没有 W&B/Echo/S3；0.031 未复现；#10618 无自有 GPU 证据。研究 B 推荐 262K 评测选题；主执行代理在合并两份研究后，把 65K 放第一、评测陷阱放第二、梯度裁剪放第三，理由写在 brief。

## 独立核查

**输入**：`evidence-draft.md`、两份研究记录。**输出**：`verification.md`。核查者没有只读摘要，而是用 comment ID 反查作者/时间，用 pulls/reviews/commits/check-runs 核对 PR 状态，并对每个数字回到原始正文。

结果：22 条主张中约 20 条完全成立；没有发现编造、死链或“merged 说成 adopted”的升级。主要修正：

1. “13 次重试”不成立：attempts 1–15 失败、attempt 16 恢复；OOM 明确记录在 1–4、6–9、11–12。已改为“15 次失败尝试后第 16 次恢复”。
2. 325710 的“单 token/op”是过度归因：那条链属于 step-324000 batch 在 324000 权重下的复现；325710 未在 325709 权重评估，353060 未复现。已改写 E14 与选题 3。
3. “32K 以上没有评测”与 E4 矛盾：65K 打包下 Paloma loss 仍会评；正确说法是“没有长上下文**能力**基准，65K 阶段只能靠 loss 判断”。
4. 阶段标签：production gate pass 不是 GPU rehearsal；approved ≠ merged；clip 触发只属于单机 rt1/rt2。
5. #10618 的状态补充为 approved/dirty、无 rack 验证；#10785 是 issue 不是 PR；4K 重评是“跑完后”的承诺，不能算逾期。

核查报告还列出无法验证的材料（W&B 曲线、Echo wiki、S3、11 机架行为、attempts 10/14）和“旧材料被误当新进展”的检查。**这是内部一致性核查，不是外部正确性保证。**

## 写作与 Humanizer

**子 Agent**：`bcd4a486-00af-4b9f-97e9-d87e7bfb4d28`。**输入**：`writer-brief.md`、`docs/pilot-001/evidence.md`、Humanizer-zh 技能文件全文。**输出**：`brief.md`、`teaching.md`、`humanizer-audit.md`（临时目录；主执行代理做最终整合后提交到 `docs/pilot-001/`）。

执行方式：

- 技能加载：没有 DSH `skill` 条目可直接加载这个本机 Codex 技能，因此按任务允许的替代方式，用 `read` 读取 `SKILL.md` 全文（408 行），在 audit 中记录“read 全文（408 行）”，而不是声称通过 skill 工具加载。面向读者的 brief/teaching 不出现本机绝对路径。
- 两阶段：先按“零预训练背景”做教学重写（概念先行、术语首现解释、一个贯穿比喻），再做 Humanizer 润色。Humanizer 只改表达，不加事实、不改数字/日期/归因/否定/完成状态。
- 修正：第一轮来自核查报告（325710 归因、15 次失败尝试、“能力基准”措辞、−15%、clip 触发范围、10-29 与 spike 的关系、4K 重评条件、#10618 状态）；第二轮来自任务所有者对 brief 的可读性验收（2026-11-02T05:41:32Z，明确要求去掉未解释的缩写、把单机架与主训练观察分开、改标题）。写作 Agent 按两轮清单改写，自报最终 1,399 / 2,196 个中文字。
- 主执行代理的最终整合：第一轮可读性反馈后，T 的第二版 brief 仍残留“混合专家/算子/0/197/机架 399”等不适合零基础读者的表述，主执行代理用同一批事实重写了 brief；第二轮 Codex 事实/记录验收（2026-11-02T05:48:01Z）又要求修正 token/存档定义、专家丢弃与 qk_mult 措辞、切换后检查与“未触发阈值”的说法、未来日期加“预计”，并把 CP/EP、capacity factor、qk_mult、详细排练数字移到可选技术附录。主执行代理据此重排 teaching 主线与附录。最终 brief 1,375 字、teaching 2,509 字；T 的 audit 记录的是它自己的 1,399 / 2,196，最终以 `docs/pilot-001/` 为准。
- 字数：brief 1,375（目标 800–1,400）；teaching 2,509（目标 1,800–2,800）。两轮可读性返工说明本轮内部的“零背景读者”验收还不够好，这是过程事实，不粉饰。

**Humanizer 审计摘要**（完整 audit 在临时目录，以下为可核对的要点）：

- 加载：`skill` 工具没有 Humanizer-zh 条目，改用 `read` 读 `SKILL.md` 全文 408 行；记录为“read 全文（408 行）”，不声称通过 skill 工具加载。
- 两阶段：阶段 A 按零基础读者重写教学；阶段 B 按 Humanizer 的优先级做表达润色（保留信息/确定程度 → 编辑范围 → 作者声音 → 具体表达问题）。
- 31 条模式只当检查线索，不当词表；没有为了“像人写的”删数字、归因、否定或限定。
- 保留检查：数字（372,000、95.3%、16.05、−0.02009、4.058e-4、127.8、0/197 等）、日期、归因（作者/代理 vs 外部观察者）、范围（65K 仍在跑）、否定与完成状态（已批准≠已合并、未开始）均保留；没有新增事实、经历、引文或来源。
- 最终计数：brief 1,375、teaching 2,509（主执行代理整合后）；Humanizer 是编辑步骤，不是事实核查，也不保证通过 AI 检测器。

**限制**：Humanizer 是编辑步骤，不是事实核查，也不保证通过任何 AI 检测器；audit 只证明读了规范并做了保留检查，不证明文字“像人”。

## 实际验证（主执行代理）

- `docs/pilot-001/tracking.json` 用 PowerShell `ConvertFrom-Json` 解析成功。
- `brief.md`、`teaching.md` 用 `[regex]::Matches($t,'[\u4e00-\u9fff]')` 计中文字数：最终 **1,375 / 2,509**，在 800–1,400 与 1,800–2,800 区间。
- `evidence.md` 的每个关键结论都有 issue/PR 永久链接；核查者按 comment ID 解析了 24 个评论链接，作者与时间匹配。
- `git diff --check` 无空白错误；`git status` 只包含本轮 docs/pilot-001 的改动。
- PR #4 的 0/0 CI 只表示仓库没有配置检查，**不等于质量测试通过**；质量依据是上述人工/子 Agent 核查与字数/JSON 检查。
- 没有运行 Marin 代码、GPU、训练或下载权重；没有向 Marin 发评论/Issue/PR。

## 人工介入（内部触发 ID 与 GitHub 评论 ID 分开）

- **启动评论**：GitHub `https://github.com/AdaChenScience/superagent-playground/issues/3#issuecomment-6569792751`；内部 trigger ID `65a0783c8fa0b963b0318e76bc9f0fb7`；2026-11-02T05:26:10Z。这是任务派发，不是“人工修改指导”。
- **第一次验收指导**：GitHub `https://github.com/AdaChenScience/superagent-playground/issues/3#issuecomment-6570006606`；内部 trigger ID `34296a4fa48177a0da77e5f1666661ee`；2026-11-02T05:41:32Z。指出 brief 仍堆有未解释缩写，要求零背景可读、单机架与主训练观察分开、改标题。
- **第二次验收指导**：GitHub `https://github.com/AdaChenScience/superagent-playground/issues/3#issuecomment-6570091548`；内部 trigger ID `e540395067c6fc53de5a02ebfe2f9954`；2026-11-02T05:48:01Z。要求修正 token/存档定义、专家丢弃与 qk_mult 措辞、切换后检查与“未触发阈值”的说法、未来日期加“预计”，并把配置细节移到可选技术附录。
- **归属**：两次验收反馈由 Codex 代表用户验收发出，不是用户亲自写稿；**终端用户额外决策 0 次**。因此本轮记录为 **2 次 Codex 验收指导、0 次终端用户决策**，不是“人工介入 0”，也不是“用户改了 2 次稿”。
- **计数定义**：只统计本轮启动后、来自任务所有者/其验收代理的额外修改指令；启动派发与子 Agent 之间的正常返工不计入。内部 trigger ID 用于宿主消息路由；GitHub 评论 ID 用于永久链接，两者不是同一个编号体系，不能混写。
- **处理**：没有向用户反问；按两次验收指导修订，主执行代理做最终整合，teaching 主线与可选技术附录分开。两次可读性返工说明内部验收仍需改进，run-record 保留这一事实。

## 框架原生能力 / 本次配置 / 模型产物

- **DSH 原生**：`subagent` 工具创建隔离上下文的子 Agent 并返回 ID；`send_message` 传递修正；运行时投递完成消息；`read`/`write`/`pwsh`/`grep` 等文件与命令工具；模型推理。DSH 本身不自带事实核查器、Humanizer 或“质量门”。
- **Super-agent 层（不是 DSH 原生）**：`orchestrator_report`/`orchestrator_ack`、ticket/PR 绑定、trigger 路由与订阅、向原 Issue 交付。这些是本部署的 Super-agent 编排层能力，不能写成“DSH 原生”。
- **本次配置**：Windows + PowerShell；`gh.exe` 以 mathlover77 只读访问；固定 14 天窗口；四个子 Agent；本机 Humanizer-zh 技能；PR #4；临时工作目录用于子 Agent 输出、主执行代理统一整合。
- **模型产物**：研究笔记、核查报告、brief/teaching/evidence/run-record 的文字与判断。它们可能出错；本轮的核查是内部复核，不是第三方复现。最终采用与否需要人类 review。

## 可复现线索

- 研究入口与永久链接都在 `evidence.md`；核查报告在 `verification.md`（本轮为临时产物，不提交仓库；其结论已写入 `evidence.md` 的“内部核查与修正”）。
- 子 Agent ID 是本会话运行时句柄，跨会话不可复现；可复现的是它们的输入（窗口、URL、约束）和输出（提交到 PR 的文档）。
- 复看顺序：`brief.md` → `teaching.md` → `evidence.md` → `tracking.json` → 本文件；对外部来源按 evidence 的链接点开。
- 本轮没有把临时研究文件提交到仓库，避免把过程稿与交付稿混在一起。

## 完成证据

- PR #4：本轮提交后更新；具体 commit 与文件列表见 PR 页面。
- Issue #3：本轮完成后在原 Issue 回复 PR 链接、三个选题、推荐题与“本轮无需决策”。
- 子 Agent 完成消息：研究 A/B、核查、写作各一条；对应输出文件在临时目录，内容已整合进交付文档。写作 Agent 自报 1,399 / 2,196；主执行代理两轮整合后为 1,375 / 2,509，差异来自可读性/结构改写，不改变事实。
- 本文件即过程记录；没有把“耗时”“效率提升”“无遗漏”写成无法核实的结论。
