---
title: 谷歌一天放出 6 篇 AI 写的数学论文，JetBrains 开源 12B 编程模型
date: 2026-11-27T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 这一期把目光放在“开放权重怎样被真正用起来”。谷歌研究院一次挂出 6 篇由 Cogentic 生成的数学论文，覆盖整数分解、离散对数与博弈论，但同行评审和 Lean 4 形式化仍未完成；JetBrains 则把 12B 的 Mellum2.1 Instruct 以 Apache 2.0 开放，靠 2.5B 激活参数把编程代理的 token 压掉一半。商汤的 27B 机器人模型和一篇多智能体论文提供了另一条线索：前者在 RoboDojo 拿到 34.7 分，后者用等预算阶梯证明“组织方式”本身能带来 18 个百分点。工程侧，openset-llm 与 Project Maya 都在把大模型压进 8GB 级硬件，微软则披露了 agent-framework 供应链被投毒的事件。想核对每条的数字与边界，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. 谷歌一次挂出 6 篇 Cogentic 论文：整数分解做到 L[1/3]，Nash 近似压到 1/5**
11 月 25 日，google-research 仓库的 cogentic_research_papers 目录一次性新增 6 篇由多智能体系统 Cogentic 生成的论文，总篇幅 274 页，覆盖整数分解、固定特征有限域离散对数、双矩阵博弈近似 Nash、log-rank 猜想与图重构；最受关注的一篇给出无条件下 L_N[1/3, O(1)] 的整数分解算法，另一篇把近似 Nash 的最坏情况 regret 从约 1/3 压到 1/5，但这些结果尚未同行评审，承诺的 Lean 4 形式化也还没随论文公开，读者应把“证明已计算机验证”与“已通过社区审查”分开看。同一目录还留着 11 月 23 日、18 日与 10 月 30 日的论文，说明这是一条持续产出的流水线，而非单次演示。
[原文](https://github.com/google-research/google-research/tree/master/cogentic_research_papers)

**2. JetBrains 开源 Mellum2.1 Instruct：12B MoE 只激活 2.5B，token 省一半**
JetBrains 把 Mellum2.1 Instruct 以 Apache 2.0 放上 Hugging Face：同样是 12B MoE、每 token 只激活 2.5B 参数，但不输出思维链，官方称 SWE-bench Verified 得分较上一代 Instruct 翻倍，token 用量约为 Thinking 版的一半。它随权重提供 MTP 头，生成速度最高提升 1.8 倍；单请求比 Qwen3.5 9B 快约 1.4 倍、高负载下快约 1.8 倍，训练阶段用 10,000 个并发沙箱收集了约 50 万条代理轨迹，适合放在子代理和工具循环里跑短任务。
[原文](https://blog.jetbrains.com/ai/2026/11/mellum2-1-instruct-is-out-bringing-faster-answers-for-coding-agents/)

**3. 商汤开源 27B 机器人 VLM：RoboDojo 34.7 分，真机抓取 60%**
11 月 23 日上线的 SenseNova-RoboRSI-27B 以 Apache 2.0 开放，11 月 26 日又补齐 RoboDojo 复现脚本。这个 27B 视觉语言模型能在 VLA 技能与“写代码控制机械臂”之间切换，在 RoboDojo 的 Agent 协议下平均 34.7 分，较 Preview 版的 10.0 分提升 247%，也高于 GPT-6.1 Sol 的 33.7；真机 10 次试验里，Piper 抓放成功 60%、Piper X 堆块成功 90%，末端执行器预测落在目标 50mm 内的比例从 24.9% 升到 49.2%。
[原文](https://github.com/OpenSenseNova/SenseNova-RoboRSI)

**4. RCI 用等预算阶梯验收多智能体：27B 自发成枢纽，准确率涨 18 个百分点**
11 月 24 日的 arXiv 论文提出“等预算阶梯”，把多智能体系统依次与单代理、算力匹配的多数投票、算力匹配的 pass@n 比较，避免把“多花 token”误算成“协作增益”；作者在 27B 模型上看到，转发被拆散的事实能让群体超过 pass@n 约三分之一，同构代理会自发形成枢纽、准确率提升 18 个百分点，把组织方式写进提示词后达到第 3 级所需 token 不到一半。两代无标签迭代后，9B 第二代同时超过 pass@n 和用基座模型对话训练的对照，且在 PlanBench、HiddenBench、Hanabi 上连续 4 到 6 代在未见问题上进步。
[原文](https://arxiv.org/abs/2611.32606)

## 开源与工程

**5. openset-llm 把 27B 三值模型塞进 8GB CPU：17–20 token/s**
11 月 26 日发布 v0.2.0 的 openset-llm 是 MIT 协议的 C++ 推理引擎，用 AVX-512/AVX2 内核在纯 CPU、8GB 内存上跑 5.95GB 的 Qwen3.8-27B 三值模型，单用户解码 17–20 token/s，官方称约为 ik_llama.cpp 的 4.2–4.3 倍、PrismML 分支的 5.9–6.1 倍。它提供 OpenAI 兼容服务，--multi 4 时总吞吐约 2.28 倍，GSM8K-40 为 39/40、编码 pass@1 为 83.1%；11 月 27 日的 0.3.2 又修了无 AVX-512 机器上 --portable 构建无法启动的问题，项目在 7 天内新增 1,171 星，说明“无 GPU 本地推理”仍是高需求场景。
[原文](https://github.com/opensetlabs/openset-llm)

**6. Project Maya v1.1.81：8GB 显存跑 744B 模型，量化保留 97.7% 精度**
11 月 25 日发布 v1.1.81 的 Project Maya 是 MIT 协议的本地推理栈，把 GLM-5.3（744B）、DeepSeek-V4.1-Flash（748B）、MiniMax-M3.1-Flash（428B）等模型按专家热度拆到 GPU 显存、系统内存和 NVMe SSD，部分模型最低 8GB 显存即可启动。官方给出的 Maya 量化把 GLM-5.3-Flash 的零样本准确率保留在 FP8 的 97.7%–99.2%，DeepSeek V4.1 Maya-S 与原模型下一 token 一致率 92.6%；双 RTX 5090 加 RTX 4090 的实测解码达到 148.6 token/s，仓库一周新增 335 星。
[原文](https://github.com/mw00/project-maya)

**7. 微软 agent-framework 被投毒：1.26.1/1.26.2 在 Python 启动时窃取凭证**
11 月 25 日微软更新事故公告：11 月 17 日，agent-framework 的 1.26.1 与 1.26.2 两个 PyPI 版本在未授权情况下被上传，包内加入 .pth 启动钩子，只要该环境启动 Python 就会执行，即使用户没有 import 这个包；恶意代码会收集环境变量、云凭证、CI/CD 与 GitHub Actions OIDC 材料，两个版本已被移除（PYSEC-2026-4388），但已经下载到 lockfile、镜像、缓存或内部镜像源的副本不会自动消失。受影响版本的指示文件分别是 agent_framework_meta.pth 与 af_runtime_diag.pth，检查 site-packages 即可确认；微软建议锁定 1.26.0，若安装过则把机器视为失陷，轮换所有可达凭证并从干净镜像重建。
[原文](https://devblogs.microsoft.com/agent-framework/microsoft-agent-framework-repository-update/)

**8. Langfuse v4.86.0 把评测搬到 trace 级：转录评估有了独立界面**
11 月 24 日发布的 Langfuse v4.86.0 增加 trace 级 transcript 评估 UI，把多轮代理对话的评分从单条 generation 提升到整条 trace，并让 MCP 图表以可交互的 MCP App 形式内嵌渲染。这个版本还修了 OpenAI Responses 输出项里的 tool call 解析、TypeScript 工具列归一化和 ingestion 路径的一致性问题，共包含 20 多项改动；对已经用 Langfuse 做代理观测的团队来说，升级后可以直接在 trace 视图里跑评估，不必再把对话拆成单步样本。
[原文](https://github.com/langfuse/langfuse/releases/tag/v4.86.0)

**9. LobeHub 重建 v2.3.7 发版 PR：114 个提交、300 个文件**
11 月 26 日，LobeHub 把 v2.3.7 的发版 PR 按 canary HEAD 重建，范围覆盖 114 个提交、300 个文件、12 位贡献者，前一个发版 PR 因 overlay 类型检查失败在 13:15Z 关闭。发版分支此前两次 canary 同步是 squash 合并，merge base 断裂导致无法增量合并，团队这次直接对齐 canary HEAD；这一版把 Channel 消息表情回应、Agent 访问申请与审批、按文件标记同步沙箱新增附件、上下文引擎的技能快照增量注入放进同一个版本；其中访问审批的 replay 漏洞被 Codex 评审指出，维护者确认会在 canary 侧单独修复，不阻塞本次发版。
[原文](https://github.com/lobehub/lobehub/pull/23962)

*本期由站点自动生成 · 信息以原文为准。*
