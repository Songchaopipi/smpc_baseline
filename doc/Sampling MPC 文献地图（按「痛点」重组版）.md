# Sampling MPC 文献地图（按「痛点」重组版）

这份笔记想回答一个具体问题：机器人为什么已经模拟了很多未来，仍然找不到一个能执行的动作？“样本不够”只是一种解释。也可能是样本形状不合动力学、两个好解被平均坏了、更新过早停止，或仿真一直在认真优化一个错误的未来。把这些原因混在一起，换一个更复杂的优化器也很难知道究竟改善了什么。

本文保留原有 A–G 分类与 57 篇 L/N 编号，编号直接链接本地 PDF。事实依据为本轮已核查的本地论文与阅读记录；A 类以 2026-11-16 的公式和实验复核为准。下文明确标出的“读后判断”和“可分辨实验”是本笔记的推断与研究建议，不表示作者已经做过这些实验。

## 先把问题放回一个控制周期

以推一个 T 形物体为例。控制器围绕当前计划产生候选，在模型里预测接触和物体运动，再根据代价保留、加权或搬移候选，最后执行一小段并重规划。PS 直接留最优，CEM 拟合精英，MPPI 用指数权重；这些操作可以共享同一仿真器，却未必搜索同一种变量，也未必执行分布均值。

因此，比较论文前应先看它究竟把什么交给搜索：每步力矩、样条控制点、接触位置、基础模型的输入命令，还是生成器的噪声。再看最后执行的是重新构造的均值，还是已经评估过的最佳样本。这两件事会改变“采得好”和“更新得好”的含义。

| 类别 | 这份笔记追问的问题 | 主要分歧 |
| --- | --- | --- |
| A 采样质量 | 为什么白噪声浪费预算，怎样才算覆盖得好？ | 时间相关、点集均匀性、状态覆盖与先验各在补什么 |
| B 多模态 | 什么时候平均会毁掉一个可行解？ | 提前择一、保留多个解，或主动提出不同策略 |
| C 更新规则 | 权重集中、分布变小，是否就接近最优？ | 软硬筛选、更新幅度、方差缩减与粒子重采样 |
| D 地形与搜索方向 | 长时域接触为何仍难，平滑和梯度各丢掉了什么？ | 搜索范围、平滑偏移、代理方向与模型导数 |
| E 约束与接触 | 哪些事应该交给结构化 solver？ | 先投影、边生成边修正、局部接触求解与多重射击 |
| F 模型与先验 | 算法在优化错误的未来时，增加采样还有多少用？ | 学提案、学动力学、改搜索变量与补反馈 |
| G 工具、预算与背景 | 怎样知道收益来自想研究的那一部分？ | 共同实现、计算口径、模型误差与理论适用范围 |

## A. 白噪声为什么浪费：先看动力学会保留哪一类输入

设想一辆转向执行器较慢的车。某一步向左打一点，下一步立刻向右打一点，控制幅值虽然不小，车可能几乎没有完成一次有效转向。有限样本下，独立噪声大量生成这种互相抵消的候选。问题出在时间结构；只把方差调大，会同时得到更多极端动作，未必更容易得到持续同向输入。

iCEM（[N01](<../papers/A_采样质量/N01_2020_Sample_efficient_Cross_Entropy_Method_for_Real_time_Planning.pdf>)）用幂律频谱有色噪声增加连贯动作出现的概率，还保留、移位精英并递减迭代样本数。Low-Frequency MPPI（[N06](<../papers/A_采样质量/N06_2024_Low_Frequency_Sampling_in_Model_Predictive_Path_Integral_Control.pdf>)）把类似的时间相关性接到 MPPI 的指数加权上，用频域采样和逆 FFT 构造相关高斯噪声。两者的共同直觉是把概率质量分给动力学更容易积累的输入方向；iCEM 的收益还包括记忆与预算安排，不能全算在噪声上。

N06 §III-A 明确没有因此降低控制序列维数。相同边际动作方差下，相关性会改变状态响应的分布；它不改变机器人本来的可达性，也不能保证每条轨迹走得更远。原文慢转向车辆的例子支持这种有限预算解释。若任务需要突然制动或快速换向，过强的低频偏置也可能漏掉必要动作。

Smooth MPPI（[N05](<../papers/A_采样质量/N05_2021_Smooth_Model_Predictive_Path_Integral_Control_without_Smoothing.pdf>)）处理的是另一处不一致：先评分原始控制、执行时再低通滤波，真正送给机器人的动作就没有按同一个模型被评分。它将平滑要求放进优化内部，但这里最容易读错的是索引。N05 §III-B、Eq.(15)–(18) 与 Algorithm 1 的动作更新是

\[
a_t^{i+1}=a_t^i+\Delta t\,u_t^{i+1}.
\]

这里的 \(t\) 是预测时刻，\(i\) 是优化迭代。累积发生在 \(i\) 轴；公式没有沿 \(t\) 轴把前一步白噪声积分到后一步。原文还在预测时间上加入动作差分惩罚，并对动作幅值施加 box 约束；按变化率尺度设置噪声，不等于施加了硬速率约束。把它与有色噪声都叫“平滑采样”，会掩盖两种不同机制。

