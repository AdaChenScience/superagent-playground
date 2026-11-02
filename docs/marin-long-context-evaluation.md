# Marin 535B-A23B 长上下文评测：首轮基线记录

> 观察时间：2026-11-02T04:42–04:55Z（北京时间 12:42–12:55）。只读查阅 marin-community/marin 的公开 Issue、PR、评论与源码链接；未运行训练、未下载权重。看板 `mtracker.oa.dev/hero-run-535b` 仅作线索（本次可访问），事实均回到 GitHub 原始记录核对。

## 一个虚构的小例子：更多前文为什么可能降低 loss

假设要补全“这个国家的首都是___”。短输入只看到这一句；长输入还包含前一句“我们正在介绍法国”。有了前文，同一个模型更容易给“巴黎”较高概率，因而在这一位置上的 loss 更低。

这是**虚构的解释性例子，不是本次实验数据**。它说明：同一批参数，前文不同，同一位置的预测也会不同。后文 4K 与 16K 评测的差异，一部分来自这种“评测条件”变化，而不是模型突然变强。

## 术语

- **token**：模型处理的最小文本单位，可能是词、词的一部分或标点。
- **context（上下文）**：模型一次能看到的 token 序列；上限即上下文窗口。
- **checkpoint**：训练到某一步保存的参数快照。
- **loss**：平均每个 token 的负对数似然，越低表示越能预测下一个 token。
- **packing**：把多篇文档首尾相接填满定长序列，减少浪费。
- **padding / masking**：padding 用占位符把短序列补到定长；masking 指定哪些位置不参与注意力或 loss。
- **macro**：先算子数据集分数再等权平均；某子集没数据会扭曲结果。
- **dropless**：MoE（混合专家）路由不因专家容量上限丢弃 token；相对的是会丢 token 的 dropping。
- **CP**：context parallelism，把一条长序列切到多张 GPU 上并行计算。
- **qk_mult**：长上下文外推时对注意力中 query-key 缩放做的修正系数。

## 时间线：5 个关键事件

### 1. 4K 到 16K（2026-10-01～10-14）

