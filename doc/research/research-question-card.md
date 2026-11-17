# 研究问题卡：哪些早期控制值得重新探索？

日期：2026-11-16。依据 Scholar `research-ideation` 的 Research Contract，以及本地 `research-idea-rubric`。本卡承接 [03 的研究取舍](../03_CEM_IDEA_REVIEW.md)，提供下一轮可执行协议；所有方法收益仍是假设。

## 选定问题

**Question：** 在逐渐增加评价时域的黑箱轨迹优化中，能否利用少量真实控制干预，识别值得恢复探索的时间块，并在相同真实计算预算下提高完整任务成功率？

**Type：** exploratory，先辨认机制；通过后再冻结方法做 confirmatory 比较。

**Hypothesis：** 后续任务暴露的新困难有时需要改变已收缩的早段控制。小批试探能识别有用的修复块，使剩余搜索预算比统一恢复、固定早段或随机块得到更好的完整任务结果。

**Why it matters：** 接近阶段的选择会决定后续接触和承载是否可能；控制前缀一旦停止扰动，后段优化难以纠正旧决定。研究对象是有限查询下恢复哪些变量，协方差膨胀本身属于已有工具。

**Current evidence：** ER-20261116-a144-01、ER-20261116-stages-02 支持这一问题值得检验；ER-20261116-trmpc-04 与 ER-20261116-icem-03 约束可声称的增量。没有“选择性重开已有效”的证据。

**Missing evidence：** 哪些时间块在相同条件下有可重复的修复收益；选块能否胜过持续探索；收益能否跨初态、参考和任务成立；探测收益能否预测后续优化而非只挑到一次偶然好样本。

**What would support it：** 同一冻结问题上存在稳定的块间收益差异；从同一初态运行完整流程后，自适应选块在等仿真预算和可接受墙钟下胜过最强简单对照；去掉收益选择、改变选择信息后，优势相应减弱。

**What would falsify it：** 固定早段、全局重启或持续探索得到相同收益；从一开始使用完整时域已解决问题；探测开销抵消收益；只在更换 cost 后成功；只改善标量成本而没有改善完整任务。

**Minimal next action：** 用已有 yaw020 失败 checkpoint，固定完整评分窗口，做下述七组等预算恢复比较；先不加入多分支、DFO 或学习器。

**Decision：** run experiment。这里是实验规划，不是启动 GPU 任务的完成记录。

## 证据记录

### ER-20261116-a144-01

- **Source：** [A144 诊断](../../../Loco_maniplation/g1-trajopt/docs/A144_DIAGNOSTICS.md)、[审计说明](../../../Loco_maniplation/g1-trajopt/docs/CEM_SEARCH_AUDIT_PLAN.md)、[verification.json](../../../Loco_maniplation/g1-trajopt/runs/yaw020_search_audit/audit/verification.json)。
- **Source type：** experiment artifact，并以 project note 解释。
- **Supports：** 末轮 1,023 个不同候选的前约 2.02 s 实际控制相同；已记录的侧面到达候选只是 1.15 s 短前缀，仍有约 9.0/8.4 mm 几何穿透。
- **Contradicts：** 把 row478/row78 称为已经完成抓取的成功解；现有记录仍不足以支持“完整成功解被单高斯拟合毁掉”。
- **Method / metric：** 保存控制、实际 rollout、几何标签、冷重放；JSON 明示 `geometry_only=true`、`full_task_success_not_inferred=true`。
- **Limitation：** 单个 seed 的失败机制；未构成新方法成功率。目标接近路径也有几何冲突，不能把冻结认作唯一原因。
- **Project relevance：** 提供最小恢复实验的起点。
- **Claim strength：** observed；仅限已记录诊断。

### ER-20261116-stages-02

- **Source：** [SBTO 采样器](../../../Loco_maniplation/g1-trajopt/src/g1_trajopt/_vendor/sbto/solvers/cem.py)、[阶段调度](../../../Loco_maniplation/g1-trajopt/src/g1_trajopt/engine/optimization.py)、[控制评分](../../../Loco_maniplation/g1-trajopt/src/g1_trajopt/engine/controller.py)。定位分别为 `get_samples`、`optimization_stages`、当前 H 的 PCHIP 与 `T/H` 缩放。
- **Source type：** project note，附本轮直接静态源码核查。
- **Supports：** 方差硬掩码停止坐标扰动；评价时域逐阶段扩展，比例缩放没有计算未来代价。
- **Contradicts：** “所有迭代从开始就评完完整 4.45 s”。
- **Method / metric：** 源码语义核查；不包含新的优化结果。
- **Limitation：** 解释当前实现，不证明恢复变量会提高性能。
- **Project relevance：** 区分算法问题、实现限制和时域策略。
- **Claim strength：** observed；研究收益仍为 speculative。

### ER-20261116-icem-03

