# papers 目录论文下载地址清单

整理日期：2026-11-17（Asia/Shanghai）。覆盖当前 `papers/` 下全部 **58 篇 PDF**，不包含 `foundations/` 下的 3 个 HTML 知识页。

链接由本地 PDF 首页、[现有文献索引](01_PAPER_MAP.md)及官方论文页面整理。49 篇使用本地首页明确的版本化 arXiv 编号；2 篇提供未固定版本的 arXiv 地址；7 篇使用论文集、OpenReview 或作者的 PDF 地址。原 PDF 文件保持不变。

- [纯下载链接 TXT](paper_download_urls.txt)：每行一个 URL，共 58 行。
- [完整 CSV](paper_downloads.csv)：编号、类别、标题、下载链接、原文入口、本地路径、版本依据及核验状态。

原文献索引包含 57 篇；本清单另纳入根目录新增的 `18709_Beyond_Softmax_and_Entro.pdf`，不擅自为它分配原索引的 L/N 编号。

## 链接核验与版本

- 版本化 arXiv 链接对应本地 PDF 首页编号；本轮没有重新下载全部 arXiv 文件。
- L23、L18 的本地 PDF 无可提取的自身 arXiv 版本号，提供索引对应的未固定版本链接，可能与本地会议版有差异。
- L28、L34、L10、L22 和新增论文已用少量响应数据确认 HTTP 200 及 PDF 文件头。
- L06、L07 的官方 OpenReview 下载入口已由原文及官方索引确认；本轮自动请求返回 HTTP 403，浏览器可能要求验证。这里保留正确入口，不将其写成已成功下载。

## 类别统计

| 分类 | 篇数 |
|---|---:|
| A_采样质量 | 7 |
| B_多模态 | 8 |
| C_更新规则 | 4 |
| D_地形与搜索方向 | 9 |
| E_约束与接触 | 5 |
| F_模型与先验 | 12 |
| G_工具预算与背景 | 12 |
| 根目录_未分类 | 1 |
| **合计** | **58** |

## A_采样质量

