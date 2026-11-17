# 完整上下文交接：Loco_maniplation → SMPC

更新：2026-11-16。本文保存整个对话的主要研究脉络、已有工程与实验、失败诊断、当前决定和资产位置。**不能只用最近的“选择性重开”覆盖此前研究。** 数字注明来源与历史口径；此次补写没有重新运行这些实验。

## 1. 接手时先明确用户究竟在做什么

最终任务是人形机器人从不同初始位置接近箱子、建立双掌接触、搬起、携物行走并放到目标支撑面。条件包括机器人—箱体相对位姿、距离、箱体尺寸、质量，抓取面、接触位置及时间也可能变化。

最终部署架构已确定：**上层条件参考生成器＋下层 SuperMimic WBT tracker**。以少量种子产生大量 kinematic/dynamic feasible 条件轨迹，再训练 diffusion/flow；以后适配 tracker、联合后训练解决交互残差。当前最关心的是**如何更快、更可靠地产出可行数据**，不是立即训练最终搬运策略。

WBT在用户设想中有三处作用：规划工厂内辅助CEM并闭环执行候选；利用可行数据训练/适配交互tracker；最终生成器下执行并联合微调。不要再次把它说成“仅部署末端才需要”，也不要自行更换SuperMimic。

当前已运行的主优化器仍是**PD目标控制节点＋Warp动力学＋原版SBTO完整协方差CEM**。研究文档中的“逐候选由SuperMimic执行参考”是另一层尚未完整落地的路线。两者不能混称已完成。

SMPC有两条工作线：通用CEM/MPPI算法创新，在多任务上验证；以及G1离线多接触轨迹优化，SMPC可以是更大规划框架中的组件。用户瞄准RSS2027，也重视有价值的中间成果；不要保证录用，不要仅拼组件便宣称创新。

## 2. 当前仓库、数据与文档在哪里

| 位置 | 用途 |
|---|---|
| `/home/songchao/G1_Locomaniplation/SMPC` | 当前通用SMPC研究目录 |
| [src/hydrax](src/hydrax) | 本窗口修复的通用实验台 |
| [Loco_maniplation/g1-trajopt](../Loco_maniplation/g1-trajopt) | G1算法修改、优化、播放、审计统一入口，已移出baseline |
| [原始运动学工作库](../Loco_maniplation/g1-trajopt/data/flat_palm_library) | IK平掌NPZ输入 |
| [优化结果工作库](../Loco_maniplation/g1-trajopt/data/optimized_library) | 控制、轨迹、分布checkpoint、冻结问题，含旧rejected |
| [历史研究档案](../Loco_maniplation/g1-trajopt/docs/research_history/contact-transfer) | 五分支、DFO、约束恢复、条件延拓的核心MD |
| [sources](../Loco_maniplation/g1-trajopt/data/sources) | 数据来源、DiPLO官方数据、旧研究/基线压缩归档 |

五个baseline保留在 `Loco_maniplation/baseline/{humanoid-dart,hydrax,McVAMP,sbto,OT-retarget}`。**baseline/hydrax与SMPC/src/hydrax是两个目录。** 原`baseline/g1-trajopt`和`baseline/contact-transfer`已不存在。旧contact-transfer源码/工作树不能按旧README直接运行，应先查`sources/legacy_research.tar.gz`等归档；旧outputs已按清理要求删除。

主要文献入口：

- [01_PAPER_MAP.md](doc/01_PAPER_MAP.md)：57篇逐篇算法机制与关联。
- [按痛点重组的文献地图](<doc/Sampling MPC 文献地图（按「痛点」重组版）.md>)：跨论文分析。
- [02_RESEARCH_AND_FRAMEWORK.md](doc/02_RESEARCH_AND_FRAMEWORK.md)：通用算法、搬运框架及组件分工。
- [03_CEM_IDEA_REVIEW.md](doc/03_CEM_IDEA_REVIEW.md)：本窗口用rubric完成的最新评估，不是新实验报告。

**阅读旧文档时要辨认日期与成熟度：**`CEM_CONTACT_RESEARCH_PLAN.md`的pp_cem是11月8日旧主线；11月15日`CEM_MULTICONTACT_RESEARCH_PLAN.md`已明确暂停它。contact-transfer旧README/PLAN中“运行中、全部产物保留”也是历史状态，后来的清理说明优先。

## 3. 早期工程为什么逐渐走向物理优化

### 3.1 DART平掌改造：几何成功，不能自动推出承载成功

来源：[FLAT_PALMS_TRACKING](../Loco_maniplation/baseline/humanoid-dart/FLAT_PALMS_TRACKING.md)。原始87条、70Hz、34381有效帧；平掌处理保留箱体、根、腿、腰，只改14个臂关节。双掌共同接触段，每掌128个边界点，离面≤20μm、法向≤0.02°、边余量≥2mm。

87条通过的是**掌面运动学几何检查**，不是全身无碰撞、摩擦承载或动力学验收。后来用户反复指出身体穿模、足底悬空、放箱悬空，正是从双臂IK升级到全身与物体联合物理优化的动机。

G1仍为29关节，质量/惯量与执行器保留；自由刚体箱子、平掌碰撞及摩擦真实参与仿真。当前GPU-CEM工程保留六边形平掌视觉，并曾将复杂掌/脚凸包碰撞简化为盒；这一选择不能无检查套用于DART、OT-retarget或McVAMP，它们各自的碰撞模型要单独核对。不能把视觉又改成胶囊或普通手。原DART与SuperMimic checkpoint格式不同，不能互换播放器。

