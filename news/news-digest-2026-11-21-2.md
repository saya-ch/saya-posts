---
title: Qwen4-Max 把单任务成本砍六成，MaxText 跨城训练流量降 65 倍
date: 2026-11-21T20:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: 本期把镜头对准“同样的能力，账怎么算”。阿里 Qwen4-Max 用 2.4T 总参数、71B 激活把单任务成本压约六成，还给出 100 万上下文与原生视觉；RadixArk 的 DaisyRose 27B 只靠强化学习后训练 18 天，就把 Terminal Bench 4.1 从 5.6 拉到 12.0。工程侧，MaxText 的 Streaming DiLoCo 在 10Gbit/s 链路上把跨城训练从 55.11 秒/步压到 5.76 秒，Gemini API 的 Tool Retrieval 让 Agent 按需加载工具 schema。研究侧，QDA 用 Agent 设计量化配方，一篇审计论文则发现 25 名人类审计员识别破坏的 AUROC 只有 0.59。竞争正从“谁更大”转向“谁把每一分算力、每一个 token 花得更值”，想看清每条的成本与代价，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. Qwen4-Max 用 2.4T 总参数换 60% 成本下降：71B 激活、100 万上下文**
阿里 11 月 16 日发布 Qwen4-Max，MoE 总参数 2.4 万亿、每 token 仅激活 710 亿，较同规模 Qwen3.8-Max 少 25% 激活参数；新 Qwen4 架构同时改注意力、残差与嵌入的稀疏设计，支持原生视觉与 100 万 token 上下文。官方称在覆盖 721 个专业领域的 Work360 上，高强度推理模式少用 35% token、单任务平均成本降约 60%，缓存命中输入价 0.075 美元/百万 token，Code Arena 排第 6。展示中它用 48 小时闭合流程筛出一种经实验验证的冷却液分子；另一项阿尔茨海默研究里执行 454 次工具调用与 3,239 次指标评估，优化 24 个候选分子。汽车产线则把工件编程从 475 分钟压到 24 分钟。开源的小尺寸 Qwen4 只给了“即将发布”的时间表。
[原文](https://www.alibabacloud.com/en/press-room/alibaba-unveils-qwen4-max)

**2. DaisyRose 27B 只做强化学习：18 天、6.3 万 GPU 小时，Terminal Bench 4.1 翻倍**
RadixArk 11 月 18 日以 Apache 2.0 放出 DaisyRose-27B，基座是 Qwen3.8-27B，架构、分词器、视觉塔与 256K 上下文全部保留，训练却跳过监督微调与蒸馏，只用强化学习。数据是约 1,200 个长程 Agent 任务，单次会话可到 100 万 token；128 张 H200 跑推理、16 张做训练，异步跑 18 天，合计约 63,000 GPU 小时。模型卡给出 Terminal Bench 4.1 从 5.6 升到 12.0、DeepSWE v1.1 从 42.2 升到 46.2、SWE Bench Pro v2 达 94.7，但 DeepSWE 仍落后 Inkling 2 Small 26 分；BF16、FP8、NVFP4 权重分别约 52、29、23GB。
[原文](https://huggingface.co/RadixArk/DaisyRose-27B)

**3. 25 名审计员看完整记录，仍只有 17% 能说清破坏手法**
11 月 16 日提交到 arXiv 的 Redwood Research 论文把一名人类专家设为“破坏者”，让其在 AI 安全实验的完整记录里植入 33 种破坏机制，再请 25 名具备机器学习与安全背景的审计员判断哪次实验被动手脚。结果人类 AUROC 仅 0.59，与随机猜测无显著差异，能完整指出具体机制的只有 17%；不允许跑实验的前沿模型反而更好，Fable 5.1 的 AUROC 为 0.73、完整识别率 32%，较弱模型则接近或低于人类。作者据此认为，专家级 Agent 的研究破坏很难靠现有审计流程发现。
[原文](https://arxiv.org/abs/2611.21832)

**4. QDA 让 Agent 自己调量化配方：4 个推理模型平均涨 0.9–2.2 个百分点**
11 月 17 日的一篇论文提出 QDA：把后训练量化配方交给 LLM Agent 迭代设计，候选方案不再跑几十个 GPU 小时，而是用推理轨迹上的输出 KL 散度在几分钟内打分，再对高分方案做长生成检查，专门抓重复循环这类失败。四个推理模型上，QDA 在 W4A16、W4A8、W3A16 分别平均比最强公开配方高 0.9、1.4、2.2 个百分点，每次搜索约 64 GPU 小时；320B 的 GLM-5.3-Flash 在 NVFP4 W4A4 下恢复 FP8 模型 98.3% 的准确率，供应商最佳 checkpoint 为 96.9%，且产物可直接跑现有 serving kernel。
[原文](https://arxiv.org/abs/2611.23388)

## 开源与工程

**5. MaxText 装上 Streaming DiLoCo：10Gbit/s 跨城训练 9.57 倍加速，流量少 65 倍**
谷歌 11 月 19 日在 MaxText v0.2.5 中落地 Streaming DiLoCo，把模型参数切成“层数+1”个分片，在 P=H=65 的配置下每步只同步一个分片，并用主机 CPU 上的同步 worker 在后台做 all-reduce，不再需要中心参数服务器。Qwen3-32B、两块 TPU v5p-32 的实测里，10Gbit/s 限速下标准数据并行要 55.11 秒/步，Streaming DiLoCo 稳定在 5.76 秒，加速 9.57 倍；65 步内跨切片流量从 4,258.8GB 降到 67.0GB。两个 MoE 模型跑 4,000 步的验证损失与同步数据并行持平甚至略好；不过这些数字来自限速模拟，真实长途链路与万亿参数规模仍未验证。
[原文](https://developers.googleblog.com/distributed-llm-training-across-metros-streaming-diloco-in-maxtext/)

**6. Gemini API 上线 Tool Retrieval：几百个工具只加载当前要用的 schema**
谷歌 11 月 18 日为 Gemini API 的 Interactions 接口推出 Tool Retrieval 预览：请求里加一个 tool_search 工具，并给长尾工具标 defer_loading，模型仍能看到工具名与描述，但参数 schema 不进上下文；需要时按名检索，API 在同一次请求内加载 schema 并完成函数调用。官方称没有单独收费，只按模型标准价计 token，且延迟加载的 schema 不破坏前缀缓存；支持 Gemini 3.6 及之后的所有 Gemini 4 模型，也能对远程 MCP server 整体延迟加载。对工具库上百个的 Agent，这直接省下每轮重复发送 schema 的输入成本。
[原文](https://blog.google/innovation-and-ai/technology/developers-tools/gemini-api-tool-retrieval/)

**7. GitHub 开源 Replica 与无障碍框架：公共贡献一年翻到 25 亿次**
GitHub 11 月 18 日的博客集中介绍两个已开源项目：Replica 是面向设计系统的原型 harness，用强化学习从每次交互与评论里学习“什么算好”，让开发者与 Copilot 并排迭代；Open Source Accessibility Framework 则给出四阶段问责模型，并推动仓库根目录出现 ACCESSIBILITY.md，集中写明无障碍目标、贡献者预期、支持环境与已知限制。GitHub 同时给出规模数字：2026 年公共贡献 25 亿次，较上年的 11.2 亿次翻倍多；过去一年代码推送 179 亿次、提交 343 亿次、合并 PR 13 亿次，平均每天 350 万次。
[原文](https://github.blog/open-source/maintainers/the-best-things-in-open-source-are-built-together/)

**8. Claude-BugHunter 冲上 GitHub 日榜：83 个技能、681 份公开报告做底座**
11 月 21 日 GitHub Trending 首位是 elementalsouls/Claude-BugHunter，一个给 Claude Code 做漏洞挖掘与外部红队的技能包：83 个技能、15 个斜杠命令，把 681 份已披露报告整理成 24 类核心漏洞的检测模式，其中 433 份单独引用、可追溯。它把流程拆成侦察、测绘排序、狩猎、验证、报告等阶段，提交前过一道 7-Question Gate，并自带 M365/Entra、Okta、vCenter、SSL VPN 等企业入口的攻击链与证据脱敏规范；同一套 SKILL.md 还能装进 Codex CLI、OpenCode、Hermes 与 Antigravity。
[原文](https://github.com/elementalsouls/Claude-BugHunter)

**9. 272 个真实微调任务测出：FLOP 估算只解释 31% 的耗时差异**
Dharma-AI 11 月 17 日在 Hugging Face 博客公布 272 个生产 SFT 任务的复盘，合计约 7,382 GPU 小时，平均 23.43 小时、中位数 15.29 小时。按 6PD 公式、MFU 取最优的 30% 时，FLOP 基线 R² 只有 0.31、MAE 12.41 小时；换更细的 Calculon 解析模型，预估中位数仅为实际墙钟的 11%。改用只看派发前信息的 CatBoost，24 个特征把 R² 拉到 0.98、MAE 1.71 小时、中位误差 0.51 小时；特征重要性第一是评估数据量，其次才是可训练参数、评估 batch 比例与 checkpoint 数量。
[原文](https://huggingface.co/blog/Dharma-AI/predicting-fine-tuning-time)

*本期由站点自动生成 · 信息以原文为准。*