这个索引也解释了 N06 的一个对比设置：动作扰动为 \(\Delta t\,\epsilon\)，小时步会压小有效探索。N06 §IV-C 在双积分器实验中将 Smooth-MPPI* 的**标准差**设为 \(\sigma/\Delta t\)，相应协方差除以 \(\Delta t^2\)。这是匹配扰动尺度的实验调整，不能写成方差除以时步，也不能推广成所有 Smooth MPPI 系统的唯一参数规则。

dsCEM（[N07](<../papers/A_采样质量/N07_2025_Sample_Efficient_and_Smooth_Cross_Entropy_Method_Model_Predictive_Control_Using_Deterministic_Samples.pdf>)）和 dsMPPI（[N08](<../papers/A_采样质量/N08_2026_Smooth_Sampling_Based_Model_Predictive_Control_Using_Deterministic_Samples.pdf>)）追问另一个浪费来源：同样数量的候选，随机点集是否恰好把某些方向采得过密、另一些方向漏掉？它们先离线优化逼近标准高斯的等权 LCD 点集，再按当前均值与协方差变换。dsCEM 用精英硬筛选；dsMPPI 改用软权重，并组合迭代、参数动量和精英缓冲。它们减少的是有限随机采样造成的更新波动，不能据此说它们只有这一种平滑来源。

两篇都显式使用时间相关结构，例如 \(C_j=\operatorname{diag}(\sigma_j)C_\rho\operatorname{diag}(\sigma_j)\)。固定 \(C_\rho\) 而只更新边缘方差，仍然是在采相关控制。N07 另有完整协方差变体。两篇最终都返回已评估最佳序列的首动作，不能把实际执行信号直接当成估计均值，再用均值估计方差解释全部结果。

实验也没有给出“dsMPPI 全面优于 dsCEM”的结论。N08 §5 的 cart-pole 与 truck backer-upper 中，dsCEM 累计 cost 通常略低；dsMPPI 的 cart-pole 整段平滑优势特指较大样本量，后半段指标优势更明显，truck 的两项平滑指标均有优势。N07 的任务则是 mountain car 与 cart-pole，完整协方差变体并非总是更好。这些证据支持组合设计的成本与平滑权衡，尚不能直接外推到人形接触控制。

C-Uniform（[N11](<../papers/A_采样质量/N11_2024_C_Uniform_Trajectory_Sampling_For_Fast_Motion_Planning.pdf>)）把“均匀”放到了动力学之后：输入空间撒得均匀，不代表到达的配置分布均匀。它离散各时刻可达配置层，通过最大流预计算状态相关动作概率。N11 §V-B 的 2 秒轨迹覆盖实验中，10000 条轨迹覆盖 2578 个网格，占可达离散空间 10.74%；同表最佳基线为 7.66%。约 40% 指相对覆盖增益，10.74% 并非任务成功率。

这个选择把一部分困难移到了离线计算。N11 §V-A 的约 14000 秒是全部层集动作概率表的预计算，不能当作一次在线 MPC 延迟。实证包含平面 Dubins 车与 F1Tenth 真车，不能删去实机证据；但其低维离散覆盖经验也不足以说明高维接触任务同样可扩展。原文没有给出状态相关非高斯提案接入 MPPI 后的密度比与新权重推导，因此覆盖改善不能自动延伸为保持原 MPPI 目标的理论结论。

Biased-MPPI（[N03](<../papers/A_采样质量/N03_2024_Biased_MPPI_Informing_Sampling_Based_Model_Predictive_Control_by_Fusing_Ancillary_Controllers.pdf>)）更直接：若辅助控制器已经知道怎么绕过去，为什么坚持从当前计划附近重新发现？它把经典、学习或运动基元控制器的候选纳入统一评分。关键代价是主动改变正则化参照。由 N03 §III-A、Eq.(10)–(14) 可写成

\[
\min_Q \mathbb E_Q[S]+\lambda D_{\mathrm{KL}}(Q\Vert Q_s),
\qquad q^*_{\mathrm{bias}}(V)\propto q_s(V)e^{-S(V)/\lambda}.
\]

物理任务代价 \(S\) 仍在，原先验 \(P\) 被 \(Q_s\) 替代。这解释了为何从 \(Q_s\) 取候选时可以直接用任务代价的指数权重。若只是换提案、还要估计原目标，则需处理密度比。实现中每个辅助控制器提供一个确定性候选，其余来自当前计划附近高斯；不能把这个有限池写成精确抽取上述连续最优分布。

**读后判断与用途边界。** A 类有至少四种不同主张：改善候选的时间形状、减少点集造成的更新误差、改变动力学后的覆盖，以及借外部知识偏置目标。它们可以叠加，部分方法还改代价、更新与执行规则。选用哪一种，应先观察失败候选究竟是来回抵消、重复拥挤、状态覆盖偏斜，还是根本没有任务知识。

**可分辨实验。** 在同一模型、相同有效动作扰动尺度与总 rollout 预算下，先独立开关时间相关矩阵，再比较随机与 LCD 点集；同时记录候选代价、分布参数变化及最终执行动作。另将差分代价和最佳样本执行单独开关，才有可能区分“搜索更有效”和“输出更平滑”。只画一条平滑曲线无法回答这些问题。

## B. 何时平均会毁掉解：先决定是否允许两个策略被混在一起

绕障的例子最直观：左绕和右绕都可行，二者控制的平均可能正好指向障碍。推物体时也类似，从物体两侧接触分别能形成有用的力矩，混合两段控制却可能哪边都没接上。非线性动力学下，控制均值的状态响应更不是两条状态轨迹的均值。因此，“每个样本都好”不足以推出“平均后也好”。