Hero 原计划 4K→8K（约 50%）→65K（约 95%）（[#8435 正文](https://github.com/marin-community/marin/issues/8435)）。10-01 mcwitt-agent 的单机架测量（[#8435 评论](https://github.com/marin-community/marin/issues/8435#issuecomment-5922369256)，2026-10-01T00:40:47Z；完整记录 [#9615](https://github.com/marin-community/marin/issues/9615)，10-14 关闭）显示：固定每步 token 数、从 step-180000 恢复、3 个种子各 100 步，8K 吞吐比 4K 低 1.6%，16K 低 3.1%；丢 token 比例在 8K、16K 分别约是 4K 的 4.5 倍和 16 倍。ClassicLarry 据此主张直接上 16K（[#9615 评论](https://github.com/marin-community/marin/issues/9615#issuecomment-5923072768)，2026-10-01T01:46:53Z）。16K 切换 [#10101](https://github.com/marin-community/marin/pull/10101) 于 10-14T02:06:04Z 合并，同日据 #9615 评论确认部署（[作者报告](https://github.com/marin-community/marin/issues/9615#issuecomment-6175246369)）。这是作者的单机架、短窗口测量，11 机架长期行为仍是外推。（观察于 2026-11-02T04:44Z）

### 2. 评测在哪个长度打包（2026-10-22）

10-22 独立预测者 bethehand 从同一 checkpoint 恢复、各跑少量步骤后对比（[#8435 评论](https://github.com/marin-community/marin/issues/8435#issuecomment-6336070902)，2026-10-22T08:14:54Z）：16K 评测的 Paloma macro 比权重几乎相同的 4K 评测低约 0.031（qk_mult 保持 1.3 时约 0.028）。因此 4K 时代的预测直接对 16K 最终值评分，会天然偏高约 0.03。ClassicLarry 回应跑完后用 4K 重评所有 checkpoint（[#8435 评论](https://github.com/marin-community/marin/issues/8435#issuecomment-6336416457)，2026-10-22T08:33:56Z），但未给预期结论；截至观察时未见重评记录。这是独立观察者基于 W&B 的测量，不是 Marin 官方评测。（观察于 2026-11-02T04:45Z）

### 3. 262K 短数据集与多长度评测（2026-10-25～26）

10-25 bethehand 预警（[#8435 评论](https://github.com/marin-community/marin/issues/8435#issuecomment-6401174668)，2026-10-25T18:01:18Z）：若 262K 阶段按训练长度打包，Paloma 的 PTB 与 WikiText-103 验证集都不足一条 262,144-token 序列，两个子集没有数据、loss 记为 0.0；Paloma 组 macro 跳过它们，`eval_dropless/paloma/macro_loss` 会变成其余 14 个子集的平均，偏高约 0.06（约 3%，可能触发 `eval_regressed`）；顶层 macro 又把 0 算进去，约低 0.14。10-26 dlwh 在源码中确认（[#8435 评论](https://github.com/marin-community/marin/issues/8435#issuecomment-6408314282)，2026-10-26T04:34:41Z），先提出“零 token 标签不计入整体 macro + 评测打包上限 65,536 token”（[#8435 评论](https://github.com/marin-community/marin/issues/8435#issuecomment-6408421426)，04:43:51Z）；约一小时后改为分长度评测：每次同时评 4K、当前训练长度、65K、262K，短语料用 padding+masking 保留最后一段（[评论](https://github.com/marin-community/marin/issues/8435#issuecomment-6409543752)，06:15:37Z）；随后又收敛为只评训练长度及以下，65K/262K 等训练到达再加（[评论](https://github.com/marin-community/marin/issues/8435#issuecomment-6410181205)，07:00:00Z）。截至观察时，未见“65K 封顶”方案进入运行的记录。（观察于 2026-11-02T04:46Z）

### 4. 设备组显存问题与 #10711 的修订（2026-10-26～30）

[#10709](https://github.com/marin-community/marin/pull/10709) 说明了长上下文评测的组织方式本身会带来显存问题。评测需要按上下文长度把 GPU 临时分成新的通信分组（mesh，设备分组），这些分组训练时没用过；创建它们会额外占用显存，评测结束回到训练时，训练要用的显存就不够。单机架 65K、CP 4 路的评测里，一次评测让每块 GPU 多占约 7.4 GiB，而当时只剩约 5.8 GiB 余量，下一步训练因此分配失败。方案是复用训练已有的 GPU 分组；据 PR 作者报告，改后同一单机架 65K、CP 4 路能连续跑两次 dropless 评测并继续训练，Paloma macro 约 2.05。只验证了单机架，11 机架未测。

[#10711](https://github.com/marin-community/marin/pull/10711) 初版把 65K/262K 评测放在新分组上，自动评审指出会复现显存问题（[评论](https://github.com/marin-community/marin/pull/10711#issuecomment-6409591208)，2026-10-26T06:18:56Z）；作者改用 [#10709](https://github.com/marin-community/marin/pull/10709) 的设备组复用，并把评测长度限制在训练长度以内（[评论](https://github.com/marin-community/marin/pull/10711#issuecomment-6410180643)，06:59:58Z）。最终设计是每个长度一套指标（如 `eval_dropless/seq_len_4096/...`），无前缀指标对应训练长度；短语料 padding+masking；整体 macro 排除零 token 标签。作者报告 3,578 个受影响的 CPU 用例通过、CI 绿；ClassicLarry 于 10-30 批准（[review](https://github.com/marin-community/marin/pull/10711#pullrequestreview-5716334825)），但追问保留 partial chunk（最后一段未填满的序列）是否改变 4K/16K 数据（[评论](https://github.com/marin-community/marin/pull/10711#issuecomment-6529741854)，2026-10-30T17:54:31Z）。截至观察：#10711 未合并，观察时 GitHub 标记 `mergeable=false`/`dirty`（瞬时状态，当时与 main 冲突），PR 正文称尚无 GPU 演练；#10709 也未合并（`blocked`，无评审）。（观察于 2026-11-02T04:47Z）

### 5. 65K 切换与后续评测（2026-11-01～02）

[#10821](https://github.com/marin-community/marin/issues/10821) 记录 16K→65K：11-01 确定在 step-372,000（约 95.3%）交接，10-31 选定 CP1×EP64（1 路上下文并行 × 64 路专家并行）、capacity factor（专家容量系数）1.5、qk_mult 1.66，而非更快的 CP4：CP1 每步比 CP4 慢 4.7%，但丢 token 少 2.5 倍，且 CP 4 路的 dropless 评测在 #10709 合并前会让下一步训练失败。部署 [#10820](https://github.com/marin-community/marin/pull/10820) 于 11-02T02:55:41Z 合并；03:25Z 启动 65K 子运行，04:03Z 运行代理自报通过全部 gate（放行检查）：中位 16.05 s/step、drop 约 3.5e-4、50 步 loss 比父运行低 0.020（[#10821 评论](https://github.com/marin-community/marin/issues/10821#issuecomment-6568749068)，2026-11-02T04:03:06Z）。这是作者报告，尚无独立复核；预计 11-05T15:00Z 左右跑完。

后续评测仍是未决点：评测按 65K 打包，Paloma macro 可能阶跃并误触发 `eval_regressed`（[#10821 的 E4](https://github.com/marin-community/marin/issues/10821)）；首次 65K eval 约 11-02T17:20Z（step 375,000），永久 checkpoint 约 11-03T07:00Z（step 378,000）；32K 以上仍无长上下文能力评测（RULER issue [#7638](https://github.com/marin-community/marin/issues/7638) 已因不活跃被自动关闭）。分长度评测要等 [#10711](https://github.com/marin-community/marin/pull/10711)，计划 65K 之后、262K 之前落地；262K 还依赖 [#10618](https://github.com/marin-community/marin/pull/10618) 等，截至观察时均未合并。（观察于 2026-11-02T04:48Z）

## 当前状态（截至 2026-11-02T04:48Z）

下表把“谁说的”和“做到哪一步”分开；“作者报告”不等于独立复核。

| 事项 | 方案/提出 | 代码 | CPU 测试 | GPU 演练 | 合并 | 主训练采用 |
|---|---|---|---|---|---|---|
| 16K 切换 | [#9615](https://github.com/marin-community/marin/issues/9615)，10-01 | [#10101](https://github.com/marin-community/marin/pull/10101) 配置改动 | 未专门报告 | 单机架 3 个种子×100 步 | 10-14 合并 | 10-14 起使用 |
| 262K 短数据集/零 token | 10-25 外部观察，10-26 确认 | [#10711](https://github.com/marin-community/marin/pull/10711) | 3,578 用例（作者报告） | 无 | 未合并 | 未采用 |
| 多长度评测 | 10-26 | [#10711](https://github.com/marin-community/marin/pull/10711) | 同上，CI 绿 | 无 | 未合并 | 未采用 |
| 设备组复用 | [#10709](https://github.com/marin-community/marin/pull/10709) | [#10709](https://github.com/marin-community/marin/pull/10709) | CI 绿 | 单机架 65K、CP 4 路 | 未合并 | 65K 采用 CP 1 路，未用 4 路 |
| 65K 切换 | [#10821](https://github.com/marin-community/marin/issues/10821)，10-31/11-01 | [#10820](https://github.com/marin-community/marin/pull/10820) | 未专门报告 | 单机架 C1/C2（作者报告） | 11-02T02:55 合并 | 11-02T03:25 起 65K |
| 262K 阶段 | 计划中 | [#10618](https://github.com/marin-community/marin/pull/10618) 等 | — | — | 未合并 | 截至观察时未见启动记录 |

## 下一次值得追踪的 3～5 个问题

1. [#10711](https://github.com/marin-community/marin/pull/10711) 何时合并（观察时与 main 冲突）、ClassicLarry 要求的单机架 4K/16K partial-chunk 对比结果，以及 GPU 显存/耗时演练是否通过。
2. 65K 首次 eval（step 375,000，约 11-02T17:20Z）的 Paloma macro 阶跃是否触发 `eval_regressed`，团队如何解释其与旧预测的可比性。
3. [#10709](https://github.com/marin-community/marin/pull/10709) 的 11 机架布局和 CP4 65K 评测是否验证；262K 阶段若用 CP4，是否仍保留全部 16 个 Paloma 子集。
4. 跑完后的 4K 重评是否给出可比的固定长度序列，从而校正约 0.03 的打包偏差和各方预测。
5. 65K 后 1.8 万步的 drop/吞吐是否与单机架外推一致；step-378,000 checkpoint 与 262K 交接是否按计划。

## 资料限制

- 只读公开 GitHub；W&B 链接与 mtracker 看板未作为事实来源。
- 未运行 Marin 代码或 GPU 任务；[#10711](https://github.com/marin-community/marin/pull/10711) 与 [#10709](https://github.com/marin-community/marin/pull/10709) 的代码结论来自 PR 作者与自动评审，未独立复跑。
- `mergeable`/`dirty` 等字段是 2026-11-02T04:48Z 的瞬时观测值，不是长期状态。
- 状态是 2026-11-02T04:48Z 的快照，后续以 tracking JSON 为基线。
