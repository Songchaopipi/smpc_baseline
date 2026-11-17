# 本轮核验笔记

日期：2026-11-16。

## 输入与范围

- `doc/01_PAPER_MAP.md`、`doc/02_RESEARCH_AND_FRAMEWORK.md`、`doc/03_CEM_IDEA_REVIEW.md`。
- 本地 `papers/` 文献 PDF、`src/hydrax` 源代码，以及相邻 `../Loco_maniplation/g1-trajopt/` 的诊断和实现。
- 未发现用户论文 `.tex` 正文；当前结论属于研究构想评审。

## 上游与安装

- Paper Reading：[sodalone/paper-reading-skill](https://github.com/sodalone/paper-reading-skill)，main；安装调用名 `paper-reading`。
- Scholar：[Galaxy-Dawn/claude-scholar，codex](https://github.com/Galaxy-Dawn/claude-scholar/tree/codex)，提交 `b672e80951a2348dd6136316d096603bf2dc5b83`。
- Scholar：45 skills、6 agent 配置、41 个支持文件（含 agent 说明和上游 AGENTS sidecar）。
- 安装状态与受保护配置哈希保存在 `/home/songchao/.local/share/codex-skill-sources/smpc-install-state.json`，不记录认证内容。
- 没有运行会合并真实配置与持久化认证的 Scholar setup.sh；使用官方 skill-installer 与不覆盖已有文件的支持文件复制。

## 三项独立审查实际发现

1. 项目证据审查：SBTO 方差硬掩码与渐进时域同时存在；不能据此断言单高斯毁掉成功抓取。Hydrax 的 `cov` 实存标准差，最终返回规则也与 SBTO 有别。
2. 文献审查：iCEM 的噪声与记忆、TR-MPC 的探索保护、BAMPC 的预算问题、BC-EvoCEM 的多 worker 管理均构成必要近邻。窄主张应是基于真实干预收益选择恢复哪些时间变量。
3. 安装审查：完整技能不依赖覆盖全局配置；Paper Reading 必须验证 PDF 和 metadata，不能仅凭 `Pipeline complete` 宣告成功。

## 一手来源

- [iCEM v1](https://arxiv.org/html/2008.06389v1)，§3.1–3.3；[正式论文集](https://proceedings.mlr.press/v155/pinneri21a.html)。
- [TR-MPC v2](https://arxiv.org/html/2605.07801v2)，§IV、§VI-A–F、附录 A–D。
- [BC-EvoCEM v2](https://arxiv.org/html/2506.02205v2)，§3、§5。
- [BAMPC v1](https://arxiv.org/html/2610.14064v1)，§III–IV。

TR-MPC 关于投影无额外误差的表述需要谨慎：无熵项时，指数倾斜后的任意成本分布一般也不属于高斯族。不能把无限维最优分布的约束保证直接赋予有限样本高斯实现。

## 实际验收与独立复核

- Paper Reading 的 26 个上游文件按 Git blob SHA1 逐一验证；独立 venv 的依赖可导入，上游 20 项测试通过。
- Scholar 的 45 个技能与固定提交的每个源文件一致；6 个 agent TOML 可解析。
- 安装前后原模型、认证、MCP 配置及全局 AGENTS 的文件哈希或不存在状态一致。
- 研究卡经独立实现复核，修正“没有成功证据”被过写为“反证成功存在”的措辞；保留其余已核实实现语义。
- 指南经独立可用性复核，补充了可直接复制的最终验证命令。
- 本轮下载 TR-MPC 的 PDF 实际可打开，共 8 页；六个下载请求均返回 200，但 hjfy 为 JS 外壳，不能等同于读过翻译。ar5iv 缓存内容和参考文献少于版本化 PDF/源码；报告以 PDF 与源码为准。
- 原图提取遇到相同 basename 的任务图片映射问题，精读报告采用按源完整路径核对的图片。运行时间表按 PDF 的 siunitx 显示结果读取，避免把秒值直接误当毫秒。
- TR-MPC arXiv v2 摘要页 Comments 声明 CDC 2026 已接收；应与已经核实正式论文集出版区分。
- 唯一精读报告已完成：8 条 Claim、8 条独立定位、6 张原图；官方 UTF-8/math 检查和 `--final` 检查均通过。根代理再次执行两个检查，并核对完整报告、PDF 运行时间表、平底解析例和两张关键结果图。
- 所有最终 Markdown 本地链接和 6 张报告图片引用均可解析；安装状态摘要已补记首次使用与最终验收。