SVG-MPPI（[L35](<../papers/B_多模态/L35_2024_Stein_Variational_Guided_Model_Predictive_Path_Integral_Control_Proposal_and_Experiments_with_Fast_Maneuvering_Vehicles.pdf>)）对此选择尽早作决定。少量引导粒子先寻找一个优良模态，再把均值和粗略方差交给 MPPI；它删去 SVGD 中促分散的部分核与先验项来促聚集。Clustering-MPPI（[N12](<../papers/B_多模态/N12_2026_Clustering_Embedded_Model_Predictive_Path_Integral_Control_Avoiding_Averaging_Induced_Failure_and_Enabling_Efficient_Cluster_Selection_for_Dynamic_Obstacles.pdf>)）则先删除碰撞 rollout，按方向特征聚类，选一个簇后再加权。二者都试图让最后的平均发生在较一致的策略内部，区别在于前者主动搬移引导粒子，后者从已有样本中辨认分组。

“选一簇”仍不是安全证书。N12 Algorithm 1 在簇内加权后直接执行，未再独立 rollout 检查所得名义控制。同簇只是特征相近，不能保证其控制平均留在非凸可行域内。这个方法有针对跨路径平均的实机证据，读它时应把“减少一种失效”与“消除所有碰撞可能”分开。

SV-MPC（[L28](<../papers/B_多模态/L28_2020_Stein_Variational_Model_Predictive_Control.pdf>)）作了相反的取舍：用多个 SVGD 粒子表示后验，核排斥帮助多个候选同时存活。它可用 score-function 和蒙特卡洛 rollout 估计梯度，不必对仿真器求导。它与 SVG-MPPI 的差异不是有没有 Stein 名字，而是是否愿意为保留多个解持续付出粒子预算。

OT-MPC（[L24](<../papers/B_多模态/L24_2026_Sampling_Based_Control_via_Entropy_Regularized_Optimal_Transport.pdf>)）也保留多条完整控制序列，但用熵正则最优传输连接低代价提议与粒子，再做各自的几何重心更新。几何关系决定哪些好候选影响哪个粒子，避免所有质量一次汇到一个全局均值；执行时取最低代价粒子。其固定提议下的下降或驻点结论，不能直接覆盖在线不断重采样的全部过程。

多样性遇到约束时，单纯排斥还可能把轨迹推出可行域。Stein-ADMM（[L09](<../papers/B_多模态/L09_2026_Contact_Implicit_Stein_Projected_ADMM_for_Discovery_of_Diverse_Contact_Rich_Manipulation_Strategies.pdf>)）将 Stein 方向放进 ADMM 的投影变量，让多策略搜索与接触隐式约束求解配合。它在推、抓、交接中发现多种策略，代价是需要导数与投影求解。把这一机制原样加到黑箱 CEM 上，并没有同一套可行性依据。

另两篇把问题放在更早的提案阶段。MTP（[L21](<../papers/B_多模态/L21_2025_Model_Tensor_Planning.pdf>)）在随机多部图上组合控制路点，再插值得到分散候选，与局部高斯样本混合；它试图让“另一种策略”先进入候选池。其实机部署（[L03](<../papers/B_多模态/L03_2026_Real_World_Deployment_of_Massively_Parallel_Sampling_Based_MPC_for_Contact_Rich_Manipulation.pdf>)）在 Hydrax/MJX 上做 Franka Push-T，也保留最佳已评估样本。MTP 的覆盖定理限于渐近线性插值，不能授予有限样条候选同样保证；L03 是工作坊部署证据。

M3P2I（[N04](<../papers/B_多模态/N04_2023_Multi_Modal_MPPI_and_Active_Inference_for_Reactive_Task_and_Motion_Planning.pdf>)）讨论的模态又不同：高层符号动作各自有代价与均值，底层分别规划后切换或融合技能。这里的“推还是拉”由人工动作模板给出，不能与同一代价下自动发现多个后验模式画等号。它降低的是技能选择和连续控制脱节的困难，模板之外的新技能仍要另找来源。

**读后判断与用途边界。** 这组工作存在真实的预算分歧。若当前就必须选边、重新发现另一边很便宜，SVG 或聚类的早选策略可能合适；若障碍会移动、后续接触尚未确定，保留多个解可能有价值，但每个解分到的细化预算会减少。这里的“可能”是任务判断，论文之间尚没有统一预算下的普遍排序。

**可分辨实验。** 让左右两条路线当前代价接近，并在执行后改变其中一条的可行性；固定总仿真量，比较早选与保留多个解的切换延迟。完整方法比较应保留各自原有执行规则；另设附加机制对照，统一重评最终控制，并分别采用最佳样本或均值执行，以分清收益来自模式表示、模式选择，还是避免执行未评估的平均。

## C. 更新集中是否真等于收敛：先区分“相信谁”与“已经找到什么”

假设一批 rollout 里只有一条稍好。低温指数权重会几乎只相信它，CEM 的精英筛选也会把分布迅速收窄。此时有效样本数下降，可能表示发现了清楚的好方向，也可能只是所有候选都很差、其中一条没那么差。分布不再变化还可能是方差耗尽。权重集中、更新变小和目标接近最优，需要不同证据。