### 3.2 平面接触三原则与ci_loco：已经进入独立任务

来源：[软约束设计](../Loco_maniplation/baseline/humanoid-dart/PLANAR_CONTACT_SOFT_CONSTRAINT_DESIGN.md)、[CI_LOCO](../Loco_maniplation/baseline/humanoid-dart/CI_LOCO.md)。代码在 `baseline/humanoid-dart/src/humanoid_dart/ci_loco/`。

研究的是保留MuJoCo状态转移，在PPO中对用户指定的连续薄层接触本构加入软正则：

- 法向：由真实间隙、同一步末速度、固定材料参数给出压力分布；比较其面积积分与MuJoCo实际法向冲量/倾覆矩，形成RN。
- 切向：**固定同一个压力分布**，构建可实现切向wrench集合T[p]，用 `RT = LambdaT − Proj_T(LambdaT − beta*nuT)` 同时衡量集合成员及最大耗散固定点。

这不是仅罚滑移，也不是从净wrench唯一反演压力，更不是罚一个自构造后恒零的互补积。MuJoCo的`condim=3,pyramidal`切向截面是diamond，与旧方形摩擦截面不同。必须逐物理子步对齐wrench、末速度、参考点和作用方向，不能拿最后一步力乘整个控制周期。

已有：保留29维动作、562维actor/700维critic、原14奖励，加`ci_normal/ci_tangent`；固定压力投影用FISTA；35项专项＋6项导入/注册/推进检查；8环境、2次PPO更新及checkpoint/play smoke。**没有长训收益或大规模吞吐证明。** pressure-gradient默认1e6 N/m³尚未与MuJoCo标定；离散投影误差证书不覆盖连续积分和任意大倾角。

另有[接触稳定性审计](../Loco_maniplation/baseline/humanoid-dart/docs/contact_stability_audit/README.md)：历史model_28000受力约10N/掌，但方向误差约44°、有效触点跨度中位约1.18mm，提示接触面积/抗扭能力不足。不能用“夹持力够大”代替稳定抓持分析。

### 3.3 固定脚闭链与可行流形研究

来源：[固定双足双臂搬箱文献](../Loco_maniplation/baseline/humanoid-dart/FIXED_FEET_BIMANUAL_LIFTING_LITERATURE_2026.md)。这是早期局部pickup简化，不是用户最终永久禁止行走。

机器人29关节＋浮动基6＋箱体6，共41维切空间。规则点若双足/双掌完整24个约束独立，局部速度空间17维；这是局部秩计算，不是全局恒定维数。单纯平面共面通常只有法向间隙＋两项法向对齐共3个约束，面内平移及转角仍自由。`ker J_c`的允许运动与`ker G`的抓取内力不同。

固定脚没有删除足底反力、摩擦、CoP、限矩和浮动基动力学；双掌共面也没有证明承载。`2μN≥mg`只是侧夹静态承载的直观筛查。双手脚标签不变时，蹲下、站直仍需不同关键构型；箱底离开支撑面本身是物体—环境接触变化。

文献分工曾核查FARO、PATACON、CoMET/Hippo、CQDC-Warp、DSMS和Caron CWC等。它们分别贡献已知模式优化、几何闭链、准动力学、动态多重射击或受力约束，不是现成G1平掌全任务解。开源状态与速度须按版本复核，不能从论文数字承诺本机几秒求解。

### 3.4 离线教师—生成—执行的完整合同

来源：[离线教师与生成器](../Loco_maniplation/baseline/humanoid-dart/OFFLINE_CONTACT_TEACHER_AND_GENERATOR.md)、[局部拾放范围](../Loco_maniplation/baseline/humanoid-dart/docs/research_box_2026/LOCAL_PICK_PLACE_SCOPE.md)、[奖励与生成规划总报告](../Loco_maniplation/baseline/humanoid-dart/BOX_MANIPULATION_REWARD_AND_GENERATIVE_PLANNING_REPORT_2026.md)。

输入已经是G1粗轨迹，无须为采用联合优化思想重新人体重定向。参考应提供初值和适度偏好；根、腿、腰、臂、箱体路径、接触位置与持续时间应有共同调整机会。初态由任务给定，不能偷偷移动机器人降低难度。全局刚体旋转属于坐标对称，不等于新的相对行为；尺寸变化要同步惯量、有限掌面与碰撞。

数据需要KIN-pass、STATIC-pass、EXEC-pass分级，没解出的wrench应留空。训练于可行样本不保证生成样本可行；逐帧手腕IK不等于全轨迹约束修正。place包含建立支撑、卸载和撤手，不是pickup倒放。数据划分按源片段/镜像组及条件区域防泄漏。

Pinocchio/CasADi/IPOPT、KOMO曾是候选教师底座，不能写成已实现并胜过SBTO。DART发布接口曾发现：默认扩增主要relative_x/y；`object_global_ori`实际调用位置误差；某`replanning.enabled`分支抛NotImplementedError。论文完整能力与本地发布实现要分开。

## 4. 两个几何规划baseline做到了哪里

### OT-retarget

来源：[README](../Loco_maniplation/baseline/OT-retarget/README.md)。先复现OT思想，再接平掌、实体箱、全身与箱体联合窗口SQP；有足底锚点、摆动脚非穿透、速度/加速度、自碰撞、整掌有限面约束。Phase-I/HiGHS先恢复可行，再跟踪风格。

