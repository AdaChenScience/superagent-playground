# 证据与核查记录：Marin 535B-A23B 试刊 001

本文件回答两个问题：每个结论能不能回到原始来源，以及它走到了哪一步。观察窗口为 **2026-10-19T05:30Z 至 2026-11-02T05:30Z（UTC，14 天）**；截至截点，主线最后一条公开活动是 2026-11-02T04:03Z 的 65K 门禁评论。只读访问 GitHub；W&B 曲线、Echo 内部事故页、S3 checkpoint 与训练日志未读取；`mtracker.oa.dev/hero-run-535b` 只作线索，未用其数字，原因是它由 LLM 生成且有两处与 GitHub 不一致。

**证据阶段**：提议 → 作者报告 → 代码完成 → CPU 测试 → GPU 演练（单机或小规模）→ 合并 → 主训练采用。**归因**：Marin 作者/代理、独立观察者、自动评审、运维日志。未单列“独立验证”的事件，就是没有独立复核；本文件的核查是内部复核，不是外部复现，也不构成正确性保证。

需要特别分清：本窗口内几乎所有训练/运行数字都来自 Marin 自己的 bot 或 agent；唯一的外部独立测量是 bethehand 的 4K/16K 打包差（以及两条独立预测），它们不是 Marin 官方 eval。自动评审（loom、Codex）是机器人，不是人类独立验证；“approve”也不等于 merge。

## 事实表