| 编号 | 论文（原文入口） | PDF 下载地址 | 本地文件 | 版本／说明 |
|---|---|---|---|---|
| N01 | [Sample-efficient Cross-Entropy Method for Real-time Planning](https://arxiv.org/abs/2008.06389) | [下载 PDF](https://arxiv.org/pdf/2008.06389v1) | [本地 PDF](<../papers/A_采样质量/N01_2020_Sample_efficient_Cross_Entropy_Method_for_Real_time_Planning.pdf>) | 本地 PDF 首页：arXiv:2008.06389v1 |
| N03 | [Biased-MPPI: Informing Sampling-Based Model Predictive Control by Fusing Ancillary Controllers](https://arxiv.org/abs/2401.09241) | [下载 PDF](https://arxiv.org/pdf/2401.09241v2) | [本地 PDF](<../papers/A_采样质量/N03_2024_Biased_MPPI_Informing_Sampling_Based_Model_Predictive_Control_by_Fusing_Ancillary_Controllers.pdf>) | 本地 PDF 首页：arXiv:2401.09241v2 |
| N05 | [Smooth Model Predictive Path Integral Control without Smoothing](https://arxiv.org/abs/2112.09988) | [下载 PDF](https://arxiv.org/pdf/2112.09988v8) | [本地 PDF](<../papers/A_采样质量/N05_2021_Smooth_Model_Predictive_Path_Integral_Control_without_Smoothing.pdf>) | 本地 PDF 首页：arXiv:2112.09988v8 |
| N06 | [Low Frequency Sampling in Model Predictive Path Integral Control](https://arxiv.org/abs/2404.03094) | [下载 PDF](https://arxiv.org/pdf/2404.03094v2) | [本地 PDF](<../papers/A_采样质量/N06_2024_Low_Frequency_Sampling_in_Model_Predictive_Path_Integral_Control.pdf>) | 本地 PDF 首页：arXiv:2404.03094v2 |
| N07 | [Sample-Efficient and Smooth Cross-Entropy Method Model Predictive Control Using Deterministic Samples](https://arxiv.org/abs/2510.05706) | [下载 PDF](https://arxiv.org/pdf/2510.05706v2) | [本地 PDF](<../papers/A_采样质量/N07_2025_Sample_Efficient_and_Smooth_Cross_Entropy_Method_Model_Predictive_Control_Using_Deterministic_Samples.pdf>) | 本地 PDF 首页：arXiv:2510.05706v2 |
| N08 | [Smooth Sampling-Based Model Predictive Control Using Deterministic Samples](https://arxiv.org/abs/2601.03893) | [下载 PDF](https://arxiv.org/pdf/2601.03893v2) | [本地 PDF](<../papers/A_采样质量/N08_2026_Smooth_Sampling_Based_Model_Predictive_Control_Using_Deterministic_Samples.pdf>) | 本地 PDF 首页：arXiv:2601.03893v2 |
| N11 | [C-Uniform Trajectory Sampling For Fast Motion Planning](https://arxiv.org/abs/2409.12266) | [下载 PDF](https://arxiv.org/pdf/2409.12266v1) | [本地 PDF](<../papers/A_采样质量/N11_2024_C_Uniform_Trajectory_Sampling_For_Fast_Motion_Planning.pdf>) | 本地 PDF 首页：arXiv:2409.12266v1 |
## B_多模态

| 编号 | 论文（原文入口） | PDF 下载地址 | 本地文件 | 版本／说明 |
|---|---|---|---|---|
| L03 | [Real-World Deployment of Massively Parallel Sampling-Based MPC for Contact-Rich Manipulation](https://arxiv.org/abs/2606.20712) | [下载 PDF](https://arxiv.org/pdf/2606.20712v1) | [本地 PDF](<../papers/B_多模态/L03_2026_Real_World_Deployment_of_Massively_Parallel_Sampling_Based_MPC_for_Contact_Rich_Manipulation.pdf>) | 本地 PDF 首页：arXiv:2606.20712v1 |
| L09 | [Contact-Implicit Stein Projected ADMM for Discovery of Diverse Contact-Rich Manipulation Strategies](https://arxiv.org/abs/2609.28299) | [下载 PDF](https://arxiv.org/pdf/2609.28299v1) | [本地 PDF](<../papers/B_多模态/L09_2026_Contact_Implicit_Stein_Projected_ADMM_for_Discovery_of_Diverse_Contact_Rich_Manipulation_Strategies.pdf>) | 本地 PDF 首页：arXiv:2609.28299v1 |
| L21 | [Model Tensor Planning](https://openreview.net/forum?id=fk1ZZdXCE3) | [下载 PDF](https://arxiv.org/pdf/2505.01059v2) | [本地 PDF](<../papers/B_多模态/L21_2025_Model_Tensor_Planning.pdf>) | 本地 PDF 首页：arXiv:2505.01059v2 |
| L24 | [Sampling-Based Control via Entropy-Regularized Optimal Transport](https://arxiv.org/abs/2605.02147v1) | [下载 PDF](https://arxiv.org/pdf/2605.02147v1) | [本地 PDF](<../papers/B_多模态/L24_2026_Sampling_Based_Control_via_Entropy_Regularized_Optimal_Transport.pdf>) | 本地 PDF 首页：arXiv:2605.02147v1 |
| L28 | [Stein Variational Model Predictive Control](https://proceedings.mlr.press/v155/lambert21a.html) | [下载 PDF](https://proceedings.mlr.press/v155/lambert21a/lambert21a.pdf) | [本地 PDF](<../papers/B_多模态/L28_2020_Stein_Variational_Model_Predictive_Control.pdf>) | 官方论文集/期刊或作者 PDF；未固定文件修订版本 |
| L35 | [Stein Variational Guided Model Predictive Path Integral Control: Proposal and Experiments with Fast Maneuvering Vehicles](https://arxiv.org/abs/2309.11040v3) | [下载 PDF](https://arxiv.org/pdf/2309.11040v3) | [本地 PDF](<../papers/B_多模态/L35_2024_Stein_Variational_Guided_Model_Predictive_Path_Integral_Control_Proposal_and_Experiments_with_Fast_Maneuvering_Vehicles.pdf>) | 本地 PDF 首页：arXiv:2309.11040v3 |
| N04 | [Multi-Modal MPPI and Active Inference for Reactive Task and Motion Planning](https://arxiv.org/abs/2312.02328) | [下载 PDF](https://arxiv.org/pdf/2312.02328v2) | [本地 PDF](<../papers/B_多模态/N04_2023_Multi_Modal_MPPI_and_Active_Inference_for_Reactive_Task_and_Motion_Planning.pdf>) | 本地 PDF 首页：arXiv:2312.02328v2 |
| N12 | [Clustering-Embedded Model Predictive Path Integral Control: Avoiding Averaging-Induced Failure and Enabling Efficient Cluster Selection for Dynamic Obstacles](https://arxiv.org/abs/2607.06499) | [下载 PDF](https://arxiv.org/pdf/2607.06499v1) | [本地 PDF](<../papers/B_多模态/N12_2026_Clustering_Embedded_Model_Predictive_Path_Integral_Control_Avoiding_Averaging_Induced_Failure_and_Enabling_Efficient_Cluster_Selection_for_Dynamic_Obstacles.pdf>) | 本地 PDF 首页：arXiv:2607.06499v1 |
## C_更新规则

| 编号 | 论文（原文入口） | PDF 下载地址 | 本地文件 | 版本／说明 |
|---|---|---|---|---|
| L31 | [Variance-Reduced Model Predictive Path Integral via Quadratic Model Approximation](https://www.roboticsproceedings.org/rss22/p108.html) | [下载 PDF](https://arxiv.org/pdf/2602.03639v2) | [本地 PDF](<../papers/C_更新规则/L31_2026_Variance_Reduced_Model_Predictive_Path_Integral_via_Quadratic_Model_Approximation.pdf>) | 本地 PDF 首页：arXiv:2602.03639v2 |
| L32 | [Variational Inference MPC using Tsallis Divergence](https://www.roboticsproceedings.org/rss17/p073.html) | [下载 PDF](https://arxiv.org/pdf/2104.00241v1) | [本地 PDF](<../papers/C_更新规则/L32_2021_Variational_Inference_MPC_using_Tsallis_Divergence.pdf>) | 本地 PDF 首页：arXiv:2104.00241v1 |
| N09 | [Sampling-based Model Predictive Control Using Trust Regions](https://arxiv.org/abs/2605.07801) | [下载 PDF](https://arxiv.org/pdf/2605.07801v2) | [本地 PDF](<../papers/C_更新规则/N09_2026_Sampling_based_Model_Predictive_Control_Using_Trust_Regions.pdf>) | 本地 PDF 首页：arXiv:2605.07801v2 |
| N17 | [Sampling-based Model Predictive Control as Sequential Monte Carlo Inference](https://arxiv.org/abs/2611.10877) | [下载 PDF](https://arxiv.org/pdf/2611.10877v1) | [本地 PDF](<../papers/C_更新规则/N17_2026_Sampling_based_Model_Predictive_Control_as_Sequential_Monte_Carlo_Inference.pdf>) | 本地 PDF 首页：arXiv:2611.10877v1 |
## D_地形与搜索方向

| 编号 | 论文（原文入口） | PDF 下载地址 | 本地文件 | 版本／说明 |
|---|---|---|---|---|
| L02 | [Global Sampling-Based Trajectory Optimization for Contact-Rich Manipulation via KernelSOS](https://arxiv.org/abs/2604.27175) | [下载 PDF](https://arxiv.org/pdf/2604.27175v1) | [本地 PDF](<../papers/D_地形与搜索方向/L02_2026_Global_Sampling_Based_Trajectory_Optimization_for_Contact_Rich_Manipulation_via_KernelSOS.pdf>) | 本地 PDF 首页：arXiv:2604.27175v1 |
| L08 | [Reference-Free Sampling-Based Model Predictive Control](https://arxiv.org/abs/2511.19204) | [下载 PDF](https://arxiv.org/pdf/2511.19204v3) | [本地 PDF](<../papers/D_地形与搜索方向/L08_2026_Reference_Free_Sampling_Based_Model_Predictive_Control.pdf>) | 本地 PDF 首页：arXiv:2511.19204v3 |
| L12 | [DiffContact: Differentiable Contact Dynamics for Policy Learning and Contact-Implicit Trajectory Optimization](https://arxiv.org/abs/2610.35852) | [下载 PDF](https://arxiv.org/pdf/2610.35852v2) | [本地 PDF](<../papers/D_地形与搜索方向/L12_2026_DiffContact_Differentiable_Contact_Dynamics_for_Policy_Learning_and_Contact_Implicit_Trajectory_Optimization.pdf>) | 本地 PDF 首页：arXiv:2610.35852v2 |
| L14 | [Full-Order Sampling-Based MPC for Torque-Level Locomotion Control via Diffusion-Style Annealing](https://arxiv.org/abs/2409.15610) | [下载 PDF](https://arxiv.org/pdf/2409.15610v1) | [本地 PDF](<../papers/D_地形与搜索方向/L14_2025_Full_Order_Sampling_Based_MPC_for_Torque_Level_Locomotion_Control_via_Diffusion_Style_Annealing.pdf>) | 本地 PDF 首页：arXiv:2409.15610v1 |
| L16 | [GRASP: Gradient-Guided Sampling for Contact-Rich Physics-Based Motion Retargeting](https://arxiv.org/abs/2611.05754) | [下载 PDF](https://arxiv.org/pdf/2611.05754v1) | [本地 PDF](<../papers/D_地形与搜索方向/L16_2026_GRASP_Gradient_Guided_Sampling_for_Contact_Rich_Physics_Based_Motion_Retargeting.pdf>) | 本地 PDF 首页：arXiv:2611.05754v1 |
| L17 | [Global Convergence of Sampling-Based Nonconvex Optimization through Diffusion-Style Smoothing](https://arxiv.org/abs/2605.16520) | [下载 PDF](https://arxiv.org/pdf/2605.16520v1) | [本地 PDF](<../papers/D_地形与搜索方向/L17_2026_Global_Convergence_of_Sampling_Based_Nonconvex_Optimization_through_Diffusion_Style_Smoothing.pdf>) | 本地 PDF 首页：arXiv:2605.16520v1 |
| L23 | [Model-Based Diffusion for Trajectory Optimization](https://arxiv.org/abs/2407.01573) | [下载 PDF](https://arxiv.org/pdf/2407.01573) | [本地 PDF](<../papers/D_地形与搜索方向/L23_2024_Model_Based_Diffusion_for_Trajectory_Optimization.pdf>) | 未固定 arXiv 版本；下载版本可能与本地会议版不同 |
| L25 | [One-Step Model Predictive Path Integral for Manipulator Motion Planning Using Configuration Space Distance Fields](https://arxiv.org/abs/2509.00836v3) | [下载 PDF](https://arxiv.org/pdf/2509.00836v3) | [本地 PDF](<../papers/D_地形与搜索方向/L25_2026_One_Step_Model_Predictive_Path_Integral_for_Manipulator_Motion_Planning_Using_Configuration_Space_Distance_Fields.pdf>) | 本地 PDF 首页：arXiv:2509.00836v3 |
| N18 | [CEM-GD: Cross-Entropy Method with Gradient Descent Planner for Model-Based Reinforcement Learning](https://arxiv.org/abs/2112.07746) | [下载 PDF](https://arxiv.org/pdf/2112.07746v1) | [本地 PDF](<../papers/D_地形与搜索方向/N18_2021_CEM_GD_Cross_Entropy_Method_with_Gradient_Descent_Planner_for_Model_Based_Reinforcement_Learning.pdf>) | 本地 PDF 首页：arXiv:2112.07746v1 |
## E_约束与接触

| 编号 | 论文（原文入口） | PDF 下载地址 | 本地文件 | 版本／说明 |
|---|---|---|---|---|
| L01 | [Approximating Global Contact-Implicit MPC via Sampling and Local Complementarity](https://arxiv.org/abs/2505.13350) | [下载 PDF](https://arxiv.org/pdf/2505.13350v2) | [本地 PDF](<../papers/E_约束与接触/L01_2025_Approximating_Global_Contact_Implicit_MPC_via_Sampling_and_Local_Complementarity.pdf>) | 本地 PDF 首页：arXiv:2505.13350v2 |
| L04 | [PRISM: Projection-Integrated Sampling-Based MPC with Bayesian Cost Tuning for Bimanual Manipulation](https://arxiv.org/abs/2608.25666) | [下载 PDF](https://arxiv.org/pdf/2608.25666v1) | [本地 PDF](<../papers/E_约束与接触/L04_2026_PRISM_Projection_Integrated_Sampling_Based_MPC_with_Bayesian_Cost_Tuning_for_Bimanual_Manipulation.pdf>) | 本地 PDF 首页：arXiv:2608.25666v1 |
| L13 | [Diffusion Predictive Control with Constraints](https://proceedings.mlr.press/v283/romer25a.html) | [下载 PDF](https://arxiv.org/pdf/2412.09342v2) | [本地 PDF](<../papers/E_约束与接触/L13_2025_Diffusion_Predictive_Control_with_Constraints.pdf>) | 本地 PDF 首页：arXiv:2412.09342v2 |
| L27 | [Shooting for Contact: Contact-Implicit Multiple Shooting for Dynamic Motion Retargeting](https://arxiv.org/abs/2608.03116v1) | [下载 PDF](https://arxiv.org/pdf/2608.03116v1) | [本地 PDF](<../papers/E_约束与接触/L27_2026_Shooting_for_Contact_Contact_Implicit_Multiple_Shooting_for_Dynamic_Motion_Retargeting.pdf>) | 本地 PDF 首页：arXiv:2608.03116v1 |
| L30 | [UniConFlow: A Unified Constrained Flow-Matching Framework for Certified Motion Planning](https://arxiv.org/abs/2506.02955v3) | [下载 PDF](https://arxiv.org/pdf/2506.02955v3) | [本地 PDF](<../papers/E_约束与接触/L30_2026_UniConFlow_A_Unified_Constrained_Flow_Matching_Framework_for_Certified_Motion_Planning.pdf>) | 本地 PDF 首页：arXiv:2506.02955v3 |
## F_模型与先验

| 编号 | 论文（原文入口） | PDF 下载地址 | 本地文件 | 版本／说明 |
|---|---|---|---|---|
| L06 | [Diffusion Model Predictive Control](https://openreview.net/forum?id=pvtgffHtJm) | [下载 PDF](https://openreview.net/pdf?id=pvtgffHtJm) | [本地 PDF](<../papers/F_模型与先验/L06_2025_Diffusion_Model_Predictive_Control.pdf>) | 官方论文集/期刊或作者 PDF；未固定文件修订版本；本轮自动访问 403 |
| L07 | [What Drives Success in Physical Planning with Joint-Embedding Predictive World Models?](https://openreview.net/forum?id=cHZn5Gdh8e) | [下载 PDF](https://openreview.net/pdf?id=cHZn5Gdh8e) | [本地 PDF](<../papers/F_模型与先验/L07_2026_What_Drives_Success_in_Physical_Planning_with_Joint_Embedding_Predictive_World_Models.pdf>) | 官方论文集/期刊或作者 PDF；未固定文件修订版本；本轮自动访问 403 |
| L15 | [Feedback-MPPI: Fast Sampling-Based MPC via Rollout Differentiation – Adios low-level controllers](https://doi.org/10.1109/LRA.2025.3630871) | [下载 PDF](https://arxiv.org/pdf/2506.14855v3) | [本地 PDF](<../papers/F_模型与先验/L15_2026_Feedback_MPPI_Fast_Sampling_Based_MPC_via_Rollout_Differentiation_Adios_low_level_controllers.pdf>) | 本地 PDF 首页：arXiv:2506.14855v3 |
| L19 | [KODA: Kinematics–Observation Data Alignment for Humanoid Learning from Human Egocentric Demonstration](https://arxiv.org/abs/2610.21152v1) | [下载 PDF](https://arxiv.org/pdf/2610.21152v1) | [本地 PDF](<../papers/F_模型与先验/L19_2026_KODA_Kinematics_Observation_Data_Alignment_for_Humanoid_Learning_from_Human_Egocentric_Demonstration.pdf>) | 本地 PDF 首页：arXiv:2610.21152v1 |
| L20 | [PhysibleMDM: Enforcing Inference-Time Physical Feasibility Constraints on Motion Diffusion Models via Noise Optimization](https://arxiv.org/abs/2610.30500v1) | [下载 PDF](https://arxiv.org/pdf/2610.30500v1) | [本地 PDF](<../papers/F_模型与先验/L20_2026_PhysibleMDM_Enforcing_Inference_Time_Physical_Feasibility_Constraints_on_Motion_Diffusion_Models_via_Noise_Optimization.pdf>) | 本地 PDF 首页：arXiv:2610.30500v1 |
| L26 | [Grounding Generative Policies in Physics: Optimization-Guided Diffusion for Robot Control](https://arxiv.org/abs/2606.24208v1) | [下载 PDF](https://arxiv.org/pdf/2606.24208v1) | [本地 PDF](<../papers/F_模型与先验/L26_2026_Grounding_Generative_Policies_in_Physics_Optimization_Guided_Diffusion_for_Robot_Control.pdf>) | 本地 PDF 首页：arXiv:2606.24208v1 |
| L29 | [Test-Time Compute for Humanoid Behavior Foundation Models](https://arxiv.org/abs/2610.14896v2) | [下载 PDF](https://arxiv.org/pdf/2610.14896v2) | [本地 PDF](<../papers/F_模型与先验/L29_2026_Test_Time_Compute_for_Humanoid_Behavior_Foundation_Models.pdf>) | 本地 PDF 首页：arXiv:2610.14896v2 |
| L34 | [Dream-MPC: Gradient-Based Model Predictive Control with Latent Imagination](https://proceedings.mlr.press/v306/spieler26a.html) | [下载 PDF](https://raw.githubusercontent.com/mlresearch/v306/main/assets/spieler26a/spieler26a.pdf) | [本地 PDF](<../papers/F_模型与先验/L34_2026_Dream_MPC_Gradient_Based_Model_Predictive_Control_with_Latent_Imagination.pdf>) | 官方论文集/期刊或作者 PDF；未固定文件修订版本 |
| N13 | [A KL-regularization Framework for Learning to Plan with Adaptive Priors](https://arxiv.org/abs/2510.04280) | [下载 PDF](https://arxiv.org/pdf/2510.04280v2) | [本地 PDF](<../papers/F_模型与先验/N13_2025_A_KL_regularization_Framework_for_Learning_to_Plan_with_Adaptive_Priors.pdf>) | 本地 PDF 首页：arXiv:2510.04280v2 |
| N15 | [Generative Models from and for Sampling-Based MPC: A Bootstrapped Approach for Adaptive Contact-Rich Manipulation](https://arxiv.org/abs/2510.14643) | [下载 PDF](https://arxiv.org/pdf/2510.14643v2) | [本地 PDF](<../papers/F_模型与先验/N15_2025_Generative_Models_from_and_for_Sampling_Based_MPC_A_Bootstrapped_Approach_for_Adaptive_Contact_Rich_Manipulation.pdf>) | 本地 PDF 首页：arXiv:2510.14643v2 |
| N20 | [PRISM: PRior-guided Imagination Sampling in world Models](https://arxiv.org/abs/2606.07974) | [下载 PDF](https://arxiv.org/pdf/2606.07974v2) | [本地 PDF](<../papers/F_模型与先验/N20_2026_PRISM_PRior_guided_Imagination_Sampling_in_world_Models.pdf>) | 本地 PDF 首页：arXiv:2606.07974v2 |
| N22 | [Dual Model Predictive Control through Sampling-Based Belief Propagation](https://arxiv.org/abs/2611.11088) | [下载 PDF](https://arxiv.org/pdf/2611.11088v1) | [本地 PDF](<../papers/F_模型与先验/N22_2026_Dual_Model_Predictive_Control_through_Sampling_Based_Belief_Propagation.pdf>) | 本地 PDF 首页：arXiv:2611.11088v1 |
## G_工具预算与背景

| 编号 | 论文（原文入口） | PDF 下载地址 | 本地文件 | 版本／说明 |
|---|---|---|---|---|
| L05 | [Do My Samples Help? Compute Budget Allocation for Real-World Sampling-Based MPC in Contact-Rich Manipulation](https://arxiv.org/abs/2610.14064) | [下载 PDF](https://arxiv.org/pdf/2610.14064v1) | [本地 PDF](<../papers/G_工具预算与背景/L05_2026_Do_My_Samples_Help_Compute_Budget_Allocation_for_Real_World_Sampling_Based_MPC_in_Contact_Rich_Manipulation.pdf>) | 本地 PDF 首页：arXiv:2610.14064v1 |
| L10 | [Cross-Entropy Randomized Motion Planning](https://roboticsproceedings.org/rss07/p22.html) | [下载 PDF](https://roboticsproceedings.org/rss07/p22.pdf) | [本地 PDF](<../papers/G_工具预算与背景/L10_2011_Cross_Entropy_Randomized_Motion_Planning.pdf>) | 官方论文集/期刊或作者 PDF；未固定文件修订版本 |
| L11 | [Derivative-free optimization methods](https://doi.org/10.1017/S0962492919000060) | [下载 PDF](https://arxiv.org/pdf/1904.11585v2) | [本地 PDF](<../papers/G_工具预算与背景/L11_2019_Derivative_free_optimization_methods.pdf>) | 本地 PDF 首页：arXiv:1904.11585v2 |
| L18 | [Generalized Schrödinger Bridge Matching](https://arxiv.org/abs/2310.02233) | [下载 PDF](https://arxiv.org/pdf/2310.02233) | [本地 PDF](<../papers/G_工具预算与背景/L18_2024_Generalized_Schr_dinger_Bridge_Matching.pdf>) | 未固定 arXiv 版本；下载版本可能与本地会议版不同 |
| L22 | [Accelerating Motion Planning via Optimal Transport](https://www.ias.informatik.tu-darmstadt.de/uploads/Team/AnThaiLe/mpot_preprint.pdf) | [下载 PDF](https://www.ias.informatik.tu-darmstadt.de/uploads/Team/AnThaiLe/mpot_preprint.pdf) | [本地 PDF](<../papers/G_工具预算与背景/L22_2023_Accelerating_Motion_Planning_via_Optimal_Transport.pdf>) | 官方论文集/期刊或作者 PDF；未固定文件修订版本 |
| L33 | [An Introduction to Zero-Order Optimization Techniques for Robotics](https://arxiv.org/abs/2506.22087v3) | [下载 PDF](https://arxiv.org/pdf/2506.22087v3) | [本地 PDF](<../papers/G_工具预算与背景/L33_2025_An_Introduction_to_Zero_Order_Optimization_Techniques_for_Robotics.pdf>) | 本地 PDF 首页：arXiv:2506.22087v3 |
| N02 | [Predictive Sampling: Real-time Behaviour Synthesis with MuJoCo](https://arxiv.org/abs/2212.00541) | [下载 PDF](https://arxiv.org/pdf/2212.00541v2) | [本地 PDF](<../papers/G_工具预算与背景/N02_2022_Predictive_Sampling_Real_time_Behaviour_Synthesis_with_MuJoCo.pdf>) | 本地 PDF 首页：arXiv:2212.00541v2 |
| N10 | [MPPI-Generic: A CUDA Library for Stochastic Trajectory Optimization](https://arxiv.org/abs/2409.07563) | [下载 PDF](https://arxiv.org/pdf/2409.07563v5) | [本地 PDF](<../papers/G_工具预算与背景/N10_2024_MPPI_Generic_A_CUDA_Library_for_Stochastic_Trajectory_Optimization.pdf>) | 本地 PDF 首页：arXiv:2409.07563v5 |
| N14 | [Sampling-based Model Predictive Control Leveraging Parallelizable Physics Simulations](https://arxiv.org/abs/2307.09105) | [下载 PDF](https://arxiv.org/pdf/2307.09105v3) | [本地 PDF](<../papers/G_工具预算与背景/N14_2023_Sampling_based_Model_Predictive_Control_Leveraging_Parallelizable_Physics_Simulations.pdf>) | 本地 PDF 首页：arXiv:2307.09105v3 |
| N16 | [Judo: A User-Friendly Open-Source Package for Sampling-Based Model Predictive Control](https://arxiv.org/abs/2506.17184) | [下载 PDF](https://arxiv.org/pdf/2506.17184v1) | [本地 PDF](<../papers/G_工具预算与背景/N16_2025_Judo_A_User_Friendly_Open_Source_Package_for_Sampling_Based_Model_Predictive_Control.pdf>) | 本地 PDF 首页：arXiv:2506.17184v1 |
| N19 | [STORM: An Integrated Framework for Fast Joint-Space Model-Predictive Control for Reactive Manipulation](https://arxiv.org/abs/2104.13542) | [下载 PDF](https://arxiv.org/pdf/2104.13542v2) | [本地 PDF](<../papers/G_工具预算与背景/N19_2021_STORM_An_Integrated_Framework_for_Fast_Joint_Space_Model_Predictive_Control_for_Reactive_Manipulation.pdf>) | 本地 PDF 首页：arXiv:2104.13542v2 |
| N21 | [Model Predictive Control via Probabilistic Inference: A Tutorial and Survey](https://arxiv.org/abs/2511.08019) | [下载 PDF](https://arxiv.org/pdf/2511.08019v4) | [本地 PDF](<../papers/G_工具预算与背景/N21_2025_Model_Predictive_Control_via_Probabilistic_Inference_A_Tutorial_and_Survey.pdf>) | 本地 PDF 首页：arXiv:2511.08019v4 |
## 根目录_未分类

| 编号 | 论文（原文入口） | PDF 下载地址 | 本地文件 | 版本／说明 |
|---|---|---|---|---|
| 未编号 | [Beyond Softmax and Entropy: Convergence Rates of Policy Gradients with f-SoftArgmax Parameterization & Coupled Regularization](https://proceedings.iclr.cc/paper_files/paper/2026/hash/17a5dab745d625415c119f127b9fd121-Abstract-Conference.html) | [下载 PDF](https://proceedings.iclr.cc/paper_files/paper/2026/file/17a5dab745d625415c119f127b9fd121-Paper-Conference.pdf) | [本地 PDF](<../papers/18709_Beyond_Softmax_and_Entro.pdf>) | ICLR 2026 官方论文集 PDF；本地首页标题与作者一致 |