历史具体成果：`sub7_largebox_029`源帧[114,174)的60帧放箱段，抱箱下降→接地→松手，独立KIN验收accepted，掌穿透约0.2901mm、箱地约0.2900mm、脚支持覆盖100%。30帧窗口/22帧前瞻改善放箱阶段可见性。**208帧完整任务未交付；只验证运动学，播放用mj_forward，不含摩擦力/动力学平衡。** README记403测试＋13子测试，是历史记录非本次重跑。500条OMOMO输入不是500条通过G1，旧数据和输出还可能已被清理。

### McVAMP柜顶取箱到货架

来源：[独立任务README](../Loco_maniplation/baseline/McVAMP/tasks/g1_cabinet_to_shelf/README.md)。六阶段 `approach/close/lift/transfer/place/release`；接近/撤手笛卡尔引导＋全身投影，持箱段RRTC搜索固定脚、CoM、双手相对位姿及箱体竖直通道。上游球碰撞筛查后，以真实凸包/掌板/箱体/货架复查。

历史默认例：0.20×0.22×0.18m箱从z=.45柜顶到z=.7975中板，488帧、11.5s，规划约162s；离散几何验收环境穿透0、接触点误差约.162mm、足底间隙.206–.747mm。**固定顺序KIN规划，未验证负载平衡、摩擦承载、连续无碰撞或自动接触发现。** 输出`q_mcvamp/T_box/phase/contact_mode`不是当前统一NPZ，不能未经转换就喂给g1-trajopt。

## 5. DiPLO与数据资产：来源、规模、质量不是一回事

来源：[数据总说明](../Loco_maniplation/g1-trajopt/data/README.md)、[工作库说明](../Loco_maniplation/g1-trajopt/data/flat_palm_library/README.md)、[全库分类](../Loco_maniplation/g1-trajopt/docs/IK_TRAJECTORY_CATALOG.md)、[DiPLO官方说明](../Loco_maniplation/g1-trajopt/data/sources/diplo/official/README.md)。

DiPLO链为 **OMOMO→OmniRetarget→29个largebox种子→898条未对称增强SBTO解→D4八对称7184解**。官方29源×8版本＝232目录，50Hz、294–572帧；`kinematic_traj.npz`是粗参考，`sbto_traj.npz`含批量tube，不是一文件一轨迹。官方说明raw solves未精筛，头尾插值未仿真，也只有一种largebox。当前`pick_place/diplo`的29条是后来适配/精修的平掌IK种子，不能说7184条都已改平掌或通过当前物理验收。

迁移/分类文档登记（本次未逐条重验）：

| 资产 | 数量与口径 |
|---|---|
| IK工作库 | 19720：pick_place3556、carry_walk4424、other_interaction11740 |
| 物体质量类别 | 声明箱体双手2626、ULTRA双手代理1938、其他箱体/代理1940、重设计道具11899、unknown1317 |
| 重设计道具 | `geometry_redesigned_unoptimized`，含门/家具/清洁/拐杖；不是已可行箱体搬运 |
| 接触/质量缺口 | 1374缺手标签＋284闪断后无有效标签；19719条缺真实质量元数据，默认0.6kg不是多重量数据覆盖 |
| 双手声明箱体动作模式 | 起箱65、放箱67、拾放3、拾搬放24、carry2246、held_adjust23、其他接触198 |
| 优化主库 | 147完整结果：63 complete、84 rejected；另1完整实验与1未完成checkpoint，即148个完整结果可播放 |

schema/geometry预检ready不等于IK/动力学成功；旧rejected也不等于文件毫无用途。carry标签按根部移动统计，不等于实际步态。旧7252运行副本曾核对硬链接后清理，不能当7252独立优化。来源变体/镜像/修复前版本不得重复算独立任务；运动数组一致也不代表物体几何元数据相同。

优化保存控制、实际状态、完整协方差mean/cov/RNG/阶段和冻结problem，便于resume或warm start。生产脚本按source hash跳过已收集来源，**收集不等于验收通过**；改条件需看`include-collected`规则，不能误跳所有条件。主配置一份YAML批量展开，不要每轨迹新造JSON。

原始起箱65条的同窗播放（该工具用`[`/`]`）：

```bash
cd /home/songchao/G1_Locomaniplation/Loco_maniplation/g1-trajopt
.venv/bin/python scripts/classify_ik_library.py \
  --play --scope working --role bimanual_box --pattern pick --speed 0.5
```

优化库播放（这个播放器用N/P、Space、R）：

```bash
env -u PYTHONPATH -u JAX_PLATFORMS MUJOCO_GL=glfw \
XLA_PYTHON_CLIENT_PREALLOCATE=false \
.venv/bin/g1-trajopt play data/optimized_library
```

## 6. CPU SBTO→GPU Hydrax→独立g1-trajopt的演进

用户要求照原SBTO复现。中间曾采用Hydrax对角CEM，后来恢复原SBTO完整协方差；两套结果不能混算同一基线。CPU环境现位于`baseline/sbto/.venv`，当前主项目使用自己的`.venv`与包内实现，不应依赖另两个仓库运行时拼接。

[VALIDATION](../Loco_maniplation/g1-trajopt/docs/VALIDATION.md)记录原SBTO revision `cbed4ede…`的独立oracle：377变量、1024候选、8轮、58→87→203→377活动维度，同一cost下候选、均值、完整cov及best tape逐元素一致。**这验证优化器，不验证native与Warp接触动力学相同。** vendor保存来源hash/许可信息；本地未推GitHub，第三方资产许可仍须按THIRD_PARTY_NOTICES处理。

