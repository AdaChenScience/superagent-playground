# Marin 535B-A23B 长上下文评测：首轮基线记录

> 观察时间：2026-11-02T04:42–04:52Z（北京时间 12:42–12:52）。只读查阅 marin-community/marin 的公开 Issue、PR、评论与源码链接；未运行训练、未下载权重。看板 `mtracker.oa.dev/hero-run-535b` 仅作线索（本次可访问），事实均回到 GitHub 原始记录核对。

## 一个虚构的小例子：更多前文为什么可能降低 loss

假设要补全“法国的首都是___”。如果前文刚说“我们聊聊欧洲面积最大的国家，它的首都是___”，读者知道话题是法国，“巴黎”几乎被锁定；如果只看到“首都是___”，而前文在谈别的国家，答案就会分散到多个城市。前文越明确，正确 token 的概率越高，loss 越低。

这是**虚构的解释性例子，不是本次实验数据**。它想说明：同一批参数，前文长短不同，测出的 loss 也会不同。后文 4K 与 16K 评测的差异，一部分正来自这种“评测条件”变化，而不是模型突然变强。

## 术语

- **token**：模型处理的最小文本单位，可能是词、词的一部分或标点。
- **context（上下文）**：模型一次能看到的 token 序列；上限即上下文窗口。
- **checkpoint**：训练到某一步保存的参数快照。
- **loss**：平均每个 token 的负对数似然，越低表示越能预测下一个 token。
- **packing**：把多篇文档首尾相接填满定长序列，减少浪费。
- **padding / masking**：padding 用占位符把短序列补到定长；masking 指定哪些位置不参与注意力或 loss。
- **macro**：先算子数据集分数再等权平均；某子集没数据会扭曲结果。
- **dropless**：MoE 路由不因专家容量上限丢弃 token；相对的是会丢 token 的 dropping。
- **CP**：context parallelism，把一条长序列切到多张 GPU 上并行计算。
- **qk_mult**：长上下文外推时对 attention 的 query-key 缩放做的修正系数。

## 时间线

### 1. 4K 与 16K 对比（2026-10-01）