Tsallis-VI MPC（[L32](<../papers/C_更新规则/L32_2021_Variational_Inference_MPC_using_Tsallis_Divergence.pdf>)）让 MPPI 式软权重与 CEM 式精英截断进入同一变形指数框架，并讨论代价变换的风险偏好。它有助于理解“用多少样本说话”本身就是一个选择：宽权重保留信息，窄权重强调尾部。统一表达式没有消除该选择，也没有让高斯、混合高斯与 Stein 粒子的投影结果变成同一件事。

Trust-Region MPC（[N09](<../papers/C_更新规则/N09_2026_Sampling_based_Model_Predictive_Control_Using_Trust_Regions.pdf>)）尝试控制每次相信新证据的幅度：给下一提议分布设 KL 上界，可另加熵下界，通过对偶问题求权重，再做高斯矩匹配。它比 dsMPPI 的启发式调温更明确地规定更新约束；随机、Sobol、Halton 与 LCD 样本仍可独立选择。

但分布层面的受约束解与有限样本拟合出的高斯必须分开。N09 §IV 的矩匹配是近似，不能默认投影后仍严格满足原 KL 界；相关精确情形还依赖目标条件。因此，信赖域更适合回答“这一轮是否跨得太大”，不能独自回答“这个局部区域是否值得继续搜索”。

VR-MPPI（[L31](<../papers/C_更新规则/L31_2026_Variance_Reduced_Model_Predictive_Path_Integral_via_Quadratic_Model_Approximation.pdf>)）处理权重退化的方式不同。若代价里有一个二次近似能解释主要变化，先将它解析地吸收入高斯提案，用梯度与曲率引导均值、协方差，再只让残差承担指数重加权。这样做希望减少样本间难以估计的权重差异；它不是把所有权重简单压平，也不是只从精英方差推测曲率。近似失准时，残差仍需真实评分来修正。

PFMP（[N17](<../papers/C_更新规则/N17_2026_Sampling_based_Model_Predictive_Control_as_Sequential_Monte_Carlo_Inference.pdf>)）把长序列一次评分改为沿预测时域逐结点提粒子、积累权重，并在 ESS 不足时重采样，追溯祖先形成轨迹。它允许指定 mean、elite 或 MAP 等更新，由此组织 PS/MPPI/CEM 的特定形式。重采样后仍维持粒子预算，只是把后续计算重新分配给较好的前缀，不能据此认为总物理仿真步数必然减少。重复祖先也不等于创造新的接触策略；这也是为何 proposal 设计仍然重要。

**读后判断与用途边界。** C 类没有一种“正确温度”能替代问题结构。Tsallis 改权重形状，信赖域管更新幅度，VR-MPPI 管重要性权重的估计难度，PFMP 管证据沿时间进入的顺序。它们都可能让更新更稳，却各自保留了局部探索、投影近似或粒子贫化的问题。

**可分辨实验。** 除最终成本外，记录权重 ESS、实际高斯间 KL、最优已评估成本，以及把相同预算从当前分布重新扩散后是否还能明显改进。如果 ESS 很高而候选全差，问题是没找到方向；若名义更新很小而扩大探索立刻获益，停滞也不能当作接近最优。上述判据是建议，不是这些论文给出的统一收敛定理。

## D. 长时域接触为何难：成功可能先要求一段看起来退步的动作

继续看 Push-T。当前接触点只能把物体推向错误方向，机器人可能需要先离开、绕到另一侧，再建立新接触。短时域只看见离目标更远，长时域虽能看见最终收益，却要求一整段动作恰好协调。样本数增加并不自动补上这种连贯的策略变化；接触切换还会使相近控制对应完全不同的结果。

MBD（[L23](<../papers/D_地形与搜索方向/L23_2024_Model_Based_Diffusion_for_Trajectory_Optimization.pdf>)）和 DIAL-MPC（[L14](<../papers/D_地形与搜索方向/L14_2025_Full_Order_Sampling_Based_MPC_for_Torque_Level_Locomotion_Control_via_Diffusion_Style_Annealing.pdf>)）先改变搜索所看到的代价形状。MBD 在不同噪声层用蒙特卡洛估计 score，由粗到细优化，不训练去噪网络；原文主要是离线轨迹优化。DIAL 将 MPPI 解释为平滑密度上的单步 score ascent，再安排迭代和预测时间双层退火，使远期保留探索、临近执行逐步细化，并在全阶接触仿真中做四足力矩控制。

退火的吸引力在于，大尺度扰动可能把相邻接触策略之间的尖锐差异平均成可利用的方向。代价也正在这里：平滑后的最好解可能偏离原问题的最好解。DIDA（[L17](<../papers/D_地形与搜索方向/L17_2026_Global_Convergence_of_Sampling_Based_Nonconvex_Optimization_through_Diffusion_Style_Smoothing.pdf>)）将这种“扩大适宜搜索的区域”与“移动最优位置”的权衡明确化，联合退火噪声和温度；其邻域收敛分析依赖次高斯尾、最优点附近强凸光滑等条件，实验主要是高维黑箱函数。它不能直接为有限预算 DIAL 或任意接触任务作全局收敛证明。

Reference-Free SMPC（[L08](<../papers/D_地形与搜索方向/L08_2026_Reference_Free_Sampling_Based_Model_Predictive_Control.pdf>)）展示了另一条常被退火名字遮住的线：位置与速度双控制点的 Hermite 样条、保留最佳已仿真轨迹、计算延迟补偿，一起支持 Go2 的无参考步态。普通 CPU 上的结果不能归因于“换一个噪声日程”。其复杂动作与 G1 展示的证据范围也要与 Go2 实机分开。