历史算法比较曾有实质适配错误：DIAL累积cost直接配温度.1使ESS≈1，节点越界、双重退火未对应；MPPI温度单位、MPPI-CMA方差平方/特征值下限、CBO持久状态也曾修正。`task_mapped`是任务适配，不是原论文默认；很多检查只有两轮29步。旧差结果不能当各理论能力排名。后来按用户要求只保留CEM产物，其他算法日志/轨迹被清理。

历史保留结果：Hydrax**对角**CEM在20°冻结任务seed0、1024候选、650轮，cost176.706512、进程370.343s、终点55.987mm/7.248°，Warp冷重放一致；complete不是完整物理验收。它不是后来的full-cov A144五条件成绩。

20°→25°冷/热启动曾实测：冷407.727s/237轮/cost110.120422，热163.350s/52轮/cost175.908661；扣缓存world-steps约8675万→3286万。达到热启动最终cost的时间289.465→154.765s，约1.87倍，但门槛事后选择，热启动最终精度更差。因此不能把总时间比2.50写成同质量2.50倍。`init-from`迁移控制/分布后在新任务重新仿真；`resume`续同问题；跨条件不能复用旧物理前缀。

GPU OOM曾定位到Warp FFI Graph随XLA缓冲地址产生重复捕获；即使graph-cache-size=1仍增长。WARP_STAGED固定staging缓冲、阶段释放Graph后，限时跨阶段检查显存稳定，候选/cost/mean/cov一致。这是工程验证，不是所有场景永不OOM保证。总samples与并行microbatch要分开；通常先降batch保留候选预算。

GPU未必明显快：CPU/GPU是否都复用同样前缀、相同实际仿真步数、长时域串行积分、碰撞复杂度、host PCHIP/decode与传输、编译/graph管理都会影响总时间。双掌/脚的凸包碰撞后来简化为盒，六边形视觉保留；不要把精度和求和差异当作万能解释。

Warp本身是动力学后端，优化控制可以直接在同Warp冷重放；原生窗口显示状态不证明native引擎同控制验证通过。历史CPU约1155.81s/522轮与GPU约405.95s/510轮不是严格同预算同结果比较；native重仿真曾摔倒，不能隐瞒这个边界。

## 7. contact-transfer五分支：实际机制与历史成绩

核心档案：[README](../Loco_maniplation/g1-trajopt/docs/research_history/contact-transfer/README.md)、[RESEARCH](../Loco_maniplation/g1-trajopt/docs/research_history/contact-transfer/RESEARCH.md)、[PLAN](../Loco_maniplation/g1-trajopt/docs/research_history/contact-transfer/PLAN.md)、[EXPLORATION_ARCHIVE](../Loco_maniplation/g1-trajopt/docs/research_history/contact-transfer/EXPLORATION_ARCHIVE.md)。原可再生outputs于11月5日清理；以下是保留MD中的历史结果，不能现场复认证旧产物。

早期目标是从20°控制分布扩增大范围条件族。十条件c00–c09：机器人环绕0…324°、36°一组，距离.34–.58m、尺寸倍率约.8–1.25、质量.3–1.4kg。连续表示13×29＋3事件＋4面内坐标＝384维，另含离散对向抓面；编码时也称385维。它与现在377维纯控制不是同一问题。`(1.86,1.86,1.96)`是identity事件映射锚点，**不是实际触碰时刻**。

| 分支 | 已做机制 | 尚未做/不能声称 |
|---|---|---|
| transport | 同抓面分块Sinkhorn，持久粒子向低cost proposal重心运输 | 不是跨条件可行分布OT，不跨面平均类别 |
| stein | 多群CMA及演化路径＋标准化参数RBF排斥，同面交互 | 没有真实接触策略核或其可靠pullback；不是精确SVGD复现 |
| kl_trust | 有限样本REPS对偶、Gaussian矩拟合，投影后KL/熵及类别KL检查 | 不仅是softmax调温，但未实现多物理约束对偶，也非后来TR-MPC原样复现 |
| progressive_signed | 结构子空间内付费正负残差探针、阻尼GN、真实/预测下降比、渐进时域 | 不是作者DFO-GN完整复现，不自动发现新接触 |
| contact_graph | 最多二抓面×三个参考onset区间，懒展开、局部DFO、beam/UCB | 不是完整脚步/手/箱支撑接触序列规划，没有条件homotopy |

v7/seed0旧筛查：CEM2/10、graph1/10，其余四类0/10；共同480s上限，但CEM每轮2048、新分支512候选。高度单项通过不是完整成功。这是当时实现/自定义协议的结果，不足以判OT/Stein/KL理论无用。

修正的`progressive_consistent`在v8完成十条件20次配对，两法各1/10、仅c00共同通过：

| c00 seed0 | 在线时间 | 含setup | 终点位置 | 终点姿态 |
|---|---:|---:|---:|---:|
| warm CEM | 487.648s | 506.868s | 1.897mm | 8.755° |
| consistent | 213.584s | 232.930s | .230mm | 6.325° |

这是约2.28倍的**单条件**正面记录，不是十条件泛化；c00/c01曾用于开发，追加多seed队列取消。修正版同时修改中心一致性、实际裁剪位移拟合、非线性外推检查及预算调度，尚未隔离各项贡献；不能把收益归于signed残差或单一GN公式，也不是作者DFO-GN原样复现。速度与精度必须来自同一成功控制hash，不能拼first-success时间和另一条最终精度。

`contact-validation-v3`有19项：末0.3s位姿/twist、双掌真实承力、法向与有限面/角点/重叠、手脚滑移、脚支撑、身体碰撞和限矩等。夹持力阈值.1N/掌、支撑合力1N，角度p95≤12°、面距p95≤.02m、最小掌面重叠≥.60、滑移p95≤.12m/s等均是当时约定，不是无条件真机安全认证。后续不能为方法改标签。

