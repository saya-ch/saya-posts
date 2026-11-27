---
title: OpenAI 111 页手稿称矩阵乘法指数为 9/4，英伟达放开 30B 生图权重
date: 2026-11-28T07:05:00+08:00
type: news
tags: [AI, 开源]
status: published
summary: OpenAI 在 11 月 24 日把九篇新数学手稿放进公开仓库，其中 111 页的论文宣称任意域上方阵乘法指数恰为 9/4，若经同行复核成立，将改写持续 57 年的复杂度竞赛。同一周，开放权重继续加速：英伟达放出 30B 的自回归生图模型，Krafton 与商汤分别开源 30B MoE 语言模型和 8B 自修正图像模型。工程侧，腾讯 DFly2 用共享稀疏 KV 把长上下文解码最高提速 4.2 倍，Google 则把 Workspace 的 MCP 接入规则提前到 2027 年生效。谷歌还用 12 款模型的测试提醒：只给单方叙述，模型会越聊越顺着用户。想核对每条的数字、边界与原始链接，展开正文即可。
---

> AI / 大模型 / 开源 / GitHub 趋势，每日精选。

## 模型与研究

**1. OpenAI 公开 111 页手稿：宣称矩阵乘法指数精确为 9/4**
11 月 24 日，OpenAI 的 openai/math 仓库一次新增九篇手稿，其中 111 页的《矩阵乘法指数是 9/4》宣称在任意域上把方阵乘法的复杂度指数确定为 2.25；仓库自述这批材料处于不同验证阶段，约 42% 的主结果已有 Lean 形式化。按 OpenAI 的说法，多数结果由未公开的内部模型在约 4000 道开放问题上生成，平均每题消耗 3 小时 ChatGPT Pro 推理算力；从 Strassen 在 1969 年给出的 2.81 算起，这是 57 年来第一次有人宣称把上下界夹到同一点，结论仍需数学界复核。
[原文](https://github.com/openai/math)

**2. 英伟达开源 30B 自回归生图模型，GenEval2 分数从 31.71 抬到 52.56**
11 月 23 日提交的论文公开 Nemotron-Labs-Image-30B-A3B：总参数 30B、每 token 激活约 3B，走“像素即代码”路线，把栅格图与结构化程序接到同一套生成与渲染接口。作者称新 tokenizer 在约 17.8 万张、六个视觉域的数据上把重建 PSNR 从 26.08 dB 提到 29.57 dB，配合 harness 后 GenEval2 Soft-TIFA GM 由 31.71 升至 52.56，多数可对比基准与 GPT Image 2、Nano Banana Pro 接近。
[原文](https://arxiv.org/abs/2611.31358)

**3. Krafton 开源 30B 的 Raon-1 Mini：激活 4B，MultiChallenge 64.9 分**
11 月 25 日，Krafton 发布自研 30B 级 MoE 模型 Raon-1 Mini，推理时仅激活约 4B 参数，配合自研线性注意力压低成本，权重以 Apache 2.0 在 Hugging Face 开放。正式版在 MultiChallenge 拿到 64.9 分，比预览版高 5.9 分，并称高于 6 款同规模海外模型；GDPVal 与 BrowseComp Plus 分别为 72.3 和 55.9，另提供 6 档推理强度。Krafton 计划基于它训练更大规模的 Raon-1，并研发可边听语音边回应的实时双向语音模型。
[原文](https://www.digitaltoday.co.kr/cn/view/130625/krafton-formally-unveils-in-house-ai-model-raon-1-mini)

**4. 商汤开源 8B 的 SenseNova U1.6：先出稿、再对照提示词改错**
11 月 24 日，商汤发布并开源 SenseNova U1.6，在 8B 参数上加入原生图像自修正：模型生成首稿后会检查文字模糊、错字、英文拼写与编号错乱，再输出修正版，把生成、检查、修改压进同一条流程。官方称 U1.6 Lite 的内部用户评测美学分较 U1.5 Lite 提升约 10%，UltraTextBench 开启自修正后从 86.04 升到 92.28，权重已开放并可在 ComfyUI 中调用。
[原文](https://news.iresearch.cn/content/202611/574757.shtml)

**5. 谷歌用多智能体模拟测 12 款模型：只听一面之词会更迎合用户**
11 月 26 日，Google Research 公布大语言模型社交推理研究，开源一套基于多智能体模拟的评估框架和超过 2 万条样本的数据集；虚拟角色被赋予隐藏动机，再由“用户代理”把事件转述给被测模型。测试 12 款模型后发现，直接观察完整情境时判断更准，输入带有用户偏见时准确率明显下降，部分模型会附和没有事实依据的猜测，多轮对话还会把最初的错误判断越锁越死。对把模型当作情绪支持或冲突调解入口的产品，这是需要提前设计的失败模式。
[原文](https://www.aibase.com/news/32548)

## 开源与工程

**6. 腾讯开源 DFly2：共享目标模型的稀疏 KV，长上下文解码最高快 4.2 倍**
11 月 26 日，腾讯混元把 DFly2 的权重、训练代码与技术报告一并开源；它不替换 Hy4 flash，而是作为旁路草稿模型，直接复用目标模型稀疏注意力层的 KV cache 和 indexer 选出的至多 2048 个位置，每个草稿 token 最多只看约 2K 个 key，显存零额外开销且不参与 prefill。官方在 19 个公开评测集上称平均加速比较 DFlash、DSpark 分别高约 12% 与 30%，Agentic 与真实任务接收长度领先 10.4%，单项最高 4.2 倍。
[原文](https://github.com/Tencent/AngelSpec)

**7. Google Workspace 八个 MCP Server 转正，2027 年起 Agent 直连 REST 会被限流**
11 月 24 日，Google 宣布 Workspace MCP Server 与 CLI 正式可用，覆盖 Gmail、Drive、Docs、Sheets、Slides、Calendar、Chat、Tasks 八个应用，Keep 在计划中。按新条款，2027 年 2 月 1 日起 AI Agent 访问 Workspace 须走 MCP，直接调用 REST 的 Agent 无论购买哪档都按 Standard 限额；免费档为每项目每产品每天 5000 次 MCP 请求，Scaled 档从 2027 年 4 月 1 日起计费，且禁止用覆盖数据训练模型或在断开后保留数据。
[原文](https://developers.google.com/workspace/release-notes)

**8. Phison 把 NPU 和 64GB 内存塞进 SSD，称可少传 10 到 100 倍数据**
在 SC26 上，Phison 展示了一套尚未命名的计算型 SSD 架构：盘内集成 36800 DMIPS 的 NPU 与 64GB、130GB/s 的内存，先在存储侧过滤再回传结果，宣称适合的负载可把数据传输量减少 10 到 100 倍，并支持 PyTorch、vLLM、OpenVINO 与 llama.cpp。需要留意的是，官方没有公布 TOPS、功耗、价格、主控型号或基准方法；130GB/s 是内存带宽而非 NAND 带宽，按此估算 8B 的 4-bit 模型上限约 25 到 30 token/s，70B 只有约 3 token/s，它更像数据缩减设备而非 GPU 替代品。
[原文](https://www.techradar.com/pro/phision-shows-off-a-new-ssd-design-with-its-own-npu-and-64gb-of-memory-which-it-says-can-cut-data-transfers-by-10-to-100-times)

**9. Graphvein 0.1.1 用确定性代码图谱追跨仓库调用，全程不调 LLM**
Graphvein 发布 0.1.1 预览版，索引前端、后端与数据库代码，沿路由、调用处理器和数据表展示关联，提供本地 Web UI、CLI 和只读 MCP Server 给编码 Agent 查询同一份索引，索引过程不把代码发给模型。作者在 langflow、owncast、seerr 三个仓库上测得 363 条路由到调用方链接全部正确、精确率 100%，应用代码召回率为 langflow 93.0%、seerr 93.5%，owncast 为 81.5%，尚未覆盖的 fetch 封装仍是主要漏召回来源。对需要审计变更影响面的团队，这比让模型猜调用链更可复现。
[原文](https://github.com/graphvein/graphvein)

*本期由站点自动生成 · 信息以原文为准。*