GRASP（[L16](<../papers/D_地形与搜索方向/L16_2026_GRASP_Gradient_Guided_Sampling_for_Contact_Rich_Physics_Based_Motion_Retargeting.pdf>)）则不愿丢掉大量已经付费算过的成本。它保存已评估及被拒候选，在降维时间坐标里拟合局部二次模型，用恢复的方向引导 tempered SMC 的 MCMC 再生，再以真实代价做 MH 校正。它不要求仿真器导数，但历史点拟合出的方向仍是代理，不是精确 score。这里的 SMC 沿人工温度轴推进，与 PFMP 沿物理预测时间的祖先重采样不同。

KernelSOS / Global-MPPI（[L02](<../papers/D_地形与搜索方向/L02_2026_Global_Sampling_Based_Trajectory_Optimization_for_Contact_Rich_Manipulation_via_KernelSOS.pdf>)）把代理放得更广：先用 LSE 平滑，再建立 KernelSOS 全局代理，通过 SDP 提出候选，最后交给 MPPI 局部细化。它与 GRASP 都在问“能否从成本样本恢复方向”，一个偏局部历史复用，一个承担更重的全局代理求解。SDP 成本随样本增长，名称中的 Global 不能替代对具体问题、有限样本和计算成本的检查。

如果模型可微，CEM-GD（[N18](<../papers/D_地形与搜索方向/N18_2021_CEM_GD_Cross_Entropy_Method_with_Gradient_Descent_Planner_for_Model_Based_Reinforcement_Learning.pdf>)）选择直接利用导数：先用 CEM 找到若干高回报候选，再做模型梯度与步长搜索，以热启动减少后续采样。它与 GRASP 的取舍很清楚：前者要可靠的模型导数，后者付出额外评估来回归局部方向。两者仍依赖初始覆盖；梯度能精修当前策略，却未必会告诉机器人应当去物体另一面。

另一些论文通过改变问题，避开对长时域盲搜的依赖。CDF-MPPI（[L25](<../papers/D_地形与搜索方向/L25_2026_One_Step_Model_Predictive_Path_Integral_for_Manipulator_Motion_Planning_Using_Configuration_Space_Distance_Fields.pdf>)）从配置空间距离场读取避障梯度，以局部几何支持单步采样；它的速度积分器规划不能直接替代力矩级接触控制。DiffContact（[L12](<../papers/D_地形与搜索方向/L12_2026_DiffContact_Differentiable_Contact_Dynamics_for_Policy_Learning_and_Contact_Implicit_Trajectory_Optimization.pdf>)）在反向传播中松弛摩擦接触的锥互补条件，前向仍保留硬接触解，为 BPTT 和 DDP 提供连续敏感度；松弛方向也不是原非光滑目标处处的精确导数。

**读后判断与用途边界。** 长时域接触同时包含搜索概率低、远期评价难、接触方向突变和模型误差积累。退火、历史代理、可微接触各处理其中一部分。若坏结果来自接触位置始终没换过，再细的局部二次模型也未必够；若候选已经找对接触而只差精度，大范围探索反而浪费预算。

**可分辨实验。** 固定控制参数化和最佳样本执行，分别增加时域、噪声范围与更新轮数，记录新接触模式首次出现的时刻、出现后能否持续改善，以及模型中的改进能否在真实系统兑现。退火实验还应在原始未平滑代价上重评候选。这样才能区分“终于看见远期收益”“找到新策略”和“只是更善于优化平滑后的问题”。

## E. 哪些事该交给 solver：可行性不必全靠高代价教育样本

若机械臂候选大多违反关节速度或加速度界限，单靠罚项会把预算用于区分不同程度的违规。PRISM-QP（[L04](<../papers/E_约束与接触/L04_2026_PRISM_Projection_Integrated_Sampling_Based_MPC_with_Bayesian_Cost_Tuning_for_Bimanual_Manipulation.pdf>)）先将宽分布采样的关节速度序列用并行 QP 投影到位置、速度、加速度、jerk 与初速边界，再 rollout 评分。它把“满足明确的离散运动学界限”交给结构化求解，将剩下的采样预算用于任务选择。

这类分工的保证范围很具体。投影保证的关节界限不自动包含碰撞、摩擦或接触稳定；最终加权与执行方式也需要核查。它与另一个以世界模型先验为主题的 PRISM（N20）只是同名，不能把两篇的机制拼在一起。

DPCC（[L13](<../papers/E_约束与接触/L13_2025_Diffusion_Predictive_Control_with_Constraints.pdf>)）与 UniConFlow（[L30](<../papers/E_约束与接触/L30_2026_UniConFlow_A_Unified_Constrained_Flow_Matching_Framework_for_Certified_Motion_Planning.pdf>)）将修正插进生成过程。前者在已训练扩散模型每次去噪时，投影动力学及状态动作约束，并依据模型误差上界收紧状态集；后者在已训练流匹配生成器的推理 ODE 中加入 QP 最小修正，再用违规片段、终端修正和样条压缩减少开销。两者都是让既有生成器适配约束，不能把免重训写成生成器无需训练。