## 8. DFO研究不是只有一个GN公式

用户明确认可昂贵黑盒动力学上的derivative-free optimization，要求继续看2019之后及近期工作。直接谱系DFO-GN→DFO-LS→DFBGN：残差插值、点集几何/历史复用、子空间模型；本地正负差分GN只是改造，不是作者算法原样复现。

已调研的计算机/数学机制包括ReLIZO查询复用、CoCD陈旧坐标、BayeSQP约束不确定性、R-AdaZO、LOZO/HiZOO/FZOO、EnKG、RSDFO-Q、稀疏LM、随机子空间SQP、可行方向/活动集识别、CatMADS和RETRO。其平滑性、PL、MFCQ、可行初态或生成先验条件不能直接搬给非光滑接触。代码和正文入口已在旧RESEARCH/ARCHIVE，不必重新从零搜。

### 8.1 重要数学发现：压缩残差改变GN曲率

旧oracle每15步把残差向量压成非负范数c=||r||。平方cost保持，但GN从JᵀJ变成JᵀuuᵀJ，u=r/||r||，丢掉Jᵀ(I−uuᵀ)J。例r(x)=(1,x)在0点，原GN刚度1，范数GN为0。这个性质已明确，**尚不能据此断言全部真实失败就是它导致**。

signed方案保留部分正交时间系数Sr及余能||Qr||，平方能量保持，GN保留更多方向。正式retain=1只有DC带符号时间均值＋余能；自适应频谱/割线选基尚未实现。中心、正负探针和trial需同一基，零点不可微、epsilon/floor与浮点误差有明确限制。完整384维中心差分约771候选，在512批下padding到1024world；更大batch可能令dense GN开销更大，不能把更多方向等同更快。

### 8.2 约束恢复与运行时分支的具体记录

| 分支 | 历史记录与限制 |
|---|---|
| Handoff12 filter/v9 | c00通过131.827s、9.775mm，精度差于基线1.897mm；c06失败 |
| tangent白化/v10 | 局部QP可用23/24，最终c00仍因法向12.610°、重叠.576失败；子问题可用不等于任务成功 |
| Contact16/v11 | 首个完整score约143s/轮，运行时回退，中止 |
| inverse-gather/v12 | 512×345步score约.677s，旧路径.671s；scalar cost差0、约束对validator差≤1.78e−6，这是观测/运行时验证 |
| v12真实优化 | c00通过177.772s、3.719mm；c06足滑.127351>.12失败，没有联合速度精度胜出 |
| Physical21 | 追加真实承力/支撑/碰撞/滑移/抬升，719项CPU测试记录；真实Warp批量对照未完成 |
| precision funnel | 可行后收紧内部精度、保护安全incumbent的计划，未完整整合验证 |

c00 v12审计97次验证、75个成功hash，最终保存已验证成功点中终点最优，没有发现验证过的更优点被最终selector丢掉；未验证/未归档候选无法排除。最低J成功点反而终点8.850mm，留下**总cost与末端精度的冲突**。

### 8.3 另一个明确性能证据

同c06、seed0的v7/v8 warm CEM，前90轮搜索与156个共同验证候选结果逐值一致。v7第93轮首次成功；v8第90轮已484.707s，被480s预算截断而失败。共同90轮decode32.674→163.709s，physics/transfer247.376→248.397s。

这说明host开销和墙钟截断能改变相同随机搜索的最终成败；没有历史CPU频率/负载证据解释decode为何慢，也不能事后加时回填成功。旧协议还修过遗漏速度项、预热账本覆盖、v6人工事件移时导致dense control差.30097rad。v7恢复source identity初始化后，才是原控制提示的主比较。

## 9. 跨条件可行分布延拓：已有完整设想，尚未实现

档案提出在固定局部接触模式m下，研究已付费数据能否建立

`h_m(z+δz,c+δc) ≈ h_m(z,c)+A_m δz+B_m δc`。

A是控制响应，B是箱位姿/尺寸/质量变化的响应；先作带可信域的违反量恢复，再优化原J。已有同条件残差FD和真实下降检查，**没有B模型、跨条件割线archive、条件oracle调度或可行分布OT投影**。

分布目标曾写为尽量小地运输源分布到新条件可行集并保留覆盖；探索高斯包含失败样本，不等于可行专家测度。逐粒子恢复不是已求得全局Wasserstein最优；同条件Sinkhorn也不是跨条件运输。接触建立/解除、滑移、换面后要检测模型失效，缩小条件步或切换模式，跨面控制不做重心平均。

五类后续命题曾分别是：DFO条件敏感度迁移；OT由可行约束决定目标边缘；KL多约束更新；接触图在条件路径上自适应换面；Stein在接触策略空间维护多样性。都未成为已验证胜者。

另有时间局部FD＋链式敏感度/多重射击后备方案：Warp恢复状态可能需41配置切空间＋41速度＋41 qacc_warmstart＝123维，而非只存q/v。对pre-forward传感器与post-integration状态需对齐；原控制维度不变，不能直接换成345×29逐步控制再称同问题。只做过文献/源码审查，未有新速度结果。

## 10. WBT辅助CEM：设计、已实现基线与未完成路线