| # | 事实 | 时间（UTC） | 永久来源 | 阶段 | 归因 | 限制 |
|---|---|---|---|---|---|---|
| E1 | 同一 step-252,000 checkpoint，16K 打包的 Paloma macro 为 2.0929/2.0926，4K 打包为 2.1234，低约 0.031；qk_mult 保持 1.3 时约 0.028。4K 口径的预测直接对 16K 终值评分，可能系统性偏高约 0.03。 | 2026-10-22T08:14:54Z | [#8435 c-6336070902](https://github.com/marin-community/marin/issues/8435#issuecomment-6336070902) | 独立观察（GPU 实测） | 外部观察者 bethehand | 单 checkpoint、单次测量；不是 Marin 官方 eval，未复现；offset 可能漂移 |
| E2 | Marin 回应会在 run 结束后用 4K 重评所有 checkpoint，但声明 4k→16k→65k 的模型在 4K 上可能与全程 4K 的模型不同。 | 2026-10-22T08:33:56Z | [#8435 c-6336416457](https://github.com/marin-community/marin/issues/8435#issuecomment-6336416457) | 团队承诺（跑完后） | Marin 作者 | 到 05:30Z 无公开记录；承诺条件是 run 结束，因此现在还不能算逾期 |
| E3 | 两条独立预测线：mindscanlabs A=1.979、B=1.935（90% 区间，10-21）；bethehand median 2.035（80% 区间，10-09）。mindscanlabs 在结果前主动更正：H1/H4 因打包变化对自家预测有利约 0.03，并固定了校正方案。 | 2026-10-21/22 | [#8435 c-6328613653](https://github.com/marin-community/marin/issues/8435#issuecomment-6328613653)；[#8435 c-6401672531](https://github.com/marin-community/marin/issues/8435#issuecomment-6401672531)；[#9977](https://github.com/marin-community/marin/issues/9977) | 独立预测 | 外部预测者 | 不是 Marin 结果；尚未计分；bethehand 的 2.035 是 10-09 的旧帖，窗口内只新增了打包偏差讨论 |
| E4 | 262K 阶段若按训练长度打包，Paloma 的 PTB 与 WikiText-103 验证集都不足一条 262,144-token 序列，产出零样本、loss 记 0.0；组内 macro 静默跳过它们，变成 14 子集均值（约 +0.06，约 3%，足以触发 `eval_regressed`）；顶层 macro 把 0 平均进去（约 −0.14）。67B 的 65K→262K A/B 给出 +0.0376 的组内变化。 | 2026-10-25T18:01:18Z | [#8435 c-6401174668](https://github.com/marin-community/marin/issues/8435#issuecomment-6401174668) | 独立代码+数据观察 | 外部观察者 bethehand | 262K 阶段未开始；0.06 是按当前值推算，67B 实测是 +0.0376 |
| E5 | dlwh 读源码确认：`TokenSeqDataset.async_len` 丢弃尾序列，`TaggedEvaluator` 把空 tag 计入总 macro。第一版修法是排除零 token tag，并把 Hero eval 打包上限设为 65,536 token。 | 2026-10-26T04:34:41Z / 04:43:51Z | [#8435 c-6408314282](https://github.com/marin-community/marin/issues/8435#issuecomment-6408314282)；[#8435 c-6408421426](https://github.com/marin-community/marin/issues/8435#issuecomment-6408421426) | 源码确认 + 代码 | Marin 作者 | 65,536 上限当天被后续方案替换；没有 run 用过 |
| E6 | 方案先改为多长度 eval（05:43 提出，06:15 实现时同时加入保留尾序列、右 padding 与 attention/loss masking，并撤销 65K 上限），再收敛为只评当前训练长度及以下（07:00），65K/262K 等训练到达后再加。 | 2026-10-26T05:43Z–07:00Z | [#8435 c-6409129788](https://github.com/marin-community/marin/issues/8435#issuecomment-6409129788)；[#8435 c-6409543752](https://github.com/marin-community/marin/issues/8435#issuecomment-6409543752)；[#8435 c-6410181205](https://github.com/marin-community/marin/issues/8435#issuecomment-6410181205) | 代码 + CPU | Marin 作者 | 只做 CPU；没有 GPU 排练；未进入主训练 |
| E7 | #10711 初版把 65K/262K eval 放在训练未用过的设备组上，自动评审判为生产 blocker（会重演 #10709 的 NCCL 显存泄漏），作者改用 #10709 的设备组复用。 | 2026-10-26T06:18:56Z / 06:59:58Z | [#10711 c-6409591208](https://github.com/marin-community/marin/pull/10711#issuecomment-6409591208)；[#10711 c-6410180643](https://github.com/marin-community/marin/pull/10711#issuecomment-6410180643) | 自动评审 + 代码 + CPU | 自动评审 / Marin 作者 | 评审是自动账号，不作为人类批准；作者报 3,578 个受影响 CPU 用例通过 |
| E8 | ClassicLarry 批准 #10711，但 8 分钟后追问：保留最后一个不满 chunk 会不会改变 4K/16K 的 eval 数据，建议在单机架上对现有 checkpoint 实测；到截点无回复、无该实验。 | 2026-10-30T17:45:57Z / 17:54:31Z | [#10711 review-5716334825](https://github.com/marin-community/marin/pull/10711#pullrequestreview-5716334825)；[#10711 c-6529741854](https://github.com/marin-community/marin/pull/10711#issuecomment-6529741854) | 人类评审通过 + 未答请求 | Marin 作者 | 批准不等于合并；请求仍未回答 |
| E9 | #10709：CP 下旧的 eval mesh 会新建训练未用过的 data/context 组，XLA 为每组建 NCCL communicator 并保留在进程内、allocator 池之外。单机架 65K、CP4×EP16 时权重搬运+eval 让每 GPU 多约 7.4 GiB，余量约 5.8 GiB，下一步训练分配 90.49 GiB 失败。修法是复用训练的 (data, expert) 组。 | 2026-10-26T04:33Z | [#10709](https://github.com/marin-community/marin/pull/10709) | 代码 + CPU + 单机 GPU | Marin 作者 | 11 机架未测；无人类评审；open/blocked |
| E10 | #10618（生产 launcher 的 CP 支持）已获 ravwojdyla 批准，但承诺的 65K CP4 rack 验证从未贴出；到截点 open、dirty、未合并。它是 262K 阶段的前置条件，不是 65K 已采用的能力。 | 2026-10-26T20:32:27Z | [#10618 review-5658575647](https://github.com/marin-community/marin/pull/10618#pullrequestreview-5658575647) | 人类评审通过 + 代码 | Marin 作者 / 评审者 | 没有自有 GPU 证据；批准不等于合并 |
| E11 | RULER 长上下文评测的接线 issue #7638 于 2026-10-24 被 stale 机器人自动关闭（`not_planned`），未实现；#10821 据此写“32K 以上没有长上下文能力基准，65K 阶段只能靠 loss 判断”。 | 2026-10-24T04:38:14Z | [#7638](https://github.com/marin-community/marin/issues/7638)；[#10821](https://github.com/marin-community/marin/issues/10821) | 关闭（未完成） | 机器人 / Marin 作者 | “没有”指没有能力基准；训练侧 loss（含 Paloma held-out loss）在 65K 打包下仍会评，不是“完全没有 eval” |
| E12 | 10-23 两次启动失败：coordinator 因凭证轮换被主动抢占，child 在启动期两次遇到 XLA `CoordinationService/WatchTasks` 连接被拒；Iris 自动重试后恢复，初始原因未确认。 | 2026-10-23T00:55Z–15:31Z | [#8506 c-6355202688](https://github.com/marin-community/marin/issues/8506#issuecomment-6355202688)；[#8506 c-6368537536](https://github.com/marin-community/marin/issues/8506#issuecomment-6368537536) | 运维日志（自动恢复） | bot / 运维 | 与 10-29 的 OOM 是两件事 |
| E13 | 10-29 事故事件：17:18:42Z 最后一步 352561；attempts 1–15 全部失败（OOM 明确记录在 attempts 1–4、6–9、11–12），第一步前报 NCCL `ncclCuMemAlloc` OOM（168–170 → 184.3/185 GiB）；单 rack cordon 不可行（Kueue 要 11 个完整 16 节点域）；人类 operator 迁移到 rack 399，attempt 16 于 20:47:17Z 从 352321 恢复。 | 2026-10-29T17:18:42Z–20:47:17Z | [#8506 c-6504151707](https://github.com/marin-community/marin/issues/8506#issuecomment-6504151707)；[#8506 c-6504948632](https://github.com/marin-community/marin/issues/8506#issuecomment-6504948632)；[#8506 c-6505610334](https://github.com/marin-community/marin/issues/8506#issuecomment-6505610334) | 运维日志（人工恢复） | bot / 运维 / 人类 operator | 根因“IMEX 保留已离开 peer 的显存”是 leading explanation，确切归属与安全恢复未确认；与梯度 spike 是不同机制，未发现两者有关联的证据；attempts 10/14 未记录 |
| E14 | #10677：step 325710 的 grad norm 4.56、step 353060 的 20.84（41.6× 中位，loss 随后抬升 0.084 并恢复）。团队**精确复现了 step-324000 的 spike**（324000 权重、batch 内 seq 186，grad 2274），并把它钉到单条 16K 序列、单个 token、单个算子（layer 35 的 attention GatedNorm 在 outlier channel 上 gate 半开；MoE 路由近 tie 让 1 ulp 的 embedding 扰动在 layer 28 翻掉 7/8 专家）。325710 从未在本步 325709 权重上评估；353060 在 348000 权重下未复现（probe 0.509），作者明说“无法命名 353060 的 culprit”。 | 2026-10-25T19:39Z–2026-10-30T06:10Z | [#10677 c-6403766348](https://github.com/marin-community/marin/issues/10677#issuecomment-6403766348)；[#10677 c-6405962929](https://github.com/marin-community/marin/issues/10677#issuecomment-6405962929)；[#10677 c-6516267482](https://github.com/marin-community/marin/issues/10677#issuecomment-6516267482) | 作者 GPU 单机探针 | Marin 作者 | 可复现的机制只对 324000 成立；325710/353060 未在本步权重复现；作者撤回过“梯度算得对”的强断言 |
| E15 | #10783 合并（commit `8aac0db3d9`）：在 optimizer 外把总梯度范数裁到 1.0，checkpoint 布局不变；CPU 110 例；两次单机恢复测试 rt1/rt2 中 clip 触发；PR 无 11 机架测试。 | 2026-10-30T13:41:20Z | [#10783](https://github.com/marin-community/marin/pull/10783) | 代码 + CPU + 单机 GPU + 合并 | Marin 作者 | 单机证据；11 机架未测 |
| E16 | `hero-gradclip-step356k` 从永久 step-356531 重启；50 步 gate 为 GO（loss 差均值 −3e-5、最大 1.8e-4；step 13.78 vs 13.96 s；drop 1.37e-3；HBM 123.2 GiB；clip 0/50，到 356727 为 0/197；MFU 30.56% 低于 30.7–32% band）。 | 2026-10-30T13:51Z–14:50Z | [#8506 c-6524447746](https://github.com/marin-community/marin/issues/8506#issuecomment-6524447746)；[#8506 c-6525675630](https://github.com/marin-community/marin/issues/8506#issuecomment-6525675630)；[#8506 c-6525676502](https://github.com/marin-community/marin/issues/8506#issuecomment-6525676502) | 在线 50 步 + 主训练采用 | Marin 作者 | clip 从未在 hero 触发，trial 只证明“无回归”，不证明“有效”；evals/checkpoint resume 未覆盖 |
| E17 | 65K 决策：step-372000（95.3% of 390,251；0.84T token / 4.7%；剩 18,251 步）；CP1×EP64、capacity factor 1.5、qk_mult 1.66。单机架 500 步对照：CP4×EP16 14.699 s/step、drop 2.41e-3、127.6 GiB；CP1×EP64 15.397 s/step、drop 9.80e-4、128.4 GiB。选 CP1：慢 4.7% 但 drop 少 2.5×，且 CP4 的 dropless eval 在 #10709 合并前会 crash 下一步训练。 | 2026-10-31 / 11-01 | [#10821](https://github.com/marin-community/marin/issues/10821) | 提议 + 单机 GPU | Marin 作者 | 11 机架 capacity factor 1.5 首次实测就是这次切换；CP4 留给 262K |
| E18 | C1/C2 单机排练：C1 从 step-360000 跑 60 步（15.571 s/step、drop 8.29e-4、三次 dropless eval、无 OOM）；C2 从 step-366000 跑 100 步并做两次 checkpoint 保存（15.518 s/step、drop 1.00e-3、127.9 GiB、最终 4.29 TB 的 step-366100 完整），但 checkpoint 未做恢复测试。 | 2026-11-01 | [#10821 C1/C2](https://github.com/marin-community/marin/issues/10821) | 单机 GPU | Marin 作者 | 只覆盖单机；恢复路径未测 |
| E19 | #10820 合并（commit `6e6924fedd`，02:55:41Z）；65K 子 run `hero-ctx65k-step372k` 03:25Z 启动、03:28Z 恢复 step-372000、约 03:34Z 第一步；04:03:06Z 六条 F2 gate 全 PASS（step 372,102，首次尝试无重试）：CE max 1.007、loss 差 −0.02009、中位 16.05 s/step、drop 4.058e-4（均值）、peak HBM 127.8 GiB、无持续 grad spike。父 control 为 14.38 s/step、loss 0.9982、drop 1.396e-3、HBM 123.2 GiB。 | 2026-11-02T02:55–04:03Z | [#10820](https://github.com/marin-community/marin/pull/10820)；[#10821 c-6568749068](https://github.com/marin-community/marin/issues/10821#issuecomment-6568749068)；[#8506 c-6568749287](https://github.com/marin-community/marin/issues/8506#issuecomment-6568749287) | 合并 + 主训练采用 + 在线 gate | Marin agent（作者方） | gate 数字无独立复核；只覆盖 103 步；不是能力结果 |
| E20 | 65K 评测安排：eval 按训练长度打包，Paloma macro 可能在切换处阶跃，`eval_regressed`（相对父 run 2%）可能误报；首次 65K eval 在 step 375,000（约 11-02T17:20Z），永久 checkpoint 在 378,000（约 11-03T07:00Z），都在截点之后；#10711 计划在切换后、262K 前落地。 | 2026-11-02 | [#10821](https://github.com/marin-community/marin/issues/10821) | 计划/风险 | Marin 作者 | 截点前没有 65K eval 结果；不要写成“已经回归”；32K 以上没有能力基准，65K 阶段只有 loss（含 Paloma held-out loss） |
| E21 | 状态：65K 子 run 最后确认 step 372,102（04:03Z），预计 11-05T15:00Z 完成；#10711 open/dirty、已批准但无 GPU；#10709 open/blocked、单机证据、11 机架未测；#10618 open/dirty、已批准但无 rack 验证；262K 未开始。 | 2026-11-02T05:30Z | [#10711](https://github.com/marin-community/marin/pull/10711)；[#10709](https://github.com/marin-community/marin/pull/10709)；[#10618](https://github.com/marin-community/marin/pull/10618)；[#10821](https://github.com/marin-community/marin/issues/10821) | 快照 | 内部读取 | mergeable/dirty 是瞬时字段，不是长期事实 |
| E22 | #10556（routing precision/frozen-weight gradients）仍 open/blocked、未合并；#10785（FP32 saved-O 单机实验）是 issue，开 8 秒后即关闭、未采用；#10428（checkpoint averaging/format SFT）与 #10388（RL readiness）是旁线或更早测量；#8827 prompt completions 在窗口内无新内容。 | 2026-10-19–11-02 | [#10556](https://github.com/marin-community/marin/pull/10556)；[#10785](https://github.com/marin-community/marin/issues/10785)；[#10428](https://github.com/marin-community/marin/issues/10428)；[#10388](https://github.com/marin-community/marin/issues/10388) | 状态 | 内部读取 | 不作为本轮视频选题；见“筛掉/暂缓” |
| E23 | 背景（窗口外）：16K 切换 PR #10101 于 2026-10-14T02:06:04Z 合并；#9615 的 4K/8K/16K 实验 10-14 关闭。它们解释 eval 口径为何变化，但不是本窗口的新进展。 | 2026-10-14 | [#10101](https://github.com/marin-community/marin/pull/10101)；[#9615](https://github.com/marin-community/marin/issues/9615) | 合并 + 背景 | Marin 作者 | 不要写成“刚切到 16K” |

## 筛掉/暂缓

- **10-29 IMEX/NCCL OOM 事故**：重要，但时间线几乎全部来自同一个 bot 读 Iris/Finelog/节点遥测，根因未完全确认，恢复靠人类 operator 迁移 rack。暂缓为独立选题；在 brief 里作为“重要变化/可靠性短讯”保留。
- **#10428 / #10388 / #10556 / #10785 / #8827**：分别是 checkpoint averaging/SFT 旁线、RL/serving readiness（测量期更早）、未合并的 routing precision 改动、当天关闭的单机实验、无新内容的 prompt completions。不是本轮主线进展。
- **#10121 / #10145 / #10610 / #10368**：性能/基础设施 PR，影响吞吐或 CP 布局，但不构成新的上下文能力或评测结论；不选为视频题。
- **通用 eval/RL/数据项**（#10757、#10828、#10541 等）：与 535B hero 主线相邻但不同线；本轮不纳入。
- **#8706 / #7626 RULER 线**：关闭/未实现，不能读成“长上下文评测已就绪”。

## 覆盖盲区

- 没有读 W&B 曲线、Echo 内部事故页、S3 checkpoint、线上 eval 日志；所有 W&B 数字都只来自 GitHub 评论中的引用。
- 没有 GPU 复跑；65K gate、#10709 的单机结果、#10783 的 rt1/rt2 都是作者报告。
- `mtracker` 可访问（HTTP 200，Last-Modified 约 05:23Z），但自述由 LLM 生成；它写“16K run stopped after step 372,058”与 #10821 的 control window（到 372,049、03:13Z cancel）不一致，且 05:23Z 的进度数字与 16.05 s/step 不一致，故未采用。
- 截点后的事件（首次 65K eval、378,000 checkpoint、run 完成、262K 启动）都不在本记录内。
- GitHub 之外可能已有未公开的 4K 重评或 65K eval；“没有公开记录”不等于内部没有发生。
- 10-29 的 attempts 10/14 没有出现在公开日志里，精确重试次数无法完全枚举。

## 内部核查与修正

核查由独立子代理重读原始评论/PR/commit 完成（逐条用 comment ID 反查作者与时间，PR 用 pulls/reviews/commits/check-runs），主执行代理按结果修正。共核对 22 条主要主张与 14 项专项检查；约 20 条完全成立，问题集中在三处解释性表述。这是内部核查，不是外部复现，也不构成正确性保证。

| 核查发现 | 处理 |
|---|---|
| 初稿把 325710 的 spike 说成“追到单 token/op” | 那是 **step-324000 batch 在 324000 权重下**的复现（seq 186、layer 35、GatedNorm channel）；325710 从未在 325709 权重复现，353060 也未复现（0.509），作者说“无法命名 353060 的 culprit”。已改写 E14 与选题 3 的措辞。 |
| 初稿写“13 次重试” | 日志支持的是 **attempts 1–15 失败、attempt 16 恢复**（20:47:17Z，352321）；OOM 明确记录在 1–4、6–9、11–12。已改为“15 次失败尝试后，第 16 次恢复”。 |
| 初稿把 #10821 的“nothing evaluates past 32K”直译成“32K 以上没有评测” | 这句与 E4 自相矛盾：65K 打包下 Paloma loss 仍会评。已改为“没有长上下文**能力**基准；65K 阶段只能靠 loss 判断”。 |
| “−15% tok/s”被当作实测 | 保留为“预估值”；实测写 **约 −10.4%**（16.05 vs 14.38 s）或 **−11.0%**（2.8645e6 vs 3.2174e6 tok/s）。 |
| 阶段标签混淆 | E19 的 gate 是 **production gate pass**，不是 GPU rehearsal；E2 是“团队承诺”，不是 proposal；#10711/#10618 是 **approved 但未 merged**；clip 触发只属于单机 rt1/rt2，hero 上为 0/50、0/197。 |
| E13 日期从 17:47Z（首条报告）开始 | 改为 **17:18:42Z**（最后一个训练步）；并注明 initiating lost-heartbeat cause 未确认。 |
| 10-29 OOM 与梯度 spike 被写成“无关” | 改为“机制不同，未发现两者有关联的证据”；Marin 也没有正式声明二者无关。 |
| #10618 只写“open” | 补充：已获 ravwojdyla 批准（10-26T20:32:27Z），但承诺的 65K CP4 rack 验证从未贴出；open/dirty；#10821 表里的 CP4 数字是 main 分支 `launch_diagnostics` 的，不是 #10618 的证据。 |
| 4K 重评被写成“逾期” | 承诺条件是 run 结束；到截点没有公开记录，但还不能算违约。 |
| #10711 CPU 用例数 3,297 / 3,574 / 3,578 | 采用最终 revision `eea42ac` 的 **3,578**（作者报告），CI `unit-tests` 10-26T07:05:19Z 绿；3,574 是中间 revision。 |
| 65K drop 有 3.5e-4 与 4.06e-4 两个数 | 写成“约 3.5e-4–4.1e-4；gate 窗口均值 4.058e-4”。 |
| #10785 被写成 PR | 它是 issue：07:14:50Z 开、8 秒后关，作者说“no measurable loss impact, not viewing as priority”；未采用。 |
| #10711 的 `HERO_FIXED_EVAL_SEQ_LENS` 若合并，在 65K 会评 4K/65K、不含 16K 档 | 已从 patch 核实；作者未在 PR 讨论。只放在“待确认”，不作为结论。 |

## 仍待确认

1. 65K gate 的原始 W&B 曲线与逐步 `train/grad_clipped` 数据；本记录只有评论中的汇总。
2. 4K 全 checkpoint 重评是否/何时发生；这会直接影响 4K 口径预测的评分。
3. #10711 的 1-rack partial-chunk 实验（ClassicLarry 10-30 请求）是否会做、结果如何。
4. #10709 的 11 机架布局是否验证；#10618 的 65K CP4 rack 验证是否补上。
5. 10-29 事故的 IMEX 根因归属与安全恢复流程是否被正式确认。
6. #10711 的固定长度列表是否会有意保留 16K 档；当前代码阅读显示不会。