这两篇也提醒读者区分连续推理理论与实际离散算法。DPCC 的可行性条件不能直接覆盖所有非凸投影与任意模型误差；UniConFlow 的连续 ODE 结论需要与数值积分和终端修正结果分别读。“给生成器加一个 QP”描述了结构，尚未说明完整闭环的递归可行性。

Sampling+C3（[L01](<../papers/E_约束与接触/L01_2025_Approximating_Global_Contact_Implicit_MPC_via_Sampling_and_Local_Complementarity.pdf>)）提供更有针对性的分工：外层只采少量末端重定位位置，内层在这些位置并行求局部 C3 接触控制，再用代价、迟滞与停滞检测决定继续接触还是换位置。Franka 的 Push-T 与三维非凸物体操作说明，局部求解器不必独自负责发现所有接触位置。它的外层也不必在整条高维力矩序列上承担全局探索。

Shooting for Contact（[L27](<../papers/E_约束与接触/L27_2026_Shooting_for_Contact_Contact_Implicit_Multiple_Shooting_for_Dynamic_Motion_Retargeting.pdf>)）进一步将长时域状态连接交给多重射击 NLP。MuJoCo 负责各区间接触转移，节点状态与控制样条共同优化，缺陷约束连接区间，无需预先指定接触日程。它是动态重定向与后续动作模仿训练的路线，不能因为用了仿真器就把它归为 CEM 或 MPPI 变体。

**读后判断与用途边界。** 当约束结构清楚，或局部 solver 已能精确完成一种接触，采样最有价值的位置可能是离散策略、重定位点和初始化。L01 的启发是减少交给采样的职责，而非宣称采样与 solver 结合后就有全局保证；L27 也不代表多重射击没有计算成本与局部求解困难。

**可分辨实验。** 报告投影前后可行候选比例、投影距离、solver 耗时和最终执行轨迹的违规量。在同一总延迟下比较“多采样”“投影后少采样”与“外层低维采样加内层求解”，才能判断收益是避免无效候选、改善初始化，还是更准确地处理了约束。

## F. 哪些事该交给模型和先验：别把优化能力与未来预测能力混为一谈

推物体的仿真若把摩擦估错，优化器可能越努力，越确信一种真实系统做不到的滑动。此时低预测代价与实际成功会分离。另一种情况则是模型够准，但从随机控制重新发现已知技能太贵。前者要求改善预测或反馈，后者更适合借助提案先验；两类论文的收益不能放进同一个“样本效率”数字里。

GPC（[N15](<../papers/F_模型与先验/N15_2025_Generative_Models_from_and_for_Sampling_Based_MPC_A_Bootstrapped_Approach_for_Adaptive_Contact_Rich_Manipulation.pdf>)）保留物理 rollout，用成功 SPC 运行的开环序列训练条件 flow，再与 CEM 样本混合、统一评分。学习部分的职责是提出有希望的动作，不是替代动力学。它的可用范围仍受成功轨迹覆盖和固定目标影响；若训练里从没出现过一种接触，先验也可能使搜索更难离开熟悉动作。

PO-MPC（[N13](<../papers/F_模型与先验/N13_2025_A_KL_regularization_Framework_for_Learning_to_Plan_with_Adaptive_Priors.pdf>)）研究规划与学习之间的循环：规划数据训练自适应先验，策略和价值以回报及 KL 正则协调更新，并比较正向与反向 KL。它的 KL 在学习阶段约束策略与规划先验的关系，区别于 N09 对一次在线分布更新的信赖域；推理仍沿用 MPPI。证据主要来自模拟 RL，不能当成无需训练的控制技巧。

PRISM-JEPA（[N20](<../papers/F_模型与先验/N20_2026_PRISM_PRior_guided_Imagination_Sampling_in_world_Models.pdf>)）冻结 JEPA 编码器与动力学，只训练从当前、目标嵌入预测动作均值和不确定性的轻量先验，再与默认高斯提案按精度融合。它便宜地利用了表征信息，同时把一个危险假设显露出来：若先验既错又过度自信，融合会压窄正确方向的探索。这与 N03 有意偏置目标的选择相邻，却不是同一种权重推导。

D-MPC（[L06](<../papers/F_模型与先验/L06_2025_Diffusion_Model_Predictive_Control.pdf>)）把更多工作交给学习，同时训练扩散动作提案与多步扩散动力学，再生成、预测和评分，执行最佳候选。它与 GPC 的差别在于 rollout 也由学习模型承担，与 MBD/DIAL 的差别在于需要离线数据和训练。测试时换奖励或微调动力学的便利，不消除模型外推风险。

JEPA-WM 的设计研究（[L07](<../papers/F_模型与先验/L07_2026_What_Drives_Success_in_Physical_Planning_with_Joint_Embedding_Predictive_World_Models.pdf>)）系统改变视觉编码、动作条件化、训练与规划器，经验上发现 CEM、L2 距离及 AdaLN/RoPE 组合有效。它说明同一个“CEM 规划”名义下，表征和动力学接口也会主导结果。Dream-MPC（[L34](<../papers/F_模型与先验/L34_2026_Dream_MPC_Gradient_Based_Model_Predictive_Control_with_Latent_Imagination.pdf>)）则在潜世界模型中从策略先验产生少量候选，再反传回报梯度，用 Q 集成不确定性惩罚抑制外推。这里比较的是学习模型、先验与梯度的组合，不能把全部收益记到更新规则名下。