必读：[447行完整研究](../Loco_maniplation/g1-trajopt/docs/SUPERMIMIC_WBT_GUIDED_CEM_RESEARCH.md)、[实验合同](../Loco_maniplation/g1-trajopt/docs/WBT_CEM_EXPERIMENT.md)、[实现计划](../Loco_maniplation/g1-trajopt/docs/superpowers/plans/2026-11-08-wbt-cem-guidance.md)。不要只留下“未来可以接tracker”。

研究路线是冻结actor的**参考输入响应→真实接触成对试探→带符号残差局部校准→DFO-GN修正候选→progressive full-cov CEM竞争精英**。actor输入Jacobian不是完整闭环任务梯度；需低频/随机/CEM方向对照。B0同WBT参考CEM，B1低频DFO，B2随机DFO，B3 actor方向＋真实校准，B4再加跨条件预测修正。150次、37.5 GPU小时是原计划搜索上限，不是完成记录。

合同：task_spec箱体目标固定，policy_reference可变；保存`reference_command`与`achieved_motion`，实际运动重新当参考未必仍可执行。整段参考加常数XY在现SuperMimic goal接口可能被消掉；需随时间变化的root参考。拟从29关节扩到33维参考knot，不能把参考维数锁成`model.nu`。actor输出后加未部署残差会绕开tracker，不能拿它证明参考可执行。

接口需对齐99维obs/730维goal、历史、上一动作、phase、normalizer、关节顺序、action scale和PD。普通tracking54000代checkpoint与WBO版本特征不同，肩肘kp曾有75与14.25差异；不能只改关节顺序便认为模型相同。缓存必须包含策略历史及真实goal流；future goal可能提前.2s消费参考，不能照搬PD前缀缓存规则。

**实际已实现**：`guidance/{artifact,export,proposals}.py`、export/benchmark脚本、`wbt_mean`与`wbt_mixture` preset。前者替换初始节点均值，后者向固定尾槽注入artifact附近候选；仍用原PD/Warp问题评分，**不是逐候选WBT闭环参考工厂已完成**。

已登记smoke仅64候选×2轮、29物理步，零抬升不构成完整任务证据；现存MD没有完整加速成功结论。artifact的source hash与当前同名NPZ不一致，尽管problem/model/顺序/网格绑定；basis是phase-window target-minus-source，不能叫actor Jacobian。用户跑过完整mixture并质疑后段趋势，未留下可信同预算完整胜出结论；不要让其再次无依据长跑。

本地tracker工程在`/home/songchao/G1_research/supermimic_vibed`，此前写过`tools/track_ik_mujoco_warp.py`用于带对应箱体跟踪粗IK。训练/播放状态需实查，不自动恢复。Workhorse、ViBe、DiPLO、Sumo、FS-MPC、ProxPI、FIBER、GPC及Guided ES/CoVO-MPC等近邻已在完整文档归类，不能重新把它们当没读过。

## 11. pp_cem：必须保存内容，但不要重复当新主线推

来源：[370行CEM_CONTACT_RESEARCH_PLAN](../Loco_maniplation/g1-trajopt/docs/CEM_CONTACT_RESEARCH_PLAN.md)。旧工作名Prediction-Powered CEM：1024候选只付256真实rollout，预测精英资格，再以已知包含概率π做真实误差校正，估计精英质量、一阶矩与完整二阶矩。

核心是PPI/ASI/HT公式 `a_hat = mean(h_tilde + ξ/π*(h−h_tilde))`。在候选池、阈值、代理冻结且π正确时raw moments条件无偏；归一化均值/cov不自动无偏，PSD修复也改变估计。原SBTO的K−1样本协方差、保留旧精英、平滑系数和tie都须对齐。

拟议增量不只是便宜预测，而是**按下一轮均值/完整cov更新误差主动补真值并停止**。动态精英阈值会替换边界样本，影响函数净项是ΔI·(T−m_boundary)，不同于固定阈值ΔI·T；在连续密度等条件下成立，接触cost原子须另处理。高杠杆二阶矩误判可能比cost RMSE重要。

计划先用全真值快照和已知阈值作离线机制实验，再未知阈值、端到端、跨条件校准；全真值诊断不能免费回流冷启动。首轮无代理档案的付费成本、CDF单调化、交叉拟合、uniform fallback、PSD恢复、精确包含概率及microbatch都已讨论。`pp_cem.py/elite_estimation.py/proxy.py`等是建议文件，**没有实现/提速结果**。

理论/近邻已经延伸至IGO、MORE/CAS-MORE、Multifidelity CE、PPI、Active Testing、ASI、Robust ASI、Fixed-Size ASI、PPAT、多保真CDF/分位数与多输出统计估计。BC-EvoCEM、GRASP、TACTIC、MTP、DynaRetarget、DiPLO、PhysicsGen也早已在该旧计划中列出，不能把本窗口再核验写成第一次发现。

11月15日用户明确不想反复重推这条线，原因是它减少更新估计成本，却不能恢复已经禁止扰动的早段或生成缺失接触。应保存为暂存路线，不因暂停就宣称数学无效，也不在新窗口原样再次推荐。

## 12. 接触模式、ROM与混合CEM研究如何演进

用户提出从起终接触关键帧出发，结合可达性/ROM寻找中间足步、接近、抓持、搬运和放置，再由CEM/MPPI连接局部段。接触模式与任务空间轨迹相互依赖；同模式仍可有蹲下/站立等关键构型，箱体—环境支撑必须纳入图。接触图不是仅存左右手/脚标签。