Hero 原计划 4K→8K（约 50%）→65K（约 95%）（[#8435 正文](https://github.com/marin-community/marin/issues/8435)）。10-01 mcwitt-agent 贴出单机架测量（[#8435 评论](https://github.com/marin-community/marin/issues/8435#issuecomment-5922369256)，2026-10-01T00:40:47Z；完整记录 [#9615](https://github.com/marin-community/marin/issues/9615)，10-14 关闭）：固定每步 token 数、从 step-180000 恢复、3 个种子各 100 步，8K 吞吐比 4K 低 1.6%，16K 低 3.1%；丢 token 比例在 8K、16K 分别是 4K 的 4.5 倍和 16 倍。ClassicLarry 据此主张直接上 16K，因为速度接近 4K（[#9615 评论](https://github.com/marin-community/marin/issues/9615#issuecomment-5923072768)，2026-10-01T01:46:53Z）。16K 切换 [#10101](https://github.com/marin-community/marin/pull/10101) 于 10-14T02:06:04Z 合并，同日 18:18:55Z 据 #9615 评论确认已部署（[作者报告](https://github.com/marin-community/marin/issues/9615#issuecomment-6175246369)）。这些是作者的单机架、短窗口测量；11 机架长期行为仍是外推（[#9615 注意事项](https://github.com/marin-community/marin/issues/9615)）。（观察于 2026-11-02T04:44Z）

### 2. 评测在哪个长度打包（2026-10-22）

10-22 独立预测者 bethehand 用同一 checkpoint 对比（[#8435 评论](https://github.com/marin-community/marin/issues/8435#issuecomment-6336070902)，2026-10-22T08:14:54Z）：step-252,000 的 16K 评测（qk_mult 1.48）得到的 Paloma macro 比同权重的 4K 评测低约 0.031；qk_mult 保持 1.3 时约 0.028。因此用 4K 时代做的预测直接对 16K 最终值评分，会天然偏高约 0.03。ClassicLarry 回应会在跑完后用 4K 重评所有 checkpoint（[#8435 评论](https://github.com/marin-community/marin/issues/8435#issuecomment-6336416457)，2026-10-22T08:33:56Z），但没有给出预期结论。这是独立观察者基于 W&B 的测量，不是 Marin 官方评测；截至观察时，未见 4K 重评的实施记录。（观察于 2026-11-02T04:45Z）

### 3. 262K 的短数据集问题（2026-10-25）

bethehand 预警（[#8435 评论](https://github.com/marin-community/marin/issues/8435#issuecomment-6401174668)，2026-10-25T18:01:18Z）：若 262K 阶段仍按训练长度打包，Paloma 的 PTB 与 WikiText-103 验证集都不足一条 262,144-token 序列，两个子集没有评测数据，loss 记为 0.0；Paloma 组 macro 跳过它们，`eval_dropless/paloma/macro_loss` 会悄悄变成其余 14 个子集的平均，偏高约 0.06（约 3%，足以触发 `eval_regressed`）；顶层 macro 又把 0 算进去，约低 0.14。10-26 dlwh 在源码中确认该行为（[#8435 评论](https://github.com/marin-community/marin/issues/8435#issuecomment-6408314282)，2026-10-26T04:34:41Z）。（观察于 2026-11-02T04:45Z）

### 4. 从“65K 封顶”到多长度评测（2026-10-26）

dlwh 先提出两个小修：零 token 标签不计入整体 macro；Hero 评测打包上限设 65,536 token，使 262K 阶段仍保留 16 个 Paloma 子集，并在 [#10711](https://github.com/marin-community/marin/pull/10711) 实现（[#8435 评论](https://github.com/marin-community/marin/issues/8435#issuecomment-6408421426)，2026-10-26T04:43:51Z）。约一小时后他改为做多长度评测（[05:43:42Z](https://github.com/marin-community/marin/issues/8435#issuecomment-6409129788)），06:15:37Z 把 [#10711](https://github.com/marin-community/marin/pull/10711) 更新为每次评测同时评 4K、当前训练长度、65K、262K，分指标名，短语料用 padding+masking 保留最后一段，撤销 65K 上限（[评论](https://github.com/marin-community/marin/issues/8435#issuecomment-6409543752)）；07:00:00Z 又收敛为只评训练长度及以下，65K/262K 等训练到达再加（[评论](https://github.com/marin-community/marin/issues/8435#issuecomment-6410181205)）。也就是说，“65K 封顶”只是一个约一个半小时的中间设计；截至观察时，未见它进入任何运行的记录。（观察于 2026-11-02T04:46Z）

### 5. 设备组与显存：通俗说明（2026-10-26）

[#10709](https://github.com/marin-community/marin/pull/10709) 说明了长上下文评测的组织方式本身会带来显存问题。评测需要按上下文长度把 GPU 临时分成新的通信分组（mesh，设备分组），这些分组训练时没有用过；创建它们会额外占用显存，评测结束回到训练时，训练要用的显存就不够了。单机架 65K 的 CP4 评测里，一次评测让每块 GPU 多占约 7.4 GiB，而当时只剩约 5.8 GiB 余量，下一步训练因此分配失败。方案是让评测复用训练已有的 GPU 分组，不再新建；据 PR 作者报告，改后同一单机架 65K CP4 能连续跑两次 dropless 评测并继续训练，Paloma macro 约 2.05。只验证了单机架，11 机架布局未测。（观察于 2026-11-02T04:46Z）

### 6. #10711 的修订与卡点

[#10711](https://github.com/marin-community/marin/pull/10711) 初版把 65K/262K 评测放在新 mesh 上，自动评审指出会复现显存问题（[#10711 评论](https://github.com/marin-community/marin/pull/10711#issuecomment-6409591208)，2026-10-26T06:18:56Z）；作者改用 [#10709](https://github.com/marin-community/marin/pull/10709) 的设备组复用，并把评测长度限制在训练长度以内（[06:59:58Z](https://github.com/marin-community/marin/pull/10711#issuecomment-6410180643)）。最终 head `eea42ac` 的设计是：每个长度一套指标（如 `eval_dropless/seq_len_4096/...`），无前缀指标仍对应训练长度；短语料 padding+masking；整体 macro 排除零 token 标签。作者报告受影响的 3,578 个 CPU 用例通过、CI 绿；ClassicLarry 于 10-30 批准（[review](https://github.com/marin-community/marin/pull/10711#pullrequestreview-5716334825)），但追问保留 partial chunk（最后一段未填满的序列）是否改变 4K/16K 数据，建议先做单机架对比（[#10711 评论](https://github.com/marin-community/marin/pull/10711#issuecomment-6529741854)，2026-10-30T17:54:31Z）。截至观察：[#10711](https://github.com/marin-community/marin/pull/10711) 未合并，观察时 GitHub 标记 `mergeable=false`/`dirty`（当时的瞬时状态，表示与 main 冲突），PR 正文称尚未做 GPU 演练；观察时 [#10709](https://github.com/marin-community/marin/pull/10709) 也未合并（`mergeable_state=blocked`，无评审）。（观察于 2026-11-02T04:47Z）

### 7. 65K 正式切换（2026-11-01～11-02）

[#10821](https://github.com/marin-community/marin/issues/10821) 记录 16K→65K 计划与执行：11-01 确定交接步为 step-372,000（390,251 步的 95.3%），10-31 选定 CP1×EP64、capacity factor（专家容量系数）1.5、qk_mult 1.66，而非更快的 CP4：CP1 每步比 CP4 慢 4.7%，但丢 token 少 2.5 倍，且 CP4 的 dropless 评测在 [#10709](https://github.com/marin-community/marin/pull/10709) 合并前会让下一步训练失败。部署 [#10820](https://github.com/marin-community/marin/pull/10820) 于 11-02T02:55:41Z 合并；03:25Z 启动 65K 子运行，04:03Z 运行代理自报通过全部 gate（放行检查）：中位步时 16.05 s、drop 约 3.5e-4、50 步 loss 比父运行低 0.020（[#10821 评论](https://github.com/marin-community/marin/issues/10821#issuecomment-6568749068)，2026-11-02T04:03:06Z）。这是作者报告，尚无独立复核；运行代理预计 11-05T15:00Z 左右跑完。（观察于 2026-11-02T04:47Z）

### 8. 后续评测安排与风险

65K 阶段有三个未决点。其一，评测仍按训练长度 65K 打包，Paloma macro 会在切换处阶跃，`eval_regressed`（比父运行高 2%）可能误报，团队需事先约定解读方式（[#10821 的 E4](https://github.com/marin-community/marin/issues/10821)）。其二，首次 65K eval 在 step 375,000（约 11-02T17:20Z），永久 checkpoint 在 378,000（约 11-03T07:00Z）。其三，长上下文能力本身没有评测：RULER 接线 issue [#7638](https://github.com/marin-community/marin/issues/7638) 于 10-24 因长期无活动被自动关闭，所以 65K 阶段目前主要看 loss（[#10821 正文](https://github.com/marin-community/marin/issues/10821)）。分长度的固定评测序列要等 [#10711](https://github.com/marin-community/marin/pull/10711)，计划在 65K 切换之后、262K 之前落地；262K 阶段还依赖 CP launcher [#10618](https://github.com/marin-community/marin/pull/10618)、[#10709](https://github.com/marin-community/marin/pull/10709) 和 [#10711](https://github.com/marin-community/marin/pull/10711)，截至观察时均未合并。（观察于 2026-11-02T04:48Z）

## 当前状态（截至 2026-11-02T04:48Z）

下表把“谁说的”和“做到哪一步”分开；“作者报告”不等于独立复核。

| 事项 | 方案/提出 | 代码 | CPU 测试 | GPU 演练 | 合并 | 主训练采用 |
|---|---|---|---|---|---|---|
| 16K 切换 | [#9615](https://github.com/marin-community/marin/issues/9615)，10-01 | [#10101](https://github.com/marin-community/marin/pull/10101) 配置改动 | 未专门报告 | 单机架 3 seed×100 步 | 10-14 合并 | 10-14 起使用 |
| 262K 短数据集/零 token | 10-25 外部观察，10-26 确认 | [#10711](https://github.com/marin-community/marin/pull/10711) | 3,578 用例（作者报告） | 无 | 未合并 | 未采用 |
| 多长度评测 | 10-26 | [#10711](https://github.com/marin-community/marin/pull/10711) | 同上，CI 绿 | 无 | 未合并 | 未采用 |
| 设备组复用 | [#10709](https://github.com/marin-community/marin/pull/10709) | [#10709](https://github.com/marin-community/marin/pull/10709) | CI 绿 | 单机架 65K CP4 | 未合并 | 65K 用 CP1，未用 CP4 |
| 65K 切换 | [#10821](https://github.com/marin-community/marin/issues/10821)，10-31/11-01 | [#10820](https://github.com/marin-community/marin/pull/10820) | 未专门报告 | 单机架 C1/C2（作者报告） | 11-02T02:55 合并 | 11-02T03:25 起 65K |
| 262K 阶段 | 计划中 | [#10618](https://github.com/marin-community/marin/pull/10618) 等 | — | — | 未合并 | 截至观察时未见启动记录 |

## 下一次值得追踪的 3～5 个问题

1. [#10711](https://github.com/marin-community/marin/pull/10711) 何时合并（观察时与 main 冲突）、ClassicLarry 要求的单机架 4K/16K partial-chunk 对比结果，以及 GPU 显存/耗时演练是否通过。
2. 65K 首次 eval（step 375,000，约 11-02T17:20Z）的 Paloma macro 阶跃是否触发 `eval_regressed`，团队如何解释其与旧预测的可比性。
3. [#10709](https://github.com/marin-community/marin/pull/10709) 的 11 机架布局和 CP4 65K 评测是否验证；262K 阶段若用 CP4，是否仍保留全部 16 个 Paloma 子集。
4. 跑完后的 4K 重评是否给出可比的固定长度序列，从而校正约 0.03 的打包偏差和各方预测。
5. 65K 后 1.8 万步的 drop/吞吐是否与单机架外推一致；step-378,000 checkpoint 与 262K 交接是否按计划。

## 资料限制

- 只读公开 GitHub；W&B 链接与 mtracker 看板未作为事实来源。看板本次可访问，但文中数字均来自 Issue/PR 正文或评论。
- 未运行 Marin 代码或 GPU 任务；[#10711](https://github.com/marin-community/marin/pull/10711) 与 [#10709](https://github.com/marin-community/marin/pull/10709) 的代码结论来自 PR 作者与自动评审，未独立复跑。
- `mergeable`/`dirty` 等 GitHub 字段是 2026-11-02T04:48Z 的瞬时观测值，不是长期状态。
- 状态是 2026-11-02T04:48Z 的快照，后续以 tracking JSON 为基线。