Dual-MPC（[N22](<../papers/F_模型与先验/N22_2026_Dual_Model_Predictive_Control_through_Sampling_Based_Belief_Propagation.pdf>)）进一步问动作能否主动帮助识别动力学。它对未知模型与噪声采样，在预测中同时传播物理状态和贝叶斯后验，优化会响应未来信息的 hyperstate 反馈策略。它不是另一个 MPPI 权重公式；样本平均近似的一致性也只针对选定的紧致策略类与固定有限时域。

对人形系统，直接搜索力矩可能还不是合适的接口。TTC（[L29](<../papers/F_模型与先验/L29_2026_Test_Time_Compute_for_Humanoid_Behavior_Foundation_Models.pdf>)）冻结行为基础模型，在闭环物理仿真中搜索输入命令，使控制器更准确地完成目标动作，并用经验误差模型安排测试预算。KODA（[L19](<../papers/F_模型与先验/L19_2026_KODA_Kinematics_Observation_Data_Alignment_for_Humanoid_Learning_from_Human_Egocentric_Demonstration.pdf>)）也在冻结控制器闭环中用 CEM 优化稀疏位姿残差，但服务于人类第一视角示范到机器人轨迹、视图与跨控制器标签的转换。共同点是搜索经过控制器；各自贡献分别在测试算力与数据转换，不能说它们发明了新的 CEM。

PhysibleMDM（[L20](<../papers/F_模型与先验/L20_2026_PhysibleMDM_Enforcing_Inference_Time_Physical_Feasibility_Constraints_on_Motion_Diffusion_Models_via_Noise_Optimization.pdf>)）选择搜索冻结运动扩散模型的初始及反向噪声，用 Sep-CMA-ES 减少穿地、滑移、自碰撞与逆动力学残差。Optimization-Guided Diffusion（[L26](<../papers/F_模型与先验/L26_2026_Grounding_Generative_Policies_in_Physics_Optimization_Guided_Diffusion_for_Robot_Control.pdf>)）也把生成过程扰动当作变量，但用非线性最小二乘与 L-BFGS 纠偏。两者优化时无需重训生成器，生成器本身仍已训练；物理代价或非凸松弛的降低，也不能直接等同于全阶闭环可执行性的认证。

Feedback-MPPI（[L15](<../papers/F_模型与先验/L15_2026_Feedback_MPPI_Fast_Sampling_Based_MPC_via_Rollout_Differentiation_Adios_low_level_controllers.pdf>)）把问题拉回执行时刻：较好的开环计划仍会在两次重规划之间受到扰动。它对加权控制解关于初态求导，得到局部反馈增益，让低频 MPPI 给出名义动作、高频环及时修正。收益来自反馈接口与频率分工，也付出额外微分成本；硬 indicator 约束不能直接提供有效梯度。

**读后判断与用途边界。** 先验负责让熟悉的好行为更容易出现，动力学模型负责预测这些行为的后果，搜索负责在当前目标下比较，反馈负责处理执行中的偏离。把职责写清楚，才能判断应训练一个提案、校准摩擦、换命令变量，还是增加重规划频率。一个强先验也可能掩盖优化器较弱，一个错误模型则可能让所有优化器一起失效。

**可分辨实验。** 固定模型比较提案，固定提案比较模型，再同时报告预测代价与真实闭环结果。对冻结控制器的命令搜索，应同时报告目标命令、优化后输入和实际运动误差；对物理纠偏生成器，应将代价下降与仿真执行成功分开。这样才知道算法改善了“会提出什么”，还是“能否正确判断它”。

## G. 预算和工具：把共同工程部件摆到比较表面

Predictive Sampling（[N02](<../papers/G_工具预算与背景/N02_2022_Predictive_Sampling_Real_time_Behaviour_Synthesis_with_MuJoCo.pdf>)）是一个值得认真保留的简单基线：在低维时间样条上固定尺度扰动，连同原序列一起评估，直接留最佳。它不做 MPPI 指数平均，也不拟合 CEM 精英分布，作者明确不主张算法突破。若新方法还没超过这个基线，先检查是不是高维表示、均值执行或实现开销吃掉了预算。

STORM（[N19](<../papers/G_工具预算与背景/N19_2021_STORM_An_Integrated_Framework_for_Fast_Joint_Space_Model_Predictive_Control_for_Reactive_Manipulation.pdf>)）早已组合 GPU 并行、协方差适应、Halton 与 B 样条平滑采样，并使用学习碰撞代价；其主体是关节空间运动学近似。Parallel-Physics MPPI（[N14](<../papers/G_工具预算与背景/N14_2023_Sampling_based_Model_Predictive_Control_Leveraging_Parallelizable_Physics_Simulations.pdf>)）把 IsaacGym 作为前向模型，保留 Halton 样条与有效权重调温，并允许随机化物理参数。它们提示一条归因底线：今天的平滑采样、热启动和并行物理闭环已有工程前史，不能把继承部件都当作新算法的效果。