曾调研的组件分工：McVAMP几何接近；KOMO已知阶段的构型/路径；DSMS多重射击全身动态精修；Sampling+C3外采接近位置、内求局部互补MPC；ROM-CIMPC联系低维步态与全阶接触；IMPACT/CRISP/GCS和STOCS-3D处理不同范围的离散/互补/物体环境接触问题。**没有决定“必须混合整数上层”，也没有把这些组合完整实现。** 固定抓面少时，简单枚举是必要基线。OMPL几何/kinodynamic规划与当前无显式有效性oracle的单射击CEM接口不同，不能直接替换。

[历史ms_cem草案](../Loco_maniplation/g1-trajopt/docs/research_history/CEM_MULTICONTACT_MIXTURE_DRAFT_20261115.md)提过签名分层精英、模式archive重注入、K≤8完整高斯、UCB预算、合并/分裂及后续事件参数化。**尚未实现；文首已标历史，部分归因修正。** 短侧面候选进精英不证明单高斯毁掉完整成功解；没进精英也可能是cost/违规排序，不能只怪候选生成。

[现行11月15日计划](../Loco_maniplation/g1-trajopt/docs/CEM_MULTICONTACT_RESEARCH_PLAN.md)才转为先验证恢复前段。它是本窗口03与action-chunk讨论的直接来源，不是迁到SMPC后才新想到。现行方案不同时叠加分层精英/新接触奖励/代理/多群，保留原物理、参考和cost，所有注入占原1024槽、仍40精英。PCHIP影响相邻区间，选节点不等于严格时间支撑；用户显式冻结的`frozen_prefix_knots`不能擅自解除。

## 13. 当前CEM与A144故障的直接证据

代码：[optimization.py](../Loco_maniplation/g1-trajopt/src/g1_trajopt/engine/optimization.py)、[controller.py](../Loco_maniplation/g1-trajopt/src/g1_trajopt/engine/controller.py)、[vendor CEM](../Loco_maniplation/g1-trajopt/src/g1_trajopt/_vendor/sbto/solvers/cem.py)、[solver_base](../Loco_maniplation/g1-trajopt/src/g1_trajopt/_vendor/sbto/solvers/solver_base.py)。

13×29残差节点经PCHIP叠加q_nom，full cov跨关节/时间更新 `Σ←.75Σ+.25cov(elites)`；默认1024候选、40精英、1个直接保留。阶段2…13逐渐增加活动维度和真实rollout/评分H，最终4.45s。T/H缩放没计算未来，**“第一轮就评完整4.45s”是错误解释**。

低于`.001 rad²`的坐标被采样硬掩码关闭，候选取均值，`std_incr=0`使其不能自己恢复。`min_std_next=.055/min_std_final=.01`比较的是**方差**不是标准差。40精英单轮cov秩≤39，但平滑混旧Σ后不必低秩，不能据此说整个采样分布只有39秩。

证据入口：[A144诊断](../Loco_maniplation/g1-trajopt/docs/A144_DIAGNOSTICS.md)、[过程审计](../Loco_maniplation/g1-trajopt/docs/CEM_SEARCH_AUDIT_PLAN.md)、[verification.json](../Loco_maniplation/g1-trajopt/runs/yaw020_search_audit/audit/verification.json)。

- 五单因素seed0：original/forward005/width120/mass200完成抬箱，yaw020仅升97.18mm、终点误差515.04mm；正常结束、冷重放一致、机器人站立，不是OOM或先摔倒。
- yaw20目标掌板在1.10–1.50s要求约21–24mm穿透箱棱；实际右掌.89s已撞边，目标到1.38s才适配。目标几何不合理与搜索受限同时存在，不能只用一句“cost错了”结束。
- population47/row478、population50/row78在H1.15s有侧面几何进展、进过精英，仍约9.0/8.4mm穿透；参与高斯更新但未占直接保留槽。它们不是完整成功抓取。
- 214轮/219136次评估，没有“双掌就位并抬升”几何标签；标签本身也不含充分承力证明。stage13新候选不能替代对同一row478的反事实续搜。
- 最后1023个不同候选前约2.02s实际控制完全相同，覆盖卡边/接近；后期没有改前段能力。但成功原条件也会收缩，不能断言唯一根因。
- 原成功条件仍有约6–8mm足底穿透。几何到达、抬箱完成、同引擎重放、严格物理验收是不同成熟度。
- 2048候选多旧精英时曾存在覆盖samples后取argmin导致控制/cost错配；A144文档记录共同wrapper已复制覆盖前最佳控制，不能又把旧bug当未修当前事实。

## 14. 最近的rubric评估与action chunk讨论

本地skill已安装并实际使用：[research-idea-rubric](.agents/skills/research-idea-rubric/SKILL.md)，reference齐全。此前误判缺失是漏查项目`.agents/skills`，无需重装。

03由实现证据、近邻、新方案反证三路审查形成：首选**按真实试探收益选择重开时间块**；多分支后续价值排第二；残差DFO留作组件。它们都是待验证建议，本窗口没有实现新优化器、修改cost或跑新G1搜索。

重开直白说是恢复已停止变化的控制节点的方差与活动状态。每块同预算试探，在同H比较，再把剩余预算用于有收益的块。必须击败取消硬掩码＋固定持续探索、统一重开、固定早段/后段、随机等大小块、完整视界从头、iCEM与TR-MPC；分支路线还需BC-EvoCEM。局部checkpoint恢复与完整从初态重跑分开。

理论若借最优臂识别，先冻结中心、H和修复核；确定性仿真的随机性来自扰动而非重复同控制的测量噪声。块的平均/批最佳改进、达标概率、首次成功时间不等价，不能直接搬界。

用户随后问action chunk能否提升SMPC。已解释：

