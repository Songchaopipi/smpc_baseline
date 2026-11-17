# 论文精读与研究方向工作流

安装与首次使用：2026-11-16。当前建议是：**先验证真实干预能否识别值得重开的控制时间块，再决定是否发展通用优化算法。** 本轮实际完成技能安装、关键近邻精读与研究问题卡；未运行新优化实验。

当前输入是 [论文地图](01_PAPER_MAP.md)、[长期框架](02_RESEARCH_AND_FRAMEWORK.md)、[CEM 构想评审](03_CEM_IDEA_REVIEW.md) 和本地 57 篇文献。没有找到作者论文正文，所以本轮属于研究构想与相关工作的分析，不能等同于论文全文审稿。既有论文地图继续作为总索引，本轮没有对全部 57 篇重新做逐篇精读。

## 已安装什么

| 来源 | 实际安装位置 | 验收 |
|---|---|---|
| [Paper Reading](https://github.com/sodalone/paper-reading-skill) main，`82a0a00fdb1a7ad3e733c17f40acb19280e4263d` | `/home/songchao/.codex/skills/paper-reading/` | 独立 venv；依赖导入与上游 20 项测试通过；TR-MPC 官方流水线实际运行 |
| [Claude Scholar](https://github.com/Galaxy-Dawn/claude-scholar/tree/codex) **codex**，`b672e80951a2348dd6136316d096603bf2dc5b83` | `/home/songchao/.codex/skills/` 下 45 个技能 | 与固定提交源文件逐一比较一致 |
| Scholar 支持文件 | `/home/songchao/.codex/{agents,scripts,utils,templates}/` | 6 个独立 agent TOML 可解析；41 个支持文件，含 agent 说明与上游 AGENTS sidecar |

技能由官方 `skill-installer` 安装。Scholar 源码保存在 `/home/songchao/.local/share/codex-skill-sources/claude-scholar/`；本次安装清单是相邻的 `smpc-install-state.json`，项目内可读摘要见 [installation.json](research/installation.json)。本次采用选择性安装路径，未生成 Scholar `setup.sh` 自有卸载状态，后续管理应使用本次清单，不能假设上游 uninstall 已掌握文件归属。

**现有模型、认证、MCP 和全局 AGENTS 文件均经安装前后哈希核对保持不变。** 上游规则只保存为 `~/.codex/AGENTS.scholar.md` 和 `AGENTS.zh-CN.scholar.md`，供参考，不会自动成为全局规则。Zotero、Obsidian 未连接，本地工作流可以独立使用。

新技能通常从**下一轮对话**开始可被 Codex 发现；新 agent 若未出现在当前会话工具中，需新建会话。本轮已直接读取所用 `SKILL.md` 并执行脚本完成首次使用。调用名使用 `$paper-reading` 等技能名，不依赖 Claude Code 的 slash command。

## 每个阶段只用适合的技能

| 要做的事 | 主要技能 | 输入 → 交付物 | 在本项目中的位置 |
|---|---|---|---|
| 精读一个最接近的算法 | `paper-reading` | arXiv ID/链接 → 单个自包含阅读报告、原文、证据定位 | 先读 TR-MPC，再读 iCEM；不拿它一次概括全部文献 |
| 验证引用是否支持比较 | `citation-verification` | 原文、版本页 → 元数据及 claim 对齐记录 | 写“已有工作没有做什么”之前先核对 |
| 提出并收窄研究方向 | `research-ideation` + 本地 `research-idea-rubric` | 文献证据、代码、资源 → Research Question Card 和可证伪命题 | 本轮已生成[研究卡](research/research-question-card.md) |
| 分析真实实验 | `results-analysis` → `results-report` | 多 seed 原始指标、预算、配置 → 统计分析、科学图、实验决策报告 | 5 seeds 先筛选；不从单个成功视频推显著性 |
| 写论文 | `ml-paper-writing` | 已验证 Claim、图表、近邻 → 章节草稿 | 目前先写问题与方法假设，结果等待实验 |
| 检查论证再改语言 | `paper-self-review` → `writing-anti-ai` 或现有 `academic-humanizer` | 完整稿和证据 → 主张审查、再做语言编辑 | 语言编辑不能创造新颖性、数字或引用 |
| 记录长期任务 | `planning-with-files` | 多步任务 → 计划、证据笔记、交付物 | 本轮记录在 [task_plan.md](research/task_plan.md) 与 [notes.md](research/notes.md) |

45 个技能全部安装不表示每次都要启用。当前只需精读、选题与证据审查；Nature 写作、审稿回复、前端开发、知识库绑定等按实际任务再使用。`paper-miner` 负责提取写作模式，不能作为新颖性或正确性的验证者。

## 本轮分析怎样改变研究决策

原有 03 的方向排序经源码核查和四篇直接近邻审查后仍合理，但研究主张要保持窄：

> 在渐进增加评价时域的黑箱轨迹优化中，用少量真实控制干预选择恢复哪些时间块，能否比固定重开和持续探索更节省完整任务的真实仿真预算？

现有证据确认特定失败条件末轮停止探索早段；没有确认一个完整成功抓取被单高斯拟合毁掉。另一方面，原接近目标有几何冲突，因此必须把优化器变化与任务表达变化分开。

新核查还发现两个影响跨平台比较的细节：Hydrax `cov` 实存标准差，SBTO 对角项是方差；Hydrax 已含初始尺度探索和标准差下界，离线入口的最终选择也与 SBTO 有差别。这些都应在公平对照前统一，不能只比较两个算法名字。具体证据、允许与禁止的表述、七组最小实验和两周停止条件均放在[研究问题卡](research/research-question-card.md)，避免复制出另一套不一致协议。

多分支预算作为备选，先证明“当前成本排名”与“等预算续搜后成功排名”有可重复差别；DFO-GN 作为局部精修组件。若重开仅胜过旧硬掩码，应定位工程修复；若跨任务胜过最强持续探索、固定早段及完整时域对照，才进一步研究一般算法与理论。

## 阅读优先级

| 优先级 | 原文与应回答的问题 | 后续决策 |
|---|---|---|
| P0，已精读 | [TR-MPC v2](https://arxiv.org/abs/2605.07801v2)：KL/熵保护已经解决到哪一步？ | 形成必要强基线，收窄“恢复哪些变量”的贡献；[阅读报告](paper_reading/2605.07801_Sampling-based_Model_Predictive_Control_Using_Trust_Regions/2605.07801_阅读报告.md) |
| P0，方法已核验 | [iCEM v1](https://arxiv.org/html/2008.06389v1)：相关扰动、精英记忆和输出选择的收益？ | 先建强基线；正式发表身份可据 [CoRL 论文集](https://proceedings.mlr.press/v155/pinneri21a.html) 补齐 |
| P1，方法已核验 | [BAMPC v1](https://arxiv.org/html/2610.14064v1)：真实仿真预算应算哪些用途？ | 定义样本、world-steps、重评与时间对齐开销 |
| P1，方法已核验 | [BC-EvoCEM v2](https://arxiv.org/html/2506.02205v2)：多个 worker 的已有管理方式？ | 只有续搜收益提供额外排序信息时再发展分支方向 |
| P2，待进一步精读 | 03 已列出的 GRASP 及 adaptive block/coordinate selection、变量重新激活、重启与协方差恢复文献 | 检查选块命题是否已有同构方法，再考虑理论贡献 |

本轮 TR-MPC 精读采用版本化 PDF 与源码。网页抓取成功仅代表收到 HTTP 响应：翻译页可能只有 JS 外壳，HTML 内容可能滞后，表格宏也可能造成显示单位差异；准确图表、版本和理论边界以阅读报告的证据定位为准。

## 可直接复用的提示词

单篇论文：

```text
使用 $paper-reading 精读 https://arxiv.org/abs/2008.06389v1。
输出根目录为 /home/songchao/G1_Locomaniplation/SMPC/doc/paper_reading。
重点核对时间相关噪声、精英复用和最佳序列输出；区分作者结果与本地复现。
每条核心主张保留原文定位，运行最终验证器。
```

继续推进选题：

```text
使用 $research-ideation 和本项目 research-idea-rubric，更新
doc/research/research-question-card.md。先读 03 和现有证据记录。
检验“真实干预收益选择重开时间块”是否超出持续探索与固定早段重开。
审查文献与实现证据分工进行；保留反例、等预算基线和停止条件。
没有实验结果时保持假设措辞，不同时加入多分支或 DFO。
```

已有多 seed 结果后：

```text
使用 $results-analysis 分析指定的结果目录，先验证运行单位、任务成功判据、
world-steps、墙钟、失败与删失、调参和输出规则是否可比。
通过后给配对区间、效应量、成功率—预算曲线，再用 $results-report 更新决策。
输入不足时明确缺什么，不编造统计量。
```

有论文正文后：

```text
使用 $paper-self-review 对指定稿件做 Claim—Evidence 审查，并用
$citation-verification 核对实际引用。技术审查与语言编辑独立进行。
将没有结果支持的贡献改为假设；证据核对完再用写作技能修改论述。
```

## 手动运行与验证

Paper Reading 当前流水线只接受能解析 arXiv ID 的输入，不能直接把本地中文文件名、普通 PDF URL 或 DOI 页面传进去。已有文献先从 01 查 arXiv ID；没有 ID 的稿件使用全文审查技能。

```bash
cd /home/songchao/G1_Locomaniplation/SMPC/doc/paper_reading
bash /home/songchao/.codex/skills/paper-reading/scripts/run_pipeline.sh \
  https://arxiv.org/abs/2008.06389v1
```

脚本只预处理并建立骨架，**不自动完成论文分析**。模型需继续读取原文与规定 references，填写同一个主报告；不要把骨架交付成精读。完成后用技能的 venv Python 运行两个验证器；例如本轮 TR-MPC：

```bash
cd /home/songchao/G1_Locomaniplation/SMPC/doc/paper_reading
/home/songchao/.codex/skills/paper-reading/.venv-paper-reading/bin/python \
  /home/songchao/.codex/skills/paper-reading/scripts/validate_report_text.py \
  --input https://arxiv.org/abs/2605.07801v2 --root . \
  --workspace-name 2605.07801_Sampling-based_Model_Predictive_Control_Using_Trust_Regions
/home/songchao/.codex/skills/paper-reading/.venv-paper-reading/bin/python \
  /home/songchao/.codex/skills/paper-reading/scripts/validate_report.py \
  --input https://arxiv.org/abs/2605.07801v2 --root . \
  --workspace-name 2605.07801_Sampling-based_Model_Predictive_Control_Using_Trust_Regions --final
```

TR-MPC 已存在完成工作区，后续读取它即可；重新执行 pipeline 可能重写骨架，应先保留人工报告。本轮没有运行新优化实验，也没有修改已有求解器或 01–03 文档。