- **Source：** [iCEM v1](https://arxiv.org/html/2008.06389v1)，§3.1–3.3；[CoRL 正式论文集](https://proceedings.mlr.press/v155/pinneri21a.html)。
- **Source type：** full paper。
- **Supports：** 时间相关扰动、精英记忆和已评价最佳序列的使用已有直接先例。
- **Contradicts：** 把上述组件本身作为本项目原创贡献。
- **Method / metric：** CEM 的采样与复用机制；本轮没有复现其数值结果。
- **Limitation：** 与本项目渐进时域的局部恢复问题不同。
- **Project relevance：** 必要强基线；发表身份可补齐为 CoRL 2020、PMLR 卷于 2021 出版。
- **Claim strength：** supported，限方法归属。

### ER-20261116-trmpc-04

- **Source：** [TR-MPC v2](https://arxiv.org/html/2605.07801v2)，§IV、§VI-A、Fig.3；本轮[唯一精读报告](../paper_reading/2605.07801_Sampling-based_Model_Predictive_Control_Using_Trust_Regions/2605.07801_阅读报告.md)。
- **Source type：** full paper / preprint version。
- **Supports：** KL 步长约束与可选熵下界用于控制提案更新、保留探索。
- **Contradicts：** 用笼统“防止 CEM 塌缩”界定新颖性。
- **Method / metric：** 对偶求解后矩匹配；已有数值控制任务证据，未直接检验 G1 搬运。
- **Limitation：** 无限维最优分布与有限样本高斯投影须分开；任意非二次成本下，关闭熵项也不保证指数倾斜分布仍是高斯。
- **Project relevance：** 必须比较探索保护；不能只战胜旧硬掩码。
- **Claim strength：** supported，限论文方法与已报告任务。

### ER-20261116-budget-05

- **Source：** [BAMPC v1](https://arxiv.org/html/2610.14064v1)，§III-A–C、Eq.(2)、Alg.1、§IV；[BC-EvoCEM v2](https://arxiv.org/html/2506.02205v2)，§3、Alg.1、§5。
- **Source type：** full paper / preprint。
- **Supports：** 控制样本、模型假设和计算用途的预算问题已有研究；多个 CEM worker 的协调与替换也已有先例。
- **Contradicts：** “首次分配采样预算”或“多个高斯按表现重生就是新方法”。
- **Method / metric：** 两篇分别研究不同预算用途与 worker 信息几何管理；不把它们混称为选块重开。
- **Limitation：** 仅核验直接相关路线，不代表穷尽所有自适应变量选择文献。
- **Project relevance：** 多分支方向只能保留为后续可修复收益的排序假设。
- **Claim strength：** supported，限方法边界。

### ER-20261116-hydrax-06

- **Source：** [Hydrax CEM](../../src/hydrax/hydrax/algs/cem.py)，`sample_knots` / `update_params`；[open_loop](../../src/hydrax/hydrax/open_loop.py)，优化循环与末轮输出选择。
- **Source type：** project note，附本轮直接静态源码核查。
- **Supports：** 当前 Hydrax 已含 `explore_fraction` 的初始尺度探索和 `sigma_min`；`cov` 实际存标准差。离线入口从末轮候选返回最佳，没有维护全程最优。
- **Contradicts：** 把两套框架的 `cov` 数值或最终返回规则直接当作相同。
- **Method / metric：** 静态代码语义；未运行 BugTrap、Push-T 或新算法。
- **Limitation：** 文件存在和接口可用不等于复现成功。
- **Project relevance：** 持续探索属于现成强对照；恢复公式需要按各框架真实参数语义实现。
- **Claim strength：** observed。

## 可写进研究讨论的主张

| Claim candidate | Source evidence | Allowed wording | Forbidden stronger wording | Next check / Decision |
|---|---|---|---|---|
| 当前实现存在早期变量停止扰动 | a144-01、stages-02 | 特定失败条件末轮已失去早期探索 | 冻结是全部失败的唯一根因 | 等预算干预；keep |
| 收益驱动选块值得研究 | a144-01、stages-02、trmpc-04 | 本项目提出待检验的修复假设 | 已提高成功率；首次解决局部最优 | 对比持续探索和固定早段；keep as hypothesis |
| 单高斯损失应单独诊断 | a144-01、budget-05 | 目前没有完整成功解被拟合毁掉的证据 | 多高斯必然解决抓取 | 固定候选池与精英反事实；revise |
| 可在 Hydrax 做外部任务验证 | hydrax-06 | 任务实现提供实验入口 | 已复现跨任务收益 | 先建立可重复基线；keep as plan |

表中 evidence 缩写对应上方完整 ER ID；没有收益结果可以提升为论文结论。

## 最小实验协议

**单位与冻结条件。** 先用同一个失败 checkpoint，重新从同一真实状态执行每个候选，全部评到相同完整 T。固定模型、参考、cost、控制表示、输出选择规则与总预算。不要把不同阶段 `T/H` 缩放后的 cost 直接排在一起。

**七组机制比较：** 不重开、全局重开、固定早段、固定后段、随机等大小块、等额真实试探选块、取消硬掩码并保持固定探索。固定早/后/随机/自适应使用相同块维数与归一化扰动尺度；全局重开另做尺度敏感性，所有组拥有相同调参额度。持续探索对照应包含 Hydrax 已有机制，而非新起一个弱版本。

**初版决策规则。** 预先固定块划分、触发条件和探测份额，例如预算的 10% 仅作为 pilot 起点。先按等额探测选一个块；可用批次最佳真实下降作代理，同时记录成功比例与成本分位数。试探候选可参加最终选择，所有试探与后续搜索都计入预算。中心、时域或扰动提案改变后，重置对应收益统计。

**避免选块评价的偏差。** 用同一批探测既挑胜者又宣称它未来收益最高，会高估选择质量。报告实际整次运行效果；若单独验证“预测后续可修复性”，需用独立续搜随机数和等量后续预算评价选定块，诊断费用另记。不能用选中批次自身的最大下降代替预测验证。

**计费。** 同时记录真实仿真 world-steps 和墙钟。候选重评、失败步、缓存失效、插值、数据传输、采样与参数更新均计费；编译与稳态耗时分列。审计记录器开销保持一致，原 348 s 与带归档审计约 766 s 不能用来比较优化器速度。

**成功标准。** 沿用并预先冻结完整任务判据，覆盖接近、接触持续与承载、抬升、足底支撑、几何穿透、终端稳定及同引擎冷重放。几何标签、手掌距离和低 cost 均不单独充当成功判据。接触力未观测时记 unknown，不补成成功。

**分两层回答。** checkpoint 实验回答“能否局部恢复”；从相同初态重新执行完整搜索才回答“整个方法是否更好”。第二层加入从头完整时域 CEM、iCEM 式强基线和 TR-MPC。先报告统一平台的机制对照，再分别记录论文设置复现；不把移植版本自动称为完整复现。

## 方向评分与处置

1–5 分；风险 5 为最高。分数属于基于现有证据的研究判断，无新效果数据，沿用 03 的锚点。

| 方向 | 新颖性 | 可实施性 | 可评价性 | 机器人意义 | 发表潜力 | 风险 | 决定 |
|---|---:|---:|---:|---:|---:|---:|---|
| 真实干预收益选择重开块 | 3 | 4 | 5 | 5 | 3 | 3 | keep，先证伪 |
| 后续可修复收益分配分支预算 | 3 | 3 | 4 | 5 | 3 | 4 | revise，先检验排序信息 |
| 残差 DFO-GN 局部精修 | 2 | 4 | 5 | 4 | 2 | 3 | revise，组件定位 |
| 接触目标与约束表达 | 2 | 4 | 4 | 5 | 2 | 3 | keep，独立任务表达实验 |

排序依据：第一项有直接实现入口；第二项受 BC-EvoCEM 等近邻约束且需证明续搜排名差异；第三项容易实现但接触切换处局部模型风险较高；第四项有明确几何问题，但不能由此声称优化器创新。

## 两周内的继续／停止决策

时间和 48 GPU 小时是沿用 03 的建议上限，尚未启动或预约资源；硬件可用性未在本轮实测。

| 时间 | 交付物 | 决策 |
|---|---|---|
| 第 1–2 天 | 冻结 checkpoint、完整成功判据、计费日志；确认方差和掩码可正确恢复 | 无有效干预或 cost 持续偏好卡边则先定位目标问题 |
| 第 3–5 天 | 七组机制比较，5 个配对种子仅筛选 | 自适应不胜固定早段／全局／持续探索，则保留最简单修复 |
| 第 6–9 天 | 从相同初态重跑全流程；建立 BugTrap / Push-T 基线，CartPole 检查退步 | 只在 yaw020 有效则定位 G1 系统修复，收窄通用主张 |
| 第 10–14 天 | 参数冻结后在新条件、新种子复核最强对照，完成关键消融 | 收益与机制均成立才扩展论文实验 |

沿用 03 的工程筛选线：对最强朴素对照，等预算成功率增加至少 15 个百分点；或成功率下降不超过 5 个百分点，双方均达到预先登记目标时，达到该目标所需墙钟减少至少 30%。这些阈值不是显著性判据。5 seeds 不支持正式泛化结论；后续样本量根据 pilot 方差与目标区间宽度确定，报告配对区间、成功率—预算曲线及所有失败。超时按右删失记录，不能只统计成功运行耗时。

## 下一轮文献补查

本轮只对直接近邻进行核验，不能宣称已证明首创。继续检索 adaptive coordinate/block selection、variable freezing/reactivation、restart covariance adaptation、progressive-horizon optimization 与 budgeted intervention。对候选原文记录“优化对象、选择信息、是否真实探测、是否改变评价时域、额外开销、公平基线”，再判断本命题是否已有同构算法。

理论只考虑冻结中心、时域和扰动提案的一轮选择；若研究有界奖励的最佳臂识别，要重新说明独立性与样本来源。该局部代理问题的界不能直接表述为整个非平稳 CEM 的成功概率或首次成功时间保证。