| chunk含义 | 当前结论 |
|---|---|
| 仅拼接多步动作 | 维数不变，改名字不加速 |
| 常值块/稀疏样条/低维基函数 | 降搜索自由度；当前PCHIP已部分做到 |
| 学习/检索协调动作段 | 可提高有效提案率；低维latent可压维，但需要可执行接口及覆盖 |
| 多步执行后才重规划 | 减少在线调用但减少反馈，对离线造轨迹不直接省物理步 |

**重开决定哪里重新搜索，chunk决定一段怎样协调变化。** 可以先对比原PCHIP、固定粗块、同参数预算非均匀/自适应细分；学习提案后置。新chunk不自动减少候选物理步数，需用更少候选/迭代完成同任务。move-blocking与自适应块长已有先例，GPC-CEM已学整段提案并真实评分。这里尚无代码和收益。

## 15. SMPC材料、技能与并行窗口新文件

papers按A–G共57PDF：7/8/4/9/5/12/12。不能把“文件在”写成“全部已复现”。A类已区分噪声时间谱、点集估计、目标平滑与输出滤波；Smooth MPPI积分索引、σ与方差、dsCEM/dsMPPI取舍、C-Uniform覆盖/预计算都曾订正，继续阅读现有地图，不要回到机械摘要。

直接近邻包括iCEM精英记忆、SV-MPC多粒子、SVG择模态、CE-MPPI聚类、TR-MPC KL/熵、BC-EvoCEM worker管理、GRASP历史二次方向、PFMP物理时域重采样。此轮核验不是这些思想首次被整个项目发现；尤其BC早在pp_cem计划就收录。应用与理论增量须具体化，不能说全机器人领域没人做过。

本次补写时，发现另一轮工作已新增以下文件，**不是本交接补写过程完成的成果**，接手时应一并读取其最新状态：

- [04_RESEARCH_SKILLS_WORKFLOW.md](doc/04_RESEARCH_SKILLS_WORKFLOW.md)
- [研究问题卡](doc/research/research-question-card.md)、[其计划](doc/research/task_plan.md)、[笔记](doc/research/notes.md)
- [TR-MPC精读报告](doc/paper_reading/2605.07801_Sampling-based_Model_Predictive_Control_Using_Trust_Regions/2605.07801_阅读报告.md)

研究卡将恢复时间块定义为exploratory，列七组最小协议，未表示GPU已启动；任务计划里仍有未完成的报告验证项。新文件也提醒Hydrax `cov`实存标准差，已有explore_fraction/sigma_min，离线末轮输出规则不同于SBTO历史best；跨框架必须对齐语义。不要覆盖并行窗口文件或冒称替它验证过。

## 16. 本窗口Hydrax安装修复：已完成且范围明确

用户按conda环境＋`pip install -e .`后，PS导入报`kernel() got an unexpected keyword argument 'grid_stride'`。根因MuJoCo/MJX被宽松约束升级4.0.1，而Warp限制1.12.x。

已在`/home/songchao/anaconda3/envs/hydrax`恢复MuJoCo/MJX3.8.0、Warp1.12.1，卸载4.x拆分core/studio/wasm。pyproject精确固定3.8.0，uv.lock声明同步；README写修复说明；PS测试PRNG断言改JAX公开API。JAX/JAXlib/CUDA13 plugin/pjrt仍0.12.0.1、Flax0.12.12，**没有完整回退到锁文件0.8.3/0.12.4**。

验证：清除PYTHONPATH后pip check通过；JAX识别4090；PS两测试2 passed/8.71s，含100轮open-loop优化；真实MuJoCo窗口运行约5s后包装自动关闭。默认JAX GPU后端，**未验证`--warp`、其他任务/算法或完整套件**。

```bash
cd /home/songchao/G1_Locomaniplation/SMPC/src/hydrax
conda activate hydrax
env -u PYTHONPATH MUJOCO_GL=glfw \
XLA_PYTHON_CLIENT_PREALLOCATE=false python examples/pendulum.py ps
```

## 17. 接手原则与尚未执行事项

1. 先读当前任务与已有文档。上面的旧研究不能当成没发生；暂停不等于无价值，旧正面结果也不能放大成泛化。
2. 用户最新请求是保存上下文。不要自动重启全库优化、训练、旧队列，或借此改模型/cost。当前新重开/chunk实验没有结果。
3. 若用户让实现，先小规模、同问题同预算，分开取消硬掩码、恢复方差、选块规则和动作表示。不要先手工改完参考再比较。
4. 工程入口统一g1-trajopt，共享YAML；源数据只读。跨条件控制分布热启动与同条件resume、native显示与物理引擎、几何与真实承力标签严格区分。
5. 保留所有失败、world-steps、host/编译/传输/缓存账目，按成功率—预算和同一候选质量比较。旧reject可并列宽松可用性评价，不为新方法偷偷改阈值。
6. 用户全局AGENTS要求每请求有界委派，实质任务2–3个非重叠角色；根agent对结论负责。本次三路分别补研究档案、WBT/工程、早期任务与数据。

沟通用中文，先结论、少术语、真实路径与复制命令。播放器优先同窗切换，核对该工具到底用N/P还是`[]`；新优化先给原轨迹播放入口。不要重复广泛调研代替读现有结论，不要以自责、保证录用或未经验证的“主要原因”搪塞用户。

本次补写只重建这一份handoff，未删除数据、未更改算法/环境、未运行GPU；MD中的历史测试数属于不同revision，不是本次重跑。旧handoff在本次写入前已不在原路径，因此此文件按现存研究材料和本对话重建，未覆盖其他窗口新增的研究文件。