MPPI-Generic（[N10](<../papers/G_工具预算与背景/N10_2024_MPPI_Generic_A_CUDA_Library_for_Stochastic_Trajectory_Optimization.pdf>)）将动力学、代价、采样与反馈做成可替换的 C++/CUDA 部件，支持 MPPI、Tube-MPPI 和 RMPPI，重点在实现与性能工程。Judo（[N16](<../papers/G_工具预算与背景/N16_2025_Judo_A_User_Friendly_Open_Source_Package_for_Sampling_Based_Model_Predictive_Control.pdf>)）以 MuJoCo 多线程 rollout、样条节点及 PS/CEM/MPPI 统一接口提供任务、优化器与调参 GUI，属于工作坊软件稿。它们的用途是让不同选择更容易在共同栈上比较；库的吞吐量不能直接替代目标机器人上的控制延迟。

BAMPC（[L05](<../papers/G_工具预算与背景/L05_2026_Do_My_Samples_Help_Compute_Budget_Allocation_for_Real_World_Sampling_Based_MPC_in_Contact_Rich_Manipulation.pdf>)）把这个问题做成实机研究：固定 GPU 预算究竟投给更多控制候选、更多状态和动力学假设，还是更频繁重规划与时间对齐？三个真实接触任务说明，答案取决于主要的 sim-to-real 误差，增加域随机化并非普遍优于控制探索。其比较固定协方差、只更新均值，不能默认等同于所有 full-CEM 实现。

这也与 L03 的部署观察呼应：重规划间隔短时，质量、摩擦差异在短预测中可能缺少足够反馈。**读后判断：** 更多物理假设只有在预测与评分能区分它们时才有价值；若延迟错位已经主导实际误差，先增加样本数可能只会继续拖慢控制。BAMPC 给出的是具体任务内的预算证据，不是各类机器人共享的固定配比。

概念上可先读概率推断综述（[N21](<../papers/G_工具预算与背景/N21_2025_Model_Predictive_Control_via_Probabilistic_Inference_A_Tutorial_and_Survey.pdf>)），理解先验乘 Boltzmann 似然、再做分布投影的关系；零阶优化导论（[L33](<../papers/G_工具预算与背景/L33_2025_An_Introduction_to_Zero_Order_Optimization_Techniques_for_Robotics.pdf>)）进一步用随机平滑与自然梯度连接 MPPI、CEM 和演化策略。Derivative-free optimization methods（[L11](<../papers/G_工具预算与背景/L11_2019_Derivative_free_optimization_methods.pdf>)）则帮助区分直接搜索、代理信赖域与高斯平滑随机搜索。这些是阅读工具，具体收敛条件仍须回到各方法原文。

Cross-Entropy Randomized Motion Planning（[L10](<../papers/G_工具预算与背景/L10_2011_Cross_Entropy_Randomized_Motion_Planning.pdf>)）在早期轨迹规划中已用精英拟合高斯或混合分布，并明确有限采样可能停在局部解。因此，“在轨迹空间用 CEM 找不同路径”本身不能作为近期方法的新意；新意应落在表示、提案、约束接口或预算安排的具体差别上。

还有两篇适合作为思路参照，但应放在直接 SMPC 证据之外。MPOT（[L22](<../papers/G_工具预算与背景/L22_2023_Accelerating_Motion_Planning_via_Optimal_Transport.pdf>)）用熵正则传输搬移配置路点，与 OT-MPC 搬移完整控制序列不同；GSBM（[L18](<../papers/G_工具预算与背景/L18_2024_Generalized_Schr_dinger_Bridge_Matching.pdf>)）研究带状态成本的分布桥，将边缘匹配与端点条件随机控制相连。它们有助于思考几何与分布控制，却没有替本文那些机器人闭环算法完成验证。

**读后判断与用途边界。** 最有解释力的比较应共享模型、控制表示、热启动、最终动作选择与延迟处理，再说明新增部分。否则“算法 A 比 B 好”很可能混合了样条降维、最佳样本执行和更快实现三种收益。工具论文的价值正在于降低这种混杂，而非自动证明某个更新公式更好。

**可分辨实验。** 固定真实控制周期和硬件，报告完整周期延迟、有效 rollout 数、每条时域长度与更新轮数；将状态随机化和控制探索分账。时间预算相同的结果回答部署选择，rollout 预算相同的结果帮助判断搜索机制，两种口径都应写清楚。这里提出的是比较原则，没有新增实验结果。

## 从这些分歧中保留什么

对一个已经跑起来的接触 SMPC，第一步应当是定位失败发生在哪一处：候选没有持续作用、不同策略被平均、分布过早收窄、远期收益没有被看见，还是仿真改进无法兑现。A–G 是诊断入口，不是彼此排斥的算法门派。比如 Biased-MPPI 可借先验跨模式，dsMPPI 同时改变点集和更新，Stein-ADMM 同时处理多样性与约束。

这份阅读给出的研究判断是：先用少量有解释力的改动建立强基线，再决定是否需要更复杂的分布或 solver。时间相关、低维表示、热启动、最佳样本执行和时间对齐值得被显式控制；只有这样，新方向的收益才有机会被看清。若它们已经解决了问题，应承认问题来自工程配置；若它们仍失败，失败轨迹才是选择下一层机制的依据。

最后保留几个会直接读错论文的定位：L08 是 Reference-Free，DIDA/Global Convergence 是 L17；L19 是 KODA，L20 是 PhysibleMDM，均不能因旧文件简称误认成别篇。N07、N08、N09 是独立论文；GRASP 与 PFMP 的 SMC 推进轴不同；L04 与 N20 的 PRISM 机制不同。PDF 主分类和编号保持原样，跨类联系在正文讨论，不重复存放论文。
